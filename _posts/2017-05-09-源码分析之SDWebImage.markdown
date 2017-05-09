---
layout: post
title:  "源码分析之SDWebImage"
subtitle: ""
date:   2017-05-09 12:50:01
categories: [tech]
---

> 源码分析之SDWebImage

**SDWebImage**，iOS开发者必然会接触到的一个关于图片的开源库，其github地址：https://github.com/rs/SDWebImage，目前有17k+个star，下面对这个库进行简单的拆解。后面简称为SD。

- SD提供了一些UI的分类，用于简化组件上图片的配置。无论UIImageView，UIButton等最后都会统一调用*UIView+WebCache.m*这个类中的这个方法。
 {% highlight javascript %}
//- (void)sd_internalSetImageWithURL:(nullable NSURL *)url
                  placeholderImage:(nullable UIImage *)placeholder
                           options:(SDWebImageOptions)options
                      operationKey:(nullable NSString *)operationKey
                     setImageBlock:(nullable SDSetImageBlock)setImageBlock
                          progress:(nullable SDWebImageDownloaderProgressBlock)progressBlock
                         completed:(nullable SDExternalCompletionBlock)completedBlock;
{% endhighlight %}

- 上述方法内部会调用*SDWebImageManager*的loadImage方法，如下
{% highlight javascript %}
//- (id <SDWebImageOperation>)loadImageWithURL:(nullable NSURL *)url
                                     options:(SDWebImageOptions)options
                                    progress:(nullable SDWebImageDownloaderProgressBlock)progressBlock
                                   completed:(nullable SDInternalCompletionBlock)completedBlock
{% endhighlight %}
该方法会返回一个遵循SDWebImageOperation的协议，之后会将该operation加入字典进行缓存。

- 继续解析上述loadImage的方法，该方法会首先判断是否这个url是否包含在失败的url中，如果是则直接进行失败回调，反之，从imageCache中获取图片信息，如果不存在，则通过imageDownloader尝试去下载图片，下载成功，则将其缓存，下载失败，则将url存入失败的url列表中。方法代码略长，就不粘贴了。

- 此处作者会将operation加到一个runningOperation的列表中，目的是为了方便对正在进行的operation操作，主要是cancel操作。

- 在对图片进行处理的时候都会异步到后台线程进行操作，避免阻塞主线程。

- **SDWebImageDownloader：**这个类主要执行图片的下载，建立一个SDWebImageDownloaderOperation，将其加入operationQueue中，同时也通过operation的依赖关系完成LIFO操作。

- **SDWebImageDownloaderOperation：**自定义的operation，用于完成网络请求及之后的图片处理，同时通过barrierQueue来确保多线程操作时候的执行顺序，通过GCD将通知都异步到主线程进行发送，确保监听者可以直接在主线程进行操作。

SDWebImage的拆解就这样，其具体的使用方式或者一些支持的额外的功能，比如Gif等均可参考其说明文档即可。


**任何技术有其优点必有其局限，保持观望，保持学习**

