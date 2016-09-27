---
title: ios7下UIScrollView和滑动返回
date: 2016-07-22 14:21:40
tags: iOS
---
问题
===
之前因为某些历史原因，外卖基本禁止了所有的滑动返回手势，这次产品过来说需要开发一些正常流程的返回操作，于是在修复菜单页的过程中遇到了一些问题。菜单页可以说是整个外卖最复杂的页面，如下图![菜单页](http://o981ibvmi.bkt.clouddn.com/QQ20160722-2@2x.png)

图中有p1,p2,p3,p4,p5，整个菜单页的结构是：
P5 最底层，UIScrollView
P4 标题，UIView
P3 内容, UIView
P2 tab, UIScrollView
P1 page, UIScrollView

层次是：p1 + p2 --> p3 + p4 --> p5

这样就有了问题，左右滑P1，其实是UIScrollView的页面切换，分别显示，菜单，点评和商家详情。
`在整个P3区域进行滑动返回操作，是不起效果的。`

<!--more-->

解决
===
查阅资料发现，滑动返回事实上也是由存在已久的UIPanGestureRecognizer来识别并响应的，它直接与UINavigationController的view（方便起见，下文中以UINavigationController.view表示）进行绑定，因此上图中存在如下关系：

UIPanGestureRecognizer  ——bind——  UIScrollView

UIScreenEdgePanGestureRecognizer ——bind——  UINavigationController.view

滑动返回无法触发，说明UIScreenEdgePanGestureRecognizer并没有接收到手势事件

UINavigationController.view —>  UIViewController.view —>  UIScrollView —>  Screen and User's finger

即UIScrollView的panGestureRecognizer先接收到了手势事件，直接就地处理而没有往下传递。

实际上这就是两个panGestureRecognizer共存的问题。

然后具体的解决方案就是：
在P1的继承自子UIScrollView里面添加如下方法

```
- (BOOL)gestureRecognizer:(UIGestureRecognizer *)gestureRecognizer shouldRecognizeSimultaneouslyWithGestureRecognizer:(UIGestureRecognizer *)otherGestureRecognizer
{
    if ([gestureRecognizer isKindOfClass:[UIPanGestureRecognizer class]]
        && [otherGestureRecognizer isKindOfClass:[UIScreenEdgePanGestureRecognizer class]]) {
        return YES;
    }
    
    return NO;
}

```
貌似解决问题，但是自己看会有相应UIScrollView的操作，如![下图展示](http://o981ibvmi.bkt.clouddn.com/QQ20160722-3@2x.png)
看到问题了吧，在点评页面，滑动返回的时候居然一边返回，一边切换到菜单页部分，这有点不能忍，然后继续分析，是因为：

`只是做到了将手势事件往下传递，而没有关闭掉在边缘时UIScrollView对事件的响应。`

事实上，对UIGestureRecognizer来说，它们对事件的接收顺序和对事件的响应是可以分开设置的，即存在接收链和响应链。接收链如上文所述，和UIView绑定，由UIView的层次决定接收顺序。

而响应链在apple君的定义下，逻辑出奇的简单，只有一个方法可以设置多个gestureRecognizer的响应关系：

```
// create a relationship with another gesture recognizer that will prevent this gesture's actions from being called until otherGestureRecognizer transitions to UIGestureRecognizerStateFailed
// if otherGestureRecognizer transitions to UIGestureRecognizerStateRecognized or UIGestureRecognizerStateBegan then this recognizer will instead transition to UIGestureRecognizerStateFailed
// example usage: a single tap may require a double tap to fail
- (void)requireGestureRecognizerToFail:(UIGestureRecognizer *)otherGestureRecognizer;
 ```
每个UIGesturerecognizer都是一个有限状态机，上述方法会在两个gestureRecognizer间建立一个依托于state的依赖关系，当被依赖的gestureRecognizer.state = failed时，另一个gestureRecognizer才能对手势进行响应。

于是，添加如下操作：

```
在创建contentView的时候添加入如下，
        UIScreenEdgePanGestureRecognizer *screenEdgePanGestureRecognizer = self.screenEdgePanGestureRecognizer;
        [_contentPageView.panGestureRecognizer requireGestureRecognizerToFail:screenEdgePanGestureRecognizer];


- (UIScreenEdgePanGestureRecognizer *)screenEdgePanGestureRecognizer
{
    UIScreenEdgePanGestureRecognizer *screenEdgePanGestureRecognizer = nil;
    if (self.navigationController.view.gestureRecognizers.count > 0)
    {
        for (UIGestureRecognizer *recognizer in self.navigationController.view.gestureRecognizers)
        {
            if ([recognizer isKindOfClass:[UIScreenEdgePanGestureRecognizer class]])
            {
                screenEdgePanGestureRecognizer = (UIScreenEdgePanGestureRecognizer *)recognizer;
                break;
            }
        }
    }
    
    return screenEdgePanGestureRecognizer;
}

```

> [iOS7下滑动返回与ScrollView共存二三事](http://www.cnblogs.com/lexingyu/p/3702742.html) 
> 
> [Demo](https://github.com/cDigger/CoExistOfScrollViewAndBackGesture/tree/master)
