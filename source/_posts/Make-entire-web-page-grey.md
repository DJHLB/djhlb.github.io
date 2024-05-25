---
title: Make entire web page grey
date: 2024-01-19 18:07:14
tags: Web
---

在HTML 的head标签里加入如下代码即可
```html
<style type="text/css">
html {
filter: progid:DXImageTransform.Microsoft.BasicImage(grayscale=1);
-webkit-filter: grayscale(100%);}
</style>
```
