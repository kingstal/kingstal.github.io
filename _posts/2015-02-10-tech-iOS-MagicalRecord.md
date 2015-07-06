---
layout: post
title: MagicalRecord
category: 技术
tags: JSON,MagicalRecord,Core Data
keywords: JSON,MagicalRecord,Core Data
description: MagicalRecord 可简化 Core Data 操作的代码，并支持将 JSON 导入 Core Data
---


## [MagicalRecord](https://github.com/magicalpanda/MagicalRecord)
iOS 开发过程中，经常采用CoreData来进行数据持久化。但在使用CoreData进行存取等操作时，代码量相对较多。而 `MagicalRecord` 正是为方便操作 CoreData 而生。

MagicalRecord 的三个目标：

1. 简化 CoreData 相关代码
2. 清晰、简单、单行获取数据
3. 当需要优化请求的时候，仍然允许修改 NSFetchRequest

### 1. 配置
使用 CocoaPods 安装后，在文件中导入：`#import <MagicalRecord/CoreData+MagicalRecord.h>`后即可使用。
第一步：在`- applicationDidFinishLaunching: withOptions:`或`-awakeFromNib`方法中，使用`MagicalRecord`类方法设置：

>'+ (void)setupCoreDataStack;
+ (void)setupAutoMigratingCoreDataStack;
+ (void)setupCoreDataStackWithInMemoryStore;
+ (void)setupCoreDataStackWithStoreNamed:(NSString *)storeName;
+ (void)setupCoreDataStackWithAutoMigratingSqliteStoreNamed:(NSString *)storeName;
+ (void)setupCoreDataStackWithStoreAtURL:(NSURL *)storeURL;
+ (void)setupCoreDataStackWithAutoMigratingSqliteStoreAtURL:(NSURL *)storeURL;'

上述方法会实例化一个`Core Data stack`。

在程序退出的时候调用类方法`+cleanUp`类清除`MagicalRecord`的配置。

> `[MagicalRecord cleanUp];`

如果为了利用苹果 iCloud Core Data 同步，可以调用以下的方法。

> `+ (void)setupCoreDataStackWithiCloudContainer:(NSString *)containerID
                              localStoreNamed:(NSString *)localStore;`
> `+ (void)setupCoreDataStackWithiCloudContainer:(NSString *)containerID
                               contentNameKey:(NSString *)contentNameKey
                              localStoreNamed:(NSString *)localStoreName
                      cloudStorePathComponent:(NSString *)pathSubcomponent;`
> `+ (void)setupCoreDataStackWithiCloudContainer:(NSString *)containerID
                               contentNameKey:(NSString *)contentNameKey
                              localStoreNamed:(NSString *)localStoreName
                      cloudStorePathComponent:(NSString *)pathSubcomponent
                                   completion:(void (^)(void))completion;`
> `+ (void)setupCoreDataStackWithiCloudContainer:(NSString *)containerID
                              localStoreAtURL:(NSURL *)storeURL;`
> `+ (void)setupCoreDataStackWithiCloudContainer:(NSString *)containerID
                               contentNameKey:(NSString *)contentNameKey
                              localStoreAtURL:(NSURL *)storeURL
                      cloudStorePathComponent:(NSString *)pathSubcomponent;`
> `+ (void)setupCoreDataStackWithiCloudContainer:(NSString *)containerID
                               contentNameKey:(NSString *)contentNameKey
                              localStoreAtURL:(NSURL *)storeURL
                      cloudStorePathComponent:(NSString *)pathSubcomponent
                                   completion:(void (^)(void))completion;'


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

> `NSArray *people = [Person MR_findAll];`
> `NSArray *peopleSorted = [Person MR_findAllSortedBy:@"LastName" ascending:YES];`
> `NSArray *peopleSorted = [Person MR_findAllSortedBy:@"LastName,FirstName" ascending:YES];`
> `NSArray *peopleSorted = [Person MR_findAllSortedBy:@"LastName:NO,FirstName" ascending:YES];`
> `Person *person = [Person MR_findFirstByAttribute:@"FirstName" withValue:@"Forrest"];`//键值条件查找，返回符合条件的所有数据

**高级查询**：使用`NSPredicate`

> `NSPredicate *peopleFilter = [NSPredicate predicateWithFormat:@"Department IN %@", @[dept1, dept2]];`
> `NSArray *people = [Person MR_findAllWithPredicate:peopleFilter];`

**返回`NSFetchRequest`**：可以自定义`NSFetchRequest`来进行特定查找



```objc
// 返回 NSFetchRequest
NSPredicate *peopleFilter = [NSPredicate predicateWithFormat:@"Department IN %@", departments];
NSFetchRequest *peopleRequest = [Person MR_requestAllWithPredicate:peopleFilter];
// 自定义 NSFetchRequest
[peopleRequest setReturnsDistinctResults:NO];
[peopleRequest setReturnPropertiesNamed:@[@"FirstName", @"LastName"]];
NSArray *people = [Person MR_executeFetchRequest:peopleRequest];
```


**返回 Entity 的数量**：

> `NSNumber *count = [Person MR_numberOfEntities];`
> `NSNumber *count = [Person MR_numberOfEntitiesWithPredicate:...];`

**聚合操作**：


```objc
// 聚合操作
NSNumber *totalCalories = [CTFoodDiaryEntry MR_aggregateOperation:@"sum:"
                                                  onAttribute:@"calories"
                                                withPredicate:predicate];

NSNumber *mostCalories  = [CTFoodDiaryEntry MR_aggregateOperation:@"max:"
                                                  onAttribute:@"calories"
                                                withPredicate:predicate];

NSArray *caloriesByMonth = [CTFoodDiaryEntry MR_aggregateOperation:@"sum:"
                                                   onAttribute:@"calories"
                                                 withPredicate:predicate
                                                       groupBy:@"month"];
```

### 5. 保存 Entity



```objc
// NSManagedObjectContext+MagicalSaves
+ (void) MR_saveOnlySelfWithCompletion:(MRSaveCompletionHandler)completion; // Asynchronously
+ (void) MR_saveToPersistentStoreWithCompletion:(MRSaveCompletionHandler)completion; // Asynchronously
+ (void) MR_saveOnlySelfAndWait; // Synchronously
+ (void) MR_saveToPersistentStoreAndWait; // Synchronously
+ (void) MR_saveWithOptions:(MRSaveContextOptions)mask completion:(MRSaveCompletionHandler)completion;

// MagicalRecord+Actions
* (void) saveWithBlock:(void(^)(NSManagedObjectContext *localContext))block; // Asynchronously
* (void) saveWithBlock:(void(^)(NSManagedObjectContext *localContext))block completion:(MRSaveCompletionHandler)completion; // Asynchronously
* (void) saveWithBlockAndWait:(void(^)(NSManagedObjectContext *localContext))block;
* (void) saveUsingCurrentThreadContextWithBlock:(void (^)(NSManagedObjectContext *localContext))block completion:(MRSaveCompletionHandler)completion;
* (void) saveUsingCurrentThreadContextWithBlockAndWait:(void (^)(NSManagedObjectContext *localContext))block;
```

下面看一个实例：



```objc
// 0. setup
[MagicalRecord setupCoreDataStackWithStoreNamed:@"Model"];

// 1. insert
for (int i = 1; i < 5; i++) {
    Person* p = [Person MR_createEntity];
    p.name = [NSString stringWithFormat:@"p%d", i];
    p.age = [NSNumber numberWithInt:i];
}

// 2. delete
Person* pd = [Person MR_findFirst];
[pd MR_deleteEntity];

// 3. update
Person* pu = [Person MR_findFirstByAttribute:@"name" withValue:@"p2"];
pu.name = @"p22";

// 4. save
[[NSManagedObjectContext MR_defaultContext] MR_saveToPersistentStoreWithCompletion:^(BOOL success, NSError* error) {
    if (success) {
        NSLog(@"***save success!***");
    }
    else {
        NSLog(@"***save failure!***");
    }
}];
```


上述主要讲了如何使用`MagicalRecord`来简化 CoreData 操作的代码，下一篇将主要介绍[如何使用`MagicalRecord`向 CoreData 导入 JSON 数据](http://kingstal.github.io/2015/02/11/tech-iOS-MagicalRecord-2.html)。




## 参考
[http://segmentfault.com/blog/lingchen/1190000002431365](http://segmentfault.com/blog/lingchen/1190000002431365)
[http://www.raywenderlich.com/56879/magicalrecord-tutorial-ios](http://www.raywenderlich.com/56879/magicalrecord-tutorial-ios)
