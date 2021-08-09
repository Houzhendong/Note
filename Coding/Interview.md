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

12. 事件和委托的区别 
    
    - 事件的声明只是在委托前面加一个event关键词，虽然你可以定义一个public，但是有了event关键词后编译器始终会把这个委托声明为private，然后添加1组add，remove方法。add对应+=，remove对应-=。这样就导致事件只能用+=，-=来绑定方法或者取消绑定方法。而委托可以用=来赋值，当然委托也是可以用+=，-=来绑定方法的。

    - 委托可以在外部被其他对象调用，而且可以有返回值（返回最后一个注册方法的返回值）。而事件不可以在外部调用，只能在声明事件的类内部被调用。我们可以使用这个特性来实现观察者模式。
13. AppDomain

    AppDomain提供了一个更安全、用途更广的处理单元，CLR可使用该单元提供应用程序之间的隔离。您可以在具有同等隔离级别（存在于单独的进程中）的单个进程中运行几个应用程序域，而不会造成进程间调用或进程间切换等方面的额外开销。在一个进程内运行多个应用程序的能力显著增强了服务器的可伸缩性。  
    AppDomain提供的隔离具有以下优点：    
    * 在一个应用程序中出现的错误不会影响其他应用程序。因为类型安全的代码不会导致内存错误，所以使用应用程序域可以确保在一个域中运行的代码不会影响进程中的其他应用程序。    
    - 能够在不停止整个进程的情况下停止单个应用程序。使用应用程序域使您可以卸载在单个应用程序中运行的代码。    
    * 在一个应用程序中运行的代码不能直接访问其他应用程序中的代码或资源。    
    * 代码行为的作用范围由它运行所在的应用程序决定
    * 向代码授予的权限可以由代码运行所在的AppDomain来控制 

    AppDomain是CLR的运行单元，它可以加载Assembly、创建对象以及执行程序。

    AppDomain是CLR实现代码隔离的基本机制。
    每一个AppDomain可以单独运行、停止；每个AppDomain有自己默认的异常处理；
    一个AppDomain的运行失败不会影响到其他的AppDomain。
    CLR在被CLR Host(windows shell or InternetExplorer or SQL Server)加载后，要创建一个默认的AppDomain，程序的入口点
    (Main方法)就是在这个默认的AppDomain中执行。
    1. AppDomain vs 进程  
    AppDomain被创建在进程中，一个进程内可以有多个AppDomain。一个AppDomain只能属于一个进程。
    2. AppDomain vs 线程  
    其实两者本来没什么好对比的。AppDomain是个静态概念，只是限定了对象的边界；线程是个动态概念，它可以运行在不同的
    AppDomain。
    一个AppDomain内可以创建多个线程，但是不能限定这些线程只能在本AppDomain内执行代码。
    CLR中的System.Threading.Thread对象其实是个soft thread，它并不能被操作系统识别；操作系统能识别的是hard thread。
    一个soft thread只属于一个AppDomain，穿越AppDomain的是hard thread。当hard thread访问到某个AppDomain时,一个
    AppDomain就会为之产生一个soft thread。
    hard thread有thread local storage(TLS)，这个存储区被CLR用来存储这个hard thread当前对应的AppDomain引用以及soft
    thread引用。当一个hard thread穿越到另外一个AppDomain时，TLS中的这些引用也会改变。
    当然这个说法很可能是和CLR的实现相关的。
    3. AppDomain vs Assembly  
    Assembly是.Net程序的基本部署单元，它可以为CLR提供用于识别类型的元数据等等。Assembly不能单独执行，它必须被加载到
    AppDomain中，然后由AppDomain创建程序集中的对象。
    一个Assembly可以被多个AppDomain加载，一个AppDomain可以加载多个Assembly。
    每个AppDomain引用到某个类型的时候需要把相应的assembly在各自的AppDomain中初始化。因此，每个AppDomain会单独保持一
    个类的静态变量。
    4. AppDomain vs 对象  
    任何对象只能属于一个AppDomain。AppDomain用来隔离对象，不同AppDomain之间的对象必须通过Proxy(reference type)或者
    Clone(value type)通信。
    引用类型需要继承System.MarshalByRefObject才能被Marshal/UnMarshal(Proxy)。
    值类型需要设置Serializable属性才能被Marshal/UnMarshal(Clone)。

# WPF
1. 依赖属性的优先级

    **从高到底**依次是：    
    - 系统强制值
    - 动画
    - 本地设置
    - 上级元素的Template设置的值
    - 隐式样式（Implicit Style）设置的值
    - Style Trigger
    - Template Trigger
    - Style Setter
    - Default Style
    - 由上级元素继承而来的值
    - 默认值

2. 依赖属性的优点    
    优化了属性的储存，减少了不必要的内存使用。
加入了属性变化通知，限制、验证等，
可以储存多个值，配合Expression以及Animation等，打造出更灵活的使用方式。

3. 什么是路由事件

    1. 路由事件是那些在可视树层次结构中向上或向下移动的事件。
WPF事件可以分为三种类型:
直接事件:-在本例中，事件在源处引发，并在源处本身处理，如“MouseEnter”事件。
冒泡事件:-它们在可视树层次结构中向上移动。例如，“MouseDown”是一个冒泡事件。
隧道事件:这些事件沿可视树层次结构下行。“PreviewKeyDown”是一个隧道事件。
    2. WPF中的路由事件是沿着VisualTree传递的，作用是用来调用应用程序的元素树上的各种监听器上的处理程序。
        - 冒泡，这种事件处理方式是从事件的激发者向上级流过去，直到到达根节点即顶层节点，一般为最外层的控件。
        - 直接，这种处理方式是在源上处理，主要用在源元素上处理。通常setter和trigger中有所体现，我个人认为VisualState可视状态可能也是直接事件处理，也是依赖属性的状态改变。和Trigger有一定的重复，但是VisualState是通过生硬的动画间接实现依赖属性的改变。
        - 隧道，又称作Preview事件，和冒泡事件处理方式相反的。元素树的根位置调用事件处理程序，依次向下直到源元素位置。   
隧道事件和冒泡事件一般成对出现。同一事件，执行时首先是隧道事件，然后是冒泡事件。

4. WPF对象的层次结构

    `Object`: 因为WPF是用.net创建的，所以WPF UI类继承的第一个类是.net对象类。  
    `DispatcherObject`:这个类确保所有WPF UI对象只能被拥有它的线程直接访问。其他不拥有它的线程必须通过dispatcher对象。  
    `DependencyObject`:WPF UI元素使用XML格式的XAML表示。在任何给定的时刻，一个WPF元素被其他WPF元素包围，而被包围的元素可以影响这个元素，这是可能的，因为这个依赖类。例如，如果一个文本框被一个面板包围，那么很有可能这个文本框可以继承面板的背景颜色。  
    `Visual`:这是一个帮助WPF UI获得其可视化表示的类。  
    `UIElement`:包含布局、输入和事件的核心子系统。  
    `FrameworkElement`:这个类支持模板、样式、绑定、资源等。
    `Control`:为自定义应用程序控件提供基础

5. 视觉树 VS 逻辑树

    * 逻辑树是视觉树的子集，也就是视觉树基本上是逻辑树的一种扩展。  
    * WPF通过逻辑树来解决依赖项属性继承和资源的问题，使用视觉树来处理渲染，事件路由，资源定位等问题。
    * 逻辑树可以认为是XAML所见的，而视觉树包含了XAML元素内部的结构。
    * 逻辑树的查找可以通过LogicalTreeHelper辅助类，视觉树的查找可以通过VisualTreeHelper辅助类，其中需要注意的是对ContentElement元素的查找，无法直接通过VisualTreeHelper进行查找，ContentElement元素并不继承Visual，而ContentElement元素的使用时需要一个ContentElement载体FrameworkContentElement。

6. 如何理解MVVM框架

    - View负责定义UI的呈现和布局，接受用户数输入  
    - ViewModel是Model For View，即View的抽象结果。实现了View绑定所需要的属性和命令，并且通过通知事件告知View状态更改。ViewModel还负责协调与Model于View的交互。ViewModel通过命令绑定传递操作，通过属性绑定传递数据。  
    - Model是应用程序非视觉的抽象，包含了数据模型和业务逻辑。

7. 基元素  
    `UIElement`,`FrameworkElement`,`ContentElement`,`FrameworkContentElement`