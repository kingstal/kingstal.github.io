---
layout: post
title: MagicalRecord(2)
category: 技术
tags: JSON,MagicalRecord,Core Data
keywords: JSON,MagicalRecord,Core Data
description: MagicalRecord 可简化 Core Data 操作的代码，并支持将 JSON 导入 Core Data
---


## [MagicalRecord](https://github.com/magicalpanda/MagicalRecord)
本篇博文主要介绍如何使用`MagicalRecord`向 CoreData 导入 JSON 数据，该过程主要分为两步：

1. 定义 JSON 导入到 NSManagedObject 的匹配规则（**无需代码**）。
2. 实施导入操作。

### 如何匹配
`MagicalRecord`利用 Xcode 数据模型工具的"User Info"值来配置导入规则，如下图所示：
![UserInfo](/assets/image/magicalrecord-2-userinfo.png)

`MagicalRecord`导入JSON时，需要对 **Entities**, **Attributes** 和 **Relationships** 进行配置。对于**Attributes** 和 **Relationships** ，默认情况下，如果 JSON 和 Model 的名称相同，那么`MagicalRecord`不需要配置。当名称不同时，需要在"User Info"下使用`mappedKeyName`来指定 JSON 中对应属性的 keypath。对于**Relationships**，还需配置`relatedByAttribute`，这个值相当于一个外键，指向另一个Entity 的主键。其实对于 CoreData 来说，并不存在主键，但是为了实现导入功能，`MagicalRecord`需要每个 Entity有一个主键，这通过`relatedByAttribute`来设置。**必须为 Entity 设置`relatedByAttribute`**，不然会出错，我在使用的时候(MagicalRecord 2.2)就因为没有设置导致 crash，找了好久才发现。

> 注：貌似使用MagicalRecord 2.3或MagicalRecord 3.0，可以不为Entity 设置`relatedByAttribute`，因为我使用CocoaPods来导入`MagicalRecord`，它的版本是2.2，不知道2.2和3.0的情况。

对于 JSON 中没有出现的属性，`MagicalRecord`不为该属性导入任何内容。

#### 1. 支持 KeyPath


```objc
// 将对应属性的 mappedKeyName 设置为 location.latitude 和 location.longitude
{
    "name": "Point Of Origin",
    "location":
    {
    "latitude": 0.00,
    "longitude": 0.00
    }
}
```


#### 2. 通过 Keys 来关联
对于**Relationships**，需要指定它目标对应的entity，用该 entity 的主键来表示 entity，即在"User Info"中设置`relatedByAttribute`的值为 entity 的主键。
![Relationships](/assets/image/magicalrecord-2-RelatedByAttribute.png)


#### 3. 与一组值关联


```objc
// 对应多个值
{
    "name": "Title of Blog post",
    "attachments": [3, 5, 100]
}
```


这时只要配置满足下面两个条件，`MagicalRecord`就能导入数据：

> **relationship** 设置为*One to Many*或*Many to Many*
> 已经指定`relatedByAttribute`为一个恰当的值


#### 4. 多个 key 对应一个属性
使用 `mappedKeyName.1 mappedKeyName.2 ... mappedKeyName.9`来配置 keypath，优先级从高到低
![Relationships](/assets/image/magicalrecord-2-mappedKeyNamesWithPriority.png)

#### 5. 日期转换（string--》NSDate）
默认：yyyy-MM-dd’T’HH:mm:ss’Z’
自定义：设置`dateFormat`
![Relationships](/assets/image/magicalrecord-2-DateAttributeWithCustomDateFormat.png)


#### 6. 导入操作回调
`MagicalRecord`允许自定义 attribute 和 relationship 的导入，通过在 Entity 中实现以下方法来自定义：

> `- (void) willImport:(id)data;`
> `- (void) didImport:(id)data;`
> `- (BOOL) shouldImport;` 


```objc
// 自定义导入
* (void)didImport:(id)data
{
    if (NO == [data isKindOfClass:[NSDictionary class]]) {
      return;
    }
  
    NSDictionary *dataDictionary = (NSDictionary *)data;

    id identifierValue = dataDictionary[@"my_identifier"];

    if ([identifierValue isKindOfClass:[NSNumber class]]) {
      NSNumber *numberValue = (NSNumber *)identifierValue;

      self.identifier = [numberValue stringValue];
    }
}
```

还可以对特定属性或关系配置导入：

> import<;attributeName>;: 
> import<;relationshipName>;:
> shouldImport<;relationshipName>;:



```objc
// 自定义属性 keywords
* (BOOL) importKeywords:(id)data;
{
    NSCharacterSet *set = [NSCharacterSet characterSetWithCharactersInString:@" ,"];
    self.keywords = [data componentsSeparatedByCharactersInSet:set];
    return YES;
}

* (BOOL) shouldImportContacts:(id)data;
{
    NSString *zipCode = [[data lastObject] valueForKey:@"zipCode"];
    return IsFormattedAsZipPlusFour(zipCode);
}
```


### 导入
配置完成后，导入就非常方便了，直接调用下面两个方法就行：

> `Person* p = [Person MR_importFromObject:dic];`
> `NSArray *people = [Person MR_importFromArray:personArray];`

到此，`MagicalRecord`的使用就介绍完了，没有仔细查看源码，只是了解了该开源库如何使用。

## 参考
[https://github.com/magicalpanda/MagicalRecord/blob/develop/Docs/Importing-Data.md](https://github.com/magicalpanda/MagicalRecord/blob/develop/Docs/Importing-Data.md)

[http://segmentfault.com/blog/lingchen/1190000002431365](http://segmentfault.com/blog/lingchen/1190000002431365)

[http://www.raywenderlich.com/56879/magicalrecord-tutorial-ios](http://www.raywenderlich.com/56879/magicalrecord-tutorial-ios)

[http://www.cimgf.com/2012/05/29/importing-data-made-easy/](http://www.cimgf.com/2012/05/29/importing-data-made-easy/)
