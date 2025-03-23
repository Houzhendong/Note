# 搭建dotnet的可观测环境

## 1. 准备Grafana + openTelemetry + Prometheus


### 文件夹结构：

![p1][pic1]


### 文件内容：

```yaml
# datasources.yaml
apiVersion: 1

datasources:
- name: Prometheus
  type: prometheus
  url: http://prometheus:9090 
  isDefault: true
  access: proxy
  editable: true

# otel.confg.yml 
extensions:
  health_check:
  otlp_encoding:
    protocol: otlp_json

receivers:
  otlp:
    protocols:
      grpc:
        endpoint: 0.0.0.0:4317
      http:
        endpoint: 0.0.0.0:4318

processors:
  batch:

exporters:
  debug:
    verbosity: detailed
  prometheus:
    endpoint: 0.0.0.0:8889
  otlphttp:
    endpoint: http://loki:3100/otlp
  rabbitmq:
    connection:
      endpoint: amqp://rabbitmq:5672
      auth:
        plain:
          username: guest
          password: guest
    encoding_extension: otlp_encoding
    routing:
      exchange: logs

service:

  pipelines:

    traces:
      receivers: [otlp]
      processors: [batch]
      exporters: [debug]

    metrics:
      receivers: [otlp]
      processors: [batch]
      exporters: [debug, prometheus]

    logs:
      receivers: [otlp]
      processors: [batch]
      exporters: [debug, otlphttp, rabbitmq]

  extensions: [health_check,otlp_encoding]


# prometheus.yml
global:
  scrape_interval: 15s
  scrape_timeout: 10s
  evaluation_interval: 15s
alerting:
  alertmanagers:
  - static_configs:
    - targets: []
    scheme: http
    timeout: 10s
    api_version: v1
scrape_configs:
- job_name: prometheus
  honor_timestamps: true
  scrape_interval: 15s
  scrape_timeout: 10s
  metrics_path: /metrics
  scheme: http
  static_configs:
  - targets:
    - localhost:9090
- job_name: OpenTelemetry
  scrape_interval: 1s # poll very quickly for a more responsive demo
  static_configs:
  - targets: ['otel-collector:8889']

```

### docker-compose.yml
```yaml
services:
  prometheus:
    image: prom/prometheus
    container_name: prometheus
    command:
      - '--config.file=/etc/prometheus/prometheus.yml'
    ports:
      - 9090:9090
    restart: unless-stopped
    volumes:
      - ./prometheus:/etc/prometheus
      - prom_data:/prometheus
  grafana:
    image: grafana/grafana
    container_name: grafana
    ports:
      - 3000:3000
    restart: unless-stopped
    environment:
      - GF_SECURITY_ADMIN_USER=admin
      - GF_SECURITY_ADMIN_PASSWORD=grafana
    volumes:
      - ./grafana:/etc/grafana/provisioning/datasources
  loki:
    image: grafana/loki:latest
    ports:
      - "3100:3100"
    command: -config.file=/etc/loki/local-config.yaml
  otel-collector:
    image: otel/opentelemetry-collector-contrib
    volumes:
      - ./otel/otel-collector-config.yaml:/etc/otelcol-contrib/config.yaml
    ports:
      - 1888:1888 # pprof extension
      - 8888:8888 # Prometheus metrics exposed by the Collector
      - 8889:8889 # Prometheus exporter metrics
      - 13133:13133 # health_check extension
      - 4317:4317 # OTLP gRPC receiver
      - 4318:4318 # OTLP http receiver
      - 55679:55679 # zpages extension
  rabbitmq:
    image: rabbitmq:4.0.7-management
    ports:
      - '4369:4369'
      - '5551:5551'
      - '5552:5552'
      - '5672:5672'
      - '25672:25672'
      - '15672:15672'
volumes:
  prom_data:
```

# C#代码

## 1. 引入依赖
```xml
<PackageReference Include="OpenTelemetry.Exporter.Console" Version="1.11.0" /> // debug
<PackageReference Include="OpenTelemetry.Exporter.OpenTelemetryProtocol" Version="1.11.0" />
<PackageReference Include="OpenTelemetry.Instrumentation.EventCounters" Version="1.5.1-alpha.1" />
<PackageReference Include="OpenTelemetry.Instrumentation.Process" Version="1.10.0-beta.1" />
<PackageReference Include="OpenTelemetry.Instrumentation.Runtime" Version="1.10.0" />
```

## 2. 示例代码
```csharp
// See https://aka.ms/new-console-template for more information
using System.Diagnostics.Metrics;
using System.Reflection.PortableExecutable;
using OpenTelemetry;
using OpenTelemetry.Metrics;
using OpenTelemetry.Resources;
using OpenTelemetry.Logs;
using RabbitMQ.Client;
using RabbitMQ.Client.Events;
using System.Text;
using Microsoft.Extensions.Logging;

using var meterProvider = Sdk.CreateMeterProviderBuilder()
    .AddProcessInstrumentation()
    .AddRuntimeInstrumentation()
    .AddOtlpExporter(opt =>
    {
        opt.Endpoint = new Uri("http://192.168.50.6:4317/");
        opt.Protocol = OpenTelemetry.Exporter.OtlpExportProtocol.Grpc;
        opt.BatchExportProcessorOptions.ExporterTimeoutMilliseconds = 1000;
        opt.BatchExportProcessorOptions.ScheduledDelayMilliseconds = 1000;
        opt.TimeoutMilliseconds = 1000;
    })
    .ConfigureResource(builder =>
    {
        builder.AddService(serviceName: "hello", autoGenerateServiceInstanceId: false, serviceInstanceId: "hello.t");
    })
    .AddEventCountersInstrumentation(config =>
    {
        config.AddEventSources(
            // "System.Runtime",
            // "Microsoft-AspNetCore",
            // "Microsoft.AspNetCore",
            // "System.Net",
            "System.Net.Sockets");
        config.RefreshIntervalSecs = 1;
    })
    .AddConsoleExporter()
    .AddMeter("TestMeter")
    .Build();

var meter = new Meter("TestMeter");
var counter = meter.CreateCounter<int>("my_test_counter", null, null, tags: [new("job", "test_job"), new("name", "test_name")]);
var connectionFactory = new ConnectionFactory()
{
    Uri = new Uri("amqp://guest:guest@localhost:5672")
};

var connection = await connectionFactory.CreateConnectionAsync();
var channel = await connection.CreateChannelAsync();
await channel.ExchangeDeclareAsync("logs", ExchangeType.Direct, false, true, null);
var queueResult = await channel.QueueDeclareAsync("test-logs", true, false, false,
    new Dictionary<string, object?>()
    {
        {"x-expires", 60*1000},
    });
await channel.QueueBindAsync(queueResult.QueueName, "logs", "otlp_logs");

// List<Task> tasks = new();1
// foreach (var i in Enumerable.Range(1, 50))
// {
//     var machine = $"Machine_{i}";
//     tasks.Add(Task.Run(async () =>
//     {
//         var c = meter.CreateCounter<int>("machine_counter", null, null, tags: [new("machine", machine)]);
//         var timer = new PeriodicTimer(TimeSpan.FromSeconds(1));
//         while (await timer.WaitForNextTickAsync())
//         {
//             c.Add(Random.Shared.Next(1, 10), [new("machine", machine)]);
//         }
//     }));
// }
var consumer = new AsyncEventingBasicConsumer(channel);
consumer.ReceivedAsync += (model, ea) =>
{
    byte[] body = ea.Body.ToArray();
    var message = Encoding.UTF8.GetString(body);
    Console.WriteLine($" [x] {message}");
    return Task.CompletedTask;
};

_ = channel.BasicConsumeAsync(queueResult.QueueName, autoAck: true, consumer: consumer);

// await Task.WhenAll(tasks);
var loggerFactory = LoggerFactory.Create(builder =>
{
    builder.AddOpenTelemetry(logging =>
    {
        logging.AddConsoleExporter();
        logging.AddOtlpExporter(opt =>
        {
            opt.Endpoint = new Uri("amqp://guest:guest@localhost:5672")
            opt.Protocol = OpenTelemetry.Exporter.OtlpExportProtocol.Grpc;
            opt.BatchExportProcessorOptions.ExporterTimeoutMilliseconds = 1000;
            opt.BatchExportProcessorOptions.ScheduledDelayMilliseconds = 1000;
            opt.TimeoutMilliseconds = 1000;
        });
    });
});

var logger = loggerFactory.CreateLogger<Program>();

while (true)
{
    counter.Add(1, [new("second_label", "label_test")]);
    logger.LogInformation("Random Id {id}", Guid.NewGuid());
    await Task.Delay(1000);
}


Console.WriteLine("Hello, World!");


```

[pic1]:data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAATMAAACqCAIAAABOJD82AAAACXBIWXMAAA7EAAAOxAGVKw4bAAAAEXRFWHRTb2Z0d2FyZQBTbmlwYXN0ZV0Xzt0AAAAXdEVYdFVzZXIgQ29tbWVudABTY3JlZW5zaG9093UNRwAAF2RJREFUeJztnX9oG/fdxz971mxzZsuRlcQJ83ZTtV7jPBpTnglcYhzRgyIxY8aeZgOpf1QdowroKpdd0R/Grkkd/IeKIDES1H90aH9M5oH1GcPoqcTgOicoxKA0GhWxe0XVjrk4rWs7kV2XYjOeP+5OOkl3kpzI0ln+vBDh9L3vL7d++/v5fu/7fd+3zpmtgCCIxnjqm2++bnUfEAQp5z9a3QEEQRRAZSKIFkFlIogWQWUiiBZBZSKIFnmqgXW9+PJv+35sLHxdXPjgfvpe/uFmA5tQhXRPeKxdkE/PjkW5ZjSIIAdKI5UplyUADNieH7A9X5bnfvrDv/31Lw1sVIByWCE1459DUSJtgqoyj/foYCO/o3Cno7sHHm0oPwW9cXWiUT3bLxufoyyR9kFNmRdeHf/dj7jo+DvJUnF22F57y00sh994N6VSUndC/8Ivf1U2fpbx3h//sPLPnNIduzc4QgAA5OIxcJgz/pkEkO4JjyGbMlisOj5Gh1eEwBUAgI/RYZZ0TfosOgAiFBhKzV6NEL6QgwAAgHxq9mokC2ByTntOLcRhxEEAyCJeqiIngmgEtRWge9fHo6vnXNeuDB4vJnbYXnvrJWI58qaqLAHgvOVCdVlCRdwrQbomRyBG+xnaz2T6h+V5jCaI+Bk6zILp54bsLO1naH8sRwwzFHDRq3ScBz5G+69GsmDvh3k/Q/uZmTRYXU5SrIAY6c/QfoaeTYHF6TYBgFpOBNEA6muzO8m3x+Ti7LBdmXiJWP7Tm+8uKMW4Bfp/ZqneZP7h5p2/swo3qBELpOLinUQ4Jh9Uc0lpDpmdC4prPGyGV6glEZ5JAAAAt8jli8n8fJgFAMje5bZ0BqJKTgTRAFVXgHaSb4/B69d+c+3Kd96HF379k2xNWepO6HUn9NWbXPpHWvXe5modIWUh4gWA3FLFbZNz2mPVCddblYM7t7oBhrpyIkjrqLU2u5O8Pg6vX3vZ9e/FSC1ZAsB5y4WaTSoPmAL6syYAQZymXoNSDrs3aNucpcOccG0uuy1MKf1MQrh2qXej/pwI0nzq2Gmwk7z++1dfeaO2LKGOUHZx4QPVe2yG11kdlPDF7pBGsxLIsz2w/kCIZikzUXGfOKXbWhPCYHKAVKph/zkRpPk08nlm/uFmzVD2fvqe+s1EePbshCcUGAaAXDyWI8pHRAAuEk1Ne4IhBwDwucp5Jjsz3x/0BawAkOf5arPH+nMiSPP5lvGZ/2x1H5QxOaddEJnCzQPIkUSr+2ZJt8sK2bsoS+SI0sho9omRL7oCH6NxByxyZNFuNIsgRxmtRrMIcrRBZSKIFkFlIogWQWUiiBZBZSKIFkFlIogWad7zzNHekz84VtLcZ7t7Nz7/smkdQJBDRJOUeVnfPXq65OzIne2dy/puAEBxIkglLYtmV3Z3Ly1/OnraMNp7cn8lKSbgsx9MpzSHyTl9dH5YRE4r55mPL84qUExgUjATQZBDTItXgArifK7zeO3cCHJkaNmO9sv6bmGeKdB37FjV7CWeegXKzO9AdA8xeoJWPkaHWSV3PLJgvZeLM0G2JAW2UjPCubOSM2ike8IJ0auRbJmLHyvzK5HqL7RYqKrQ1cG1QorgzDD2wBEaXEttWK0EAPDz/hkQf0x+3i96FCFHFE2dNVFD8tRjQZAo8BkAkMzvEgCka9Lnct6emhvzf84EhtYli8rKDEavh8zOyk+x2L0e60aMnmKFhnxeig6r26EAGE0w42dE9XpIbpaJZAEop1tIcchalFfFZnKOoYsm4LIAQJkJfiHMAeWALqshydBhsHuDI4FgLi5de6lE1W4gbU5To9mnP/pY/vnz5qO6iql66tU0v6vMkNvM6/R9xRwmp42QbPUAuOitHGGuvuJScPEjB0hIz4kutexcJAt2hxXS8WKLPb0yp0w2w4uefUCZjXxGHBK3UvMsAEBiiS+5LimLHD2aOmYG+s7Iv+5jbqniqVfT/K4iAxe9Ou8NhgLDgos0AIDkBgQAACvrW0Nn61tAMup160vlJ0h1Fk+oaIXEGwEKORJL/Eg/BSxr7ydySzN1tYEcWZqqTGFiWRgq72zvyNOroeSpV9P8TiVDIswkAOze4LRrZWwRoOuUTD99hq7NTBagPnEazpAy6QEU565KsPHUoMMOYCb4TLiu+pGjSwvWZv0rD8o+NQqoeOrVNL+rmkEMa7N3uS1ixCtWTrqGxDgz+/l6F3nRJCQ6xfWhUhJLvE6yewfK6TZBYok3OsoeP5LuiSAjVs/dzurNXrNBingRRJUWrAB9+tNn91lC2VNP2fyOnU8P+cS1WYUMRUOTrdTMFAsAkalZkCovhriFegDy6Vhqa0ihW+wMDb6QJ2gFAODn/QDZmZkzk75AcAQAxBeolJTgFjmDh+Ti6KKC1KJJbiPPdR6PGn9YJYMr969CcNvGoCEgUidNGjPvbO/c+GK9bEd7gc92946CLAHsDut6kkFZIrVpXjR7xHeuCzsQ+BiNDymRekDvPATRInhyGkG0CCoTQbQIKhNBtAgqE0G0CCoTQbQIKhNBtAgqE0G0CCoTQbRII/cAvfjyb/t+bCx8XVz44H76Xv7hZgObUIZ0T3gMSfXjVwhy6GikMuWyBIAB2/MDtufL8txPf/i3v/5lf/Wi8JCjh6oyj/foYCOvtM28o7sHHm18rVjqxtWJRvUMQY4yasq88Or4737ERcffSZaKs8P22ltuYjn8xrtK5h4AALoT+hd++auy8bOM9/74h5V/5pTuyF4IL/jHUUxg2AgAjmBoUHSjK+TJp2fH8I3xSFuitgJ07/p4dPWc69qVQZlXT4fttbdeIpYjb6rKEgDOWy5UlyVUxL0SpGtypCc142doP0PHYWTCSQIb9M+mtiAXZ+ipOa7oo8fQ/lnO5JHsAhCkvVBfm91Jvj0mF2eH7crES8Tyn958d6HqUcr+n1mq3QbIP9y883elOSM1YoFUVDpVzMZTIPp9lOcRffS429m84QxazCHtSNUVoJ3k22Pw+rXfXLvynffhhV//JFtTlroTet0JffUml/6RVr0n98jjVjd0ZkIy5pK1YRUNPgAAgO8DWKneIIIcPmqtze4kr4/D69dedv17MVJLlgBw3nKhZpPKA6aAzCMPyLM9+XW+Mk+lfzkOm0jbUcdOg53k9d+/+sobtWUJdYSyiwsfqN5jM7zO6nKKOqMcVuBulw+YbIYvWt0hSNvSyOeZ+YebNUPZ++l76jcTYQa8otud7L0gXCTJh6S1WbmPnuxtJQjSXqDbCIJoEdw3iyBaBJWJIFoElYkgWgSViSBaBJWJIFoElYkgWgSViSBaBJWJIFoElYkgWqR57wIb7T1Z9pa+z3b3jvgLwhBEjSYp87K+e/S0QZ5yZ3vnsr4bjvzb+xBEkZZFsyu7u5eWPx09bRjtPXlQbZDuieCTmR4Ua7B7g9OuQ3XczOScDgRDgUm3iXRPNLfzJud0wGdvXnvtSPOi2UoEcd489zQ83sipbU89yhcaXJtp3Yvf7Q4rpGdpwSdpimlRL5DHpcUrQIWR87nO47VzI/tj/QHalx1aWjZmXtZ3C/NMgb5jx6pmb7ynnsk57bHqAIqHPIUXtgMAQC5ebShWbIh0TfosOgCArdT/Zsn/tugACF/AKlalVLndG7RtptYtVmPxMKrKT83H6DArhAnWLhBaEYtQvtDg2nyWHLHopMqlbITwH+fuxWJwUaghn4pz5CBEiu1STGBovXDeVRrzjYU+lLaY2rBaCfF/B4h5Ku0mkMelldFs3UieetKvxYQzNzUX9K/Io1nJU0/89WWoqlGuyTntIblZJpIFoJxuIcWhTwkpQDGBSTevfCZbsSHSNekzcUIP7S7nSvQq/UAWzapXrrOcWvAzQaU+2r3BYkOuPgCwez3WjRg9xQrd8HkpOswCAHRZbTBL+zmgfCGHz87ORKaYVW/QvCSq8WKxTg+ZFUJc0j3h0YHcBJHN8MO2ARKyHADY+4lccoYDSv7Dul13xb9EXVZDkqHDYPcGRwLBXFy69lKJsDZnF4eNpkazT3/0sfzz581HdRVrgKee3RsMBYKhQDDkpQCAHCAhPScKj52LZIVZmZQC7HwayAHFJRPFhsiLpmIPE9Hy0a9K5fl0XBxkxAUbYc0GACgzwc+Lv+VcJMqCyWkrpgAXvZUjzOIqy1YqIgiGzeQELyVFTE4bwS+IgzwXiabypfcTS7xO3yf8mGaCz7AAwAbDxR+2mHUrNc+KRUque3oP1SqZhmnqmBnoOyP/uo+55ZN66iXCjDzKMup160vlwaN8VsY92NT1q7nyVTYEhq7NTFXTE6XKSzuQnRvzzxW/mnoNW2vlZtklKSvrW0MVIlxZ36rWDaisUw6byTnMdmATlNnIZ4SRvBilA+TVXQ+RBtNUZQoTy8JQeWd7R55ejcZ76hnOkGXakKeQZ/T5TTWzzMqGqNIe1miuauUyuk4Zy7pYktIn/jkgatekXANxSgdrpbfZ+fTQCAXQT+SWZkCQpf4W7RfjZ/d+mkKeiBaszfpXHpR9ahRovKdeYonXWZxuYcChnG5TaQpQIxbgFhXXkBQbKumhvdBVxeaqVS4je5fbKjREul1UaQqQriEjn9nfckv2LrdF2MQHm6R7UNI05QtNiH3mFjlDv8/cIwaoRr1O+iNCXjTp9tUa8kS0YAXo058+u88SjffUY2do8IXEoJSf9wNk5SnViis2VNJDPkYDABtPDXqktdl6K5fBRaZmQWoon54tS5FWa/cFF5mKMQFPyAIA+VQ8lR+syJK9y/V4yGxcnDOHY2Ypf47PV+RGDowmeec913k8avxhlQyu3L8KwS3SHChfqD9TIW/SPeGEKHqFtpomjZl3tndufLFetqO9wGe7eyjLZkMxDiIXn6lIdlg3btEoy5bTvGgWd663HtlGBWGbRMl4KWze4Of9+EBSA6ATNIJoETw5jSBaBJWJIFoElYkgWgSViSBaBJWJIFoElYkgWgSViSBaBJWJIFqkkXuAXnz5t30/Nha+Li58cD99L/9ws4FNHCit9tRCkCKNVKZclgAwYHt+wPZ8WZ776Q//9te/NLDRJ0M8f4gGGYjWUFXm8R4dbOSVtpl3dPfAo42vFUvduDrRqJ4hyFFGTZkXXh3/3Y+46Pg7yVJxdthee8tNLIffeDelUlJ3Qv/CL39VNn6W8d4f/7Dyz0rbC7s3aNuMcaZh0dxNPMdIuic8hmzKYLHqhEOJCj50sjzCXu3FnwvWeHJ7uzLPO+nrcCgwlJq9GgEAAKNkrlEsWHTZk9wMSs5PUUzAnCmkCx1T8cJDkHpRWwG6d308unrOde3KoMyrp8P22lsvEcuRN1VlCQDnLReqyxIq4l4ZOssQRBnaz9DxTaun6PNtNEHEz9BhtuBDR/sZ2h8Dx6TkFSDlmU2BxRNyFa5FMwHJ846h/bOcycNQkAgzM+k88DHaLx1H7LLaYI72M3Sc11kcdgAAivGQnNBcHEYmyv0Kipic0w6Y9zO0n6FRlsgTor42u5N8e0wuzg7blYmXiOU/vfnuQtWjlP0/s1RvMv9w887f1SZ2+aKlXTy1RZglW49cUvxdr+JDJ+bJ3uW25Nc6AwE1zfVEKkzoSNeQkb8ldSmT6zql+kcn+/l6Fd86BNkXVVeAdpJvj8Hr135z7cp33ocXfv2TbE1Z6k7odSf01Ztc+kd9Dmzc6gYYlG7UbXJX3rkKz7sqA5vMhI4YlrxFACC/rqo9NjjbO+0Jhur1E0EQdWqtze4kr4/D69dedv17MVJLlgBw3nKhZpPqA2Ypah55j+VDB/DYDuIKju9qdnWCM6XJOe3x2dGtHHkS6thpsJO8/vtXX3mjtiyhjlB2ceGDqvel2R2QLqeSR95j+dAB7NdcrwC3yEGxOQl+LS+5MNu9w+XxLYa1yJPTyOeZ+YebNUPZ++l71etIr5kDwREAAH7er7SO8jg+dAAqnndc9FYuULI2W052bixeaE5yrMvOLfDBkUBwBCAXj+UIM4B8CRdycQajWeSJ0JTbiN0btG3W8bIgBGl7cN8sgmgRVCaCaBFNRbMIgojgmIkgWgSViSBaBJWJIFoElYkgWgSViSBaBJWJIFqkee8CG+09WfaWvs929/AFYQiiSJOUeVnfPXq65EjXne2dy/puwLf3IYgSLYtmV3Z3Ly1/OnraMNp7slV9qBPKF6piZdBkTM7pQNHqAWlbWjnP1LA4SddkaP9HxhCkYbR4Baggzuc6j9fOjSBHhuatAJVxWd8tzDMF+o4dU8/bnp56pcbT4gm4B47Q4Fpqw2olxGpB7MxjGjIgh5XD8tSkDT312Eyui7womTOYCX5BEHyX1bDE0H5mnidGAkFz4Rqj6yNFU5X59Ecfyz9/3nxUd9F29NRjM7zQBwDKbOQzCamteRYAILHEl1z39GplFQppAk2NZgN9Z+RfH3Nu2T6eeoklfqSfApa19xO5pZk6uoocGZqqTGFiWRgq72zvyNPrpY089dh4atBhBzATfCa8/x4gbUwL5pn+lQdlnzoKtZOnnuyRDHc7qzd7zYZ0HFd3kBJasDb76U+f3X+hdvLUIweK1XCLnMFDcnE0JUNKaZLbyHOdx6PGH1bJ4Mr9qxDcVtDOnnom57QLIvgeFKSMJo2Zd7Z3bnyxXrajvcBnu3vqsmxv7A7repJBWSLlNC+axZ3rZQjbD/gYja/VRSpB7zwE0SKHZQ8QghwtUJkIokVQmQiiRVCZCKJFUJkIokVQmQiiRVCZCKJFUJkIokUOVplfPWP78pLnQJs4HJDuiSCDngRI/RzI7ry9zlMAsNd18uuz545t46Y8BNk3B6LMz4fHv7315Tdn+7//ycLXZ853w3sA8Oi/Xvzu6v3vrS4dRIsI0mY0WJlfPWP7+uy5767eF4ZK4d8Hv5jo+uTmbufJ7UtXOj+5+X3u5lPbayoVkJJFXcFmrmBsVzQfMDmnPae4tN5q0QknKm8PCKVKbfWSMXAIx5dlrgWy05LVXPAqU0QoJjC0Xjj8KfjfRcFdoz8Isk8aPM/c6zq513l6r/P0budJABD+FW91nt7rPPXwwot7XWq+z6Rr0mfiZvwM7Wfms2LKSE9KSCm1qCNImKP9zEwarJ6gu3DtKnrYGR3mjJ+h/cxMWi8VpBiPdT3OyJ3ySlzwRBFW8cVjM7yu4P1l7ycki7Da/UGQfdBIZX55yfPwwovfnO3/5mz/V8/YhH+Fiy8veYTg9gf/M6oa0JIXTZCKimeIE9E5TjC2k1KAjaegYAMpekByi1xefi3zsMvFxbGOi94S/CNJ15CRjwXFU1dcJMkb+ymAlfUtycNO6EdVX7zEEq/T9wGAYEWZYevtD4Lsg0ZGsx2ry189Y1NtaXvt5M3ZauX7DF2bmfLYb3O1mMKtbujMBICSPVdVioZ3JbZd/Fp+sJcENjIVYwLBkKPgH63ki1foBpvJOcx2YBOU2chngvvtC4LUQyOV+e3tL57aXhMWZisx3HyndhX6s3INlKeouubVQtL8AOj0MtNK4pRuIyPYyAb9LADFBCbd/NXbir54Rdj59NAIBYBWlMjB0cho9nurS52f3FS81ft/U7VXZdkMrytMzOwuJ1maApRD0TVPDeOgVJV3WPBZ5ha5PDEsPVck3YNEbkluKCCGtcq+eLI3gnGLnKHfZ+4RbZoRpPE0eG22+8P3vrt6f/uZS3udpwHgqe0vOlaXv//JQn2lE2EGvEFfwAoAwMdoAE6eIn8pSB3ksuAOBHViQRYAIDs3NgvTnmDIAVB89wnFBEQHynx6dowFACVfPDnZu1yPh8yi4x1yYLSp24jw1ESaNB5I/U6I4hMR5MDAfbOPA+WwbtxCWSIHSMve0ndYEUJfft6PU0zkIGnTaBZBDjkYzSKIFkFlIogWQWUiiBZBZSKIFkFlIogWQWUiiBZBZSKIFkFlIogWQWUiiBZBZSKIFkFlIogWQWUiiBZBZSKIFkFlIogWQWUiiBZBZSKIFkFlIogW+X9qDNfVcGfxtwAAAABJRU5ErkJggg==