# 方法1：使用资源字典
1. 创建资源字典StringResource.xaml，包含需要支持多国语言的字符串
```xml
<ResourceDictionary xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
        xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
        xmlns:sys="clr-namespace:System;assembly=mscorlib">
        <sys:String x:Key="WinTitle">MainWindow</sys:String>
        <sys:String x:Key="TblText">Support multi language demo.</sys:String>
        <sys:String x:Key="BtnOK">OK</sys:String>
        <sys:String x:Key="HdNo">No.</sys:String>
        <sys:String x:Key="HdName">Name</sys:String>
        <sys:String x:Key="HdGender">Gender</sys:String>
        <sys:String x:Key="HdDept">Dept</sys:String>
        <sys:String x:Key="HdEmail">Email</sys:String>
        <sys:String x:Key="HdTel">Tel</sys:String>
        <sys:String x:Key="MsgShowTime">Now time is:{0}</sys:String>
</ResourceDictionary>
```
2. 在app.xaml中使用MergedDictionary
```xml
<Application x:Class="LocalizationDemo.App"
               xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
               xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
               StartupUri="MainWindow.xaml" Exit="Application_Exit">
      <Application.Resources>
          <ResourceDictionary>
              <ResourceDictionary.MergedDictionaries>
                  <ResourceDictionary Source="Resources\StringResource.xaml" />
                  <ResourceDictionary Source="Resources\StringResource.zh-CN.xaml" />
             </ResourceDictionary.MergedDictionaries>
         </ResourceDictionary>
     </Application.Resources>
</Application>

```
3. 在程序中使用资源字典中的字符串
例如：  
> .xaml文件:    `Text="{StaticResource TblText}"`  
> .cs文件:  `string s = Application.Current.FindResource("MsgShowTime").ToString()`
4. 在程序启动时根据当前区域加载对应的资源字典文件
```C#
List<ResourceDictionary> dictionaryList = new List<ResourceDictionary>();
        foreach (ResourceDictionary dictionary in Application.Current.Resources.MergedDictionaries)
        {
            dictionaryList.Add(dictionary);
        }
        string requestedCulture = string.Format(@"Resources\StringResource.{0}.xaml", Culture);
        ResourceDictionary resourceDictionary = dictionaryList.FirstOrDefault(d => d.Source.OriginalString.Equals(requestedCulture));
        if (resourceDictionary == null)
        {
            requestedCulture = @"Resources\StringResource.xaml";
            resourceDictionary = dictionaryList.FirstOrDefault(d => d.Source.OriginalString.Equals(requestedCulture));
        }
        if (resourceDictionary != null)
        {
            Application.Current.Resources.MergedDictionaries.Remove(resourceDictionary);
            Application.Current.Resources.MergedDictionaries.Add(resourceDictionary);
        }
```
# 方法2：使用.resx资源文件
这种方式和Winform支持多国语言保持一致，相对较容易实现，新增支持语言需要重新编译程序，所有的.resx文件必须放在同一个主程序集中。
1. 添加字符串  
在资源文件Resources.resx中添加字符串资源，并将访问修饰符设置为Public。
2. 在程序中使用资源文件  
> .xaml文件，引入名称空间：`xmlns:props="clr-namespace:LocalizationDemo.Properties"；`  
> 使用方式：`Text="{x:Static props:Resources.TblText}"`

> .cs文件，使用方式：`string s = Properties.Resources.MsgShowTime;`  
> 3.  新增语言资源文件  
> 以新增简体中文为例，复制资源文件Resources.resx，重命名为Resources.zh-CN.resx，将值翻译为中文保存。
> 4. 设置程序运行语言环境  
> ` LocalizationDemo.Properties.Resources.Culture = new CultureInfo("zh-CN");`

参考：<https://www.cnblogs.com/horan/archive/2012/04/20/wpf-multilanguage.html>