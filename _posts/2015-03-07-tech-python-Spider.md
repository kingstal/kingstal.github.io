---
layout: post
title: Python爬虫
category: 技术
tags: Python,爬虫
keywords: Python,爬虫
description: 介绍如何使用 Python 来写爬虫
---

上个学期使用 JAVA 做了简单的爬虫项目，期间遇到了许多问题，最近在全面学习`Python`，发现利用`Python`写爬虫非常方便，现在就把如何利用`Python`完成爬虫记录下来。


### 获取静态网页的数据
这个比较简单，数据暴露在页面的源代码中，只需要获取源代码，然后匹配相应的标签就可以获取想要的数据。
上代码：


    {% highlight Python %}
    # 获取煎蛋网的图片 url 保存到本地文件
    from pyquery import PyQuery as pq #利用 pyquery 来解析

    fileToSave = open('imglinks.txt','w')

    for page_number in range(6100,6110): #网页的第6100~6110
        url = "http://jandan.net/pic/page-" + str(page_number)
        html_content = pq(url)

        jpg_container = []
        for anchor in html_content('#comments p>img'): #图片的标签路径
            anchor = html_content(anchor)
            jpg_link = anchor.attr('src') #获取图片标签的 src 属性即为 url
            jpg_container.append(jpg_link)

            fileToSave.write(jpg_link+'\n')
        print(page_number)

    fileToSave.close
    {% endhighlight %}
