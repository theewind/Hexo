---
title: 前端代码学习之scss
date: 2016-06-22 18:39:03
tags:
- css
- 前端
---

今天看到一个网站，主要都是前端动画[http://hakim.se](http://hakim.se),刚好自己最近对前端比较好奇，然后就学习了其中一个demo, [cloudy-spiral](http://codepen.io/hakimel/full/aIhkf),然后这里主要记录自己在学习前端过程中的一些小的知识点，不断更新中。。。

<!--more-->

scss是什么
===
sass是css3的一个扩展，增加了规则嵌套、变量、混合、选择器集成等。通过使用命令行的工具或WEB框架插件，可以把它转成标准的、格式良好的css代码。

scss即使sass的新语法，是Sassy css的简写，是css3语法的超集，也就是说所有有效的CSS3样式也同样适合于SASS。

#### 一、什么是SASS
SASS是一种CSS的开发工具，提供了许多便利的写法，大大节省了设计者的时间，使得CSS的开发，变得简单和可维护。
本文总结了SASS的主要用法。我的目标是，有了这篇文章，日常的一般使用就不需要去看官方文档了。
####二、安装和使用
#####2.1 安装
SASS是Ruby语言写的，但是两者的语法没有关系。不懂Ruby，照样使用。只是必须先安装Ruby，然后再安装SASS。
假定你已经安装好了Ruby，接着在命令行输入下面的命令：
gem install sass
然后，就可以使用了。

#####2.2 使用
SASS文件就是普通的文本文件，里面可以直接使用CSS语法。文件后缀名是.scss，意思为Sassy CSS。
下面的命令，可以在屏幕上显示.scss文件转化的css代码。（假设文件名为test。）
sass test.scss
如果要将显示结果保存成文件，后面再跟一个.css文件名。
sass test.scss test.css
SASS提供四个编译风格的选项：

* nested：嵌套缩进的css代码，它是默认值。
* expanded：没有缩进的、扩展的css代码。
* compact：简洁格式的css代码。
* compressed：压缩后的css代码。

生产环境当中，一般使用最后一个选项。
sass –style compressed test.sass test.css
你也可以让SASS监听某个文件或目录，一旦源文件有变动，就自动生成编译后的版本。
// watch a file
sass –watch input.scss:output.css
// watch a directory
sass –watch app/sass:public/stylesheets
SASS的官方网站，提供了一个[在线转换器。你可以在那里，试运行下面的各种例子。](http://sass-lang.com/try.html)

#####2.3 基本用法
具体可以参考：
>http://www.frostsky.com/2014/07/sass-scss/


sass 和 scss的区别呢
===
Sass 和 SCSS 其实是同一种东西，我们平时都称之为 Sass，两者之间不同之处有以下两点：
文件扩展名不同，Sass 是以“.sass”后缀为扩展名，而 SCSS 是以“.scss”后缀为扩展名

语法书写方式不同，Sass 是以严格的缩进式语法规则来书写，不带大括号({})和分号(;)，而 SCSS 的语法书写和我们的 CSS 语法书写方式非常类似。

```
先来看一个示例：
Sass 语法
$font-stack: Helvetica, sans-serif  //定义变量
$primary-color: #333 //定义变量

body
  font: 100% $font-stack
  color: $primary-color	 
  
SCSS语法
$font-stack: Helvetica, sans-seri;
$primary-color: #333;

body {
  font: 100% $font-stack;
  color: $primary-color;
}

编译出来的 CSS

body {
  font: 100% Helvetica, sans-serif;
  color: #333;
}
```


