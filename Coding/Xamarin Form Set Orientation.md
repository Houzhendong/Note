# Xamarin form强制横竖屏

## Android

```c#
public void ForceLandscape()
{     
    GetActivity().RequestedOrientation = ScreenOrientation.Landscape;    
}
public void ForcePortrait()
{
	GetActivity().RequestedOrientation = ScreenOrientation.Portrait;
}
public Activity GetActivity()
{
    var activity = (Activity)Forms.Context;
    return activity;
}
```

## IOS

```c#
public void ForceLandscape()
{
    UIDevice.CurrentDevice.SetValueForKey(new NSNumber((int)UIInterfaceOrientation.LandscapeLeft), new NSString("orientation"));
}

public void ForcePortrait()
{
	UIDevice.CurrentDevice.SetValueForKey(new NSNumber((int)UIInterfaceOrientation.Portrait), new NSString("orientation"));
}
```



