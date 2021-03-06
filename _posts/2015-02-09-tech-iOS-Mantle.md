---
layout: post
title: Mantle
category: 技术
tags: JSON,Mantle
keywords: JSON,Mantle
description: Mantle 用于 JSON 和 Model 之间的转换
---


## [Mantle](https://github.com/Mantle/Mantle)
Mantle 是一个Model 层框架，支持将 JSON 解析为 Model 对象，也可以反向操作，即将 Model 对象序列化为 JSON。同时，支持 Core  Data 的序列化和反序列化。下面分两个部分介绍

### 1. JSON <---> Model
Mantle 提供了一个基类：MTLModel，如果想使用 Mantle 的各种功能，那么所创建的模型必须是这个类的子类。为了实现这两者的转换，还必须遵守`<MTLJSONSerializing>`协议，该协议常用的有以下两个方法：

> > `+ (NSDictionary *)JSONKeyPathsByPropertyKey;`

> 该方法是必须实现的，它返回一个字典，用于 Model property 和 JSON key 值之间的匹配。

下面看一个实例。

```objc
// Member.h
@interface Member : MTLModel<MTLJSONSerializing>
@property (nonatomic, retain) NSString * memberID;
@property (nonatomic, retain) NSString * mobilePhone;
@property (nonatomic, retain) NSDate * createDate;
@property (nonatomic, retain) NSNumber *goldNumber;
@property (nonatomic, assign) NSUInteger age;
@property (nonatomic, assign) BOOL isVip;
@property (nonatomic, retain) NSURL *url;
```




```objc
// Member.m
//当property 和 json keypath 的名称相同时，可忽略
- (NSDictionary *)JSONKeyPathsByPropertyKey{
    return @{@"memberID" : @"id",//支持 keypath，如“location.latitude”
        @"mobilePhone" : @"phone",
        @"createDate" : @"date"
    };
}
```



> > `+ (NSValueTransformer *)JSONTransformerForKey:(NSString *)key;`

> 有时候 model 和 JSON 之间的类型并不相同，该方法可以指定如何来进行转换。

具体实现：


```objc
//属性值转换
- (NSValueTransformer *)JSONTransformerForKey:(NSString *)key{
    if ([key isEqualToString:@"createDate"]) {
        return [MTLValueTransformer reversibleTransformerWithForwardBlock:^id(NSString *string) {
            //NSString-->NSDate
            return [self.dateFormatter dateFromString:string];
        } reverseBlock:^id(NSDate *date) {
            //NSDate-->NSString
            return [self.dateFormatter stringFromDate:date];
        }];
    }
    else{
        return nil;
    }
}

- (NSDateFormatter *)dateFormatter {
    NSDateFormatter *dateFormatter = [[NSDateFormatter alloc] init];
    dateFormatter.dateFormat = @"yyyy-MM-dd";
    return dateFormatter;
}
```

> `+ (instancetype)reversibleTransformerWithForwardBlock:(MTLValueTransformerBlock)forwardBlock reverseBlock:(MTLValueTransformerBlock)reverseBlock;`

第一个Block的返回的值是 `JSON`-->`model`转换的结果，第二Block的返回值是`model`-->`JSON`转换的结果。当然只需要序列化，那么就实现单向转换即可，使用下列API：

> `+ (instancetype)reversibleTransformerWithBlock:(MTLValueTransformerBlock)transformationBlock;`

Mantle 还提供了另一种转换方式，同样的实现上述功能：


```objc
// 针对特定的属性进行转换
- (NSValueTransformer *)createDateJSONTransformer{
    return [MTLValueTransformer reversibleTransformerWithForwardBlock:^id(NSString *string) {
        return [self.dateFormatter dateFromString:string];
    } reverseBlock:^id(NSDate *date) {
         return [self.dateFormatter stringFromDate:date];
    }];
}
```

方法命名规则是：`+<key>JSONTransformer`，另外对于BOOL和NSURL类型的有更快捷的方法：


```objc
// URL 转换的快捷方法
- (NSValueTransformer *)urlJSONTransformer{
    return [NSValueTransformer valueTransformerForName:MTLURLValueTransformerName];
}

// BOOL 类型转换的快捷方法
- (NSValueTransformer *)isVipJSONTransformer{
    return [NSValueTransformer valueTransformerForName:MTLBooleanValueTransformerName];
}
```


> 空对象处理

有时候 JSON 传过来的对象值为空，这时需要对空对象进行处理。假定`@"isVip" : NSNull.null`


```objc
// 这种问题其实只针对于非指针类型，像float，bool，double
* (void)setNilValueForKey:(NSString *)key{
    if ([key isEqualToString:@"isVip"]) {
        self.isVip = 0;
    }
    else{
        [super setNilValueForKey:key];
    }
}
```

**这时，就可以调用`MTLJSONAdapter`的类方法实现 JSON <---> model 的互相转换。**

> 1. `+ (id)modelOfClass:(Class)modelClass fromJSONDictionary:(NSDictionary *)JSONDictionary error:(NSError **)error;`
> 2. `+ (NSArray *)modelsOfClass:(Class)modelClass fromJSONArray:(NSArray *)JSONArray error:(NSError **)error;`
> 3. `+ (NSDictionary *)JSONDictionaryFromModel:(MTLModel<MTLJSONSerializing> *)model;`
> 4. `+ (NSArray *)JSONArrayFromModels:(NSArray *)models;`

具体参看如下：


```objc
// 调用MTLJSONAdapter的类方法
NSDictionary *response = @{@"id" : @"1",
                      @"phone" : @"xxxxxx",
                      @"date" : @"2014-09-09",
                      @"goldNumber" : @2,
                      @"age" : @"18",
                      @"url" : @"http://bawn.github.io/",
                      @"isVip" : NSNull.null
                      };
Member *member = [MTLJSONAdapter modelOfClass:[Member class] fromJSONDictionary:response error:nil];

NSDictionary *dictionary = [MTLJSONAdapter JSONDictionaryFromModel:member];
```

### 2. Core Data 相关
Mantle提供了一个专门操作Core Data的协议`<MTLManagedObjectSerializing>`，常见的方法有以下几个：

> 1. `+ (NSString *)managedObjectEntityName;`  
> 该方法是必须实现的，`返回此类对应的 Entity 类别`。
>
> 2. `+ (NSDictionary *)managedObjectKeysByPropertyKey;`
> 该方法是必须实现的，返回一个字典，用于匹配 model property 和 entity property 的 key，即 model 和 Entity 属性的映射关系。
>
> 3. `+ (NSValueTransformer *)entityAttributeTransformerForKey:(NSString *)key;`
> 类似于 model 和 JSON 之间的转换，该方法用于 model 和 entity 之间的属性值转换。
>
> 4. `+ (NSSet *)propertyKeysForManagedObjectUniquing;`
> 唯一性检查。
>
> 5. `+ (NSDictionary *)relationshipModelClassesByPropertyKey;`
> 当 Entity 具有`relationship`的属性时，可以用该方法来进行匹配。


### 3. 调用`MTLManagedObjectAdapter`实现`MTLModel`和`NSManagedObject`的转换

> 1. `+ (id)modelOfClass:(Class)modelClass fromManagedObject:(NSManagedObject *)managedObject error:(NSError **)error;`
> `NSManagedObject`——>`MTLModel`
> 2. `+ (id)managedObjectFromModel:(MTLModel<MTLManagedObjectSerializing> *)model insertingIntoContext:(NSManagedObjectContext *)context error:(NSError **)error;`
> 'MTLModel'——>`NSManagedObject`


使用 Mantle 实现 JSON 的持久化，需要进行两次转化：`JSON`--》`MTLModel`--》`NSManagedObject`。这时我们可以借助于另一个开源库 `MagicalRecord`，它可以极大的方便 Core Data 的使用，同时也支持将 JSON 导入 Core Data。下一篇将会介绍[如何使用MagicalRecord 来进行 Core Data 操作](http://kingstal.github.io/2015/02/10/tech-iOS-MagicalRecord.html)。





## 参考
[http://bawn.github.io/ios/2014/12/11/Mantle.html](http://bawn.github.io/ios/2014/12/11/Mantle.html)

[https://github.com/Mantle/Mantle](https://github.com/Mantle/Mantle)

[http://robinchao.github.io/blog/2014/11/08/mantle-jian-dan-shi-yong-jiao-cheng/](http://robinchao.github.io/blog/2014/11/08/mantle-jian-dan-shi-yong-jiao-cheng/)
