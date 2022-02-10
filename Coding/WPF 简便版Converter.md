WPF程序在binding过程中会用到Converter来辅助，下面这种实现可以更方便地使用Converter

```C#
using System;
using System.Globalization;
using System.Windows.Data;
using System.Windows.Markup;

namespace Example
{
    public abstract class BaseValueConverter<T> : MarkupExtension, IValueConverter
        where T : class , new()
    {
        private static Lazy<T> converter = new Lazy<T>(() => new T());

        public override object ProvideValue(IServiceProvider serviceProvider) => converter.Value;

        public abstract object Convert(object value, Type targetType, object parameter, CultureInfo culture);

        public abstract object ConvertBack(object value, Type targetType, object parameter, CultureInfo culture);
    }
}
```

使用方法：

1. 继承`BaseValueConverter`，实现抽象方法，在泛型中填写派生类本身

```C#
public class BooleanInvertConverter: BaseValueConverter<BooleanInvertConverter>
    {
        public override object Convert(object value, Type targetType, object parameter, CultureInfo culture)
        {
            return !(bool)value;
        }

        public override object ConvertBack(object value, Type targetType, object parameter, CultureInfo culture)
        {
            throw new NotImplementedException();
        }
    }
```

2. 在xaml中使用

```xml
<Button IsEnable="{Binding SomeBooleanValue, Converter={namespace:BooleaninvertConverter}}">
```


由于实现了MarkupExtension抽象类，在xaml文件中使用时可以直接用`namespace:SomeConverter`的方式来调用，不需要在Resource中先声明