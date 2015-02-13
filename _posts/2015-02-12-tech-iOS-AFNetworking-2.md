---
layout: post
title: AFNetworking2
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



## 参考

[https://github.com/AFNetworking/AFNetworking](https://github.com/AFNetworking/AFNetworking)

[http://www.raywenderlich.com/59255/afnetworking-2-0-tutorial](http://www.raywenderlich.com/59255/afnetworking-2-0-tutorial)

[http://nshipster.cn/afnetworking-2/](http://nshipster.cn/afnetworking-2/)
