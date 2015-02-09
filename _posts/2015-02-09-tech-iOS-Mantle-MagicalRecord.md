---
layout: post
title: Mantle 和 MagicalRecord 结合，用于操作 Core Data
category: 技术
tags: JSON,Mantle,MagicalRecord,Core Data
keywords: JSON,Mantle,MagicalRecord,Core Data
description: Mantle 用于 JSON 和 Model 之间的转换，而 MagicalRecord 则方便将转换后的 Model 用于 Core Data 操作
---

# Mantle
Mantle 是一个模型框架，支持将 JSON 解析为 Model 对象，也可以反向操作，即将 Model 对象序列化为 JSON。同时，支持 Core  Data 的序列化和反序列化。下面分两个部分介绍

### JSON <--> Model
Mantle 提供了一个基类：MTLModel，如果想使用 Mantle 的各种功能，那么所创建的模型必须是这个类的子类。为了实现这两者的转换，还必须遵守`<MTLJSONSerializing>`协议，该协议常用的有以下两个方法：
> 1. `+ (NSDictionary *)JSONKeyPathsByPropertyKey;`
> 该方法是必须实现的，它返回一个字典，用于 Model property 和 JSON key 值之间的匹配。

下面看一个实例。
> Member.h

    
    {% highlight objective-c %}
    @interface Member : MTLModel<MTLJSONSerializing>
    @property (nonatomic, retain) NSString * memberID;
    @property (nonatomic, retain) NSString * mobilePhone;
    @property (nonatomic, retain) NSDate * createDate;
    @property (nonatomic, retain) NSNumber *goldNumber;
    @property (nonatomic, assign) NSUInteger age;
    @property (nonatomic, assign) BOOL isVip;
    @property (nonatomic, retain) NSURL *url;
    {% endhighlight %}


> Member.m


    {% highlight objective-c %}
    //当property 和 json keypath 的名称相同时，可忽略
    + (NSDictionary *)JSONKeyPathsByPropertyKey{
        return @{@"memberID" : @"id",
            @"mobilePhone" : @"phone",
            @"createDate" : @"date"
        };
    }
    {% endhighlight %}



> 2. `+ (NSValueTransformer *)JSONTransformerForKey:(NSString *)key;`
> 有时候 model 和 JSON 之间的类型并不相同，该方法可以指定如何来进行转换。

具体实现：




### Core Data 相关
Mantle提供了一个专门操作Core Data的协议`<MTLManagedObjectSerializing>`，常见的方法有以下几个：
> 1. `+ (NSString *)managedObjectEntityName;`  
> 该方法是必须实现的，`返回此类对应的 Entity 类别`。
> 
> 2. `+ (NSDictionary *)managedObjectKeysByPropertyKey;`
> 该方法是必须实现的，返回一个字典，用于匹配 model property 和 entity property 的 key。
> 
> 3. `+ (NSValueTransformer *)entityAttributeTransformerForKey:(NSString *)key;`
> 类似于 model 和 JSON 之间的转换，该方法用于 model 和 entity 之间的属性值转换。

具体实现：


