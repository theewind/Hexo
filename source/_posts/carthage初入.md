---
title: carthage初入
date: 2016-08-04 15:11:34
tags:
---

carthage是什么
===
> Carthage
英 [ˈkɑ:θidʒ]  美 [ˈkɑrθɪdʒ] 
n.
迦太基（非洲北部，今突尼斯的奴隶制城邦，腓尼基人所建，公元146年被罗马帝国所灭，见Punic Wars）
网络
迦太基古城;  迦太基遗址;  迦太基城

carthage是一个iOS的包依赖管理工具，相对于CocoaPods，他的目标是去中心化的依赖管理器。
它没有总项目的列表，这能够减少维护工作并且避免任何中心化带来的问题（如中央服务器宕机）。不过，这样也有一些缺点，就是项目的发现将更困难，用户将依赖于Github的趋势页面或者类似的代码库来寻找项目。

<!--more-->

carthage和 CocoaPods
===

CocoaPods默认会自动创建并更新你的应用程序和所有依赖的Xcode workspace。Carthage使用xcodebuild来编译框架的二进制文件，但如何集成它们将交由用户自己判断。CocoaPods的方法更易于使用，但Carthage更灵活并且是非侵入性的。

CocoaPods的目标在它的README文件描述如下：

…为提高第三方开源库的可见性和参与度，创建一个更中心化的生态系统。

与之对照，Carthage创建的是去中心化的依赖管理器。它没有总项目的列表，这能够减少维护工作并且避免任何中心化带来的问题（如中央服务器宕机）。不过，这样也有一些缺点，就是项目的发现将更困难，用户将依赖于Github的趋势页面或者类似的代码库来寻找项目。

CocoaPods项目同时还必须包含一个podspec文件，里面是项目的一些元数据，以及确定项目的编译方式。Carthage使用xcodebuild来编译依赖，而不是将他们集成进一个workspace，因此无需类似的设定文件。不过依赖需要包含自己的Xcode工程文件来描述如何编译。

最后，我们创建Carthage的原因是想要一种尽可能简单的工具——一个只关心本职工作的依赖管理器，而不是取代部分Xcode的功能，或者需要 让框架作者做一些额外的工作。CocoaPods提供的一些特性很棒，但由于附加的复杂性，它们将不会被包含在Carthage当中。
使用
===

1 安装

	brew install carthage
	
2 创建cartfile文件

	vim Cartfile
	
3 添加依赖

```
#Require version 2.5 or later
github "ReactiveCocoa/ReactiveCocoa" >= 2.5

#Require version 1.x
github "Mantle/Mantle" ~> 1.0
```

4 执行

	carthage update

5 添加framework到project中
	
	在Build目录下会生成对应的iOS， Mac， tvOS，watchOS四个平台的各自framework集合，以及他们的dSYM文件，后续的添加到工程中使用即可。
	
6 支持版本iOS8