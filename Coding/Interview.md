# C#/.Net
1. 简述 private、 protected、 public、 internal 修饰符的访问权限。  
    private :   私有成员, 在类的内部才可以访问。

    protected : 保护成员，该类内部和继承类中可以访问。

    public :    公共成员，完全公开，没有访问限制。

    internal:   在同一命名空间内可以访问。
2. Override与重载的区别  
    重载是方法的名称相同。参数或参数类型不同，进行多次重载以适应不同的需要  

    Override 是进行基类中函数的重写。实现多态。

3. 单例模式
单线程模式
```C#
public class Singleton
{
    private static Singleton instance = null;

    public Singleton() { }
    public static Singleton Instance 
    {
        get
        {
            if(instance == null)
                instance = new Singleton();
            return instance;
        }
    }
}
```
线程安全模式
```C#
public class Singleton
{
    private volatile static Singleton instance = null;
    private static readonly object objLock = new object();

    public Singleton() { }
    public static Singleton Instance 
    {
        get
        {
            if(instance == null)
            {
                lock(objLock)
                {
                    if(instance == null)
                        instance = new Singleton();
                    return instance;
                }
            }
        }
    }
}
```
4. 什么是虚方法，和抽象方法有什么区别  
    虚方法必须有默认的实现，可以在子类中override。   
    抽象方法没有实现，它只存在于抽象类中，继承与它的子类必须实现抽象方法。
5. `const`和`readonly`的区别

    const变量只能且必须在声明时指定值，之后无法更改  
    readonly变量可以在声明时和构造函数中指定初始值

6. `string`和`StringBuilder`的区别  
    `string`是不可变的对象。当改变其值或者追加新字符串时，它会删除旧的值，然后在内存中创建一个保存了新值的新的`string`实例。  
    `StringBuilder`是可变对象。每当append和replace时它会创建一个新的实例，旧的对象用于对字符串进行任何操作。

7. 装箱和拆箱  
    将值类型转换为引用类型为装箱，将引用类型转换为值类型为拆箱

8. 什么是依赖注入，如何实现的

    依赖注入是一个设计模式中的原则。为了避免创建一个类A的对象还要创建他的依赖类B，将依赖类B作为参数传递给A的构造函数。
9. `using`的用法

    使用命名空间  
    定义资源范围代码块，在using代码块中的资源会在执行代码块执行结束时立即垃圾回收

10. 什么是序列化

    将对象转化为字节流的过程称为序列化，序列化的对象应该实现`ISerialization`接口

11. abstract class和interface有什么区别?

    | Abstract class | Interface |
    | :----:| :----:|
    |包含了声明和定义两部分 | 只包含声明部分 |
    |不支持多继承 | 支持多继承 | 
    |有构造函数 | 没有构造函数|
    |可以包含static成员 | 不可以包含static成员 |
    |可以包含不同的访问权限 | everything is public |
    |用来实现类的核心功能 | 用来实现类的次要功能 |
    | 可以包含方法、字段、常量 | 只能包含方法(属性也是方法)|

# WPF
1. 依赖属性的优先级

    从高到底依次是：动画→绑定→本地值→自定义Style Trigger→自定义Template Trigger→自定义Style Setter→默认Style Trigger→默认Style Setter→继承值→默认值

2. 什么是路由事件

    1. 路由事件是那些在可视树层次结构中向上或向下移动的事件。
WPF事件可以分为三种类型:
直接事件:-在本例中，事件在源处引发，并在源处本身处理，如“MouseEnter”事件。
冒泡事件:-它们在可视树层次结构中向上移动。例如，“MouseDown”是一个冒泡事件。
隧道事件:这些事件沿可视树层次结构下行。“PreviewKeyDown”是一个隧道事件。
    2. WPF中的路由事件是沿着VisualTree传递的，作用是用来调用应用程序的元素树上的各种监听器上的处理程序。
    - 冒泡，这种事件处理方式是从源元素向上级流过去，直到到达根节点即顶层节点，一般为最外层的控件。
    - 直接，这种处理方式是在源上处理，主要用在源元素上处理。通常setter和trigger中有所体现，我个人认为VisualState可视状态可能也是直接事件处理，也是依赖属性的状态改变。和Trigger有一定的重复，但是VisualState是通过生硬的动画间接实现依赖属性的改变。
    - 隧道，又称作Preview事件，和冒泡事件处理方式相反的。元素树的根位置调用事件处理程序，依次向下直到源元素位置。
隧道事件和冒泡事件一般成对出现。同一事件，执行时首先是隧道事件，然后是冒泡事件。

3. WPF对象的层次结构

    `Object`: 因为WPF是用.net创建的，所以WPF UI类继承的第一个类是.net对象类。  
    `DispatcherObject`:这个类确保所有WPF UI对象只能被拥有它的线程直接访问。其他不拥有它的线程必须通过dispatcher对象。  
    `DependencyObject`:WPF UI元素使用XML格式的XAML表示。在任何给定的时刻，一个WPF元素被其他WPF元素包围，而被包围的元素可以影响这个元素，这是可能的，因为这个依赖类。例如，如果一个文本框被一个面板包围，那么很有可能这个文本框可以继承面板的背景颜色。  
    `Visual`:这是一个帮助WPF UI获得其可视化表示的类。  
    `UIElement`:包含布局、输入和事件的核心子系统。  
    `FrameworkElement`:这个类支持模板、样式、绑定、资源等。
    `Control`:为自定义应用程序控件提供基础

4. 视觉树 VS 逻辑树

    * 逻辑树是视觉树的子集，也就是视觉树基本上是逻辑树的一种扩展。  
    * WPF通过逻辑树来解决依赖项属性继承和资源的问题，使用视觉树来处理渲染，事件路由，资源定位等问题。
    * 逻辑树可以认为是XAML所见的，而视觉树包含了XAML元素内部的结构。
    * 逻辑树的查找可以通过LogicalTreeHelper辅助类，视觉树的查找可以通过VisualTreeHelper辅助类，其中需要注意的是对ContentElement元素的查找，无法直接通过VisualTreeHelper进行查找，ContentElement元素并不继承Visual，而ContentElement元素的使用时需要一个ContentElement载体FrameworkContentElement。
