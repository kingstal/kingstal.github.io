---
layout: post
title: AFNetworking
category: 技术
tags: AFNetworking
keywords: AFNetworking
description: AFNetworking使用说明
---

iOS开发中往往会涉及到网络数据处理，在iOS 7之前，iOS提供的网络架构叫 NSURLConnection；在iOS7中，Apple更新了iOS中的网络架构 NSURLSession。`AFNetworking`则是基于这两个框架而设计的第三方开源库。


## 体系结构

### NSURLConnection
AFNetworking 包含的组件：
- `AFURLConnectionOperation` - NSOperation 的子类，负责管理 NSURLConnection 并且实现其 delegate 方法。
- `AFHTTPRequestOperation` - AFURLConnectionOperation 的子类，用于生成 HTTP 请求，可以区别可接受的和不可接受的状态码及内容类型。 2.0 版本中的最大区别是，你可以直接使用这个类，而不用继承它，原因可以在“序列化”一节中找到。
- `AFHTTPRequestOperationManager` - 封装常见 HTTP web 服务操作的类，包括创建请求、响应序列化、网络可达性监测、安全性和请求操作管理。通过 AFHTTPRequestOperation 由 NSURLConnection 支持。

### NSURLSession (iOS 7 / Mac OS X 10.9)
AFNetworking 包含的组件：
-	`AFURLSessionManager` - 创建、管理基于 NSURLSessionConfiguration 对象的 NSURLSession 对象的类， 也可以管理 session 的数据、下载/上传任务，实现 session 和其相关联的任务的 delegate 方法。因为 NSURLSession API 设计中奇怪的空缺，任何和 NSURLSession 相关的代码都可以用 AFURLSessionManager 改善。
-	`AFHTTPSessionManager` - AFURLSessionManager 的子类，包装常见的 HTTP web 服务操作，通过 AFURLSessionManager 由 NSURLSession 支持。


## 序列化

AFNetworking 2.0 新构架的突破之一是使用序列化来创建请求、解析响应。可以通过序列化的灵活设计将更多业务逻辑转移到网络层，并更容易定制之前内置的默认行为。

### 请求序列化
> `<AFURLRequestSerialization>`

符合这个协议的对象用于处理请求，它将请求参数转换为 query string 或是 entity body 的形式，并设置必要的 header。

-	AFHTTPRequestSerializer
-	AFJSONRequestSerializer
-	AFPropertyListRequestSerializer

### 响应序列化
> `<AFURLResponseSerialization>`

符合这个协议的对象用于验证、序列化响应及相关数据，转换为有用的形式，比如 JSON 对象、图像、甚至基于 Mantle 的模型对象。

-	AFHTTPResponseSerializer
-	AFJSONResponseSerializer
-	AFXMLParserResponseSerializer
-	AFXMLDocumentResponseSerializer (Mac OS X)
-	AFPropertyListResponseSerializer
-	AFImageResponseSerializer
-	AFCompoundResponseSerializer

## Additional Functionality
-	AFSecurityPolicy
-	AFNetworkReachabilityManager

## 使用

### `AFHTTPRequestOperationManager`

> > `GET` Request

    {% highlight objective-c %}
    // GET
    AFHTTPRequestOperationManager* manager = [AFHTTPRequestOperationManager manager];
    [manager GET:@"http://example.com/resources.json"
        parameters:nil
        success:^(AFHTTPRequestOperation* operation, id responseObject) {
             NSLog(@"JSON: %@", responseObject);
        }
        failure:^(AFHTTPRequestOperation* operation, NSError* error) {
             NSLog(@"Error: %@", error);
        }];
    {% endhighlight %}


> > `POST` URL-Form-Encoded Request

    {% highlight objective-c %}
    // POST
    AFHTTPRequestOperationManager* manager = [AFHTTPRequestOperationManager manager];
    NSDictionary* parameters = @{ @"foo" : @"bar" };
    [manager POST:@"http://example.com/resources.json"
        parameters:parameters
        success:^(AFHTTPRequestOperation* operation, id responseObject) {
              NSLog(@"JSON: %@", responseObject);
        }
        failure:^(AFHTTPRequestOperation* operation, NSError* error) {
              NSLog(@"Error: %@", error);
        }];
    {% endhighlight %}

> > `POST` Multi-Part Request

    {% highlight objective-c %}
    // Multi-Part Request
    AFHTTPRequestOperationManager* manager =
        [AFHTTPRequestOperationManager manager];
    NSDictionary* parameters = @{ @"foo" : @"bar" };
    NSURL* filePath = [NSURL fileURLWithPath:@"file://path/to/image.png"];
    [manager POST:@"http://example.com/resources.json"
       parameters:parameters
       constructingBodyWithBlock:^(id<AFMultipartFormData> formData) {
            [formData appendPartWithFileURL:filePath name:@"image" error:nil];
        }
          success:^(AFHTTPRequestOperation* operation, id responseObject) {
              NSLog(@"Success: %@", responseObject);
        }
          failure:^(AFHTTPRequestOperation* operation, NSError* error) {
              NSLog(@"Error: %@", error);
        }];
    {% endhighlight %}


### `AFURLSessionManager`
> > Creating a Download Task

    {% highlight objective-c %}
    // Creating a Download Task
    NSURLSessionConfiguration* configuration =
        [NSURLSessionConfiguration defaultSessionConfiguration];
    AFURLSessionManager* manager =
        [[AFURLSessionManager alloc] initWithSessionConfiguration:configuration];

    NSURL* URL = [NSURL URLWithString:@"http://example.com/download.zip"];
    NSURLRequest* request = [NSURLRequest requestWithURL:URL];

    NSURLSessionDownloadTask* downloadTask
        = [manager downloadTaskWithRequest:request
                                  progress:nil
                               destination:^NSURL*(NSURL* targetPath, NSURLResponse* response) {
                                   NSURL* documentsDirectoryURL = [[NSFileManager defaultManager] URLForDirectory:NSDocumentDirectory
                                                                               inDomain:NSUserDomainMask
                                                                      appropriateForURL:nil
                                                                                 create:NO
                                                                                  error:nil];
                                   return [documentsDirectoryURL URLByAppendingPathComponent:[response suggestedFilename]];
                               }
                         completionHandler:^(NSURLResponse* response, NSURL* filePath, NSError* error) {
                             NSLog(@"File downloaded to: %@", filePath);
                         }];
    [downloadTask resume];
    {% endhighlight %}

> > Creating an Upload Task

    {% highlight objective-c %}
    // Creating an Upload Task
    NSURLSessionConfiguration* configuration = [NSURLSessionConfiguration defaultSessionConfiguration];
    AFURLSessionManager* manager = [[AFURLSessionManager alloc] initWithSessionConfiguration:configuration];

    NSURL* URL = [NSURL URLWithString:@"http://example.com/upload"];
    NSURLRequest* request = [NSURLRequest requestWithURL:URL];
    NSURL* filePath = [NSURL fileURLWithPath:@"file://path/to/image.png"];

    NSURLSessionUploadTask* uploadTask
        = [manager uploadTaskWithRequest:request
                                fromFile:filePath
                                progress:nil
                       completionHandler:^(NSURLResponse* response, id responseObject, NSError* error) {
                           if (error) {
                               NSLog(@"Error: %@", error);
                           } else {
                               NSLog(@"Success: %@ %@", response, responseObject);
                           }
                       }];
    [uploadTask resume];
    {% endhighlight %}

> > Creating an Upload Task for a Multi-Part Request, with Progress

    {% highlight objective-c %}
    // Creating an Upload Task for a Multi-Part Request, with Progress
    NSMutableURLRequest* request
        = [[AFHTTPRequestSerializer serializer] multipartFormRequestWithMethod:@"POST"
                                                                     URLString:@"http://example.com/upload"
                                                                    parameters:nil
                                                     constructingBodyWithBlock:^(id<AFMultipartFormData> formData) {
                                                         [formData appendPartWithFileURL:[NSURL fileURLWithPath:@"file://path/to/image.jpg"]
                                                                                    name:@"file"
                                                                                fileName:@"filename.jpg"
                                                                                mimeType:@"image/jpeg"
                                                                                   error:nil];
                                                     }
                                                                         error:nil];

    AFURLSessionManager* manager
        = [[AFURLSessionManager alloc] initWithSessionConfiguration:[NSURLSessionConfiguration defaultSessionConfiguration]];
    NSProgress* progress = nil;

    NSURLSessionUploadTask* uploadTask
        = [manager uploadTaskWithStreamedRequest:request
                                        progress:&progress
                               completionHandler:^(NSURLResponse* response, id responseObject, NSError* error) {
                                   if (error) {
                                       NSLog(@"Error: %@", error);
                                   } else {
                                       NSLog(@"%@ %@", response, responseObject);
                                   }
                               }];

    [uploadTask resume];
    {% endhighlight %}

> > Creating a Data Task

    {% highlight objective-c %}
    // Creating a Data Task
    NSURLSessionConfiguration* configuration = [NSURLSessionConfiguration defaultSessionConfiguration];
    AFURLSessionManager* manager = [[AFURLSessionManager alloc] initWithSessionConfiguration:configuration];

    NSURL* URL = [NSURL URLWithString:@"http://example.com/upload"];
    NSURLRequest* request = [NSURLRequest requestWithURL:URL];

    NSURLSessionDataTask* dataTask
        = [manager dataTaskWithRequest:request
                     completionHandler:^(NSURLResponse* response, id responseObject, NSError* error){
                         if (error) {
                             NSLog(@"Error: %@", error);
                         } else {
                             NSLog(@"%@ %@", response, responseObject);
                         }
                     }];
    [dataTask resume];
    {% endhighlight %}


### AFHTTPRequestOperation
尽管生成请求最好使用`AFHTTPRequestOperationManager`，但`AFHTTPRequestOperation`也可以独立使用。

> > 使用`AFHTTPRequestOperation`进行 GET 请求

    {% highlight objective-c %}
    // 1. urlsting->url->urlRequest
    NSURL* URL = [NSURL URLWithString:@"http://example.com/resources/123.json"];
    NSURLRequest* request = [NSURLRequest requestWithURL:URL];
    // 2 新建 AFHTTPRequestOperation 并设置 responseSerializer：AFJSONResponseSerializer、AFPropertyListResponseSerializer等
    AFHTTPRequestOperation* op = [[AFHTTPRequestOperation alloc] initWithRequest:request];
    op.responseSerializer = [AFJSONResponseSerializer serializer];
    // 3 设置 completion block
    [op setCompletionBlockWithSuccess:^(AFHTTPRequestOperation* operation, id responseObject) {
        // serializer 会解析接收到的数据，返回一个responseObject
        NSLog(@"JSON: %@", responseObject);
    } failure:^(AFHTTPRequestOperation* operation, NSError* error) {
        NSLog(@"Error: %@", error);
    }];
    // 4 开始
    [operation start];//或    [[NSOperationQueue mainQueue] addOperation:op];
    {% endhighlight %}

> > 批处理

    {% highlight objective-c %}
    // 批处理
    NSMutableArray* mutableOperations = [NSMutableArray array];
    for (NSURL* fileURL in filesToUpload) {
        NSURLRequest* request = [[AFHTTPRequestSerializer serializer] multipartFormRequestWithMethod:@"POST" URLString:@"http://example.com/upload" parameters:nil constructingBodyWithBlock:^(id<AFMultipartFormData> formData) {
            [formData appendPartWithFileURL:fileURL name:@"images[]" error:nil];
        }];

        AFHTTPRequestOperation* operation = [[AFHTTPRequestOperation alloc] initWithRequest:request];

        [mutableOperations addObject:operation];
    }

    NSArray* operations = [AFURLConnectionOperation batchOfRequestOperations:@[... ] progressBlock:^(NSUInteger numberOfFinishedOperations, NSUInteger totalNumberOfOperations) {
        NSLog(@"%lu of %lu complete", numberOfFinishedOperations, totalNumberOfOperations);
    } completionBlock:^(NSArray* operations) {
        NSLog(@"All operations in batch complete");
    }];
    [[NSOperationQueue mainQueue] addOperations:operations waitUntilFinished:NO];
    {% endhighlight %}


### 最佳实践
`AFHTTPRequestOperation`用于创建一次性的网络操作，而`AFHTTPRequestOperationManager`和`AFHTTPSessionManager`可方便的用于和单个 Web 服务终端交互。在项目中使用使用`AFHTTPSessionManager`可以使得网络通信代码复用：

> 1. 为每一个 Web 服务创建一个子类。如在写一个社交网络整合的应用，可以为 Twitter、Facebook 等各建一个子类。
> 2. 在每一个子类中创建一个类方法返回共享单例，可以节省资源。


    {% highlight %}
    //单例
    + (WeatherHTTPClient*)sharedWeatherHTTPClient
    {
        static WeatherHTTPClient* _sharedWeatherHTTPClient = nil;

        static dispatch_once_t onceToken;
        dispatch_once(&onceToken, ^{
            _sharedWeatherHTTPClient = [[self alloc] initWithBaseURL:[NSURL URLWithString:WorldWeatherOnlineURLString]];
            });

            return _sharedWeatherHTTPClient;
        }
    {% endhighlight %}

## UIKit 扩展

-	AFNetworkActivityIndicatorManager：在请求操作开始、停止加载时，自动开始、停止状态栏上的网络活动指示图标。


    {% highlight objective-c %}
    // AFNetworkActivityIndicatorManager
    [AFNetworkActivityIndicatorManager sharedManager].enabled = YES;
    {% endhighlight %}

-	UIImageView+AFNetworking（`setImageWithURLRequest:`）：增加了 imageResponseSerializer 属性，可以轻松地让远程加载到 image view 上的图像自动调整大小或应用滤镜。比如，AFCoreImageSerializer 可以在 response 的图像显示之前应用 Core Image filter。


    {% highlight objective-c %}
    // UIImageView+AFNetworking
    NSURL* url = [NSURL URLWithString:daysWeather.weatherIconURL];
    NSURLRequest* request = [NSURLRequest requestWithURL:url];
    UIImage* placeholderImage = [UIImage imageNamed:@"placeholder"];

    __weak UITableViewCell* weakCell = cell;

    [cell.imageView setImageWithURLRequest:request
                          placeholderImage:placeholderImage
                                   success:^(NSURLRequest* request, NSHTTPURLResponse* response, UIImage* image) {
                                       weakCell.imageView.image = image;
                                       [weakCell setNeedsLayout];
                                   }
                                   failure:nil];// 两个 block 都是可选的。若没有 success，imageView 自动设置远程图片，若有，则必须明确设置图片
    {% endhighlight %}

-	UIButton+AFNetworking：与 UIImageView+AFNetworking 类似，从远程资源加载 image 和 backgroundImage。

-	UIActivityIndicatorView+AFNetworking：根据指定的请求操作和会话任务的状态自动开始、停止 UIActivityIndicatorView。

-	UIProgressView+AFNetworking：自动跟踪某个请求或会话任务的上传/下载进度。 

-   UIWebView+AFNetworking (新): 为加载 URL 请求提供了更强大的API，支持进度回调和内容转换。

## 参考

[https://github.com/AFNetworking/AFNetworking](https://github.com/AFNetworking/AFNetworking)

[http://www.raywenderlich.com/59255/afnetworking-2-0-tutorial](http://www.raywenderlich.com/59255/afnetworking-2-0-tutorial)

[http://nshipster.cn/afnetworking-2/](http://nshipster.cn/afnetworking-2/)
