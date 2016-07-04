---
title: FBRetainCycleDetector代码学习
date: 2016-06-30 17:39:28
tags:
- iOS
- 内存检测
---
[FBMemoryProfiler](https://github.com/facebook/FBMemoryProfiler)是Facebook开源的一款用于分析iOS内存使用和检测循环引用的工具库，可以让你在应用程序内运行循环应用检测。底层主要由[FBAllocationTracker](https://github.com/facebook/FBAllocationTracker)和[FBRetainCycleDetector](https://github.com/facebook/FBRetainCycleDetector#filters)两块来实现功能。

FBAllocationTracker
---
通过在main.m中添加

	[[FBAllocationTrackerManager sharedManager] startTrackingAllocations];
	-->FB::AllocationTracker::beginTracking
	-->    replaceSelectorWithSelector([NSObject class],
                                @selector(alloc),
                                @selector(fb_newAlloc),
                                FBClassMethod);

          replaceSelectorWithSelector([NSObject class],
                                sel_registerName("dealloc"),
                                @selector(fb_newDealloc),
                                FBInstanceMethod);

就开启了内存检测，主要是通过替换对象的**+alloc** 和** -delloc** 方法，替换后

	alloc-->fb_newAlloc是IMP-->
	fb_originalAlloc-->alloc的IMP
	所以[nsobject alloc]-->fb_newAlloc的IMP-->.....

<!--more-->

FBRetainCycleDetector
---
对于通过objc_setAssociatedObject添加的对象，FBRetainCycleDetector也是可以检测的，这里就需要使用`fishbook`来进行c函数的hook,[fishhook](https://github.com/facebook/fishhook)是一个非常简单的库，它能动态替换运行在IOS模拟器或设备上Mach-o文件的符号表。是facebook的一个开源工具.![官网的一个截图为](https://camo.githubusercontent.com/18243516844d12b1bd158ce3687635d6e48d2e2e/687474703a2f2f692e696d6775722e636f6d2f4856587148437a2e706e67)
官网的一张图很好的解释了fishhook的原理：
> dyld链接2种符号，lazy和non-lazy，fishhook可以重新链接/替换本地符号。如图所示，__DATA区有两个section和动态符号链接相关：__nl_symbol_ptr 、__la_symbol_ptr。__nl_symbol_ptr为一个指针数组，直接对应non-lazy绑定数据。__la_symbol_ptr也是一个指针数组，通过dyld_stub_binder辅助链接。的section头提供符号表的偏移量。图示中，1061是间接符号表的偏移量，*（偏移量+间接符号地址）=16343，即符号表偏移量。符号表中每一个结构都是一个nlist结构体，其中包含字符表偏移量。通过字符表偏移量最终确定函数指针。fishhook就是对间接符号表的偏移量动的手脚，提供一个假的nlist结构体，从而达到hook的目的。

具体的源码想要看懂，还是需要对mach_o文件结构有比较深入的理解的。使用过程还是如下

```
int main(int argc, char *argv[]) {
    @autoreleasepool {
        [FBAssociationManager hook];
        [[FBAllocationTrackerManager sharedManager] startTrackingAllocations];
        [[FBAllocationTrackerManager sharedManager] enableGenerations];
        return UIApplicationMain(argc, argv, NSStringFromClass([NVApplication class]), NSStringFromClass([NVAppDelegate class]));
    }
}

```
`enableGenerations`开始追踪类的实例对象，就像instruments中的make generations一样，每个新创建的实例都会记录在对应的generations中

跟踪官方的demo（testObjectsRetainedBySomeObjectWillBeFetched），可以看到，通过object创建FBObjectiveCObject，然后调用allRetainedObjects-->_unfilteredRetainedObjects-->FBGetObjectStrongReferences-->FBGetStrongReferencesForClass

同时也包括[super allRetainedObjects]获取associations的强引用对象，最后通过如下代码生成FBObjectiveCGraphElement，就可以进行循环引用的搜索了。

```
- (NSSet *)allRetainedObjects
{
  NSArray *unfiltered = [self _unfilteredRetainedObjects];
  return [self filterObjects:unfiltered];
}

- (NSArray *)_unfilteredRetainedObjects
{
  Class aCls = object_getClass(self.object);
  if (!self.object || !aCls) {
    return nil;
  }

  NSArray *strongIvars = FBGetObjectStrongReferences(self.object, self.configuration.layoutCache);

  NSMutableArray *retainedObjects = [[[super allRetainedObjects] allObjects] mutableCopy];

  for (id<FBObjectReference> ref in strongIvars) {
    id referencedObject = [ref objectReferenceFromObject:self.object];

    if (referencedObject) {
      NSArray<NSString *> *namePath = [ref namePath];
      [retainedObjects addObject:FBWrapObjectGraphElementWithContext(referencedObject,
                                                                     self.configuration,
                                                                     namePath)];
    }
  }

```

其他代码学习
===
判断一个对象实例的iVar的类型

```
- (FBType)_convertEncodingToType:(const char *)typeEncoding
{
  if (typeEncoding[0] == '{') {
    return FBStructType;
  }

  if (typeEncoding[0] == '@') {
    // It's an object or block

    // Let's try to determine if it's a block. Blocks tend to have
    // @? typeEncoding. Docs state that it's undefined type, so
    // we should still verify that ivar with that type is a block
    if (strncmp(typeEncoding, "@?", 2) == 0) {
      return FBBlockType;
    }

    return FBObjectType;
  }

  return FBUnknownType;
}

以及对ivar的操作
    _name = @(ivar_getName(ivar));
    _type = [self _convertEncodingToType:ivar_getTypeEncoding(ivar)];
    _offset = ivar_getOffset(ivar);

```
`日常我们写一个property时，系统会默认给我们生成一个对应的_property的ivar，如果property前面添加了_,或者__呢，经试验系统还是会默认在这些前面继续加_的`

那如何区分那些是weak，那些是strong呢，这里就需要对Class的 Ivar Layout有一定的了解

```
struct class_ro_t {
    uint32_t flags;
    uint32_t instanceStart;
    uint32_t instanceSize;
#ifdef __LP64__
    uint32_t reserved;
#endif
    const uint8_t * ivarLayout; // <- 记录了哪些是 strong 的 ivar

    const char * name;
    const method_list_t * baseMethods;
    const protocol_list_t * baseProtocols;
    const ivar_list_t * ivars;

    const uint8_t * weakIvarLayout; // <- 记录了哪些是 weak 的 ivar
    const property_list_t *baseProperties;
};
```
所以FBRetainCycleDetector中是通过取 

	 const uint8_t *fullLayout = class_getIvarLayout(aCls);
来获取强引用类型的。

>[FBRetainCycleDetector分析](http://www.alonemonkey.com/2016/05/15/fbretaincycledetector-analyse/)

>[Facebook如何降低应用中的FOOMs（运行在前台内存不足）](http://www.cocoachina.com/cms/wap.php?action=article&id=14098)

>[Objective-C Class Ivar Layout 探索](http://blog.sunnyxx.com/2015/09/13/class-ivar-layout/)