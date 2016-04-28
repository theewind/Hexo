---
title: hexo 基础
date: 2016-04-28 17:44:20
tags: hexo
---

### tags的用法

基本使用方法是：

	tags: hexo

如果一行多个标签，使用
  	
  	tags: [hexo, hexo1, hexo2, ...]
  	
如果一篇文章属于多个标签

	tags: 
	- hexo
	- hexo2
	- hexo3
	
### new 和 new page
最终生成的静态文档都在public下

- new: 统一在归档目录 archive下
- new page: 新建一个目录，比如about，然后在theme/yilia/_config.yml 的 menu下可以添加新的模块，比如 about: /about	
