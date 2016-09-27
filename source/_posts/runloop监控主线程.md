---
title: runloop监控主线程
date: 2016-09-12 15:37:09
tags:
- runLoop
- iOS
---

前几天读了关于微信监控内存卡顿的文章，其中介绍可以同过监控主线程runloop的时间来记录卡顿，当cpu为100%且runloop运行2秒以上时，就会把当前堆栈的所有信息给dump下来，然后上传发服务器，后续再继续进行分析。

<!--more-->

然后自己对这里面的一些知识点比较感兴趣，这里做个简单点记录。

###记录内存使用

```
+ (float)usedSizeOfMemory {
    task_vm_info_data_t taskInfo;
    mach_msg_type_number_t infoCount = TASK_VM_INFO_COUNT;
    kern_return_t kernReturn = task_info(mach_task_self(), TASK_VM_INFO_PURGEABLE, (task_info_t)&taskInfo, &infoCount);

    if (kernReturn != KERN_SUCCESS) {
        return 0.0f;
    }
    return ((taskInfo.internal + taskInfo.compressed - taskInfo.purgeable_volatile_pmap) / (1024.0 * 1024.0));
}
```
###记录cpu使用

```
-(float) cpu_usage
{
    kern_return_t kr;
    task_info_data_t tinfo;
    mach_msg_type_number_t task_info_count;

    task_info_count = TASK_INFO_MAX;
    kr = task_info(mach_task_self(), TASK_BASIC_INFO, (task_info_t)tinfo, &task_info_count);
    if (kr != KERN_SUCCESS) {
        return -1;
    }

    task_basic_info_t      basic_info;
    thread_array_t         thread_list;
    mach_msg_type_number_t thread_count;

    thread_info_data_t     thinfo;
    mach_msg_type_number_t thread_info_count;

    thread_basic_info_t basic_info_th;
    uint32_t stat_thread = 0; // Mach threads

    basic_info = (task_basic_info_t)tinfo;

    // get threads in the task
    kr = task_threads(mach_task_self(), &thread_list, &thread_count);
    if (kr != KERN_SUCCESS) {
        return -1;
    }
    if (thread_count > 0)
        stat_thread += thread_count;

    long tot_sec = 0;
    long tot_usec = 0;
    float tot_cpu = 0;
    int j;

    for (j = 0; j < thread_count; j++)
    {
        thread_info_count = THREAD_INFO_MAX;
        kr = thread_info(thread_list[j], THREAD_BASIC_INFO,
                         (thread_info_t)thinfo, &thread_info_count);
        if (kr != KERN_SUCCESS) {
            return -1;
        }

        basic_info_th = (thread_basic_info_t)thinfo;

        if (!(basic_info_th->flags & TH_FLAGS_IDLE)) {
            tot_sec = tot_sec + basic_info_th->user_time.seconds + basic_info_th->system_time.seconds;
            tot_usec = tot_usec + basic_info_th->system_time.microseconds + basic_info_th->system_time.microseconds;
            tot_cpu = tot_cpu + basic_info_th->cpu_usage / (float)TH_USAGE_SCALE * 100.0;
        }

    } // for each thread

    kr = vm_deallocate(mach_task_self(), (vm_offset_t)thread_list, thread_count * sizeof(thread_t));
    assert(kr == KERN_SUCCESS);

    return tot_cpu;
}
```

###使用runloop监控主线程

```
1. 合适的地方添加一个子线程
    [NSThread detachNewThreadSelector: @selector(newThreadProcess)
                             toTarget: self
                           withObject: nil];
2. 添加监控方法
- (void)newThreadProcess
{
    @autoreleasepool {
        //获得当前thread的Runloop
        NSRunLoop *mainLoop = [NSRunLoop mainRunLoop];
        NSRunLoop* myRunLoop = [NSRunLoop currentRunLoop];
        
        //设置Run loop observer的运行环境
        CFRunLoopObserverContext context = {0,(__bridge void *)(self),NULL,NULL,NULL};
        
        //创建Run loop observer对象
        //第一个参数用于分配observer对象的内存
        //第二个参数用以设置observer所要关注的事件，详见回调函数myRunLoopObserver中注释
        //第三个参数用于标识该observer是在第一次进入runloop时执行还是每次进入run loop处理时均执行
        //第四个参数用于设置该observer的优先级
        //第五个参数用于设置该observer的回调函数
        //第六个参数用于设置该observer的运行环境
        CFRunLoopObserverRef observer =CFRunLoopObserverCreate(kCFAllocatorDefault,kCFRunLoopAllActivities, YES, 0, &myRunLoopObserver, &context);
        if(observer)
        {
            //将Cocoa的NSRunLoop类型转换成CoreFoundation的CFRunLoopRef类型
            CFRunLoopRef cfRunLoop = [mainLoop getCFRunLoop];
            
            //将新建的observer加入到当前thread的runloop
            CFRunLoopAddObserver(cfRunLoop, observer, kCFRunLoopDefaultMode);
        }

        [myRunLoop runUntilDate:[NSDate distantFuture]];

    }

}

3. 监控方法 
void myRunLoopObserver(CFRunLoopObserverRef observer,CFRunLoopActivity activity,void *info)
{
    switch (activity) {
            //The entrance of the run loop, before entering the event processing loop.
            //This activity occurs once for each callto CFRunLoopRun and CFRunLoopRunInMode
        case kCFRunLoopEntry:
            NSLog(@"run loop entry");
            break;
            //Inside the event processing loop before any timers are processed
        case kCFRunLoopBeforeTimers:
            NSLog(@"run loop before timers");
            break;
            //Inside the event processing loop before any sources are processed
        case kCFRunLoopBeforeSources:
            NSLog(@"run loop before sources");
            break;
            //Inside the event processing loop before the run loop sleeps, waiting for a source or timer to fire.
            //This activity does not occur ifCFRunLoopRunInMode is called with a timeout of 0 seconds.
            //It also does not occur in a particulariteration of the event processing loop if a version 0 source fires
        case kCFRunLoopBeforeWaiting:
            NSLog(@"run loop before waiting");
            break;
            //Inside the event processing loop after the run loop wakes up, but before processing the event that woke it up.
            //This activity occurs only if the run loopdid in fact go to sleep during the current loop
        case kCFRunLoopAfterWaiting:
            NSLog(@"run loop after waiting");
            break;
            //The exit of the run loop, after exiting the event processing loop.
            //This activity occurs once for each callto CFRunLoopRun and CFRunLoopRunInMode
        case kCFRunLoopExit:
            NSLog(@"run loop exit");
            break;
            /*
             A combination of all the precedingstages
             case kCFRunLoopAllActivities:
             break;
             */
        default:
            break;
    }
}

可以通过kCFRunLoopBeforeWaiting和kCFRunLoopAfterWaiting的时间差，可以得知一次runloo的时间，然后就处理时间规则。

```
###dump所有线程信息
此处可以参考[Demo](https://github.com/ming1016/DecoupleDemo)，简单直接使用PLCrashReporter进行收集，也可以参考他的[源码](https://opensource.plausible.coop/src/projects/PLCR/repos/plcrashreporter/browse/Source/PLCrashLogWriter.m?at=refs%2Ftags%2F1.0#694) 手动dump所有线程的信息。


> [深入理解RunLoop](http://blog.ibireme.com/2015/05/18/runloop/) 这篇讲解的最深刻，作者功力深厚，开源YYKit系列
> 
> [检测iOS的APP性能的一些方法](http://www.jianshu.com/p/802cb5210dc4)
> 
> [优化UITableViewCell高度计算的那些事](http://blog.sunnyxx.com/2015/05/17/cell-height-calculation/)  其实最新的demo中，已经去除了Runloop
> 
> [iOS学习笔记12—Runloop](http://blog.csdn.net/jjunjoe/article/details/8313016)
> 
> [微信iOS卡顿监控系统](http://mp.weixin.qq.com/s?__biz=MzAwNDY1ODY2OQ==&mid=207890859&idx=1&sn=e98dd604cdb854e7a5808d2072c29162&scene=21#wechat_redirect)


