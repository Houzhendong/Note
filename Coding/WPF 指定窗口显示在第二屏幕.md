# WPF窗口显示在扩展屏

要指定WPF程序的window显示在第二屏幕，可以用下列两种方法：

1. 引入`System.Window.Forms`命名空间，将window的`Top`和`Left`属性写为第二屏幕的Top和Left

   ```c#
   public void RunAtSecondScreen()
   {
       if(System.Windows.Forms.SystemInformation.MonitorCount > 1)       
       {
           var workingArea = System.Windows.Forms.Screen.AllScreens.Where(s => !s.Primary).FirstOrDefault().WorkingArea;
           var secondWindow = new MainWindow
           {
               WindowStartupLocation = WindowStartupLocation.Manual,
               Top = workingArea.Top,
               Left = workingArea.Left  
           };
           secondWindow.Show();    
       }
   }
   ```

   

   

2. 调用Win32Api

   ```c#
   public partial class MyWindow : Window
   {
       readonly Rectangle screenRectangle;
   
       public MyWindow( System.Windows.Forms.Screen screen )
       {
           screenRectangle = screen.WorkingArea;
           InitializeComponent();
       }
   
       [DllImport( "user32.dll", SetLastError = true )]
       static extern bool MoveWindow( IntPtr hWnd, int X, int Y, int nWidth, int nHeight, bool bRepaint );
   
       protected override void OnSourceInitialized( EventArgs e )
       {
           base.OnSourceInitialized( e );
           var wih = new WindowInteropHelper( this );
           IntPtr hWnd = wih.Handle;
           MoveWindow( hWnd, screenRectangle.Left, screenRectangle.Top, screenRectangle.Width, screenRectangle.Height, false );
       }
   
       void Window_Loaded( object sender, RoutedEventArgs e )
       {
           WindowState = WindowState.Maximized;
       }
   }
   ```

需要注意，使用这两种方法时，如果要显示的window需要最大化，那么要在Loaded事件中更改`WindowState`，而不是一开始在设计时写好，不然两种方法都不会生效。