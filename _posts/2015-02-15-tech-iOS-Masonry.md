---
layout: post
title: Masonry
category: 技术
tags: Masonry,Autolayout
keywords: Masonry,Autolayout
description: Masonry的介绍与使用
---


`Masonry`是一个轻量级的布局框架,拥有自己的描述语法,采用更优雅的链式语法封装自动布局,简洁明了,并具有高可读性,而且同时支持 iOS 和 Max OS X。

[Masonry介绍与使用实践(快速上手Autolayout)](http://adad184.com/2014/09/28/use-masonry-to-quick-solve-autolayout/)这篇博客已经详细介绍了`Masonry`的语法和使用方法。下面主要记录在学习过程中自认为需要注意的几个要点：


### 语法


    {% highlight objective-c %}
    // 为 view1 添加约束
    [view1 mas_makeConstraints:^(MASConstraintMaker *make) {
    make.edges.equalTo(superview).with.insets(padding);
    }];
    {% endhighlight %}


约束可以是：
> `.equalTo`

> `.lessThanOrEqualTo`

> `.greaterThanOrEqualTo`

> `mas_updateConstraints`：更新

> `mas_remakeConstraints`：删除


### `Masonry`与`NSLayoutAttrubute`对照
| MASViewAttribute | NSLayoutAttribute |
| :-------- | --------:|
| `view.mas_left`  | `NSLayoutAttributeLeft`|
| `view.mas_right`  | `NSLayoutAttributeRight`|
| `view.mas_top`  | `NSLayoutAttributeTop`|
| `view.mas_bottom`  | `NSLayoutAttributeBottom`|
| `view.mas_leading`  | `NSLayoutAttributeLeading`|
| `view.mas_trailing`  | `NSLayoutAttributeTrailing`|
| `view.mas_width`  | `NSLayoutAttributeWidth`|
| `view.mas_height`  | `NSLayoutAttributeHeight`|
| `view.mas_centerX`  | `NSLayoutAttributeCenterX`|
| `view.mas_centerY`  | `NSLayoutAttributeCenterY`|
| `view.mas_baseline`  | `NSLayoutAttributeBaseline`|


### 注意点

#### 1. 参数可以是数组
> `make.height.equalTo(@[view1, view3]); //can pass array of views`

#### 2. 自动装箱操作`mas_equalTo`
使得原始数据类型和结构体能作为参数，构成约束
> `make.top.mas_equalTo(42);`
> `make.edges.mas_equalTo(UIEdgeInsetsMake(10, 0, 10, 0));`
> `make.left.mas_equalTo(view).mas_offset(UIEdgeInsetsMake(10, 0, 10, 0));`


#### 3. 组合

> `edges`:top, left, bottom, right

> `size`:width and height

> `center`:centerX and centerY


**注**：第三方开源库[MMPlaceHolder](https://github.com/adad184/MMPlaceHolder)可以非常方便的显示`UIView`的大小，使用也非常方便，一行代码即可：
> `[yourView showPlaceHolder];`  `showPlaceHolderWithAllSubviews`
效果图：
![MMPlaceHolder](/assets/image/masonry-MMPlaceHolder.png)


### 参考
[https://github.com/Masonry/Masonry](https://github.com/Masonry/Masonry)

[http://adad184.com/2014/09/28/use-masonry-to-quick-solve-autolayout/](http://adad184.com/2014/09/28/use-masonry-to-quick-solve-autolayout/)
