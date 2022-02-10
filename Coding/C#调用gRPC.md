# C#调用gRPC

## 配置服务端
1. 使用Visual Studio开发  
    服务端部署在Asp.net程序上, 所以只需要使用Visual Studio 创建gRPC项目即可.

2. 使用VsCode开发  
    1. 在项目中添加`Grpc.AspNetCore`nuget包
    2. 编写proto文件
    3. 在项目文件中添加:
        ```xml
        <ItemGroup>
            <Protobuf Include="[path]\[protofile].proto" GrpcServices="Server" />
        </ItemGroup>
        ```
    4. 编译项目, 程序会自动生成proto文件对应的类

通过override基类的方法来完成gRPC服务端的逻辑

## 配置客户端

1. 使用Visual Studio 开发
    
    * **方法一**  
        右键项目依赖项→管理连接的服务  
        在服务应用中添加服务引用, 选择文件时指定proto文件(proto文件是gRPC通信的contract,所以内容要与服务端保持一致),类型选择客户端, 程序会自动生成客户端代码

    * **方法二**  
        添加nuget包`Google.Protobuf`, `Grpc.Net.ClientFactory`, `Grpc.Tools`  
        在项目文件中添加:  
        ```xml
        <ItemGroup>
            <Protobuf Include="[path]\[protofile].proto" GrpcServices="Client" />
        </ItemGroup>
        ```  
        编译项目, 自动生成client类
2. 使用Vscode 开发

    同1.中的方法二


客户端示例代码
```C#
using var channel = GrpcChannel.ForAddress("https://localhost:7042");
var client = new Greeter.GreeterClient(channel);
var reply = await client.SayHelloAsync(new HelloRequest { Name = "GreeterClient" });
Console.WriteLine("Greeting: " + reply.Message);
```



**参考**:  [C#调用gRPC](https://docs.microsoft.com/zh-cn/aspnet/core/tutorials/grpc/grpc-start?view=aspnetcore-6.0&tabs=visual-studio)

