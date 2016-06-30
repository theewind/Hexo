---
title: css中的伪元素after和before的默认顺序
date: 2016-06-27 18:33:18
tags:
- 前端 
- CSS
---

今天在学习CSS动画的时候，看到很多动画都是采用了`伪元素 :before和:after`，他们是干什么的以及怎么用，可以自行google，我这里主要是对他们的顺序进行一下探讨。
乍感觉一下，after应该是在原元素的前面，而before应该是在后面，实际上不是这样的，after和before都算是他的子元素，所以默认情况下，他们都是在原始元素的前面，借用网友的一张![图片](http://www.alixixi.com/web/UploadPic/2011-9/20119211128385.jpg)，他是在这样显示的，如果还有不相信的。

<!--more-->
如果想自己测试下：可以试验下如下的代码

```
#star-six {         
		width: 0;        
		height: 0;      
		border-left: 50px solid transparent;  
		border-right: 50px solid transparent;      
		border-bottom: 100px solid red;     
		position: relative;
	}
	
	#star-six:after {   
		content:'';
		width: 0;
		height: 0;
		border-left: 50px solid transparent;
		border-right: 50px solid transparent;         
		border-top: 100px solid green;
		position: absolute;         
		content: "";         
		top: 30px;
		left: -50px;
		z-index:auto
	}
	
		#star-six:before {   
		content:'';
		width: 0;
		height: 0;
		border-left: 50px solid transparent;
		border-right: 50px solid transparent;         
		border-bottom: 100px solid yellow;
		position: absolute;         
		content: "";         
		top: 30px;
		left: -50px;
			z-index:auto;
	}
```
运行结果如下：![](http://o981ibvmi.bkt.clouddn.com/css_after_before.png)

跟上面的图片结果是一样的。那么如果想修改after和before的顺序呢，只要添加z-index:就行，比如-1等。所以很多动画效果都可以通过这个来实现，[参考网站](http://zaole.net).

> http://justcoding.iteye.com/blog/2032627
> http://www.alixixi.com/web/a/2011090273706.shtml