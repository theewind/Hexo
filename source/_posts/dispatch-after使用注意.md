---
title: dispatch_after使用注意
date: 2016-08-03 09:42:50
tags: iOS
---

注意
---
开发中，经常需要dispatch_after进行延迟处理，dispatch_after能让我们添加进队列的任务延时执行，该函数并不是在指定时间后执行处理，而只是在指定时间追加处理到dispatch_queue该方法的第一个参数是time，第二个参数是dispatch_queue，第三个参数是要执行的block。dispatch_time_t有两种形式的构造方式，第一种相对时间：DISPATCH_TIME_NOW表示现在，NSEC_PER_SEC表示的是秒数，它还提供了NSEC_PER_MSEC表示毫秒。
第二种是绝对时间，通过dispatch_walltime函数来获取，dispatch_walltime需要使用一个timespec的结构体来得到dispatch_time_t。

**注意：不是一定在给定时间后执行相关的任务，而是在一定时间后，将执行的操作加入到队列中，然后队列里面再分配执行的时间，所以即使给了个 `0秒` 也不会立即执行，而是会追加一个Runloop的时间，一般是1/60s**

测试代码如下
---
```
    dispatch_after(dispatch_time(DISPATCH_TIME_NOW, (int64_t)(0 * NSEC_PER_SEC)), dispatch_get_main_queue(), ^{
        NSLog(@"in after");
    });
    
    NSLog(@"viewDidLoad");
    
打印的顺序是这样：
2016-08-03 09:41:06.237 testDispatchAfter[29364:9056216] viewDidLoad
2016-08-03 09:41:06.244 testDispatchAfter[29364:9056216] in after

```
那有什么问题呢？
---
所以如果我们在使用dispatch_after的时候，比如有时候需要判断条件，特别是有些针对版本的判断，比如<= iOS7上，需要进行一定的延迟处理：
比如

```
    NSString *string = @"测试";
    dispatch_after(dispatch_time(DISPATCH_TIME_NOW, (int64_t)(0 * NSEC_PER_SEC)), dispatch_get_main_queue(), ^{
        NSString *afterString = [@"name in after" stringByAppendingString:string];
        NSLog(@"%@", afterString);
    });
    
    NSLog(@"%@", string);
    string = nil;

```

虽然测试的过程中，没有crash。但是我们在代码逻辑里还是要避免这样的操作，**因为是有可能，在dispatch_after的block执行之前，释放了某些变量，导致block出错的。**
