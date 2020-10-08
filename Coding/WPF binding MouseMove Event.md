# 1.添加附加属性
```c#
public class ElementAttachedProperties
{
    public static readonly DependencyProperty MouseMoveCommandProperty =
                DependencyProperty.RegisterAttached("MouseMoveCommand", 
                typeof(ICommand), typeof(ElementAttachedProperties),
                new FrameworkPropertyMetadata(new PropertyChangedCallback(MouseMoveCommandChanged)));

    public static void MouseMoveCommandChanged(DependencyObject d, DependencyPropertyChangedEventArgs e)
    {
        FrameworkElement element = d as FrameworkElement;

        element.MouseMove += Element_MouseMove;
    }
    
    private static void Element_MouseMove(object sender, MouseEventArgs e)
    {
        FrameworkElement element = sender as FrameworkElement;

        ICommand command = GetMouseMoveCommand(element);

        command.Execute(e.GetPosition(element));
    }

    public static void SetMouseMoveCommand(UIElement element, ICommand value)
    {
        element.SetValue(MouseMoveCommandProperty, value);
    }

    public static ICommand GetMouseMoveCommand(UIElement element)
    {
        return (ICommand)element.GetValue(MouseMoveCommandProperty);
    }
}
```
# 2. 在View中绑定命令
```c#
local:ElementAttachedProperties.MouseMoveCommand="{Binding MouseMoveCommand}"
```
# 3.ViewModel实现命令
```c#
MouseMoveCommand = new RelayParameterizedCommand((paramter) => MouseMove((Point)paramter));

private void MouseMove(Point paramter)
{
    PositionString = $"X : {(int)paramter.X} , Y : {(int)paramter.Y}";
}
```