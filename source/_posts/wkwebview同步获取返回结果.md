---
title: wkwebview同步获取返回结果
date: 2016-06-21 17:46:50
tags:
- iOS
- WKWebView
---

今天在测试UIWebView和WKWebView的时候，发现如下区别

1. UIWebView执行stringByEvaluatingJavaScriptFromString方法，因为直接返回结果，所以是``同步``的。
   WKWebView执行evaluateJavaScript：completionHandler是在block中获取执行结果的，所以他是``异步``进行的，
   这样在进行性能测试的时候，发现如果不将WKWebView他修改为同步，是无法进行对比的，然后自己就通过如下方法
   
   ```
   - (NSString *)stringByEvaluatingJavaScriptFromString:(NSString *)javascript {
    __block NSString *res = nil;
    __block BOOL finish = NO;
    
    [self.webView evaluateJavaScript:javascript completionHandler:^(NSString *result, NSError *error){
        res = result;
        finish = YES;
    }];
    
    while(!finish) {
        [[NSRunLoop currentRunLoop] runMode:NSDefaultRunLoopMode beforeDate:[NSDate distantFuture]];
    }
    return res;
}
   ```
   将WKWebView修改为同步状态，然后进行对比，主要是测试JavaScript函数执行100，300，500，1000，3000次，统计时间，可以发现在相同JS功能情况下，WKWebView的效率还没有UIWebView的效率高，如![下图](http://o981ibvmi.bkt.clouddn.com/uiwebview_wkwebview_jscore.png)
   可以看出，JavascriptCore的效率是最高的，而WKWebView改成同步以后，实际效果是最差的，分析原因也可能是因为，WKWebView不是这样用的，毕竟在无限等待中，所以结果也可能是受影响的。
   
 > 
 http://www.jianshu.com/p/3a59107aa2d2
 http://stackoverflow.com/questions/26778955/wkwebview-evaluate-javascript-return-value