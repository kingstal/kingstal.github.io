---
layout: post
title: KVO & KVC
category: 技术
tags: KVO,KVC
keywords: KVO,KVC
description: 介绍 KVO 和 KVC
---


## **KVC**
`<NSKeyValueCoding>`(KVC)是一种非正式协议，提供了一种通过对象名字(key)直接访问属性的机制。

### 语法


```objc
// get
* (id)valueForKey:(NSString *)key
* (id)valueForKeyPath:(NSString *)keyPath
//举例：person 对象有一个address 属性，address有一个属性为 city
[person valueForKeyPath:@"address.city"]
// set
* (void)setValue:(id)value forKey:(NSString *)key
* (void)setValue:(id)value forKeyPath:(NSString *)keyPath
// validate
* (BOOL)validateValue:(inout id *)ioValue forKey:(NSString *)key error:(out NSError **)outError
* (BOOL)validateValue:(inout id *)ioValue forKeyPath:(NSString *)key error:(out NSError **)outError
```


### Collection operators 集合运算符
集合运算符允许使用 keypath 和运算符对集合中的对象进行运算。
小例子：

> `NSArray *a = @[@4, @84, @2];
NSLog(@"max = %@", [a valueForKeyPath:@"@max.self"]);`

#### 简单的集合运算符

1. @avg `NSNumber *transactionAverage = [transactions valueForKeyPath:@"@avg.amount"];`
2. @count `NSNumber *numberOfTransactions = [transactions valueForKeyPath:@"@count"];`
3. @max `NSDate *latestDate = [transactions valueForKeyPath:@"@max.date"];`
4. @min
5. @sum


#### 对象运算符

1. @distinctUnionOfObjects `NSArray *payees = [transactions valueForKeyPath:@"@distinctUnionOfObjects.payee"];`
2. @unionOfObjects `NSArray *payees = [transactions valueForKeyPath:@"@unionOfObjects.payee"];`

#### 数组和集合运算符
1. @distinctUnionOfArrays `NSArray *payees = [arrayOfTransactionsArrays valueForKeyPath:@"@distinctUnionOfArrays.payee"];`
2. @unionOfArrays
3. @distinctUnionOfSets


## **KVO**

`<NSKeyValueObserving>`或`KVO`是一种非正式协议，提供了一种常用机制用于对象之间的观察和通知状态变化。

### Subscribing 订阅


发出通知的对象添加观察者



```objc
// 注册观察者
* (void)addObserver:(NSObject *)observer
         forKeyPath:(NSString *)keyPath
            options:(NSKeyValueObservingOptions)options
            context:(void *)context
```



> `observer`: 观察者，必须实现方法， `observeValueForKeyPath:ofObject:change:context:`

> `keyPath`: 所观察的`property`相对于接收者的** keypath**，不能为 nil

> `options`: `NSKeyValueObservingOptions`值的组合，表明通知中包含哪些内容

> `context`: 在`observeValueForKeyPath:ofObject:change:context:`中传递给观察者的任意值



### `NSKeyValueObservingOptions`



> `NSKeyValueObservingOptionNew`: `change dictionary`包含新的属性值

>`NSKeyValueObservingOptionOld`: `change dictionary`包含旧的属性值

> `NSKeyValueObservingOptionInitial`: 在注册方法返回前，观察者会收到一个通知。在`change dictionary`通常会包含`NSKeyValueChangeNewKey`实体，如果声明了`NSKeyValueObservingOptionNew`，但不包含`NSKeyValueChangeOldKey`实体（在初始通知中被观察属性当前的值可能是旧的，但定于观察者来说是新的）。

>`NSKeyValueObservingOptionPrior`:


### Responding 响应

观察者对通知做出响应


```objc
// 响应变化
* (void)observeValueForKeyPath:(NSString *)keyPath
                  ofObject:(id)object
                    change:(NSDictionary *)change
                   context:(void *)context
```



### 正确的`Context`声明

建议采用以下方式声明：`static void * XXContext = &XXContext;`。
它是一个静态值，储存它自身的指针，对于本身没意义，但对于`<NSKeyValueObserving>`非常完美，这样声明父类和子类都能观察到同一个对象的变化。

> `static void * PrivateKVOContext = &PrivateKVOContext;`



### 更好的 KeyPath

由于拼写错误等原因，直接传递字符串作为**KeyPath**并不是好的选择，聪明的方法是使用以下方式传递**KeyPath**：

> `NSStringFromSelector(@selector(propertyName))`


### Unsubscribing 取消订阅

当观察者完成监听后，调用`removeObserver:forKeyPath:context:`取消订阅。通常在`-observeValueForKeyPath:ofObject:change:context:`或`-dealloc`中调用。



### 属性自动通知

通过覆盖`+automaticallyNotifiesObserversForKey:`返回`NO`可实现手动通知。


```objc
// 在 setter 方法中手动调用 willChangeValueForKey 和 didChangeValueForKey
* (void)setLComponent:(double)lComponent;
{
    if (_lComponent == lComponent) {
        return;
    }
    [self willChangeValueForKey:@"lComponent"];
    _lComponent = lComponent;
    [self didChangeValueForKey:@"lComponent"];
}
```


### 属性依赖

有时候观察的属性依赖于其他的属性，这时候可以通过下面两种方式来声明依赖关系：

> `+ (NSSet *)keyPathsForValuesAffectingValueForKey:(NSString *)key`
> `+ (NSSet *)keyPathsForValuesAffecting<Key>`

```objc
// color 依赖于redComponent、greenComponent、blueComponent 三个属性
- (NSSet *)keyPathsForValuesAffectingColor
{
    return [NSSet setWithObjects:@"redComponent", @"greenComponent", @"blueComponent", nil];
}
```


### 我的实践

> 声明 Context：`static void * PrivateKVOContext = &PrivateKVOContext;`

> 添加观察者：`[labColor addObserver:self forKeyPath:NSStringFromSelector(@selector(color)) options:NSKeyValueObservingOptionInitial context:&PrivateKVOContext];`

```objc
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
```


```objc
// 取消订阅
-(void)dealloc{
    [_labColor removeObserver:self forKeyPath:NSStringFromSelector(@selector(color))];
}
```


### **常见误区**
集合是不能被关注的，KVO 只能关注关系而非集合。举例说明，假如有一个`ContactList`对象，我们能使用`-addObserver:forKeyPath:`关注它的`contacts`属性，而非直接让数组`contacts`作为被观察者。

## 参考

[http://nshipster.com/key-value-observing/](http://nshipster.com/key-value-observing/)

[http://www.objc.io/issue-7/key-value-coding-and-observing.html](http://www.objc.io/issue-7/key-value-coding-and-observing.html)
