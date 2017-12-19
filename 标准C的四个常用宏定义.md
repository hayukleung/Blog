---
title: 标准C的四个常用宏定义
date: 2012-11-03 21:58:16
tags: c
---

[原文地址](http://www.cnblogs.com/liangxiaxu/archive/2012/11/03/2752232.html)

[TOC]

## 四个常用宏

- __FILE__ // 当前文件名
- __DATE__ // 编译日期
- __LINE__ // 编译行数
- __TIME__ // 编译时间

<!--more-->

### 测试代码

```c
// liangxiaxu@126.com
#include <stdio.h>
 
int main(void)
{           
    printf("hello, gcc!\n");
    printf("%s\n", __FILE__);  
    printf("%s\n", __DATE__);
    printf("%d\n", __LINE__);
    printf("%s\n", __TIME__);
    return 0; 
}
// vim: set tabstop=4 shiftwidth=4 expandtab:
```

### 运行结果

```console
[hayuk@localhost qinghua]$ ./hello 
hello, gcc!
hello.c
Nov  2 2012
9
07:56:26
```