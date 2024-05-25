---
layout: post
title: CentOS 编译vim no terminal library found
date: 2021-06-25 16:08:34
tags:
---

./configure 时如果遇到：

```bash
no terminal library found
checking for tgetent()… configure: error: NOT FOUND!
      You need to install a terminal library; for example ncurses.
      Or specify the name of the library with –with-tlib.
```

Ubuntu解决方法：

```sudo apt install libncurses5-dev```

CentOS 下：

```yum install ncurses-devel.x86_64```
