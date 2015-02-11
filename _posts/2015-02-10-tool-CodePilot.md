---
layout: post
title: CodePilot
category: 工具
tags: iOS,CodePilot
keywords: iOS,CodePilot
description: 如何在 XCode 6上安装 CodePilot
---


今天在写代码的时候，发现查找 project 中文件的时候不是很方便，就想到使用 Code Pilot。其实以前也打算配置，但那时并没有真正开始
写代码，所以配置失败也没放在心上，今天终于搞定了，打算把该过程记下来：

1. 从[GitHub CodePilot](https://github.com/macoscope/CodePilot)下载该项目。
2. 打开 Xcode 编译该项目。这里有一点需要注意，需要将`Scheme`从`Build Package`换成CodePilot`，不然编译失败。
3. 编译成功后，从`Products`文件夹下找到`CodePilot3.xcplugin`，拷贝到插件目录。 GitHub 上说的目录为`~/Library/Application\ 
Support/Developer/Shared/Xcode/Plug-ins`，通过实践发现那样没有效果。后来在该项目的`issue #11`中发现，原来在 Xcode 5之后，
需要将插件放在 Xcode 程序文件夹下，原文是这么说的`In Xcode5, you should put the .xcplungin in the PlunIns folder within 
Xcode bundle instead of the system library path.`，即`/Applications/Xcode.app/Contents/PlugIns/`。
4. 经过以上3个步骤，即可在`File`菜单栏下找到CodePilot，可以打开 Keyboard->快捷键，为其设置一个自己熟悉的快捷键。
