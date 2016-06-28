---
title: 如何在自己的网页中添加ga分析
date: 2016-06-23 16:58:54
tags: 前端
---

在学习前端的过程中，自己都想统计用户的访问量，下面代码可以做到最轻量级的追踪：

```
/*<script type="text/javascript">

  var _gaq = _gaq || [];
  _gaq.push(['_setAccount', 'UA-XXXXX-X']);
  _gaq.push(['_trackPageview']);

  (function() {
    var ga = document.createElement('script'); ga.type = 'text/javascript'; ga.async = true;
    ga.src = ('https:' == document.location.protocol ? 'https://ssl' : 'http://www') + '.google-analytics.com/ga.js';
    var s = document.getElementsByTagName('script')[0]; s.parentNode.insertBefore(ga, s);
  })();

</script>
*/

```

> 具体可以参考如下
 https://developers.google.com/analytics/devguides/collection/gajs/
 
 
