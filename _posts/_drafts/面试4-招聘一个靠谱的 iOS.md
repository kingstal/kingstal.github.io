1.iOS程序的完整启动过程及原理

> 1. 先执行main函数，main内部会调用UIApplicationMain函数
> 2. 在UIApplicationMain函数里面创建UIApplication对象，创建UIApplication的delegate对象—–PYAppDelegate并开启一个消息循环。
> 3. 为应用程序创建一个UIWindow对象（继承自UIView），设置为PYAppDelegate的window属性，创建控制器设置为UIWindow的rootViewController属性(根控制器)。展示UIWindow，展示之前会将添加rootViewController的view到UIWindow上面。

2.Category和继承的区别

> Objective-C不支持多重继承，但是通过类别（Category）和协议（Protocol）可以实现代码复用和扩展。

Category，可以动态的为已经存在的类添加新的行为。

Category的使用场景：

1. 当在定义类的时候，在某些情况下（例如需求变更），可能想要为其中的某个或几个类中添加方法。
2. 一个类中包含了许多不同的方法需要实现，而这些方法需要不同团队的成员实现。
3. 当在使用基础类库中的类时，可能希望这些类实现一些你需要的方法。

> 需要注意的问题：

1. Category可以访问原始类的实例变量，但不能添加变量，如果想添加变量，可以考虑通过继承创建子类。
2. Category可以重载原始类的方法，但不推荐这么做，这么做的后果是你再也不能访问原来的方法。如果确实要重载，正确的选择是创建子类。
3. 和普通接口有所区别的是，在分类的实现文件中可以不必实现所有声明的方法，只要你不去调用它。

> Protocol，简单来说就是一系列不属于任何类的方法列表，其中声明的方法可以被任何类实现。

3.C语言中普通变量与静态变量在内存中存储的位置，类比OC中变量

> 在C语言中，内存主要分为如下5个存储区：

（1）栈(Stack)：位于函数内的局部变量（包括函数实参），由编译器负责分配释放，函数结束，栈变量失效。

> （2）堆(Heap)：由程序员用malloc/calloc/realloc分配，free释放。如果程序员忘记free了，则会造成内存泄露，程序结束时该片内存会由OS回收。
> 
> （3）全局区/静态区(Global Static Area)： 全局变量和静态变量存放区，程序一经编译好，该区域便存在。并且在C语言中初始化的全局变量和静态变量和未初始化的放在相邻的两个区域（在C++中，由于全局变量和静态变量编译器会给这些变量自动初始化赋值，所以没有区分了）。由于全局变量一直占据内存空间且不易维护，推荐少用。程序结束时释放。
> 
> （4）C风格字符串常量存储区： 专门存放字符串常量的地方，程序结束时释放。
> 
> （5）程序代码区：存放程序二进制代码的区域。



1.基础 `@property``

- `@property` 后面可以有哪些修饰符？

> 1. 线程安全: atomic,nonatomic
> 2. 访问权限: readonly,readwrite
> 3. 内存管理(ARC): assign,strong,weak,copy   内存管理(MRC): assign,retain,copy
> 4. 指定方法名称: setter=,getter=

- 什么情况使用 `weak` 关键字，相比 `assign` 有什么不同？

> 1. 在ARC中,出现循环引用的时候,必须要有一端使用weak,比如:自定义View的代理属性
> 2. 自身已经对它进行一次强引用,没有必要在强引用一次,此时也会使用weak,自定义View的子控件属性一般也使用weak
> 3. weak当对象销毁的时候,指针会被自动设置为nil,而assign不会。assigin 可以用于非OC对象,而weak必须用于OC对象

- 怎么用 `copy` 关键字？

> 1. 对于字符串和block的属性一般使用copy
> 2. 字符串使用copy是为了防止外部把字符串内容改了,影响该属性
> 3. block使用copy是在MRC遗留下来的,在MRC中,方法内部的block是在在栈区的,使用copy可以把它放到堆区.在ACR中对于block使用copy还是strong效果是一样的。

- 这个写法会出什么问题： `@property (copy) NSMutableArray *array;`

> 添加,删除,修改数组内的元素的时候,程序会因为找不到对应的方法而崩溃。因为copy就是复制一个不可变NSArray的对象。
> 
> 该属性使用了同步锁，会在创建时生成一些额外的代码用于帮助编写多线程程序，这会带来性能问题，通过声明nonatomic可以节省这些虽然很小但是不必要额外开销。

- 如何让自己的类用 `copy` 修饰符？

> 1. 你是说让我的类也支持copy的功能吗? 如果面试官说是: 遵守NSCopying协议,实现 - (id)copyWithZone:(NSZone *)zone; 方法。重写copy关键字的setter时，需要调用一下传入对象的copy方法，然后赋值给该setter的方法对应的成员变量。
> 2. 如果面试官说否,是属性中如何使用copy,在使用字符串和block的时候一般都使用copy

- @property` 的本质是什么？ivar、getter、setter 是如何生成并添加到这个类中的

> 在普通的OC对象中,@property在编译时编译器自动帮生成一个私有的成员变量和setter与getter方法的声明和实现。反编译property大致生成了五个东西：
> 
> - OBJC_IVAR_$类名$属性名称 该属性的偏移量
> - setter与getter方法对应的实现函数
> - ivar_list 就是成员变量列表
> - method_list 方法列表
> - prop_list 属性列表
> 
> 也就是说每次在增加一个属性,系统都会在ivar_list中添加一个成员变量的描述,在method_list中增加setter与getter方法的描述,在属性列表中增加一个属性的属性描述,然后计算该属性在对象中的偏移量,然后生成setter与getter方法对应的实现,在setter方法方法中从偏移量的位置开始赋值,在getter方法中从偏移量开始取值,为了能够读取正确字节数,系统对象偏移量的指针类型进行了类型强转.

- `@protocol` 和 `category` 中如何使用 `@property`

> 1. 在protocol中使用property只会生成setter和getter方法声明,使用属性的目的,是希望遵守协议的对象的实现该属性.
> 2. category 使用 @property 也是只会生成setter和getter方法的声明,如果真的需要给category增加属性的实现,需要借助于运行时的两个函数 objc_setAssociatedObject  objc_getAssociatedObject

- runtime如何实现weak变量的自动置nil？

> runtime 对注册的类， 会进行布局，对于 weak 对象会放入一个 hash 表中。 用 weak 指向的对象地址作为 key，当此对象的引用计数为0的时候会 dealloc， 进而在这个 weak 表中找到此对象地址为键的所有 weak 对象，从而设置为 nil

2.weak属性需要在dealloc中置nil么？

> 不需要,在ARC环境无论是强指针还是弱指针都无需在deallco设置为nil,ARC会自动帮我们处理.

3.`@synthesize`和@`dynamic`分别有什么作用？

> @property有两个对应的词，一个是@synthesize，一个是@dynamic。如果@synthesize和@dynamic都没写，那么默认的就是@syntheszie var = _var;
> 
> @synthesize的语义是如果你没有手动实现setter方法和getter方法，那么编译器会自动为你加上这两个方法。
> 
> @dynamic告诉编译器,属性的setter与getter方法由用户自己实现，不自动生成。（当然对于readonly的属性只需提供getter即可）。假如一个属性被声明为@dynamic var，然后你没有提供@setter方法和@getter方法，编译的时候没问题，但是当程序运行到instance.var =someVar，由于缺setter方法会导致程序崩溃；或者当运行到 someVar = var时，由于缺getter方法同样会导致崩溃。编译时没问题，运行时才执行相应的方法，这就是所谓的动态绑定。

4.ARC下，不显示指定任何属性关键字时，默认的关键字都有哪些？

> 对应基本数据类型默认关键字是: atomic,readwrite,assign
> 
> 对于普通的OC对象: atomic,readwrite,strong

5.用@property声明的NSString（或NSArray，NSDictionary）经常使用copy关键字，为什么？如果改用strong关键字，可能造成什么问题？

> 1. 使用copy的目的是为了让本对象的属性不受外界影响,使用copy无论给我传入是一个可变对象还是不可变对象,本身持有的就是一个不可变的副本.
> 2. 如果我们使用是strong,那么这个属性就有可能指向一个可变对象,如果这个可变对象在外部被修改了,那么会影响该属性.

6.@synthesize合成实例变量的规则是什么？假如property名为foo，存在一个名为_foo的实例变量，那么还会自动合成新变量么？

> 如果没有指定实例变量的名称，就会默认自动生成一个与属性同名的并在前面加一条下划线的实例变量.
> 
> 如果指定了实例变量名称，刚好与属性自动合成的实例变量名称相同，那就该属性就不自动合成实例变量了。
> 
> 默认的是 @synthesize foo = _foo; 属性就不会自动生成实例变量了.
> 
> 如果使用 @synthesize foo = _foo1; 属性不会自动生成实例变量,而且成员变量被修改为_foo1, 使用点语法set属性成员变量的值会赋值给_foo1,而不是_foo.

7.在有了自动合成属性实例变量之后，@synthesize还有哪些使用场景？

> @synthesize主要就是用来生成setter,getter方法的实现,在@property被增强之后,其实已经很少使用@synthesize了.

8.objc中向一个nil对象发送消息将会发生什么？

> 在Objective-C中向nil发送消息是完全有效的——只是在运行时不会有任何作用:
> 
> - 如果一个方法返回值是一个对象，那么发送给nil的消息将返回0(nil)。例如：Person * motherInlaw = [ aPerson spouse] mother];如果spouse对象为nil，那么发送给nil的消息mother也将返回nil。
> - 如果方法返回值为指针类型，其指针大小为小于或者等于sizeof(void*)，float，double，long double 或者long long的整型标量，发送给nil的消息将返回0。
> - 如果方法返回值为结构体,发送给nil的消息将返回0。结构体中各个字段的值将都是0。
> - 如果方法的返回值不是上述提到的几种情况，那么发送给nil的消息的返回值将是未定义的。

9.objc中向一个对象发送消息[obj foo]和objc_msgSend()函数之间有什么关系？

> 该方法编译之后就是objc_msgSend()函数调用. ((void ()(id, SEL))(void )objc_msgSend)((id)obj, sel_registerName("foo"));

10.什么时候会报unrecognized selector的异常？

> 当类或者对象使用到某个声明过的方法，而这个方法不管自己类还是在父类里都没有实现的时候。

11.一个objc对象如何进行内存布局？（考虑有父类的情况）

> 1. 所有父类的成员变量和自己的成员变量都会存放在该对象所对应的存储空间中.
> 2. 每一个objc对象都是一个类的实例。每一个对象内部都一个isA指针,指向他的类对象,类对象中存放着本对象的对象方法列表和成员变量的列表,它内部也有一个isA指针指向元对象(meta class),元对象内部存放的是类方法列表,类对象内部还有一个superclass的指针,指向他的父类对象
> 3. 根对象就是NSobject，如下图所示。

![iOS就业-objc对象内存布局](/assets/image/iOS就业-objc对象内存布局.png)

12.一个objc对象的isa的指针指向什么？有什么作用？

> 指向他的类对象,从而可以找到对象上的方法

13.下面的代码输出什么？

``` objc
@implementation Son : Father
- (id)init
{
    self = [super init];
    if (self) {
        NSLog(@"%@", NSStringFromClass([self class]));
        NSLog(@"%@", NSStringFromClass([super class]));
    }
    return self;
}
@end
```

> 输出的结果都是:Son,原因:super 和 self 都是指向的本实例对象的,不同的是：super调用的跳过本类方法,直接调用父类的方法。父类方法的class方法本来都是在基类中实现的,所以无论使用self和super调用都是一样的.

14.runtime如何通过selector找到对应的IMP地址？（分别考虑类方法和实例方法）

> 每一个类对象中都一个方法列表,方法列表中记录着方法的名称,方法实现,以及参数类型,其实selector本质就是方法名称,通过这个方法名称就可以在方法列表中找到对应的方法实现.

15.使用runtime Associate方法关联的对象，需要在主对象dealloc的时候释放么？

> 在ARC下不需要; 在MRC中,对于使用retain或copy策略的需要

16.objc中的类方法和实例方法有什么本质区别和联系？

> - 类方法
>   1. 类方法是属于类对象的
>   2. 类方法只能通过类对象调用
>   3. 类方法中的self是类对象
>   4. 类方法可以调用其他的类方法
>   5. 类方法中不能访问成员变量
>   6. 类方法中不能直接调用对象方法



> - 实例方法
>   1. 实例方法是属于实例对象的
>   2. 实例方法只能通过实例对象调用
>   3. 实例方法中的self是实例对象
>   4. 实例方法中可以访问成员变量
>   5. 实例方法中直接调用实例方法
>   6. 实例方法中也可以调用类方法(通过类名)

17._objc_msgForward函数是做什么的，直接调用它将会发生什么？

> 跟objc动态运行机制中的`forwardingTargetForSelector:`和`forwardInvocation:`方法有点相似。猜测是该方法编译后的源码，都用于消息转发机制。

18.能否向编译后得到的类中增加实例变量？能否向运行时创建的类中添加实例变量？为什么？

> 因为编译后的类已经注册在 runtime 中，类结构体中的 objc_ivar_list 实例变量的链表 和 instance_size 实例变量的内存大小已经确定，同时runtime 会调用 class_setIvarLayout 或 class_setWeakIvarLayout 来处理 strong weak 引用。所以不能向存在的类中添加实例变量，
> 
> 运行时创建的类是可以添加实例变量，调用 class_addIvar 函数。但是得在调用 objc_allocateClassPair 之后，objc_registerClassPair 之前，原因同上。

19.runloop和线程有什么关系？

> 1. 每一个线程中都一个runloop,只有主线的的runloop默认是开启的,其他线程的runloop是默认没有开启的
> 2. 可以通过CFRunLoopRun() 函数来开启一个事件循环
> 3. 看SDWebImage源码的时候见到有这么用过.

20.runloop的mode作用是什么？

> - NSDefaultRunLoopMode 缺省情况下，将包含所有操作，并且大多数情况下都会使用此模式
> - NSConnectionReplyMode 此模式用于处理NSConnection的回调事件
> - NSModalPanelRunLoopMode 模态模式，此模式下，RunLoop只对处理模态相关事件
> - NSRunLoopCommonModes 此模式用于配置”组模式”，一个输入源与此模式关联，则输入源与组中的所有模式相关联。

21.以+ scheduledTimerWithTimeInterval...的方式触发的timer，在滑动页面上的列表时，timer会暂定回调，为什么？如何解决？

> 当前主线程的 RunLoop 正在以 `UITrackingRunLoopMode`的模式运行。 这个时候 RunLoop 只会处理与`UITrackingRunLoopMode` “绑定”的源， 比如触摸、滚动等事件；而 NSTimer 是默认“绑定”到 `NSRunLoopDefaultMode` 上的， 所以 Timer 是事情是不会被 RunLoop 处理的，定时器被暂停了！
> 
> 常见的解决方案是把Timer“绑定”到 `NSRunLoopCommonModes`模式上:`[[NSRunLoop currentRunLoop] addTimer:timer forMode:NSRunLoopCommonModes];`
> 
> 这样这个Timer就可以和当前组中的两种模式 `UITrackingRunLoopMode` 和 `kCFRunLoopDefaultMode`相关联了。 RunLoop在这两种模式下，Timer都可以正常运行了。
> 
> 

22.猜想runloop内部是如何实现的？

> 是一个死循环，如果事件队列中存放在事件,那就取出事件,执行相关代码；如果没有事件,就挂起,等有事件了,立即唤醒事件循环,开始执行.
> 
> runloop 本质上就是 event loop 的实现。



23.objc使用什么机制管理对象内存？

> - MRC 手动引用计数  
> - ARC 自动引用计数,现在通常使用自动引用计数

24.ARC通过什么方式帮助开发者管理内存？

> 通过编译器在编译的时候,插入如内管理的代码

25.不手动指定autoreleasepool的前提下，一个autorealese对象在什么时刻释放？（比如在一个vc的viewDidLoad中创建）

> 在每次事件循环开始创建自动释放池,在每次事件结束销毁自动释放池.
> 
> 以viewDidLoad方法为例,可以理解为在viewDidLoad方法开始执行之前创建自动释放池,在viewDidLoad方法执行之后销毁自动释放池。

26.BAD_ACCESS在什么情况下出现？

> 1. 死循环了
> 2. 访问一个僵尸对象 对一个已经释放的对象执行了release

27.苹果是如何实现autoreleasepool的？

> autoreleasepool以一个队列数组的形式实现,主要通过下列三个函数完成.
> 
> - objc_autoreleasepoolPush
> - objc_autoreleasepoolPop
> - objc_aurorelease
> 
> 看函数名就可以知道，对autorelease分别执行push，和pop操作。销毁对象时执行release操作。

28.使用block时什么情况会发生引用循环，如何解决？

> 只要是一个对象对该block进行了强引用,在block内部有直接使用到该对象.
> 
> 解决方法是将该对象使用`__weak`或者`__block`修饰符修饰之后再在block中使用。
> 
> 1. id **weak weakSelf = self;或者** weak __typeof(&*self)weakSelf = self该方法可以设置宏
> 2. id __block weakSelf = self;

29.在block内如何修改block外部变量？

> 通过 `__bock`修饰的外部变量,可以在block内部修改

30.使用系统的某些block api（如UIView的block版本写动画时），是否也考虑引用循环问题？

> 一般不用考虑,因为官方文档中没有告诉我们要注意发生强引用,所以推测系统控件一般没有对这些block进行强引用,所以我们可以不用考虑循环强引用的问题

31.GCD的队列（dispatch_queue_t）分哪两种类型？

> 串行队列 Serial Dispatch Queue
> 
> 并行队列 Concurrent Dispatch Queue

32.如何用GCD同步若干个异步调用？（如根据若干个url异步加载多张图片，然后在都下载完成后合成一张整图）

> 使用Dispatch Group将block加到Global Group Queue,这些block如果全部执行完毕，再通知组在Main Dispatch Queue中执行合成图片的过程。
> 
> ``` objective-c
> dispatch_queue_t queue = dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0); 
> dispatch_group_t group = dispatch_group_create(); 
> dispatch_group_async(group, queue, ^{ /*加载图片1 */ }); dispatch_group_async(group, queue, ^{ /*加载图片2 */ }); dispatch_group_async(group, queue, ^{ /*加载图片3 */ }); dispatch_group_notify(group, dispatch_get_main_queue(), ^{ // 合并图片 });
> ```

33.dispatch_barrier_async的作用是什么？

> barrier:是障碍物的意思,在多个并行任务中间,他就像是一个隔离带,把前后的并行任务分开.
> 
> dispatch_barrier_async 作用是在并行队列中，等待前面操作并行任务完成，再执行dispatch_barrier_async中的任务，如果后面还有并行任务,会开始执行后续的并行任务.

34.苹果为什么要废弃dispatch_get_current_queue？

> 容易误用造成死锁

35.以下代码运行结果如何？

``` objc
- (void)viewDidLoad
{
    [super viewDidLoad];
    NSLog(@"1");
    dispatch_sync(dispatch_get_main_queue(), ^{
        NSLog(@"2");
    });
    NSLog(@"3");
}
```

> 只能输出1,然后线程主线程死锁

36.addObserver:forKeyPath:options:context:各个参数的作用分别是什么，observer中需要实现哪个方法才能获得KVO回调？

``` objective-c
// 添加键值观察
/**
 1. 调用对象：要监听的对象
 2. 参数
 1> 观察者，负责处理监听事件的对象
 2> 观察的属性
 3> 观察的选项
 4> 上下文
 */
[self.person addObserver:self forKeyPath:@"name" options:NSKeyValueObservingOptionNew | NSKeyValueObservingOptionOld context:@"Person Name"];
```

``` objective-c
// NSObject 分类方法，意味着所有的 NSObject 都可以实现这个方法！
// 跟协议的方法很像，分类方法又可以称为“隐式代理”！不提倡用，但是要知道概念！
// 所有的 kvo 监听到事件，都会调用此方法
/**
 1. 观察的属性
 2. 观察的对象
 3. change 属性变化字典（新／旧）
 4. 上下文，与监听的时候传递的一致
 可以利用上下文区分不同的监听！
*/
- (void)observeValueForKeyPath:(NSString *)keyPath ofObject:(id)object change:(NSDictionary *)change context:(void *)context {
    NSLog(@"睡会 %@", [NSThread currentThread]);
    [NSThread sleepForTimeInterval:1.0];
    NSLog(@"%@ %@ %@ %@", keyPath, object, change, context);
}
```

37.如何手动触发一个value的KVO

> 给这个value设置一个值,就可以触发了

38.若一个类有实例变量NSString *_foo，调用setValue:forKey:时，可以以foo还是_foo作为key？

> 都可以

39.KVC的keyPath中的集合运算符如何使用？

> - 必须用在集合对象上或普通对象的集合属性上
> - 简单集合运算符有@avg， @count ， @max ， @min ，@sum，
> - 格式 @”@sum.age”或 @”集合属性.@max.age”

40.apple用什么方式实现对一个对象的KVO？

如何关闭默认的KVO的默认实现，并进入自定义的KVO实现？

> [如何自己动手实现 KVO](http://www.cocoachina.com/ios/20150313/11321.html)

41.IBOutlet连出来的视图属性为什么可以被设置成weak?

> 因为视图已经对它有一个强引用了

42.IB中User Defined Runtime Attributes如何使用？

> User Defined Runtime Attributes 是一个不被看重但功能非常强大的的特性，它能够通过KVC的方式配置一些你在interface builder 中不能配置的属性。当你希望在IB中作尽可能多得事情，这个特性能够帮助你编写更加轻量级的viewcontroller.

43.如何调试BAD_ACCESS错误

> - 重写object的`respondsToSelector`方法，显示出现EXEC_BAD_ACCESS前访问的最后一个object
> - 通过NSZombieEnabled
> - 设置全局断点快速定位问题代码所在行

44.lldb（gdb）常用的调试命令？

> - breakpoint 设置断点定位到某一个函数
> - n 断点指针下一步
> - po打印对象
> 
> 参考[工具篇：LLDB调试器](http://southpeak.github.io/blog/2015/01/25/gong-ju-pian-:lldbdiao-shi-qi/)





