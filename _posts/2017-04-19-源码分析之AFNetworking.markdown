---
layout: post
title:  "源码拆解之AFNetworking"
subtitle: ""
date:   2017-04-19 18:54:01
categories: [tech]
---

> AFNetworking的源码拆解

所有使用Object-C开发的iOS程序员接触到的网络库一定会有AFNetworking的身影，目前这个开源库大概有29k个star，具体的github地址为：https://github.com/AFNetworking/AFNetworking。

下面具体拆解一下：

- **AFURLSessionManager:** 该类作为整个网络库中最重要的类，也是AFHTTPSessionManager的基类，其主要做了下面这些事情。

	* 承载了整个Session的创建
	
	*  task的生成，包括普通的task，uploadTask，downloadTask，以及一组相关的方法
	
	*  提供默认的responseSerializer, 以及securityPolicy
	
	*  delegateQueue的创建
	
	*  NSURLSessionDelegate, NSURLSessionTaskDelegate，NSURLSessionDataDelegate, NSURLSessionDownloadDelegate等代理方法的实现。
	 
	*  以block的方式提供给上层调用者，让其可以控制一些网络请求的阶段以及数据处理。
	
	* 提供了两个内部类，**AFURLSessionManagerTaskDelegate**和**_AFURLSessionTaskSwizzling**，其中AFURLSessionManagerTaskDelegate实现了NSURLSessionTaskDelegate, NSURLSessionDataDelegate, NSURLSessionDownloadDelegate三个代理方法，主要用来控制网络请求中回调的线程控制，比如completionBlock在主线程，以及一些progress的控制。_AFURLSessionTaskSwizzling这个类主要是针对iOS7和iOS8上面NSURLSessionTask实现方式不同，而通过runtime做了一些方法替换，使得在iOS7和iOS8上面调用的API能够得到统一。

- **AFHTTPSessionManager：**其作为AFURLSessionManager的子类，主要针对HTTP的一系列请求做了一些封装，包括GET，POST，PUT，PATCH，DELETE，HEAD，同时也提供了一系列的方法可供上层调用

	*  提供一系列初始化方法，包括基于BaseURL， Configuration，等
	
	*  提供了默认的requestSerializer和responseSerializer，同时提供其set方法
	
	*  提供了securityPolicy的set方法
	
	*  封装成完整的request以及dataTask，同时触发网络请求resume，并将dataTask做为返回值抛出，使得上层执行dataTask的操作

- **AFHTTPRequestSerializer系列：**该组类实现了AFURLRequestSerialization协议，具体拆解如下：

	* 提供了一些属性用于配置HTTP请求的一些header，以及其构建的方式
	
	* 提供了根据这些参数配置request请求的方法
	
	* 封装了一系列的serializer类，用于支持不同的contentType
	
* **AFHTTPResponseSerializer系列：**该组类实现了AFURLResponseSerialization协议，具体拆解如下：

	* 提供了acceptableStatusCodes属性可以自定义可接受的状态码，默认为200-299
	
	* 内部封装了校验response的方法，用于校验response的正确性
	
	* 封装了一系列的serializer类，用于支持不同的contentType

- **AFSecurityPolicy：**该类主要用于支持HTTPS的SecurityPolicy的配置，具体拆解如下：

	* 提供了SSL的校验方式
	
	* 提供了配置可信任证书的属性
	
	* 提供了校验证书以及域名的属性
	
	* 提供了根据severTrust和domain来检查服务器端发来的证书是否可信的方法

- **AFNetworkReachabilityManager：**用于检测当前网络状态

- **UIKit+AFNetworking：**这组系列不做拆分，主要就是提供了一些列的category，用于将AF的一些状态通过关联对象的方式绑定至UI上。

AF的实现主线非常清晰， 我并没有特别细的去拆分每一个属性具体做什么事情，根据每个模块具体做了什么可以很清晰的看到作者的主线思想，也可以很便捷的去剖析每个类，非常优秀的网络库，特别好的封装。


**任何技术有其优点必有其局限，保持观望，保持学习**

