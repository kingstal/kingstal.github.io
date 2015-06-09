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

1. 登录SAE，进入[http://sae.sina.com.cn/?m=dashboard](我的首页)，点击**创建新应用** ，创建一个新的应用`MJdeDouBan`。 开发语言选择Python。
2. 按照入门指南创建`config.yaml`、`index.wsgi`、`server.py`，其中内容根据实际情况。这时候访问[http://mjdedouban.sinaapp.com](http://mjdedouban.sinaapp.com)就能看到`Flask`服务器开始工作了。

### 使用`virtualenv`管理依赖关系
使用`virtualenv`可以很方便的管理 Python 开发时所需要的第三方包。
1. 创建一个全新的Python虚拟环境目录ENV，启动虚拟环境。

```
$ virtualenv --no-site-packages ENV
$ source ENV/bin/activate
(ENV)$
```
可以看到命令行提示符的前面多了一个(ENV)的前缀，现在已经在一个全新的虚拟环境中了。
2. 使用pip安装应用所依赖的包并导出依赖关系到requirements.txt。

```
(ENV)$ pip install Flask Flask-Cache Flask-SQLAlchemy
(ENV)$ pip freeze > requirements.txt
```
编辑`requirements.txt`文件，删除一些sae内置的模块，eg. flask, jinja2, wtforms。
3. 使用`dev_server/bundle_local.py`工具，将所有requirements.txt中列出的包导出到本地目录virtualenv.bundle目录中。如果文件比较多的话，推荐压缩后再上传。

```
(ENV)$ bundle_local.py -r requirements.txt
(ENV)$ cd virtualenv.bundle/
(ENV)$ zip -r ../virtualenv.bundle.zip .
```
4. 将virutalenv.bundle目录或者virtualenv.bundle.zip拷贝到应用的目录下
5. 修改`index.wsgi`文件，在导入其它模块之前，将virtualenv.bundle目录或者virtualenv.bundle.zip添加到module的搜索路径中，示例代码如下：

```
import os
import sys

app_root = os.path.dirname(__file__)

# 两者取其一
sys.path.insert(0, os.path.join(app_root, 'virtualenv.bundle'))
sys.path.insert(0, os.path.join(app_root, 'virtualenv.bundle.zip'))
```
到此，所有的依赖包已经导出并加入到应用的目录里了。

**注意：**
删除requirements.txt中的wsgiref==0.1.2这个依赖关系，否则可能导致 bundle_local.py导出依赖包失败。







