---
layout: post
title: AFNetworking
category: 技术
tags: AFNetworking
keywords: AFNetworking
description:
---

在iOS 7之前，iOS提供的网络架构叫NSURLConnection； AFNetworking 包含的组件：
- AFURLConnectionOperation - NSOperation 的子类，负责管理 NSURLConnection 并且实现其 delegate 方法。
- AFHTTPRequestOperation - AFURLConnectionOperation 的子类，用于生成 HTTP 请求，可以区别可接受的和不可接受的状态码及内容类型。 2.0 版本中的最大区别是，你可以直接使用这个类，而不用继承它，原因可以在“序列化”一节中找到。
- AFHTTPRequestOperationManager - 包装常见 HTTP web 服务操作的类，通过 AFHTTPRequestOperation 由 NSURLConnection 支持。

在iOS 7后，Apple更新了iOS中的网络基础架构叫NSURLSession

-	AFURLSessionManager - 创建、管理基于 NSURLSessionConfiguration 对象的 NSURLSession 对象的类， 也可以管理 session 的数据、下载/上传任务，实现 session 和其相关联的任务的 delegate 方法。因为 NSURLSession API 设计中奇怪的空缺，任何和 NSURLSession 相关的代码都可以用 AFURLSessionManager 改善。
-	AFHTTPSessionManager - AFURLSessionManager 的子类，包装常见的 HTTP web 服务操作，通过 AFURLSessionManager 由 NSURLSession 支持。

> 总的来说：为了支持新的 NSURLSession API 以及旧的未弃用且还有用的 NSURLConnection，AFNetworking 2.0 的核心组件分成了**request operation** 和 **session 任务**。AFHTTPRequestOperationManager 和 AFHTTPSessionManager 提供类似的功能， 在需要的时候（比如在 iOS 6 和 7 之间转换），它们的接口可以相对容易的互换。
>
> 之前所有绑定在 AFHTTPClient的功能，比如序列化、安全性、可达性，被拆分成几个独立的模块，可被基于 NSURLSession 和 NSURLConnection 的 API 使用。

# 体系结构

## NSURLConnection

-	AFURLConnectionOperation
-	AFHTTPRequestOperation
-	AFHTTPRequestOperationManager

## NSURLSession (iOS 7 / Mac OS X 10.9)

-	AFURLSessionManager
-	AFHTTPSessionManager

## 序列化

AFNetworking 2.0 新构架的突破之一是使用序列化来创建请求、解析响应。可以通过序列化的灵活设计将更多业务逻辑转移到网络层，并更容易定制之前内置的默认行为。

### `<AFURLRequestSerialization>`
符合这个协议的对象用于处理请求，它将请求参数转换为 query string 或是 entity body 的形式，并设置必要的 header。

-	AFHTTPRequestSerializer
-	AFJSONRequestSerializer
-	AFPropertyListRequestSerializer

### `<AFURLResponseSerialization>`
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

## UIKit 扩展

-	AFNetworkActivityIndicatorManager：在请求操作开始、停止加载时，自动开始、停止状态栏上的网络活动指示图标。

-	UIImageView+AFNetworking：增加了 imageResponseSerializer 属性，可以轻松地让远程加载到 image view 上的图像自动调整大小或应用滤镜。比如，AFCoreImageSerializer 可以在 response 的图像显示之前应用 Core Image filter。

-	UIButton+AFNetworking (新)：与 UIImageView+AFNetworking 类似，从远程资源加载 image 和 backgroundImage。

-	UIActivityIndicatorView+AFNetworking (新)：根据指定的请求操作和会话任务的状态自动开始、停止 UIActivityIndicatorView。

-	UIProgressView+AFNetworking (新)：自动跟踪某个请求或会话任务的上传/下载进度。 UIWebView+AFNetworking (新): 为加载 URL 请求提供了更强大的API，支持进度回调和内容转换。

## 参考

[https://github.com/AFNetworking/AFNetworking](https://github.com/AFNetworking/AFNetworking)

[http://www.raywenderlich.com/59255/afnetworking-2-0-tutorial](http://www.raywenderlich.com/59255/afnetworking-2-0-tutorial)

[http://nshipster.cn/afnetworking-2/](http://nshipster.cn/afnetworking-2/)
