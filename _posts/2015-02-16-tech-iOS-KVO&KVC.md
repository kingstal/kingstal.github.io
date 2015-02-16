---
layout: post
title: KVO & KVC
category: 技术
tags: KVO,KVC
keywords: KVO,KVC
description: 介绍 KVO 和 KVC
---

`<NSKeyValueObserving>`或`KVO`是一种非正式协议，提供了一种常用机制用于对象之间的观察和通知状态变化。

### **KVO**


#### Subscribing 订阅
发出通知的对象添加观察者


    {% highlight objcetive-c %}
    // 注册观察者
    - (void)addObserver:(NSObject *)observer
             forKeyPath:(NSString *)keyPath
                options:(NSKeyValueObservingOptions)options
                context:(void *)context
    {% endhighlight %}



> `observer`: 观察者，必须实现方法， `observeValueForKeyPath:ofObject:change:context:`

> `keyPath`: 所观察的`property`相对于接收者的** keypath**，不能为 nil

> `options`: `NSKeyValueObservingOptions`值的组合，表明通知中包含哪些内容

> `context`: 在`observeValueForKeyPath:ofObject:change:context:`中传递给观察者的任意值


#### `NSKeyValueObservingOptions`

> `NSKeyValueObservingOptionNew`: `change dictionary`包含新的属性值

>`NSKeyValueObservingOptionOld`: `change dictionary`包含旧的属性值

> `NSKeyValueObservingOptionInitial`: 在注册方法返回前，观察者会收到一个通知。在`change dictionary`通常会包含`NSKeyValueChangeNewKey`实体，如果声明了`NSKeyValueObservingOptionNew`，但不包含`NSKeyValueChangeOldKey`实体（在初始通知中被观察属性当前的值可能是旧的，但定于观察者来说是新的）。

>`NSKeyValueObservingOptionPrior`:


#### Responding 响应
观察者对通知做出响应

    {% highlight objective-c %}
    // 响应变化
    - (void)observeValueForKeyPath:(NSString *)keyPath
                      ofObject:(id)object
                        change:(NSDictionary *)change
                       context:(void *)context
    {% endhighlight %}


#### 正确的`Context`声明
建议采用以下方式声明：`static void * XXContext = &XXContext;`。
它是一个静态值，储存它自身的指针，对于本身没意义，但对于`<NSKeyValueObserving>`非常完美，这样声明父类和子类都能观察到同一个对象的变化。

> `static void * PrivateKVOContext = &PrivateKVOContext;`


#### 更好的 KeyPath

由于拼写错误等原因，直接传递字符串作为**KeyPath**并不是好的选择，聪明的方法是使用以下方式传递**KeyPath**：

> `NSStringFromSelector(@selector(propertyName))`


#### Unsubscribing 取消订阅
当观察者完成监听后，调用`removeObserver:forKeyPath:context:`取消订阅。通常在`-observeValueForKeyPath:ofObject:change:context:`或`-dealloc`中调用。


#### 属性自动通知
通过覆盖`+automaticallyNotifiesObserversForKey:`返回`NO`可实现手动通知。


    {% highlight objective-c %}
    // 在 setter 方法中手动调用 willChangeValueForKey 和 didChangeValueForKey
    - (void)setLComponent:(double)lComponent;
    {
        if (_lComponent == lComponent) {
            return;
        }
        [self willChangeValueForKey:@"lComponent"];
        _lComponent = lComponent;
        [self didChangeValueForKey:@"lComponent"];
    }
    {% endhighlight %}


#### 属性依赖
有时候观察的属性依赖于其他的属性，这时候可以通过下面两种方式来声明依赖关系：

> `+ (NSSet *)keyPathsForValuesAffectingValueForKey:(NSString *)key`
> `+ (NSSet *)keyPathsForValuesAffecting<Key>`

    {% highlight objective-c %}
    // color 依赖于redComponent、greenComponent、blueComponent 三个属性
    + (NSSet *)keyPathsForValuesAffectingColor
    {
        return [NSSet setWithObjects:@"redComponent", @"greenComponent", @"blueComponent", nil];
    }
    {% endhighlight %}



#### 我的实践

> 声明 Context：`static void * PrivateKVOContext = &PrivateKVOContext;`

> 添加观察者：`[labColor addObserver:self forKeyPath:NSStringFromSelector(@selector(color)) options:NSKeyValueObservingOptionInitial context:&PrivateKVOContext];`

>   {% highlight objective-c %}
    // 响应变化
    -(void)observeValueForKeyPath:(NSString *)keyPath ofObject:(id)object change:(NSDictionary *)change context:(void *)context{
        if (context == &PrivateKVOContext) {
            // Observe values here
            if ([keyPath isEqualToString:NSStringFromSelector(@selector(color))]) {
                [self colorDidChange:change];
            }
            } else {
                [super observeValueForKeyPath:keyPath ofObject:object change:change context:context];
        }
    }

    // 取消订阅
    -(void)dealloc{
        [_labColor removeObserver:self forKeyPath:NSStringFromSelector(@selector(color))];
    }
    {% endhighlight %}


### 参考

[http://nshipster.com/key-value-observing/](http://nshipster.com/key-value-observing/)
