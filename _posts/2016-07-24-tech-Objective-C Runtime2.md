---
layout: post
title: Objective-C Runtime 特性2：Method Swizzling
category: 技术
tags: Objective-C Runtime
keywords: Objective-C Runtime
description:
---

学习runtime机制不得不理解Method Swizzling。Method Swizzling是改变一个selector的实际实现的技术。通过这一技术，我们可以在运行时通过修改类的分发表中selector对应的函数，来修改方法的实现。

一个常见的例子就是在开发中我们需要跟踪程序中每一个view controller展示给用户的次数：当然，我们可以在每个view controller的viewDidAppear中添加跟踪代码；但是这太过麻烦，需要在每个view controller中写重复的代码。创建一个子类可能是一种实现方式，但需要同时创建UIViewController, UITableViewController, UINavigationController及其它UIKit中view controller的子类，这同样会产生许多重复的代码。

这种情况下，我们就可以使用Method Swizzling，如在代码所示：

```objc
#import <objc/runtime.h>

@implementation UIViewController (Tracking)

// Invoked whenever a class or category is added to the Objective-C runtime
+ (void)load
{
    static dispatch_once_t onceToken;
    dispatch_once(&onceToken, ^{
        Class class = [self class];
        // When swizzling a class method, use the following:
        // Class class = object_getClass((id)self);

        // the method might not exist in the class, but in its superclass
        SEL originalSelector = @selector(viewWillAppear:);
        SEL swizzledSelector = @selector(xxx_viewWillAppear:);

        Method originalMethod = class_getInstanceMethod(class, originalSelector);
        Method swizzledMethod = class_getInstanceMethod(class, swizzledSelector);

        // class_addMethod will fail if original method already exists
        BOOL didAddMethod = class_addMethod(class,
            originalSelector,
            method_getImplementation(swizzledMethod),
            method_getTypeEncoding(swizzledMethod));

        // the method doesn’t exist and we just added one
        if (didAddMethod) {
            class_replaceMethod(class,
                swizzledSelector,
                method_getImplementation(originalMethod),
                method_getTypeEncoding(originalMethod));
        }
        else {
            method_exchangeImplementations(originalMethod, swizzledMethod);
        }
    });
}

#pragma mark - Method Swizzling

- (void)xxx_viewWillAppear:(BOOL)animated
{
    [self xxx_viewWillAppear:animated];
    NSLog(@"viewWillAppear: %@", self);
}
```

![Method Swizzling](/assets/image/iOS-Runtime-Method Swizzling.png)
**说明**：
1. 第3步中如果 originalMethod 已经存在，则添加失败。
2. 失败则直接交换 originalMethod 和 swizzledMethod；若成功，则说明该类本身没有实现 originalMethod，第2步中`Method originalMethod = class_getInstanceMethod(class, originalSelector);`获取都是父类的 originalMethod，这时候 originalSelector 已指向 swizzledMethod，我们需要的就是将 swizzledSelector 指向父类的 originalMethod。

