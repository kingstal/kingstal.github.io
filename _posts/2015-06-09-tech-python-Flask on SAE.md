---
layout: post
title: 在 SAE 上搭建 Flask 服务器
category: 技术
tags: python, flask, SAE 
keywords: python, flask, SAE
description:
---


最近打算自己做一个 和豆瓣相关的 App，数据从网站抓取解析之后返回给客户端，需要一个服务器完成数据的解析过程，想到使用 SAE 来搭建一个免费的服务器，使用的框架是`Flask`。

### 创建应用Flask服务器

1. 登录SAE，进入[我的首页](http://sae.sina.com.cn/?m=dashboard)，点击**创建新应用** ，创建一个新的应用`MJdeDouBan`。 开发语言选择Python。
2. 按照入门指南创建`config.yaml`、`index.wsgi`、`server.py`，其中内容根据实际情况。这时候访问[http://mjdedouban.sinaapp.com](http://mjdedouban.sinaapp.com)就能看到`Flask`服务器开始工作了。

### 使用第三方包
网上有介绍可以使用`virtualenv`来管理 Python 开发时所需要的第三方包，但在配置过程中出现问题，我这里采用比较傻瓜的方法来配置使用第三方包。

1. 登陆SAE并来到“**代码管理**”页面，点击“**操作**”下的**“上传代码包**”上传需要使用的第三方包。**注意**：只支持上传`zip`、 `gz`、`tar.gz`三种代码包，文件大小不能超过20MB，同名文件将会被覆盖。上传后的目录结构将跟压缩包内的目录结构保持一致。

2. 再回到“操作”那，选择编辑代码，进入在线编辑代码平台。在左侧的`index.wsgi`中加入如下内容：

```python
import os
import sys

root = os.path.dirname(__file__)
sys.path.insert(0, os.path.join(root, 'cssselect-0.9.1'))#为刚才上传后出现在编辑平台左侧的目录名称，必须一致。
```

到此，所有的依赖包已经导出并加入到应用的目录里了。








