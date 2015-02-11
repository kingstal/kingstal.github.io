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
在文件中导入：`#import <MagicalRecord/CoreData+MagicalRecord.h>`
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



    {% highlight objective-c %}
    // 返回 NSFetchRequest
    NSPredicate *peopleFilter = [NSPredicate predicateWithFormat:@"Department IN %@", departments];
    NSFetchRequest *peopleRequest = [Person MR_requestAllWithPredicate:peopleFilter];
    // 自定义 NSFetchRequest
    [peopleRequest setReturnsDistinctResults:NO];
    [peopleRequest setReturnPropertiesNamed:@[@"FirstName", @"LastName"]];
    NSArray *people = [Person MR_executeFetchRequest:peopleRequest];
    {% endhighlight %}


**返回 Entity 的数量**：

> `NSNumber *count = [Person MR_numberOfEntities];`
> `NSNumber *count = [Person MR_numberOfEntitiesWithPredicate:...];`

**聚合操作**：


    {% highlight objective-c %}
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
    {% endhighlight %}

### 5. 保存 Entity



    {% highlight objective-c %}
    // NSManagedObjectContext+MagicalSaves
    - (void) MR_saveOnlySelfWithCompletion:(MRSaveCompletionHandler)completion; // Asynchronously
    - (void) MR_saveToPersistentStoreWithCompletion:(MRSaveCompletionHandler)completion; // Asynchronously
    - (void) MR_saveOnlySelfAndWait; // Synchronously
    - (void) MR_saveToPersistentStoreAndWait; // Synchronously
    - (void) MR_saveWithOptions:(MRSaveContextOptions)mask completion:(MRSaveCompletionHandler)completion;

    // MagicalRecord+Actions
    + (void) saveWithBlock:(void(^)(NSManagedObjectContext *localContext))block; // Asynchronously
    + (void) saveWithBlock:(void(^)(NSManagedObjectContext *localContext))block completion:(MRSaveCompletionHandler)completion; // Asynchronously
    + (void) saveWithBlockAndWait:(void(^)(NSManagedObjectContext *localContext))block;
    + (void) saveUsingCurrentThreadContextWithBlock:(void (^)(NSManagedObjectContext *localContext))block completion:(MRSaveCompletionHandler)completion;
    + (void) saveUsingCurrentThreadContextWithBlockAndWait:(void (^)(NSManagedObjectContext *localContext))block;
    {% endhighlight %}


要进行 Core Data 操作，必须有相应的 Entity，接[上一篇的案例]，我们新建如下实体：
![实体](/Users/Arthur/Dropbox/blog/kingstal.github.io/assets/image/mantlemagicalrecord2-1.png)


    {% highlight objective-c %}
    // MemberManaged.h
    @interface MemberManaged : NSManagedObject
    @property(nonatomic, retain) NSString *memberID;
    @property(nonatomic, retain) NSString *mobilePhone;
    @property(nonatomic, retain) NSDate *createDate;
    @property(nonatomic, retain) NSNumber *goldNumber;
    @property(nonatomic, retain) NSNumber *age;
    @property(nonatomic, retain) NSNumber *isVip;
    @property(nonatomic, retain) NSString *url;
    {% endhighlight %}

## Mantle 和 MagicalRecord 的结合
### 实现`<MTLManagedObjectSerializing>`协议中的相应方法：

> `Member.m`



    {% highlight objective-c %}
    //表示Member类对应的实体类是MemberManaged
    + (NSString *)managedObjectEntityName{
        return @"MemberManaged";
    }

    //表示Member类向MemberManaged类转换的字段映射，因为Member类的字段名是相同，所以这里返回nil
    + (NSDictionary *)managedObjectKeysByPropertyKey{
        return nil;
    }

    //表示Member的url向MemberManaged的url字段值转换，由 Model——>Entity
    + (NSValueTransformer *)entityAttributeTransformerForKey:(NSString *)key{
        if ([key isEqualToString:@"url"]) {
            return [MTLValueTransformer reversibleTransformerWithBlock:^id(NSURL *url) {
                return url.absoluteString;
            }];
        }
        else{
            return nil;
        }
    }
    {% endhighlight %}





## 参考
[http://segmentfault.com/blog/lingchen/1190000002431365](http://segmentfault.com/blog/lingchen/1190000002431365)
[http://www.raywenderlich.com/56879/magicalrecord-tutorial-ios](http://www.raywenderlich.com/56879/magicalrecord-tutorial-ios)
