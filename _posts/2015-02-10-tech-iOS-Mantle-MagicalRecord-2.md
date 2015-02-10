---
layout: post
title: 双剑合璧：Mantle 与 MagicalRecord 的配合（2）
category: 技术
tags: JSON,Mantle,MagicalRecord,Core Data
keywords: JSON,Mantle,MagicalRecord,Core Data
description: Mantle 用于 JSON 和 Model 之间的转换，而 MagicalRecord 则方便将转换后的 Model 用于 Core Data 操作
---


## MagicalRecord

### 1. 配置
在`- applicationDidFinishLaunching: withOptions:`或`-awakeFromNib`方法中，使用`MagicalRecord`类方法设置：

>'+ (void)setupCoreDataStack;
+ (void)setupAutoMigratingCoreDataStack;
+ (void)setupCoreDataStackWithInMemoryStore;
+ (void)setupCoreDataStackWithStoreNamed:(NSString *)storeName;
+ (void)setupCoreDataStackWithAutoMigratingSqliteStoreNamed:(NSString *)storeName;
+ (void)setupCoreDataStackWithStoreAtURL:(NSURL *)storeURL;
+ (void)setupCoreDataStackWithAutoMigratingSqliteStoreAtURL:(NSURL *)storeURL;'

上述方法会实例化一个`Core Data stack`。

在程序退出的时候调用类方法`+cleanUp`类清除`MagicalRecord`的配置。
如果为了利用苹果 iCloud Core Data 同步，可以调用以下的方法。



> `+ (void)setupCoreDataStackWithiCloudContainer:(NSString *)containerID
                              localStoreNamed:(NSString *)localStore;

+ (void)setupCoreDataStackWithiCloudContainer:(NSString *)containerID
                               contentNameKey:(NSString *)contentNameKey
                              localStoreNamed:(NSString *)localStoreName
                      cloudStorePathComponent:(NSString *)pathSubcomponent;

+ (void)setupCoreDataStackWithiCloudContainer:(NSString *)containerID
                               contentNameKey:(NSString *)contentNameKey
                              localStoreNamed:(NSString *)localStoreName
                      cloudStorePathComponent:(NSString *)pathSubcomponent
                                   completion:(void (^)(void))completion;

+ (void)setupCoreDataStackWithiCloudContainer:(NSString *)containerID
                              localStoreAtURL:(NSURL *)storeURL;

+ (void)setupCoreDataStackWithiCloudContainer:(NSString *)containerID
                               contentNameKey:(NSString *)contentNameKey
                              localStoreAtURL:(NSURL *)storeURL
                      cloudStorePathComponent:(NSString *)pathSubcomponent;

+ (void)setupCoreDataStackWithiCloudContainer:(NSString *)containerID
                               contentNameKey:(NSString *)contentNameKey
                              localStoreAtURL:(NSURL *)storeURL
                      cloudStorePathComponent:(NSString *)pathSubcomponent
                                   completion:(void (^)(void))completion;`


### 2. 创建 Entity【插入到 managedObjectContext】：

> `Person *myPerson = [Person MR_createEntity];`默认的 context
> `Person *myPerson = [Person MR_createEntityInContext:otherContext];`特定的 context


### 3. 删除 Entity【从managedObjectContext 中删除】：

> `[myPerson MR_deleteEntity];`
> `[myPerson MR_deleteEntityInContext:otherContext];`
> `[Person MR_truncateAll];`删除所有 Entity
> `[Person MR_truncateAllInContext:otherContext];`

### 4. 查询 Entity
**基础查询**：返回的结果通常是`NSArray`。


### 5.  




## 参考
[http://segmentfault.com/blog/lingchen/1190000002431365](http://segmentfault.com/blog/lingchen/1190000002431365)
[http://www.raywenderlich.com/56879/magicalrecord-tutorial-ios](http://www.raywenderlich.com/56879/magicalrecord-tutorial-ios)

