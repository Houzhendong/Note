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

# 