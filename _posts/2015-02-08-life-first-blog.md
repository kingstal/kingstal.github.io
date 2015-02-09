---
layout: post
title: 第一篇博客
category: 生活
tags:
keywords:
description:
---

2015-02-10 22：11 

从下午忙到现在，终于把博客的平台搞定了，这其中很感谢[suyan](https://github.com/suyan)，整个框架都是他提供的，基本没做改动，
希望在以后的使用过程中能够有机会对这个模板进行修改。

同时，也希望自己以后能过持之以恒的把这个博客写下去。

### 搭建过程中遇到的小问题也 mark 一下：

- 使用 Disqus 来为博客添加评论，以前没接触过，原来它是需要注册的，注册之后就会有一个 shortname，然后在配置中在相应位置改为自己的 shortname 就 OK 了。

- 配置 avatar 时，通过在文件目录下上传图片，然后在配置中在相应位置改为 avatar 的文件路径。

- google analytics 设置，首先需要注册google analytics 服务，会得到一个'Tracking ID'，然后在配置文件中改为该 ID 即可。 

- 代码高亮设置，配置文件中`pygments`的设置改为`highlighter: pygments`，在写代码块的时候应在头尾加上

> `{``% highlight python %``}` 、 `\{% endhighlight %\}` 。


