---
layout: post
title: "Go import C"
date: "2014-07-17 12:36"
description: ""
category: 
tags: []
---

Go 中调用 C 代码库，自己实现 C 静态库，过程如下：

C 语言部分
==========

interface.h

``` C

int Test(int a);

```

interface.c

``` C

#include "interface.h"

int Test(int a)
{
    return a;
}

```

先将 C 部分编译成静态库

``` sh

$gcc -c interface.c
$ar -r interface.a interface.o
ar: creating archive interface.a

```

Go 语言部分
===========

main.go

``` go

package main

// #cgo CFLAGS:
// #cgo LDFLAGS: -L. libinterface.a
// #include "interface.h"
import "C"

import "fmt"

func main() {
	fmt.Println(C.Test(100))
}

```

编译并运行

``` sh

$go build main.go
$./main
100

```

