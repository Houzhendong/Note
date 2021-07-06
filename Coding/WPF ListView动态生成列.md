# WPF ListView动态生成列
主要思路:在写界面的xaml文件时，使用Converter返回一个动态的GridView绑定到`ListView.View`上，然后定义一个ColumnConfig类来作为绑定的参数，类中定义了`Column`的`Header`和要绑定的属性名

MainWindows.xaml
```xml
<Window x:Class="DynamicGridView.MainWindow"
        xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
        xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
        xmlns:d="http://schemas.microsoft.com/expression/blend/2008"
        xmlns:mc="http://schemas.openxmlformats.org/markup-compatibility/2006"
        xmlns:local="clr-namespace:DynamicGridView"
        mc:Ignorable="d"
        Title="MainWindow" Height="450" Width="800">
    <Window.Resources>
        <local:ConfigToDynamicGridViewConverter x:Key="ConfigToDynamicGridViewConverter" />
    </Window.Resources>
    <ListView ItemsSource="{Binding Products}"
              View="{Binding ColumnConfig, Converter={StaticResource ConfigToDynamicGridViewConverter}}" />
</Window>
```    
ViewModel
```C#
 public class ViewModel
    {
        public ColumnConfig ColumnConfig { get; set; }
        public IEnumerable<Product> Products { get; set; }

        public ViewModel()
        {
            Products = new List<Product>
            {
                new Product { Name = "Some product", Attributes = "Very cool product" },
                new Product { Name = "Other product", Attributes = "Not so cool one" }
            };
            ColumnConfig = new ColumnConfig
            {
                Columns = new List<Column>
                {
                    new Column { Header = "Name", DataField = "Name" },
                    new Column { Header = "Attributes", DataField = "Attributes" }
                }
            };
        }
    }

    public class ColumnConfig
    {
        public IEnumerable<Column> Columns { get; set; }
    }

    public class Column
    {
        public string Header { get; set; }
        public string DataField { get; set; }
    }

    public class Product
    {
        public string Name { get; set; }
        public string Attributes { get; set; }
    }
```   
Converter
```C#
    public class ConfigToDynamicGridViewConverter : IValueConverter
    {
        public object Convert(object value, Type targetType, object parameter, CultureInfo culture)
        {
            var config = value as ColumnConfig;
            if (config != null)
            {
                var grdiView = new GridView();
                foreach (var column in config.Columns)
                {
                    var binding = new Binding(column.DataField);
                    grdiView.Columns.Add(new GridViewColumn { Header = column.Header, DisplayMemberBinding = binding });
                }
                return grdiView;
            }
            return Binding.DoNothing;
        }

        public object ConvertBack(object value, Type targetType, object parameter, CultureInfo culture)
        {
            throw new NotSupportedException();
        }
    }
```