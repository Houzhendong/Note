# WPF无边框窗口最大化

​	WPF窗口设置`WindowsStyle = none`之后，窗口最大化会覆盖任务栏，使用以下代码可以解决

1. 在Window的构造函数中增加

   ```c#
     SourceInitialized += (sender, e) =>
     {
         handle = new WindowInteropHelper(this).Handle;
         HwndSource.FromHwnd(handle).AddHook(new HwndSourceHook(WndProc));    
     };
   ```

2. 实现`WndProc`方法

   ```c#
    private IntPtr WndProc(IntPtr hwnd, int msg, IntPtr wParam, IntPtr lParam, ref bool handled)
   {
        switch (msg)
        {       
            case 0x0024:
                WmGetMinMaxInfo(hwnd, lParam);
                handled = true;
                break;
        } 
        return IntPtr.Zero;
   }
    
   private void WmGetMinMaxInfo(IntPtr hwnd, IntPtr lParam)
   {
   	MINMAXINFO mmi = (MINMAXINFO)Marshal.PtrToStructure(lParam, typeof(MINMAXINFO));
   
   	int MONITOR_DEFAULTTONEAREST = 0x00000002;
   	IntPtr monitor = Win32Helper.MonitorFromWindow(hwnd, MONITOR_DEFAULTTONEAREST);
   
   	if (monitor != IntPtr.Zero)
   	{
   		MONITORINFO monitorInfo = new MONITORINFO();
           Win32Helper.GetMonitorInfo(monitor, monitorInfo);
           RECT rcWorkArea = monitorInfo.rcWork;
           RECT rcMonitorArea = monitorInfo.rcMonitor;
           
           mmi.ptMaxPosition.x = Math.Abs(rcWorkArea.left - rcMonitorArea.left);
           mmi.ptMaxPosition.y = Math.Abs(rcWorkArea.top - rcMonitorArea.top);
           mmi.ptMaxSize.x = Math.Abs(rcWorkArea.right - rcWorkArea.left);
           mmi.ptMaxSize.y = Math.Abs(rcWorkArea.bottom - rcWorkArea.top);
       }
           
       Marshal.StructureToPtr(mmi, lParam, true); 
   }
   ```

3. 定义`Win32Helper`帮助类和Win32Api结构体

   ```c#
   public class Win32Helper
   {
       [DllImport("user32")]    
       internal static extern bool GetMonitorInfo(IntPtr hMonitor, MONITORINFO lpmi);
       
       [DllImport("user32")]
       internal static extern IntPtr MonitorFromWindow(IntPtr hwnd, int flags);
   }
   
   [StructLayout(LayoutKind.Sequential)]    
   public struct POINT    
   {
       public int x;
       public int y; 
   }
       
   [StructLayout(LayoutKind.Sequential)]
   public struct MINMAXINFO
   {
       public POINT ptReserved;
       public POINT ptMaxSize;
       public POINT ptMaxPosition;
       public POINT ptMinTrackSize;
       public POINT ptMaxTrackSize;
   }
   
   [StructLayout(LayoutKind.Sequential, CharSet = CharSet.Auto)]
   public class MONITORINFO 
   {    
       public int cbSize = Marshal.SizeOf(typeof(MINMAXINFO));
       public RECT rcMonitor = new RECT();
       public RECT rcWork = new RECT();
       public int dwFlags = 0;    
   }
   
   [StructLayout(LayoutKind.Sequential, Pack = 0)]
   public struct RECT    
   {
       public int left;
       public int top;
       public int right;
       public int bottom;    
   }
   ```

   

