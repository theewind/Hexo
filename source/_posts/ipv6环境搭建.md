---
title: ipv6环境搭建
date: 2016-09-24 16:05:36
tags:
---
自从5月初Apple明文规定所有开发者在6月1号以后提交新版本需要支持IPV6-Only的网络，大家便开始热火朝天的研究如何支持IPV6，以及应用中哪些模块目前不支持IPV6。

IPV6是什么，可以参考网上的文章
> [iOS应用如何支持IPV6-b](http://www.cnblogs.com/isItOk/p/5621530.html)，

如何用自己的Mac搭建ipv6测试环境，可以参考：
> [Mac电脑搭建IPV6测试环境](http://jingyan.baidu.com/article/e75057f2f33cffebc91a89a3.html)

<!--more-->


这里需要注意的是，如果mac电脑本身就是通过wifi连接网络的，需要使用一个USB Ethernet或者雷电转接口，保证电脑是通过网线连接的。比如自己的电脑就是这样的![网络设置](http://o981ibvmi.bkt.clouddn.com/QQ20160924-0@2x.png)
确保你的USB Ethernet网络是排在第一位的，如果不是的话，可以先删除Wifi，确保有线网络是第一位，然后再创建一个。
之后按照上面文章的配置就可以了。

如果应用发布的时候因为ipv6被拒，可以参考

> [ipv6审核问题大集合](https://github.com/wg689/Solve-App-Store-Review-Problem)

[TUTORIAL: HOW TO TEST YOUR APP FOR IPV6 COMPATIBILITY](http://www.brianjcoleman.com/tutorial-how-to-test-your-app-for-ipv6-compatibility/)

[iOS应用支持IPV6，就那点事儿](http://www.jianshu.com/p/a6bab07c4062)