# WPF程序在运行时更换主题/Style

1. 在app.xaml文件中建立一个独立的ResourceDictionary

   ```xaml
   <ResourceDictionary>
       <ResourceDictionary.MergedDictionaries>
           <ResourceDictionary>
               <ResourceDictionary.MergedDictionaries>
                   <ResourceDictionary>
                       <ResourceDictionary.MergedDictionaries>
                           <ResourceDictionary Source="ThemeUrl.xaml"/>
                       </ResourceDictionary.MergedDictionaries>
                   </ResourceDictionary>
                </ResourceDictionary.MergedDictionaries>
             </ResourceDictionary>
        </ResourceDictionary.MergedDictionaries>
   </ResourceDictionary>
   ```

2. 新建一个App类提供更换主题方法

   ```c#
   public class Theme
   {
       ResourceDictionary ThemeDictionary => Application.Current.Resource.MergedDictionaries[0];
       
       public void ChangeTheme(Uri url)
       {
           ThemeDictionary.MergedDictionaries.Clear();
           ThemeDictionary.MergedDictionaries.Add(new ResourceDictionary(){Source = url});
       }
   }
   ```

   因为更换时要清空MergedDictionaries，所以包含主题的ResourceDictionary要单独写在一个MergedDictionaries里。

3. 运行中切换资源文件

   ```c#
   var app = new AppTheme(); 
   app.ChangeTheme(new System.Uri("ThemeUri.xaml",UriKind.RelativeOrAbsolute));
   ```

   正常编写xaml时使用style时使用DynamicResource即可

   