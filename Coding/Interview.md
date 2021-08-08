# C#/.Net
1. AppDomain

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

2. 事件和委托的区别 
    
    - 事件的声明只是在委托前面加一个event关键词，虽然你可以定义一个public，但是有了event关键词后编译器始终会把这个委托声明为private，然后添加1组add，remove方法。add对应+=，remove对应-=。这样就导致事件只能用+=，-=来绑定方法或者取消绑定方法。而委托可以用=来赋值，当然委托也是可以用+=，-=来绑定方法的。

    - 委托可以在外部被其他对象调用，而且可以有返回值（返回最后一个注册方法的返回值）。而事件不可以在外部调用，只能在声明事件的类内部被调用。我们可以使用这个特性来实现观察者模式。大概就是这么多。下面是一段测试代码。

# WPF
1. 如何理解MVVM框架

    View负责定义UI的呈现和布局，接受用户数输入  
    ViewModel是Model For View，即View的抽象结果。实现了View绑定所需要的属性和命令，并且通过通知事件告知View状态更改。ViewModel还负责协调与Model于View的交互。ViewModel通过命令绑定传递操作，通过属性绑定传递数据。  
    Model是应用程序非视觉的抽象，包含了数据模型和业务逻辑。

2. 