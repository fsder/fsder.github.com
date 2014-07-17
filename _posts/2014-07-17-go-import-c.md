---
layout: post
title: "Go import C"
date: "2014-07-17 12:36"
description: ""
category: 
tags: []
---

Go 中调用 C 代码库，自己实现 C 静态库，过程如下

C 语言部分
==========

interface.h


先将 C 部分编译成静态库

``` sh

$gcc -c interface.c
$ar -r interface.a interface.o
ar: creating archive interface.a

```

Go 语言部分
===========

main.go


编译并运行

``` sh

$go build main.go
$./main
100

```

