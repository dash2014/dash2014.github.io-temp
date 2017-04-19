---
layout: post
title:  "源码拆解之AFNetworking的线程管理"
subtitle: ""
date:   2017-04-19 20:54:01
categories: [tech]
---

> AFNetworking的线程管理

AF主要操作了这样几个线程来处理整个网络请求，具体拆解如下：

- **请求任务创建线程：**在生成Task的时候，统一调用url_session_manager_create_task_safely来生成task，这个方法主要是为了修复iOS8以下并发队列中生成task，会创建不同的taskIdentifier，起初的completionHandler会被新的所替换，第一次的请求数据返回会在第二个completionHandler中处理，因此在该方法中低于iOS8以下的会统一在url_session_manager_creation_queue这个单队列中创建。具体代码如下：

{% highlight javascript %}
static dispatch_queue_t url_session_manager_creation_queue() {
    static dispatch_queue_t af_url_session_manager_creation_queue;
    static dispatch_once_t onceToken;
    dispatch_once(&onceToken, ^{
        af_url_session_manager_creation_queue = dispatch_queue_create("com.alamofire.networking.session.manager.creation", DISPATCH_QUEUE_SERIAL);
    });

    return af_url_session_manager_creation_queue;
}

static void url_session_manager_create_task_safely(dispatch_block_t block) {
    if (NSFoundationVersionNumber < NSFoundationVersionNumber_With_Fixed_5871104061079552_bug) {
        // Fix of bug
        // Open Radar:http://openradar.appspot.com/radar?id=5871104061079552 (status: Fixed in iOS8)
        // Issue about:https://github.com/AFNetworking/AFNetworking/issues/2093
        dispatch_sync(url_session_manager_creation_queue(), block);
    } else {
        block();
    }
}
{% endhighlight %}

- **网络回调队列：**创建session的时候，NSURLSession提供了的方法sessionWithConfiguration:delegate:delegateQueue:中可以指明代理方法的回调队列，具体代码如下：

{% highlight javascript %}
    self.operationQueue = [[NSOperationQueue alloc] init];
    self.operationQueue.maxConcurrentOperationCount = 1;
    self.session = [NSURLSession sessionWithConfiguration:self.sessionConfiguration delegate:self delegateQueue:self.operationQueue];
{% endhighlight %}

- **数据处理线程：**该线程是在网络请求完成时的回调里，用于完成反序列化，配置userInfo等操作，具体的定义和调用如下：
	
	{% highlight javascript %}
	 static dispatch_queue_t url_session_manager_processing_queue() {
	    static dispatch_queue_t af_url_session_manager_processing_queue;
	    static dispatch_once_t onceToken;
	    dispatch_once(&onceToken, ^{
	        af_url_session_manager_processing_queue = dispatch_queue_create("com.alamofire.networking.session.manager.processing", DISPATCH_QUEUE_CONCURRENT);
	    });
	    return af_url_session_manager_processing_queue;
	}
	{% endhighlight %}
	
	{% highlight javascript %}
		- (void)URLSession:(__unused NSURLSession *)session
	              task:(NSURLSessionTask *)task
	didCompleteWithError:(NSError *)error
	{
	    //我是省略的
	    if (error) {
	    //我是省略的  
	    } else {
	        dispatch_async(url_session_manager_processing_queue(), ^{
			   //我来处理数据
	        });
	    }
	}
	{% endhighlight %}

- **完成回调线程：**此处作者采用了dispatch_group的方式，如果上层定义了completionGroup则采用，否则采用内置的url_session_manager_completion_group，异步到mainQueue完成回调。具体代码如下：

	{% highlight javascript %}
	 dispatch_group_async(manager.completionGroup ?: url_session_manager_completion_group(), manager.completionQueue ?: dispatch_get_main_queue(), ^{
	                if (self.completionHandler) {
	                    self.completionHandler(task.response, responseObject, serializationError);
	                }
	
	                dispatch_async(dispatch_get_main_queue(), ^{
	                    [[NSNotificationCenter defaultCenter] postNotificationName:AFNetworkingTaskDidCompleteNotification object:task userInfo:userInfo];
	                });
	            });
	{% endhighlight %}

AFNetworking处理线程的方式，保证了网络请求任务的并发，在建立task的时候用锁保证了数据的一致性，同时又由于Session的delegateQueue是单队列，保证了任务处理的FIFO，在请求完成处理数据的过程中异步到数据处理线程，处理完成后异步到主线程，保证了流畅度，但个人不是很理解completionGroup的作用，继续理解ing...

**任何技术有其优点必有其局限，保持观望，保持学习**

