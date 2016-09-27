---
title: UIKit主线程检查
date: 2016-07-18 10:57:04
tags: iOS
---

iOS开发过程中，很多时候都会使用GCD及NSOperation等涉及到多线程的操作，这样就算是一个老手，也可能会遇到非主线程刷新UI的操作，而结果就是遇到各种不确定的问题，[这篇文章:线程安全类的设计](https://www.objccn.io/issue-2-4/)向我们解释了为什么UIKit是非线程安全的，然后也介绍了一个[UIKit主线程检查的机制](https://gist.github.com/steipete/5664345)，今天自己又复习下这个内容。

<!--more-->

method swizzle
===
主要原理就是采用：method swizzle 方法，替换系统的`-setNeedsLayout, -setNeedDisplay, -setNeedsDisplayInRect`,三个方法，确保他们是在主线程进行的。[微信读书](http://wereadteam.github.io/2016/05/03/WeRead-Performance/?from=groupmessage&isappinstalled=0)的优化方案里面也是这样用的，但是自己重新看了下Main thread guard之后，还是重新发现了一些新的东西。

首先，之前替换方法都是采用orginSel，newSel就可以了，因为可以通过SEL获取到IMP，那如果是一个blcok呢，其实也是可以的，因为block最终都算是一个function，可以根据获取到IMP，后续操作节本都差不多

	IMP impl = imp_implementationWithBlock(block);
	
\__attribute__((constructor))
===
解释一下：__attribute__((constructor)) 在main() 之前执行,__attribute__((destructor)) 在main()执行结束之后执行.
这些是属于黑魔法的范围，一般我是是采用在+（load）方法进行替换，而这里是曹永__attribute__((constructor))的方式，在函数main之前就执行的。	

	
> 这里自己无耻的贴下代码，本身没什么难度，只需要在替换的函数里面进行thread的判断：是否是主线程。

```
// Taken from the commercial iOS PDF framework http://pspdfkit.com.
// Copyright (c) 2014 Peter Steinberger, PSPDFKit GmbH. All rights reserved.
// Licensed under MIT (http://opensource.org/licenses/MIT)
//
// You should only use this in debug builds. It doesn't use private API, but I wouldn't ship it.

// PLEASE DUPE rdar://27192338 (https://openradar.appspot.com/27192338) if you would like to see this in UIKit.

#import <objc/runtime.h>
#import <objc/message.h>

// Compile-time selector checks.
#if DEBUG
#define PROPERTY(propName) NSStringFromSelector(@selector(propName))
#else
#define PROPERTY(propName) @#propName
#endif

// http://www.mikeash.com/pyblog/friday-qa-2010-01-29-method-replacement-for-fun-and-profit.html
BOOL PSPDFReplaceMethodWithBlock(Class c, SEL origSEL, SEL newSEL, id block) {
    NSCParameterAssert(c);
    NSCParameterAssert(origSEL);
    NSCParameterAssert(newSEL);
    NSCParameterAssert(block);
    
    if ([c instancesRespondToSelector:newSEL]) return YES; // Selector already implemented, skip silently.

    Method origMethod = class_getInstanceMethod(c, origSEL);

    // Add the new method.
    IMP impl = imp_implementationWithBlock(block);
    if (!class_addMethod(c, newSEL, impl, method_getTypeEncoding(origMethod))) {
        PSPDFLogError(@"Failed to add method: %@ on %@", NSStringFromSelector(newSEL), c);
        return NO;
    }else {
        Method newMethod = class_getInstanceMethod(c, newSEL);

        // If original doesn't implement the method we want to swizzle, create it.
        if (class_addMethod(c, origSEL, method_getImplementation(newMethod), method_getTypeEncoding(origMethod))) {
            class_replaceMethod(c, newSEL, method_getImplementation(origMethod), method_getTypeEncoding(newMethod));
        }else {
            method_exchangeImplementations(origMethod, newMethod);
        }
    }
    return YES;
}

SEL _PSPDFPrefixedSelector(SEL selector) {
    return NSSelectorFromString([NSString stringWithFormat:@"pspdf_%@", NSStringFromSelector(selector)]);
}

#define PSPDFAssert(expression, ...) \
do { if(!(expression)) { \
NSLog(@"%@", [NSString stringWithFormat: @"Assertion failure: %s in %s on line %s:%d. %@", #expression, __PRETTY_FUNCTION__, __FILE__, __LINE__, [NSString stringWithFormat:@"" __VA_ARGS__]]); \
abort(); }} while(0)

void PSPDFAssertIfNotMainThread(void) {
    PSPDFAssert(NSThread.isMainThread, @"\nERROR: All calls to UIKit need to happen on the main thread. You have a bug in your code. Use dispatch_async(dispatch_get_main_queue(), ^{ ... }); if you're unsure what thread you're in.\n\nBreak on PSPDFAssertIfNotMainThread to find out where.\n\nStacktrace: %@", NSThread.callStackSymbols);
}

__attribute__((constructor)) static void PSPDFUIKitMainThreadGuard(void) {
    @autoreleasepool {
        for (NSString *selStr in @[PROPERTY(setNeedsLayout), PROPERTY(setNeedsDisplay), PROPERTY(setNeedsDisplayInRect:)]) {
            SEL selector = NSSelectorFromString(selStr);
            SEL newSelector = NSSelectorFromString([NSString stringWithFormat:@"pspdf_%@", selStr]);
            if ([selStr hasSuffix:@":"]) {
                PSPDFReplaceMethodWithBlock(UIView.class, selector, newSelector, ^(__unsafe_unretained UIView *_self, CGRect r) {
                    // Check for window, since *some* UIKit methods are indeed thread safe.
                    // https://developer.apple.com/library/ios/#releasenotes/General/WhatsNewIniPhoneOS/Articles/iPhoneOS4.html
                    /*
                     Drawing to a graphics context in UIKit is now thread-safe. Specifically:
                     The routines used to access and manipulate the graphics context can now correctly handle contexts residing on different threads.
                     String and image drawing is now thread-safe.
                     Using color and font objects in multiple threads is now safe to do.
                     */
                    if (_self.window) PSPDFAssertIfNotMainThread();
                    ((void ( *)(id, SEL, CGRect))objc_msgSend)(_self, newSelector, r);
                });
            }else {
                PSPDFReplaceMethodWithBlock(UIView.class, selector, newSelector, ^(__unsafe_unretained UIView *_self) {
                    if (_self.window) {
                        if (!NSThread.isMainThread) {
#pragma clang diagnostic push
#pragma clang diagnostic ignored "-Wdeprecated-declarations"
                            dispatch_queue_t queue = dispatch_get_current_queue();
#pragma clang diagnostic pop
                            // iOS 8 layouts the MFMailComposeController in a background thread on an UIKit queue.
                            // https://github.com/PSPDFKit/PSPDFKit/issues/1423
                            if (!queue || !strstr(dispatch_queue_get_label(queue), "UIKit")) {
                                PSPDFAssertIfNotMainThread();
                            }
                        }
                    }
                    ((void ( *)(id, SEL))objc_msgSend)(_self, newSelector);
                });
            }
        }
    }
}
```

	
