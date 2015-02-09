---
layout: post
title: Markdown语法测试
category: 技术
tags: Markdown
description: 测试Markdown的语法，同时留作以后参考
---




# h1 1阶标题

## h2.1 2阶标题


### h3.1 3阶标题

> This is the first level of quoting.区块应用

> > This is nested blockquote.区块嵌套

> Back to the first level.


### h3.2

> ## 这是一个2阶标题。  引用的区块内也可以使用其他的 Markdown 语法
> 
> 1.   这是第一行列表项。
> 2. 这是第二行列表项。
> 
> 给出一些例子代码：
> 

### h3.3 列表

- Red
- Green
- Blue

1. Bird
2. McHale
3. Parish


列表项目可以包含多个段落，每个项目下的段落都必须缩进 4 个空格或是 1 个制表符：

1.  This is a list item with two paragraphs. Lorem ipsum dolor
    sit amet, consectetuer adipiscing elit. Aliquam hendrerit
    mi posuere lectus.

    Vestibulum enim wisi, viverra nec, fringilla in, laoreet
    vitae, risus. Donec sit amet nisl. Aliquam semper ipsum
    sit amet velit.

2.  Suspendisse id sem consectetuer libero luctus adipiscing.

## h2.2


代码区块很简单，只要简单地缩进 4 个空格或是 1 个制表符就可以

    
    +(NSDictionary *)JSONKeyPathsByPropertyKey {
      return @{
        @"date" : @"dt", //将JSON字典里dt键对应的值，赋值给date属性
        @"locationName" : @"name",
        @"humidity" : @"main.humidity",
        @"temperature" : @"main.temp", 
        @"tempHigh" : @"main.temp_max",
        @"tempLow" : @"main.temp_min",
      };
    }
  
  
  
  
-------------------------

一行中用三个以上的星号、减号、底线来建立一个分隔线


> 链接

This is [我的github](https://github.com/kingstal) inline link.
指向本机文件的路径[我的头像](/assets/image/avatar.png)。

插入图片

![我的头像](/assets/image/avatar.png)


*斜体*
**粗体**
***粗体+斜体***

代码`print`

*斜体*
**粗体**



    {% highlight ruby %}
    def foo
        puts 'foo'
    end
    {% endhighlight %}
    

    {% highlight object-c %}
    -(void)getName{
        return @"wangminjun";
    }
    {% endhighlight %}


    {% highlight python %}
    import this
    print 'something'
    {% endhighlight %}
    
