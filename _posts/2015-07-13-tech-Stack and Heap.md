---
layout: post
title: Stack and Heap Objects in Objective-C
category: 技术
tags: Stack, Heap
keywords: Stack, Heap
description: Stack, Heap
---


## Stack
stack 是存储`local variables`和`temporary values`的内存区域。每个线程运行时对应一个stack。当函数被调用，一个`stack frame`被 push 进 stack，函数的本地数据在那保存。当函数结束时，`stack frame`被销毁。所有的过程都是自动的。

## Heap
The heap is what you access when you call malloc and free.
heap 能随时被 alloc 和销毁。全局和静态变量保存在heap中，直到应用退出。为了访问创建在heap 中的数据，最少要求有一个保存在stack中的指针，因为CPU通过stack中的指针访问heap中的数据。

可以认为stack 中的一个指针仅仅是一个整型变量，保存了heap 中特定内存地址的数据。实际上，它有一点点复杂，但这是它的基本结构。

简而言之，操作系统使用stack段中的指针值访问heap段中的对象。如果stack 对象的指针没有了，则heap 中的对象就不能访问。这也是内存泄露的原因。

## Block

- `NSStackBlock`：位于stack
- `NSMallocBlock`：位于heap
- `NSGlobalBlock`：是code segment的一部分
具体可参考[https://www.mikeash.com/pyblog/friday-qa-2010-01-15-stack-and-heap-objects-in-objective-c.html](https://www.mikeash.com/pyblog/friday-qa-2010-01-15-stack-and-heap-objects-in-objective-c.html)



## 参考
[https://www.mikeash.com/pyblog/friday-qa-2010-01-15-stack-and-heap-objects-in-objective-c.html](https://www.mikeash.com/pyblog/friday-qa-2010-01-15-stack-and-heap-objects-in-objective-c.html)
[http://mikixiyou.iteye.com/blog/1595230](http://mikixiyou.iteye.com/blog/1595230)

