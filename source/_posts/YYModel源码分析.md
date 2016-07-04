---
title: YYModel源码分析
date: 2016-07-02 15:22:47
tags: iOS
---

根据JSON生成Model是一个iOS开发经常用到的内容，这方面开源的也很多，比如我们公司用的Mantel，但其实效率很一般，最近看了[YYModel](https://github.com/ibireme/YYModel)，区区五个文件，功能简单且高效，于是就认真学习一下人家是怎么实现的。

在我想来，主要就是通过runtime方法获取property和ivar就可以了，通过判断每个ivar的类型，然后根据不同的类型给每个property赋值，应该算是不太难的。在网上看到了[这篇文章](http://blog.csdn.net/woaifen3344/article/details/39301203)，就是我想的这个样子，但是YYModel的作者可不是这么认为的。

作者首先通过runtime方法，生成YYClassInfo，其中YYClassInfo主要是取了YYClassIvarInfo，YYClassMethodInfo，YYClassPropertyInfo。然后通过YYClassInfo构造YYModelMeta。

YYModelMeta里面包括YYModelPropertyMeta
这个过程中，我还有些疑问的，为啥一定要构造这么多中间类呢？咱们继续往下看。

实际的操作过程中，YYClassIvarInfo和YYClassMethodInfo都没有用到，主要就是用到了YYClassPropertyInfo，由此构造YYModelPropertyMeta，再生成YYModelMeta。

作者都采用了CoreFoundition类型，比如CFStringRef，CFArrayRef等的操作，相对于 Foundation 的方法来说，CoreFoundation 的方法有更高的性能，用 CFArrayApplyFunction() 和 CFDictionaryApplyFunction() 方法来遍历容器类能带来不少性能提升，以及赋值也是采用objc_sendMsg的方法，就是为了快，省去很多OC方法的查找，这就不难理解，作者为什么用那么多的中间类来记录Method，Ivar，property的setter和getter了。


作者在[文章中](http://blog.ibireme.com/2015/10/23/ios_model_framework_benchmark/)对各个开源库进行了对比，图文声茂，功底了得，膜拜一把。

<!--more-->

其他知识点：
---
### 1. nonnull
- NS_ASSUME_NONNULL_BEGIN
- NS_ASSUME_NONNULL_END

在这两个宏之间的代码，所有简单指针对象都被假定为nonnull，因此我们只需要去指定那些nullable的指针
### 2. 零依赖
自己的类甚至都不用添加<YYModel>的协议，怎么做到呢，其实就是先用respondersToSelector判断是否有<YYModel>协议里面的函数，如果有就通过(id<YYModel>)cls 来调用，如果不这样写，而是直接cls来调用，编译器会有warning的提示，同时也可以采用performSelector来实现。

```
// Get black list
    NSSet *blacklist = nil;
    if ([cls respondsToSelector:@selector(modelPropertyBlacklist)]) {
        NSArray *properties = [(id<YYModel>)cls modelPropertyBlacklist];
        if (properties) {
            blacklist = [NSSet setWithArray:properties];
        }
    }
```
### 3. copy记得释放
    Method *methods = class_copyMethodList(cls, &methodCount);
    free(methods);
    objc_property_t *properties = class_copyPropertyList(cls, &propertyCount);
    free(properties);
    
### 4. 信号量-读写锁    
    static dispatch_semaphore_t lock;
    lock = dispatch_semaphore_create(1);

    if (!info) {
        info = [[YYClassInfo alloc] initWithClass:cls];
        if (info) {
            dispatch_semaphore_wait(lock, DISPATCH_TIME_FOREVER);
            CFDictionarySetValue(info.isMeta ? metaCache : classCache, (__bridge const void *)(cls), (__bridge const void *)(info));
            dispatch_semaphore_signal(lock);
        }
    }

>
附: YYModel 性能优化的几个 Tip：
1. 缓存
Model JSON 转换过程中需要很多类的元数据，如果数据足够小，则全部缓存到内存中。

2. 查表
当遇到多项选择的条件时，要尽量使用查表法实现，比如 switch/case，C Array，如果查表条件是对象，则可以用 NSDictionary 来实现。

3. 避免 KVC
Key-Value Coding 使用起来非常方便，但性能上要差于直接调用 Getter/Setter，所以如果能避免 KVC 而用 Getter/Setter 代替，性能会有较大提升。

4. 避免 Getter/Setter 调用
如果能直接访问 ivar，则尽量使用 ivar 而不要使用 Getter/Setter 这样也能节省一部分开销。

5. 避免多余的内存管理方法
在 ARC 条件下，默认声明的对象是 __strong 类型的，赋值时有可能会产生 retain/release 调用，如果一个变量在其生命周期内不会被释放，则使用 __unsafe_unretained 会节省很大的开销。

访问具有 __weak 属性的变量时，实际上会调用 objc_loadWeak() 和 objc_storeWeak() 来完成，这也会带来很大的开销，所以要避免使用 __weak 属性。

创建和使用对象时，要尽量避免对象进入 autoreleasepool，以避免额外的资源开销。

6. 遍历容器类时，选择更高效的方法
相对于 Foundation 的方法来说，CoreFoundation 的方法有更高的性能，用 CFArrayApplyFunction() 和 CFDictionaryApplyFunction() 方法来遍历容器类能带来不少性能提升，但代码写起来会非常麻烦。

7. 尽量用纯 C 函数、内联函数
使用纯 C 函数可以避免 ObjC 的消息发送带来的开销。如果 C 函数比较小，使用 inline 可以避免一部分压栈弹栈等函数调用的开销。

8. 减少遍历的循环次数
在 JSON 和 Model 转换前，Model 的属性个数和 JSON 的属性个数都是已知的，这时选择数量较少的那一方进行遍历，会节省很多时间。


> [runtime之玩转成员变量](http://www.cnblogs.com/develop-SZT/p/5348364.html)
> 
> [runtime Method](http://blog.csdn.net/woaifen3344/article/details/50505808)
> 
> [iOS JSON 模型转换库评测](http://blog.ibireme.com/2015/10/23/ios_model_framework_benchmark/)