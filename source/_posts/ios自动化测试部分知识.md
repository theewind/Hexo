---
title: ios自动化测试部分知识
date: 2016-09-05 11:06:59
tags: 
- iOS
- 自动化
---

最近在看关于iOS自动化测试的内容，其中比较常用的有KIF等，然后自己看他的实现时，发现有个beforeAll和AfterAll的方法，因为在当初写case的时候，不能知道所有case开始，和结束的时机，于是就对它比较好奇，看了他的实现。

<!--more-->

```
- (void)beforeAll
{
    [system simulateDeviceRotationToOrientation:UIDeviceOrientationLandscapeLeft];
    [[viewTester usingIdentifier:@"Test Suite TableView"] scrollByFractionOfSizeHorizontal:0 vertical:-0.2];
}

- (void)afterAll
{
    [system simulateDeviceRotationToOrientation:UIDeviceOrientationPortrait];
    [viewTester waitForTimeInterval:0.5];
}

+ (void)setUp
{
    KIFEnableAccessibility();
    [self performSetupTearDownWithSelector:@selector(beforeAll)];
}

+ (void)tearDown
{
    [self performSetupTearDownWithSelector:@selector(afterAll)];
}

+ (void)performSetupTearDownWithSelector:(SEL)selector
{
    KIFTestCase *testCase = [self testCaseWithSelector:selector];
    if ([testCase respondsToSelector:selector]) {
#pragma clang diagnostic push
#pragma clang diagnostic ignored "-Warc-performSelector-leaks"
        [testCase performSelector:selector];
#pragma clang diagnostic pop
    }

    if (testCase->_stoppingException) {
        [testCase->_stoppingException raise];
    }
}
```
总体是一个，beforeAll调用performSetupTearDownWithSelector然后调用setUp方法。
于是自己就很困惑，记得之前看setUp方法是每个case开始都会调用，这跟beforeAll的时机是不吻合的。

但是如果自己对比你会发现：有两个方法

	- (void)setUp
	+ (void)setUp
	
由此，自己才明白，原理，类方法，是只运行一一次，在所有的测试执行前执行一次，实例方法是所有的case执行前都会执行一次
> The class method (+ (void)setUp) is only run once during the entire test run.
The instance method (- (void)setUp) is the one in the default template; it's run before every single test. Hopefully, in a hypothetical future version of Xcode, this comment will have been changed to // Put setup code here. This method is called before the invocation of each test method in the class.

> [What is the purpose of XCTestCase's setUp method?](http://stackoverflow.com/questions/21038375/what-is-the-purpose-of-xctestcases-setup-method)
> 
> [iOS UI Testing with KIF](https://www.raywenderlich.com/61419/ios-ui-testing-with-kif)
> 
> [基于 KIF 的 iOS UI 自动化测试和持续集成](http://tech.meituan.com/iOS-UITest-KIF.html)