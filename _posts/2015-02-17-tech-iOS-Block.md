---
layout: post
title: Block
category: 技术
tags: Block
keywords: Block
description: 介绍 Block
---


## 声明 block

- 作为 **local variable**：

> `returnType (^blockName)(parameterTypes) = ^returnType(parameters) {...};`


- 作为 **property**:

> `@property (nonatomic, copy) returnType (^blockName)(parameterTypes);`


- 作为 **method parameter**:

> `- (void)someMethodThatTakesABlock:(returnType (^)(parameterTypes))blockName;`


- 作为 **an argument to a method call**:

> `[someObject someMethodThatTakesABlock:^returnType (parameters) {...}];`


- 作为 **typedef**:

> `typedef returnType (^TypeName)(parameterTypes);
TypeName blockName = ^returnType(parameters) {...};`






## 参考

[http://nshipster.com/key-value-observing/](http://nshipster.com/key-value-observing/)

[http://www.objc.io/issue-7/key-value-coding-and-observing.html](http://www.objc.io/issue-7/key-value-coding-and-observing.html)
