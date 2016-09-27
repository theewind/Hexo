---
title: iOS代码编译速度优化
date: 2016-08-11 10:26:29
tags: iOS
---

cleaner源码编译速度优化方案
---
1. 参考最新的cleaner，主要是清理头文件之间的依赖，尽量都采取直接依赖，而不是通过间接依赖，比A依赖B，B依赖C。而A如果依赖C的时候，最好是直接通过采用A依赖C的操作，而不是A->B->C的操作。

cleaner要求，在所有的.h文件中，基本都采用@class，@protocol的形式添加声明，然后在.m文件中添加#import的操作，这样是可以最小限度的减少依赖环节，但是也是有弊端的：

比如常用的model嵌套，如果采用@class的声明的方式，在vm或者v中使用model的时候，都要将他的子model的头文件也添加上，还是比较繁琐的。但是采用这样的操作，可以最大限度的降低头文件依赖和编译时间，所以平台也是在大力推广。

2. 同时为了解决各个模块混乱依赖的问题，公司也要求将prefix.h文件给去掉，但是个人感觉还是可以添加一最常用的，比如 UIKit，NSFoundation等

3. pch的，苹果在xcode6中已经不建议使用了，原因可能是，大家总是把一些头文件都放到prefix中，就导致各个头文件都会包含所有prefix中所包含的，每个文件依赖增多，编译时间也会增加。参考[Code Smells in Objective-C](http://qualitycoding.org/objective-c-code-smells/)

<!--more-->


其他优化
---
1. 将Debug Information Format改为DWARF

在工程对应Target的Build Settings中，找到Debug Information Format这一项，将Debug时的DWARF with dSYM file改为DWARF。

这一项设置的是是否将调试信息加入到可执行文件中，改为DWARF后，如果程序崩溃，将无法输出崩溃位置对应的函数堆栈，但由于Debug模式下可以在XCode中查看调试信息，所以改为DWARF影响并不大。这一项更改完之后，可以大幅提升编译速度。

2. 将Build Active Architecture Only改为Yes

在工程对应Target的Build Settings中，找到Build Active Architecture Only这一项，将Debug时的No改为Yes。

这一项设置的是是否仅编译当前架构的版本，如果为No，会编译所有架构的版本。需要注意的是，此选项在Release模式下必须为Yes，否则发布的ipa在部分设备上将不能运行。这一项更改完之后，可以显著提高编译速度。

3. Enable Modules (C and Objective-C) 改为YES
打开Modules之后，可以采用@improt的形式导入依赖，但是这个功能主要是对系统的framework和library启作用，而对于我们常用的cocoapod继承的库是不生效的。

4. Don’t compile your project code or use static libraries compiled with -O4 since it tells Clang to enable Link Time Optimizations (LTO) making the linking stage much slower. Use -O3 at most.

延伸阅读
---

关于DWARF与dSYM

DWARF is a debugging file format used by many compilers and debuggers to support source level debugging. It addresses the requirements of a number of procedural languages, such as C, C++, and Fortran, and is designed to be extensible to other languages. DWARF is architecture independent and applicable to any processor or operating system. It is widely used on Unix, Linux and other operating systems, as well as in stand-alone environments.

DWARF与dSYM的关系是，DWARF是文件格式，而dSYM往往指一个单独的文件。在Xcode中如果不做特殊制定，debug information是被保存在executable文件中，可以使用dsymutil从executable中提取dSYM文件。

dsymutil

dsymutil is a tool to manipulate archived DWARF debug symbol files. 使用dsymutil可以对dSYM文件进行如下操作：从exe_path中提取成dSYM文件、将executable或者object文件中的symbol table dump出来、更新dSYM文件以让dSYM文件包含最新的accelerator tables and other DWARF optimizations。

Debug Info Format

在Xcode中可以选择DWARF和DWARF with dSYM file，推荐的设置是Debug用DWARF；Release使用DWARF with dSYM file。

使用dSYM file

如果我们有若干的build，有若干dSYM文件，而名字又有点乱，想知道哪个dSYM跟哪个build匹配，从而可以使用它们呢？办法就是查看UUID。

下面就是看怎么加载dSYM file了

关于dSYM和debug，这里有更多的信息。 


> http://www.cnblogs.com/whyandinside/archive/2013/04/28/3048366.html
> 
> [Speeding up Xcode Builds](https://tech.zalando.de/blog/speeding-up-xcode-builds/)
> 
[Modules and Precompiled Headers](http://useyourloaf.com/blog/modules-and-precompiled-headers/)
>
[Code Smells in Objective-C](http://qualitycoding.org/objective-c-code-smells/)
>
[Introduction to Objective-C Modules](https://stoneofarc.wordpress.com/2013/06/25/introduction-to-objective-c-modules/)
>
[Shaving off 50% waiting time from the iOS Edit-Build-Test cycle](https://labs.spotify.com/2013/11/04/shaving-off-time-from-the-ios-edit-build-test-cycle/)