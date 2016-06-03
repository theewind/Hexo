---
title: iOS横屏应用旋转无动画
date: 2016-05-14 14:43:33
tags: iOS
---

最近开发一个仅支持横屏的应用，发现进行180°旋转的时候没有动画，Google也没有找到有用的信息，显示的情况跟下面链接中的比较像[无动画效果](http://stackoverflow.com/questions/32848456/ios-9-orientation-auto-rotation-animation-not-working-but-always-on-main-thread/37222383#37222383). 最后在苹果的官网资料上找到了灵感：[https://developer.apple.com/library/ios/technotes/tn2244/_index.html](https://developer.apple.com/library/ios/technotes/tn2244/_index.html) 
自己就是根据这个里面的内容，全部进行了操作，最后发现是LaunchScreen.stroyboard的问题，在
![设置](http://7xthb9.com1.z0.glb.clouddn.com/launchScreen.png) 中对Main Interface 和 Launch Screen File都设置成了空，将Launch Images Source 另外指定了图片，然后就解决了此问题。

我的解决方案不一定完全使用所有的情况，这里仅仅是做一个记录，有需要的可以试一下。