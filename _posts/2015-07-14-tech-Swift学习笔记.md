---
layout: post
title: Swift学习笔记
category: 技术
tags: Swift
keywords: Swift
description: Swift
---

## 基础部分

### 常量和变量

```swift
let maximumNumberOfLoginAttempts = 10
var currentLoginAttempt = 0
```

- 类型标注（type annotation）

```swift
var welcomeMessage: String
```

- 输出常量和变量

```swift
println("The current value of friendlyWelcome is \(friendlyWelcome)")
// Swift 用字符串插值（string interpolation）的方式把常量名或者变量名当做占位符加入到长字符串中，Swift 会用当前常量或变量的值替换这些占位符
```

- 类型别名（type aliases）

```swift
typealias AudioSample = UInt16
```

- 元组（tuples）
把多个值组合成一个复合值。元组内的值可以是任意类型，并不要求是相同类型。

```swift
et http404Error = (404, "Not Found")
// http404Error 的类型是 (Int, String)，值是 (404, "Not Found")
let (statusCode, statusMessage) = http404Error
println("The status code is \(statusCode)")
// 可以将一个元组的内容分解（decompose）成单独的常量和变量，输出 "The status code is 404"
let (justTheStatusCode, _) = http404Error
println("The status code is \(justTheStatusCode)")
// 只需要一部分元组值，分解的时候可以把要忽略的部分用下划线（_）标记，输出 "The status code is 404"
println("The status code is \(http404Error.0)")
// 通过下标来访问元组中的单个元素，输出 "The status code is 404"

let http200Status = (statusCode: 200, description: "OK")
// 定义元组的时候给单个元素命名
println("The status code is \(http200Status.statusCode)")
// 给元组中的元素命名后，可以通过名字来获取这些元素的值，输出 "The status code is 200"
```

- 可选类型（optionals）
来处理值可能缺失的情况。可选类型表示：`有值，等于x`或`没有值`

```swift
let possibleNumber = "123"
let convertedNumber = possibleNumber.toInt()
// convertedNumber 被推测为类型 "Int?"， 或者类型 "optional Int"
```

可以使用`if`语句来判断一个可选是否包含值。如果可选类型有值，结果是`true`；如果没有值，结果是`false`。
当确定可选类型_确实_包含值之后，可以在可选的名字后面加一个感叹号（`!`）来获取值。这个惊叹号表示“我知道这个可选有值，请使用它。”这被称为可选值的_强制解析（forced unwrapping）_：

```swift
if convertedNumber != nil {
    println("\(possibleNumber) has an integer value of \(convertedNumber!)")
} else {
    println("\(possibleNumber) could not be converted to an integer")
}
// 输出 "123 has an integer value of 123"
```

使用_可选绑定（optional binding）_来判断可选类型是否包含值，如果包含就把值赋给一个临时常量或者变量。可选绑定可以用在`if`和`while`语句中来对可选类型的值进行判断并把值赋给一个常量或者变量。

```swift
if let actualNumber = possibleNumber.toInt() {
    println("\(possibleNumber) has an integer value of \(actualNumber)")
} else {
    println("\(possibleNumber) could not be converted to an integer")
}
// 输出 "123 has an integer value of 123"
```

这段代码可以被理解为：
“如果`possibleNumber.toInt`返回的可选`Int`包含一个值，创建一个叫做`actualNumber`的新常量并将可选包含的值赋给它。”
如果转换成功，`actualNumber`常量可以在`if`语句的第一个分支中使用。它已经被可选类型_包含的_值初始化过，所以不需要再使用`!`后缀来获取它的值。
可以在可选绑定中使用常量和变量。如果想在`if`语句的第一个分支中操作`actualNumber`的值，可以改成`if var actualNumber`，这样可选类型包含的值就会被赋给一个变量而非常量。

可以给可选变量赋值为`nil`来表示它没有值。

> 注意：
`nil`不能用于非可选的常量和变量。如果你的代码中有常量或者变量需要处理值缺失的情况，请把它们声明成对应的可选类型。
> 注意：
Swift 的`nil`和 Objective-C 中的`nil`并不一样。在 Objective-C 中，`nil`是一个指向不存在对象的指针。在 Swift 中，`nil`不是指针——它是一个确定的值，用来表示值缺失。_任何_类型的可选状态都可以被设置为`nil`，不只是对象类型。

第一次被赋值之后，可以确定一个可选类型_总会_有值，这种类型的可选状态被定义为_隐式解析可选类型（implicitly unwrapped optionals）_。把想要用作可选的类型的后面的问号（`String?`）改成感叹号（`String!`）来声明一个隐式解析可选类型。

一个隐式解析可选类型其实就是一个普通的可选类型，但是可以被当做非可选类型来使用，并不需要每次都使用解析来获取可选值。

```swift
let assumedString: String! = "An implicitly unwrapped optional string."
println(assumedString)  // 不需要感叹号
// 输出 "An implicitly unwrapped optional string."
```

可以把隐式解析可选类型当做一个可以自动解析的可选类型。你要做的只是声明的时候把感叹号放到类型的结尾，而不是每次取值的可选名字的结尾。

仍然可以把隐式解析可选类型当做普通可选类型来判断它是否包含值：

```swift
if assumedString {
    println(assumedString)
}
// 输出 "An implicitly unwrapped optional string."
```

- 断言
断言会在运行时判断一个逻辑条件是否为`true`。
可以使用全局`assert`函数来写一个断言。向`assert`函数传入一个结果为`true`或者`false`的表达式以及一条信息，当表达式为`false`的时候这条信息会被显示：

```swift
let age = -3
assert(age >= 0, "A person's age cannot be less than zero")
// 因为 age < 0，所以断言会触发
```

### 基本运算符

- 空合运算符(Nil Coalescing Operator)
如果`a`包含一个值就进行解封，否则就返回一个默认值`b`.这个运算符有两个条件:
1. 表达式`a`必须是Optional类型
2. 默认值`b`的类型必须要和`a`存储值的类型保持一致

空合并运算符是对以下代码的简短表达方法

    a != nil ? a! : b

上述代码使用了三目运算符。当可选类型`a`的值不为空时，进行强制解封(`a!`)访问`a`中值，反之当`a`中值为空时，返回默认值b。无疑空合运算符(`??`)

- 区间运算符
闭区间运算符（`a...b`）定义一个包含从`a`到`b`(包括`a`和`b`)的所有值的区间，`b`必须大于`a`。
半开区间（`a..<b`）定义一个从`a`到`b`但不包括`b`的区间。

### 字符串和字符
- 字符串索引 (String Indices)
每一个字符串都有一个关联的索引(*index*)类型，`String.index`，它对应着字符串中的每一个字符的位置。
前面提到，不同的字符可能会占用不同的内存空间数量，所以要知道字符的确定位置，就必须从字符串开头遍历每一个 Unicode 标量到字符串结尾。因此，Swift 的字符串不能用整数(integer)做索引。
使用`startIndex`属性可以获取字符串的第一个字符。使用`endIndex`属性可以获取最后一个字符的末尾位置。如果字符串是空值，`startIndex`和`endIndex`是相等的。
通过调用`String.Index`的`predecessor()`方法，可以立即得到前面一个索引，调用`successor()`方法可以立即得到后面一个索引。任何一个字符串的索引都可以通过锁链作用的这些方法来获取另一个索引，也可以调用`advance(start:n:)`函数来获取。

```swift
let greeting = "Guten Tag"
greeting[greeting.startIndex]
// G
greeting[greeting.endIndex.predecessor()]
// g
greeting[greeting.startIndex.successor()]
// u
let index = advance(greeting.startIndex, 7)
greeting[index]
// a
greeting[greeting.endIndex] // 错误
greeting.endIndex.successor() // 错误
```

使用`characters`属性的`indices`会创建一个包含全部索引的范围(`Range`)，用来在一个字符串中访问分立的字符。

```swift
for index in greeting.characters.indices {
    print("\(greeting[index]) ", appendNewline: false)
}
// 打印输出 "G u t e n   T a g !"
```

- 插入和删除 (Inserting and Removing)
调用`insert(_:atIndex:)`方法可以在一个字符串的指定索引插入一个字符。

```swift
var welcome = "hello"
welcome.insert("!", atIndex: welcome.endIndex)
// welcome now 现在等于 "hello!"
```

调用`splice(_:atIndex:)`方法可以在一个字符串的指定索引插入一个字符串。

```swift
welcome.splice(" there".characters, atIndex: welcome.endIndex.predecessor())
// welcome 现在等于 "hello there!"
```

调用`removeAtIndex(_:)`方法可以在一个字符串的指定索引删除一个字符。

```swift
welcome.removeAtIndex(welcome.endIndex.predecessor())
// welcome 现在等于 "hello there"
// 翻译的人解释：最后还有一个换行符，所以这里删除的是 !
```

调用`removeRange(_:)`方法可以在一个字符串的指定索引删除一个子字符串。

```swift
let range = advance(welcome.endIndex, -6)..<welcome.endIndex
welcome.removeRange(range)
// welcome 现在等于 "hello"
```

- 比较字符串 (Comparing Strings)

```swift
// 字符串/字符相等
if quotation == sameQuotation {
    print("These two strings are considered equal")
}

// 前缀/后缀相等 (Prefix and Suffix Equality)
for scene in romeoAndJuliet {
    if scene.hasPrefix("Act 1 ") {
        ++act1SceneCount
    }
}
```

### 集合类型


