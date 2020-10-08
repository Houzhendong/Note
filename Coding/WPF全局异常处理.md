```c#
//UI线程未捕获异常处理事件（UI主线程）
this.DispatcherUnhandledException += App_DispatcherUnhandledException;
//非UI线程未捕获异常处理事件(例如自己创建的一个子线程)
AppDomain.CurrentDomain.UnhandledException += CurrentDomain_UnhandledException;
//Task线程内未捕获异常处理事件
TaskScheduler.UnobservedTaskException += TaskScheduler_UnobservedTaskException;
```