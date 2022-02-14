# 在WPF程序中使用Generic Host

1. 添加nuget包`Microsoft.Extensions.Hosting`
2. 删掉`App.xaml`文件中的`StartupUri`
3. 编辑`App.xaml.cs`文件

    ```C#
    private IHost _host;

    public App()
    {
        _host = new HostBuilder()
                    .ConfigureServices((hostContext, services) =>
                    {
                        // 这里初始化依赖注入
                    })
                    .Build();

    protected override async void OnStartup(StartupEventArgs e)
    {
        await _host.StartAsync();
        var mainWindow = _host.Services.GetService<MainWindow>();
        mainWindow.Show();
        base.OnStartup(e);
    }

    protected override async void OnExit(ExitEventArgs e)
    {
        using (host)
        {
            await host.StopAsync();
        }

        base.OnExit(e);
    }
    ```

# Generic Host 使用Configure

使用`Configration`，可以保存一些应用配置参数到文件中，在程序运行时读取相应参数

### 添加config文件

* 向项目添加`appsettings.json`文件

* 设置文件属性为`Content(内容)`、`如果较新则复制`,或者直接编辑项目文件, 添加:
    ```xml
    <ItemGroup>
        <Content Include="appsettings.json">
            <CopyToOutputDirectory>Always</CopyToOutputDirectory>
        </Content>
    </ItemGroup>
    ```

### 添加nuget引用

> `Microsoft.Extensions.Configuration`  
> `Microsoft.Extensions.Configuration.Json`

### 初始化Configuration

```C#
Host.CreateDefaultBuilder(args)
//--------------
    .ConfigureAppConfiguration((context, config) =>
        {
            config.AddJsonFile("appsettings.json", optional: true, reloadOnChange: true)
                  .SetBasePath(context.HostingEnvironment.ContentRootPath);
        })
//----------------
    .Build();
```

### 使用config读取配置

可以通过`IConfiguration[key]`或者`IConfiguration.GetValue()`来访问节点参数


```C#
using Microsoft.Extensions.Configuration;
using Microsoft.Extensions.DependencyInjection;
using Microsoft.Extensions.Hosting;

using IHost host = Host.CreateDefaultBuilder(args).Build();

// Ask the service provider for the configuration abstraction.
IConfiguration config = host.Services.GetRequiredService<IConfiguration>();

// Get values from the config given their key and their target type.
int keyOneValue = config.GetValue<int>("KeyOne");
bool keyTwoValue = config["KeyTwo"];
string keyThreeNestedValue = config.GetValue<string>("KeyThree:Message");

// Write the values to the console.
Console.WriteLine($"KeyOne = {keyOneValue}");
Console.WriteLine($"KeyTwo = {keyTwoValue}");
Console.WriteLine($"KeyThree:Message = {keyThreeNestedValue}");

// Application code which might rely on the config could start here.

await host.RunAsync();

// This will output the following:
//   KeyOne = 1
//   KeyTwo = True
//   KeyThree:Message = Thanks for checking this out!
```

# Generic Host集成Serilog

### 添加nuget引用

> `Serilog.Extensions.Hosting`  
> `serilog.sinks.console` `Serilog.Sinks.xxx`

### 初始化Serilog

```C#
IHost host = Host.CreateDefaultBuilder()
                 .UseSerilog((context, config) =>
                 {
                    config.Enrich.FromLogContext()
                          .MinimumLevel.Debug()
                          .MinimumLevel.Override("Microsoft", Serilog.Events.LogEventLevel.Debug)
                          .WriteTo.Console(outputTemplate: "[{Timestamp:HH:mm:ss} {Level:u3}] {SourceContext}.{Method} {Message:lj}{NewLine}{Exception}");
                 })
                 .Build();
```

### 使用log

在配置中override`Microsoft log`后 ,直接使用`Microsoft.Extensions.Logging`提供的`ILogger<T>`接口记录的日志将由Serilog来处理

```C#
ILogger<Test> logger = host.Serivces.GetService<ILogger<Test>>();
logger.LogDebug("Test");
```

# Generic Host注册gRPC Client

使用Generic Host 提供的Ioc容器,可以注册`gRPC Client`,无需每次实例化Client.


```C#
IHost host = Host.CreateDefaultBuilder()
                .ConfigureServices((context, services) =>
                {
                    services.AddGrpcClient<Greeter.GreeterClient>(opt => opt.Address = new Uri("https://localhost:5001"));
                }).Build();
```
使用

```C#
Greeter.GreeterClient client = host.Services.GetRequiredService<Greeter.GreeterClient>();
var res = client.SayHello(new HelloRequest { Name = "Test" });

Console.WriteLine($"{res.Message}");
```

原本手动实例化的写法:
```C#
var channel = GrpcChannel.ForAddress("https://localhost:5001");
var client = new Greeter.GreeterClient(channel);

var res = client.SayHello(new HelloRequest { Name = "Test" });

Console.WriteLine($"{res.Message}");
```