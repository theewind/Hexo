layout: React Native 学习之F8app代码阅读
title: study
date: 2016-06-07 23:04:01
tags: React Native
---
主要是先参考如下篇文章：

> http://www.open-open.com/lib/view/open1461512056952.html

第一次接触redux，概念还比较模糊，同时js开发水平也一般，对一些语法学习还不是很懂，这里做个记录：
代码中的很多function func(): type的用法，我之前接触的基本都是无返回值类型的，这里添加一个 `:Action`类型，貌似就是确定了返回值类型的，不知道理解的对不对。

<!--more-->

```
/* from js/actions/filter.js */
function applyTopicsFilter(topics): Action {
  return {
    type: 'APPLY_TOPICS_FILTER',
    topics,
  };
}
```
redux的这个流程图不错
![组件](http://static.open-open.com/lib/uploadImg/20160424/20160424233655_731.png)

越看越不懂，任重而道远