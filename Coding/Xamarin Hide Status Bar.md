# xamarin隐藏状态栏的方式

在Xamarin.Android项目的OnCreate方法中添加：

`Window.SetFlags(WindowManagerFlags.LayoutNoLimits, WindowManagerFlags.LayoutNoLimits);`

在Xamarin.Android项目的Style.xml中添加：

`<item name="android:fitsSystemWindows">true</item>`

