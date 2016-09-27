---
title: perform执行时机
date: 2016-09-17 22:51:42
tags: iOS
---

在iOS开发中，大家都经常用到performSelector函数，但是他们的执行时机是怎么样的，这里做个简单的介绍。

首先如果我们在主线程调用和在子线程调用下面的方法有什么区别呢？

    [self performSelector:@selector(testttt) withObject:nil];

试过后，就知道在主线程会执行testttt函数，而在子线程是不会执行的呢，为什么呢？这里就要涉及到runloop了，因为主线程的runloop默认是运行的，但是子线程的runloop默认是没有的，需要自己去创建并执行，通常调用如下

	NSRunLoop *runLoop = [NSRunLoop currentRunLoop];
	子线程在没有第一次调用runloop是会先创建，然后在run。
> 更多可以参考[这里](http://ios.jobbole.com/85759/)	
<!--more-->

那么对于，`performSelector:withObject:` 和 `performSelector:withObject:afterDelay:0` 有什么区别呢？

通过如下代码可以测试：

```
    NSLog(@"beigi");
    
    [self performSelector:@selector(testttt) withObject:nil];
    [self performSelector:@selector(testPer) withObject:nil withObject:nil];
    
    [self performSelector:@selector(testttt) withObject:nil afterDelay:0];
    [self performSelector:@selector(testPer) withObject:nil afterDelay:0];
    
    NSLog(@"end");

	打印顺序为：
	2016-09-17 23:01:08.430 [76256:1232843] beigi
	2016-09-17 23:01:08.431 [76256:1232843] test tttt
	2016-09-17 23:01:08.431 [76256:1232843] test
	2016-09-17 23:01:08.431 [76256:1232843] end

	2016-09-17 23:01:08.470 [76256:1232843] test tttt
	2016-09-17 23:01:08.470 [76256:1232843] test
```
可以发现，`performSelector:withObject:`是直接调用了方法，而`performSelector:withObject:afterDelay:0` 即使是延迟0s也是在end之后执行的，这是为什么呢？

原因就是：前者是在当前runloop中调用的，后者是在先一个runloop方法中调用的。所以如果在代码中，想要通过时间：afterDelay来控制某些动画之类的，就需要注意了。

>
http://ios.jobbole.com/85759/

http://blog.ibireme.com/2015/05/18/runloop/