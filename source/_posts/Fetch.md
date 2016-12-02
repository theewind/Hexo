title: iOS 开发记录
date: 2016-11-22 18:29:39
tags: iOS
---
background Fetch 注意
---
使用background Fetch 主要记得要回调completionHandler(UIBackgroundFetchResultNewData)等

调试有两种方式
1 程序进入后台，选择xcode - debug - simulate background fetch，
2 选择scheme的 launch due to a background fetch event 这是谁debug调试的时候，先执行fetch功能，然后自己通过主动启动应用，查看是否执行了fetch

离线缓存
---
参考个大厂商的缓存，一种是看过即缓存，比如网易新闻，还有一种直接是离线下载，比如腾讯新闻。
针对这两个实现，自己大概想了下，一个是基于webview的缓存实现，另外一种，应该是通过服务下载资源到本地，然后存储到数据库中，比如coredata等。

webview的缓存实现主要两种，一个html5的manifest文件，也可以通过NSURLCache实现。
具体可以参考：[iOS开发网络——数据缓存](http://www.cocoachina.com/ios/20161107/17981.html)

还有一个就是采用NSURLProtocol实现存储，可以参考[https://github.com/rnapier/RNCachingURLProtocol](https://github.com/rnapier/RNCachingURLProtocol) 就是将请求过的内容都缓存下来。

至于为什么说离线是通过服务下载的，因为他们可以把没有看过的内容都离线到本地，那就只能是是通过文件下载来实现。

> http://www.jcodecraeer.com/a/anzhuokaifa/androidkaifa/2015/0417/2736.html
>  https://github.com/rnapier/RNCachingURLProtocol
>  https://my.oschina.net/u/2340880/blog?sort=time&p=9&temp=1479982318849 晖少的博客