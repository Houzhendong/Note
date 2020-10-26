ActualWidth等只读属性不可以直接绑定到ViewModel，可以采用依赖属性的方法来绑定，例如：

```c#
class SizeObserver
{
    #region " Observe "

    public static bool GetObserve(FrameworkElement elem)
    {
        return (bool)elem.GetValue(ObserveProperty);
    }

    public static void SetObserve(FrameworkElement elem, bool value)
    {
        elem.SetValue(ObserveProperty, value);
    }

    public static readonly DependencyProperty ObserveProperty =
        DependencyProperty.RegisterAttached("Observe", typeof(bool), typeof(SizeObserver),
        new UIPropertyMetadata(false, OnObserveChanged));

    static void OnObserveChanged(DependencyObject depObj, DependencyPropertyChangedEventArgs e)
    {
        FrameworkElement elem = depObj as FrameworkElement;
        if (elem == null)
            return;

        if (e.NewValue is bool == false)
            return;

        if ((bool)e.NewValue)
            elem.SizeChanged += OnSizeChanged;
        else
            elem.SizeChanged -= OnSizeChanged;
    }

    static void OnSizeChanged(object sender, RoutedEventArgs e)
    {
        if (!Object.ReferenceEquals(sender, e.OriginalSource))
            return;

        FrameworkElement elem = e.OriginalSource as FrameworkElement;
        if (elem != null)
        {
            SetObservedWidth(elem, elem.ActualWidth);
            SetObservedHeight(elem, elem.ActualHeight);
        }
    }

    #endregion

    #region " ObservedWidth "

    public static double GetObservedWidth(DependencyObject obj)
    {
        return (double)obj.GetValue(ObservedWidthProperty);
    }

    public static void SetObservedWidth(DependencyObject obj, double value)
    {
        obj.SetValue(ObservedWidthProperty, value);
    }

    // Using a DependencyProperty as the backing store for ObservedWidth.  This enables animation, styling, binding, etc...
    public static readonly DependencyProperty ObservedWidthProperty =
        DependencyProperty.RegisterAttached("ObservedWidth", typeof(double), typeof(SizeObserver), new UIPropertyMetadata(0.0));

    #endregion

    #region " ObservedHeight "

    public static double GetObservedHeight(DependencyObject obj)
    {
        return (double)obj.GetValue(ObservedHeightProperty);
    }

    public static void SetObservedHeight(DependencyObject obj, double value)
    {
        obj.SetValue(ObservedHeightProperty, value);
    }

    // Using a DependencyProperty as the backing store for ObservedHeight.  This enables animation, styling, binding, etc...
    public static readonly DependencyProperty ObservedHeightProperty =
        DependencyProperty.RegisterAttached("ObservedHeight", typeof(double), typeof(SizeObserver), new UIPropertyMetadata(0.0));

    #endregion
}
```

注意：绑定ViewModel的属性时，用OneWayReserve绑定会导致绑定无效，要用TwoWay绑定，但是本质上绑定的还是一个只读属性，所以ViewModel的属性在声明时不要调用OnPropertyChanged方法，这样可以实现单向绑定的目的