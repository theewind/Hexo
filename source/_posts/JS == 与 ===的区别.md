---
title: JS == 与 ===的区别
date: 2016-04-28 17:58:08
tags: JavaScript
---

#### 1、对于string,number等基础类型，==和===是有区别的

- 1）不同类型间比较，==之比较“转化成同一类型后的值”看“值”是否相等，===如果类型不同，其结果就是不等
- 2）同类型比较，直接进行“值”比较，两者结果一样

#### 2、对于Array,Object等高级类型，==和===是没有区别的
- 进行“指针地址”比较

#### 3、基础类型与高级类型，==和===是有区别的
- 1）对于==，将高级转化为基础类型，进行“值”比较
- 2）因为类型不同，===结果为false
