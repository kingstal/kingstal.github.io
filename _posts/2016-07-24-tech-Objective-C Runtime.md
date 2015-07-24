---
layout: post
title: Objective-C Runtime 特性1：基本概念
category: 技术
tags: Objective-C Runtime
keywords: Objective-C Runtime
description:
---

Objective-C语言是一门动态语言，它将很多静态语言在编译和链接时期做的事放到了运行时来处理。

这种特性意味着Objective-C不仅需要一个编译器，还需要一个运行时系统来动态得创建类和对象、进行消息传递和转发。这个运行时系统即Objective-C Runtime。Objective-C Runtime其实是一个Runtime库，它基本上是用C和汇编写的，这个库使得C语言有了面向对象的能力。

### objc_class

Objective-C类是由Class类型来表示的，它实际上是一个指向objc_class结构体的指针。它的定义如下：
`typedef struct objc_class *Class;`

在objc/runtime.h中objc_class结构体的定义如下：

```objc
struct objc_class {
    Class isa  OBJC_ISA_AVAILABILITY;

#if !__OBJC2__
    Class super_class                       OBJC2_UNAVAILABLE;  // 父类
    const char *name                        OBJC2_UNAVAILABLE;  // 类名
    long version                            OBJC2_UNAVAILABLE;  // 类的版本信息，默认为0
    long info                               OBJC2_UNAVAILABLE;  // 类信息，供运行期使用的一些位标识
    long instance_size                      OBJC2_UNAVAILABLE;  // 该类的实例变量大小
    struct objc_ivar_list *ivars            OBJC2_UNAVAILABLE;  // 该类的成员变量链表
    struct objc_method_list **methodLists   OBJC2_UNAVAILABLE;  // 方法定义的链表
    struct objc_cache *cache                OBJC2_UNAVAILABLE;  // 方法缓存
    struct objc_protocol_list *protocols    OBJC2_UNAVAILABLE;  // 协议链表
#endif

} OBJC2_UNAVAILABLE;
```

1. isa：在Objective-C中，所有的类自身也是一个对象，这个对象的Class里面也有一个isa指针，它指向metaClass(元类)。
2. super_class：指向该类的父类，如果该类已经是最顶层的根类(如NSObject或NSProxy)，则super_class为NULL。
3. cache：用于缓存最近使用的方法。一个接收者对象接收到一个消息时，它会根据isa指针去查找能够响应这个消息的对象。在实际使用中，这个对象只有一部分方法是常用的，很多方法其实很少用或者根本用不上。这种情况下，如果每次消息来时，都是methodLists中遍历一遍，性能势必很差。这时，cache就派上用场了。在每次调用过一个方法后，这个方法就会被缓存到cache列表中，下次调用的时候runtime就会优先去cache中查找，如果cache没有，才去methodLists中查找方法。这样，对于那些经常用到的方法的调用，但提高了调用的效率。

### objc_object

objc_object是表示一个类的实例的结构体，它的定义如下(objc/objc.h)：

```objc
struct objc_object {
    Class isa  OBJC_ISA_AVAILABILITY;
};

typedef struct objc_object *id;
```

这个结构体只有一个成员，即指向其类的isa指针。这样，当我们向一个Objective-C对象发送消息时，运行时库会根据实例对象的isa指针找到这个实例对象所属的类。Runtime库会在类的方法列表及父类的方法列表中去寻找与消息对应的selector指向的方法。找到后即运行这个方法。

常见的id，它是一个objc_object结构类型的指针。它的存在可以让我们实现类似于C++中泛型的一些操作。该类型的对象可以转换为任何一种对象，有点类似于C语言中void *指针类型的作用。

### 元类(Meta Class)

类自身也是一个对象，我们可以向这个对象发送消息(即调用类方法)。既然是对象，那么它也是一个objc_object指针，它包含一个指向其类的一个isa指针。这就引出了meta-class的概念。

`meta-class是一个类对象的类。`

当我们向一个对象发送消息时，runtime会在这个对象所属的这个类的方法列表中查找方法；而向一个类发送消息时，会在这个类的meta-class的方法列表中查找。

meta-class也是一个类，也可以向它发送一个消息，那么它的isa又是指向什么呢？为了不让这种结构无限延伸下去，Objective-C的设计者让所有的meta-class的isa指向基类的meta-class，以此作为它们的所属类。即，任何NSObject继承体系下的meta-class都使用NSObject的meta-class作为自己的所属类，而基类的meta-class的isa指针是指向它自己。这样就形成了一个完美的闭环。
![iOS-Runtime-meta-class](/assets/image/iOS-Runtime-meta-class.png)

### objc_ivar

Ivar是表示实例变量的类型，其实际是一个指向objc_ivar结构体的指针，其定义如下：

```objc
typedef struct objc_ivar *Ivar;

struct objc_ivar {
    char *ivar_name                 OBJC2_UNAVAILABLE;  // 变量名
    char *ivar_type                 OBJC2_UNAVAILABLE;  // 变量类型
    int ivar_offset                 OBJC2_UNAVAILABLE;  // 基地址偏移字节
#ifdef __LP64__
    int space                       OBJC2_UNAVAILABLE;
#endif
}
```

### objc_property_t

objc_property_t是表示Objective-C声明的属性的类型，其实际是指向objc_property结构体的指针，其定义如下

```objc
typedef struct objc_property *objc_property_t;
```

### objc_property_attribute_t

objc_property_attribute_t定义了属性的特性(attribute)，它是一个结构体，定义如下：

```objc
typedef struct {
    const char *name;           // 特性名
    const char *value;          // 特性值
} objc_property_attribute_t;
```

### 关联对象(Associated Object)

关联对象类似于成员变量，不过是在运行时添加的。我们通常会把成员变量(Ivar)放在类声明的头文件中，或者放在类实现的@implementation后面。但这有一个缺点，我们不能在分类中添加成员变量。如果我们尝试在分类中添加新的成员变量，编译器会报错。

我们可能希望通过使用(甚至是滥用)全局变量来解决这个问题。但这些都不是Ivar，因为他们不会连接到一个单独的实例。因此，这种方法很少使用。

Objective-C针对这一问题，提供了一个解决方案：即关联对象(Associated Object)。

可以把关联对象想象成一个Objective-C对象(如字典)，这个对象通过给定的key连接到类的一个实例上。不过由于使用的是C接口，所以key是一个void指针(const void *)。我们还需要指定一个内存管理策略，以告诉Runtime如何管理这个对象的内存。这个内存管理的策略可以由以下值指定：

```objc
OBJC_ASSOCIATION_ASSIGN
OBJC_ASSOCIATION_RETAIN_NONATOMIC
OBJC_ASSOCIATION_COPY_NONATOMIC
OBJC_ASSOCIATION_RETAIN
OBJC_ASSOCIATION_COPY
```

当宿主对象被释放时，会根据指定的内存管理策略来处理关联对象。如果指定的策略是assign，则宿主释放时，关联对象不会被释放；而如果指定的是retain或者是copy，则宿主释放时，关联对象会被释放。我们甚至可以选择是否是自动retain/copy。当我们需要在多个线程中处理访问关联对象的多线程代码时，这就非常有用了。

我们将一个对象连接到其它对象所需要做的就是下面两行代码：

```objc
static char myKey;

objc_setAssociatedObject(self, &myKey, anObject, OBJC_ASSOCIATION_RETAIN);
```

在这种情况下，self对象将获取一个新的关联的对象anObject，且内存管理策略是自动retain关联对象，当self对象释放时，会自动release关联对象。另外，如果我们使用同一个key来关联另外一个对象时，也会自动释放之前关联的对象，这种情况下，先前的关联对象会被妥善地处理掉，并且新的对象会使用它的内存。

`id anObject = objc_getAssociatedObject(self, &myKey);`

我们可以使用objc_removeAssociatedObjects函数来移除一个关联对象，或者使用objc_setAssociatedObject函数将key指定的关联对象设置为nil。

### SEL

SEL又叫选择器，是表示一个方法的selector的指针，其定义如下：

`typedef struct objc_selector *SEL;`

方法的selector用于表示运行时方法的名字。Objective-C在编译时，会依据每一个方法的名字、参数序列，生成一个唯一的整型标识(Int类型的地址)，这个标识就是SEL。

两个类之间，不管它们是父类与子类的关系，还是之间没有这种关系，只要方法名相同，那么方法的SEL就是一样的。每一个方法都对应着一个SEL。所以在Objective-C同一个类(及类的继承体系)中，不能存在2个同名的方法，即使参数类型不同也不行。相同的方法只能对应一个SEL。这也就导致Objective-C在处理相同方法名且参数个数相同但类型不同的方法方面的能力很差。

当然，不同的类可以拥有相同的selector，这个没有问题。不同类的实例对象执行相同的selector时，会在各自的方法列表中去根据selector去寻找自己对应的IMP。

工程中的所有的SEL组成一个Set集合，Set的特点就是唯一，因此SEL是唯一的。因此，如果我们想到这个方法集合中查找某个方法时，只需要去找到这个方法对应的SEL就行了，SEL实际上就是根据方法名hash化了的一个字符串，而对于字符串的比较仅仅需要比较他们的地址就可以了，可以说速度上无语伦比！！但是，有一个问题，就是数量增多会增大hash冲突而导致的性能下降（或是没有冲突，因为也可能用的是perfect hash）。但是不管使用什么样的方法加速，如果能够将总量减少（多个方法可能对应同一个SEL），那将是最犀利的方法。那么，我们就不难理解，为什么SEL仅仅是函数名了。

本质上，SEL只是一个指向方法的指针（准确的说，只是一个根据方法名hash化了的KEY值，能唯一代表一个方法），它的存在只是为了加快方法的查询速度。

我们可以在运行时添加新的selector，也可以在运行时获取已存在的selector，我们可以通过下面三种方法来获取SEL:

1. sel_registerName函数
2. Objective-C编译器提供的@selector()
3. NSSelectorFromString()方法

### IMP

IMP实际上是一个函数指针，指向方法实现的首地址。其定义如下：

`id (*IMP)(id, SEL, ...)`

这个函数使用当前CPU架构实现的标准的C调用约定。第一个参数是指向self的指针(如果是实例方法，则是类实例的内存地址；如果是类方法，则是指向元类的指针)，第二个参数是方法选择器(selector)，接下来是方法的实际参数列表。

前面介绍过的SEL就是为了查找方法的最终实现IMP的。由于每个方法对应唯一的SEL，因此我们可以通过SEL方便快速准确地获得它所对应的IMP，查找过程将在下面讨论。取得IMP后，我们就获得了执行这个方法代码的入口点，此时，我们就可以像调用普通的C语言函数一样来使用这个函数指针了。

通过取得IMP，我们可以跳过Runtime的消息传递机制，直接执行IMP指向的函数实现，这样省去了Runtime消息传递过程中所做的一系列查找操作，会比直接向对象发送消息高效一些。

### Method
Method用于表示类定义中的方法，则定义如下：

```objc
typedef struct objc_method *Method;

struct objc_method {
    SEL method_name                     OBJC2_UNAVAILABLE;  // 方法名
    char *method_types                  OBJC2_UNAVAILABLE;
    IMP method_imp                      OBJC2_UNAVAILABLE;  // 方法实现
}
```

我们可以看到该结构体中包含一个SEL和IMP，实际上相当于在SEL和IMP之间作了一个映射。有了SEL，我们便可以找到对应的IMP，从而调用方法的实现代码。

### objc_method_description

objc_method_description定义了一个Objective-C方法，其定义如下：

`struct objc_method_description { SEL name; char *types; };`

### 方法调用流程

在Objective-C中，消息直到运行时才绑定到方法实现上。编译器会将消息表达式[receiver message]转化为一个消息函数的调用，即objc_msgSend。这个函数将消息接收者和方法名作为其基础参数，如以下所示：

`objc_msgSend(receiver, selector)`

如果消息中还有其它参数，则该方法的形式如下所示：

`objc_msgSend(receiver, selector, arg1, arg2, ...)`

这个函数完成了动态绑定的所有事情：

1. 首先它找到selector对应的方法实现。因为同一个方法可能在不同的类中有不同的实现，所以我们需要依赖于接收者的类来找到的确切的实现。
2. 它调用方法实现，并将接收者对象及方法的所有参数传给它。
3. 最后，它将实现返回的值作为它自己的返回值。

消息的关键在于前面讨论过的结构体objc_class，这个结构体有两个字段是我们在分发消息的关注的：

1. 指向父类的指针
2. 一个类的方法分发表，即methodLists。

当我们创建一个新对象时，先为其分配内存，并初始化其成员变量。其中isa指针也会被初始化，让对象可以访问类及类的继承体系。

下图演示了这样一个消息的基本框架：
![iOS-Runtime-messaging](/assets/image/iOS-Runtime-messaging.gif)

当消息发送给一个对象时，objc_msgSend通过对象的isa指针获取到类的结构体，然后在方法分发表里面查找方法的selector。如果没有找到selector，则通过objc_msgSend结构体中的指向父类的指针找到其父类，并在父类的分发表里面查找方法的selector。依此，会一直沿着类的继承体系到达NSObject类。一旦定位到selector，函数会就获取到了实现的入口点，并传入相应的参数来执行方法的具体实现。如果最后没有定位到selector，则会走消息转发流程，这个我们在后面讨论。

- 隐藏参数

objc_msgSend有两个隐藏参数：

1. 消息接收对象
2. 方法的selector

这两个参数为方法的实现提供了调用者的信息。之所以说是隐藏的，是因为它们在定义方法的源代码中没有声明。它们是在编译期被插入实现代码的。

虽然这些参数没有显示声明，但在代码中仍然可以引用它们。我们可以使用self来引用接收者对象，使用_cmd来引用选择器。如下代码所示：

```objc
- strange
{
    id  target = getTheReceiver();
    SEL method = getTheMethod();

    if ( target == self || method == _cmd )
        return nil;
    return [target performSelector:method];
}
```

- 获取方法地址

Runtime中方法的动态绑定让我们写代码时更具灵活性，如我们可以把消息转发给我们想要的对象，或者随意交换一个方法的实现等。不过灵活性的提升也带来了性能上的一些损耗。毕竟我们需要去查找方法的实现，而不像函数调用来得那么直接。当然，方法的缓存一定程度上解决了这一问题。

我们上面提到过，如果想要避开这种动态绑定方式，我们可以获取方法实现的地址，然后像调用函数一样来直接调用它。特别是当我们需要在一个循环内频繁地调用一个特定的方法时，通过这种方式可以提高程序的性能。

NSObject类提供了methodForSelector:方法，让我们可以获取到方法的指针，然后通过这个指针来调用实现代码。我们需要将methodForSelector:返回的指针转换为合适的函数类型，函数参数和返回值都需要匹配上。

我们通过以下代码来看看methodForSelector:的使用：

```objc
void (*setter)(id, SEL, BOOL);
int i;

setter = (void (*)(id, SEL, BOOL))[target methodForSelector:@selector(setFilled:)];
for (i = 0 ; i < 1000 ; i++)
    setter(targetList[i], @selector(setFilled:), YES);
```

这里需要注意的就是函数指针的前两个参数必须是id和SEL。

当然这种方式只适合于在类似于for循环这种情况下频繁调用同一方法，以提高性能的情况。另外，methodForSelector:是由Cocoa运行时提供的；它不是Objective-C语言的特性。

----------------------------------------

### 消息转发(message forwarding)

在 Objective-C 中，[object foo] 语法并不会立即执行 foo 这个方法的代码。它是在运行时给 object 发送一条叫 foo 的消息。
当一个对象无法接收某一消息时，通常情况下，程序会在运行时挂掉并抛出 unrecognized selector sent to … 的异常。但在异常抛出前，Objective-C 的运行时会给你三次拯救程序的机会：

1. Method resolution  动态方法解析
2. Fast forwarding    备用接收者
3. Normal forwarding  完整转发

- Method Resolution

首先，Objective-C 运行时会调用 `+resolveInstanceMethod:` 或者 `+resolveClassMethod:`，让你有机会提供一个函数实现。如果你添加了函数并返回 YES， 那运行时系统就会重新启动一次消息发送的过程。

```objc
void fooMethod(id obj, SEL _cmd)
{
    NSLog(@"Doing foo");
}

+ (BOOL)resolveInstanceMethod:(SEL)aSEL
{
    if(aSEL == @selector(foo:)){
        class_addMethod([self class], aSEL, (IMP)fooMethod, "v@:");
        return YES;
    }
    return [super resolveInstanceMethod];
}
```

如果 resolve 方法返回 NO ，运行时就会移到下一步：**消息转发（Message Forwarding）**。

PS：iOS 4.3 加入很多新的 runtime 方法，主要都是以 imp 为前缀的方法，比如 `imp_implementationWithBlock()` 用 block 快速创建一个 imp 。
上面的例子可以重写成:

```objc
IMP fooIMP = imp_implementationWithBlock(^(id _self) {
    NSLog(@"Doing foo");
});

class_addMethod([self class], aSEL, fooIMP, "v@:");
```

- Fast forwarding
如果目标对象实现了 `-forwardingTargetForSelector:` ，Runtime 这时就会调用这个方法，给你把这个消息转发给其他对象的机会。

```objc
- (id)forwardingTargetForSelector:(SEL)aSelector
{
    if(aSelector == @selector(foo:)){
        return alternateObject;
    }
    return [super forwardingTargetForSelector:aSelector];
}
```

只要这个方法返回的不是 nil 和 self，整个消息发送的过程就会被重启，当然发送的对象会变成你返回的那个对象。否则，就会继续 Normal Fowarding 。

这里叫 Fast ，只是为了区别下一步的转发机制。因为这一步不会创建任何新的对象，但下一步转发会创建一个 NSInvocation 对象，所以相对更快点。

- Normal forwarding
这一步是 Runtime 最后一次给你挽救的机会。首先它会发送 `-methodSignatureForSelector:`消息获得函数的参数和返回值类型。如果 `-methodSignatureForSelector:` 返回 nil ，Runtime 则会发出 `-doesNotRecognizeSelector:` 消息，程序这时也就挂掉了。如果返回了一个函数签名，Runtime 就会创建一个 NSInvocation 对象并发送 `-forwardInvocation:` 消息给目标对象。

NSInvocation 实际上就是对一个消息的描述，包括selector 以及参数等信息。所以你可以在 `-forwardInvocation:` 里修改传进来的 NSInvocation 对象，然后发送 `-invokeWithTarget:` 消息给它，传进去一个新的目标：

```objc
- (NSMethodSignature*)methodSignatureForSelector:(SEL)selector
{
    NSMethodSignature* signature = [super methodSignatureForSelector:selector];

    if (!signature)
        signature = [alternateObject methodSignatureForSelector:selector];

    return signature;
}

- (void)forwardInvocation:(NSInvocation *)invocation
{
    SEL sel = invocation.selector;

    if([alternateObject respondsToSelector:sel]) {
        [invocation invokeWithTarget:alternateObject];
    }
    else {
        [self doesNotRecognizeSelector:sel];
    }
}
```

- 总结
Objective-C 中给一个对象发送消息会经过以下几个步骤：

1. 在对象类的 *dispatch table* 中尝试找到该消息。如果找到了，跳到相应的函数IMP去执行实现代码；
2. 如果没有找到，Runtime 会发送 `+resolveInstanceMethod:` 或者 `+resolveClassMethod:` 尝试去 resolve 这个消息；
3. 如果 resolve 方法返回 NO，Runtime 就发送 `-forwardingTargetForSelector:` 允许你把这个消息转发给另一个对象；
4. 如果没有新的目标对象返回， Runtime 就会发送 `-methodSignatureForSelector:` 和 `-forwardInvocation:` 消息。你可以发送 `-invokeWithTarget:` 消息来手动转发消息或者发送 `-doesNotRecognizeSelector:` 抛出异常。


### 参考
[http://tech.glowing.com/cn/objective-c-runtime/](http://tech.glowing.com/cn/objective-c-runtime/)

[Objective-C Runtime系列](http://southpeak.github.io/blog/2014/10/25/objective-c-runtime-yun-xing-shi-zhi-lei-yu-dui-xiang/)















