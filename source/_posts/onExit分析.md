---
title: onExit分析
date: 2016-07-25 12:01:47
tags: iOS
---

defer引发的OnExit

在swift中，是经常用到defer的，在作用域结束的时候，执行defer的命令，但是在Objective C中是无法使用的，不过有个宏 `onExit`可以达到这样的效果，比如

```
if (1) {
	@onExit {
 		NSLog(@"ffffff");
	};
}
```
就会在if作用域之后打印fffffff，那么他是怎么实现的呢，主要是用到了 `__attribute__`变量属性，具体的onExit的定义如下：

```
#ifndef onExit
#define onExit \
ext_keywordify \
__strong ext_cleanupBlock_t metamacro_concat(ext_exitBlock_, __LINE__) __attribute__((cleanup(ext_executeCleanupBlock), unused)) = ^
#endif
```
1. ext_keywordify 
2. ext_clearupBlock_t
3. metamacro_concat
4. clearup

其中ext_keywordify 在debug下就是一个@autoreleasepool, release下是@try..cache...finally
这里强制要求添加一个@，否则编译器会报错

ext_clearupBlock_t 是一个typedef void (^ext_cleanupBlock_t)();

metamacro_concat(A, B) 就是 metamacro_concat(A, B) A ## B ，将AB连接起来，例如

	metamacro_concat(ext_exitBlock_, __LINE__)
	拼接为：ext_exitBlock_19

clearup 

> http://draveness.me/defer/