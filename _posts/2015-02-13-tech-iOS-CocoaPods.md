---
layout: post
title: CocoaPods
category: 技术
tags: CocoaPods
keywords: CocoaPods
description: CocoaPods使用说明
---

在iOS开发过程中，使用CocoaPods可以方便的管理第三方开源库。
CocoaPods是一个用来帮助我们管理第三方依赖库的工具。它可以解决库与库之间的依赖关系，下载库的源代码，同时通过创建一个Xcode的workspace来将这些第三方库和我们的工程连接起来，供我们开发使用。


[用CocoaPods做iOS程序的依赖管理](http://blog.devtang.com/blog/2014/05/25/use-cocoapod-to-manage-ios-lib-dependency/)这篇博客对整个CocoaPods的使用都做了详细的介绍。
下面主要是记录一下我在使用过程中用到的一些命令：

### CocoaPods 管理
1. 查看本机上所有已经安装的gem包
> `gem list`

2. 更新Cocoapods
> `[sudo] gem install cocoapods --pre`//安装的时候有`sudo`，更新时也必须有

3. 删除某指定版本的gem包  
> `gem uninstall [gemname] --version=[ver]`


### CocoaPods 使用
#### 1. 新建
Podfile：Podfile这个文件是用来用来申明项目代码相关性的，cd进入.xcodeproj文件所在的目录，通过以下命令来创建一个Podfile
> `pod init`

#### 2. 编辑


    // 最简单的 podfile 文件
    platform :ios, '8.1'

    pod "AFNetworking", "~> 2.0"//申明需要不同版本的库，大部分情况下，申明到minor

    pod 'Y', :git => 'https://github.com/NSHipster/Y.git', :commit => 'b4dc0ffee'//对于那些不在CocoaPods公共Git仓库中的库，可以用任何一个Git仓库取代，并且还可以指定具体的commit, branch或者tag。

#### 3. 安装
> `pod install`

> `pod install --verbose --no-repo-update`

**注：**CocoaPods在执行pod install和pod update时，会默认先更新一次podspec索引。使用--no-repo-update参数可以禁止其做索引更新操作。

#### 4. 更新
> `pod update`

> `pod update --verbose --no-repo-update`

#### 5. 查找第三方库
> `pod search json`
