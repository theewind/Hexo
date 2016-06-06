---
title: rac_willDeallocSignal的加载时机
date: 2016-06-06 14:46:23
tags: RAC
---
今天看了下RAC的rac_willDeallocSignal的方法实现。我最初的猜测是NSObject (RACDeallocating)添加load方法，然后在里面进行method swzziling，添加dealloc方法，但是发现源码不是这样的。他只是通过常用的category 添加 associated 来进行的。先看他的简单实现：

<!--more-->

```
- (RACSignal *)rac_willDeallocSignal {
	RACSignal *signal = objc_getAssociatedObject(self, _cmd);
	if (signal != nil) return signal;

	RACReplaySubject *subject = [RACReplaySubject subject];

	[self.rac_deallocDisposable addDisposable:[RACDisposable disposableWithBlock:^{
		[subject sendCompleted];
	}]];

	objc_setAssociatedObject(self, _cmd, subject, OBJC_ASSOCIATION_RETAIN);

	return subject;
}

```
每次我们调用一个object的rac_willDeallocSignal，先判断这个是否已经添加过了rac_willDeallocSignal的_cmd对应的关联对象，如果添加了就直接返回，如果没有添加，则生成一个subject，然后调用self.rac_deallocDisposable，在这个里面进行方法替换，如下：

```
- (RACCompoundDisposable *)rac_deallocDisposable {
	@synchronized (self) {
		RACCompoundDisposable *compoundDisposable = objc_getAssociatedObject(self, RACObjectCompoundDisposable);
		if (compoundDisposable != nil) return compoundDisposable;

		swizzleDeallocIfNeeded(self.class);

		compoundDisposable = [RACCompoundDisposable compoundDisposable];
		objc_setAssociatedObject(self, RACObjectCompoundDisposable, compoundDisposable, OBJC_ASSOCIATION_RETAIN_NONATOMIC);

		return compoundDisposable;
	}
}
```
里面的重点是swizzleDeallocIfNeeded方法。如下：

```
static NSMutableSet *swizzledClasses() {
	static dispatch_once_t onceToken;
	static NSMutableSet *swizzledClasses = nil;
	dispatch_once(&onceToken, ^{
		swizzledClasses = [[NSMutableSet alloc] init];
	});
	
	return swizzledClasses;
}

static void swizzleDeallocIfNeeded(Class classToSwizzle) {
	@synchronized (swizzledClasses()) {
		NSString *className = NSStringFromClass(classToSwizzle);
		if ([swizzledClasses() containsObject:className]) return;

		SEL deallocSelector = sel_registerName("dealloc");

		__block void (*originalDealloc)(__unsafe_unretained id, SEL) = NULL;

		id newDealloc = ^(__unsafe_unretained id self) {
			RACCompoundDisposable *compoundDisposable = objc_getAssociatedObject(self, RACObjectCompoundDisposable);
			[compoundDisposable dispose];

			if (originalDealloc == NULL) {
				struct objc_super superInfo = {
					.receiver = self,
					.super_class = class_getSuperclass(classToSwizzle)
				};

				void (*msgSend)(struct objc_super *, SEL) = (__typeof__(msgSend))objc_msgSendSuper;
				msgSend(&superInfo, deallocSelector);
			} else {
				originalDealloc(self, deallocSelector);
			}
		};
		
		IMP newDeallocIMP = imp_implementationWithBlock(newDealloc);
		
		if (!class_addMethod(classToSwizzle, deallocSelector, newDeallocIMP, "v@:")) {
			// The class already contains a method implementation.
			Method deallocMethod = class_getInstanceMethod(classToSwizzle, deallocSelector);
			
			// We need to store original implementation before setting new implementation
			// in case method is called at the time of setting.
			originalDealloc = (__typeof__(originalDealloc))method_getImplementation(deallocMethod);
			
			// We need to store original implementation again, in case it just changed.
			originalDealloc = (__typeof__(originalDealloc))method_setImplementation(deallocMethod, newDeallocIMP);
		}

		[swizzledClasses() addObject:className];
	}
}
```
此swizziling方法，跟我们平时常用的写法不一致，但是总体思路都是通过runtime进行替换。

###总结
通过这样的形式，可以达到动态配置swizzling method的形式，如果采用+load()方法，那么进行替换的方法，不管有没有用都会执行，其实算是一种强制替换，而通过associated的形式，可以达到懒替换的效果，只在你调用的时候才进行替换，使用更方便。

###后记
对某个类的方法进行监控，可以通过method swizzling的方式，也可以采用RAC的方法，比如对NSNotificationCenter进行监控，也可以通过如下方法

```
    self.addObserverDisposable = [[center rac_signalForSelector:@selector(addObserver:selector:name:object:)]
     subscribeNext:^(RACTuple *args) {}
     
         self.removeNotificationsDisposable = [[center rac_signalForSelector:@selector(removeObserver:)]
     subscribeNext:^(RACTuple *args) {}

    self.removeSingleNotificationDisposable = [[center rac_signalForSelector:@selector(removeObserver:name:object:)]
     subscribeNext:^(RACTuple *args) {}

```
通过RAC的方式，监控某个方法，只需要在某个必执行的方法里面调用监控就可以了，当被调用的方法发现被掉哦那个，就会执行自己的{}方法。
至于他是如何执行，可以参考他的实现及解释。