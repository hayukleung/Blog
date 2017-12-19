---
title: C语言编译过程
date: 2012-11-03 21:36:13
tags: c
---
[原文地址](http://www.cnblogs.com/liangxiaxu/archive/2012/11/03/2752207.html)

[TOC]

```console
[hayuk@localhost qinghua]$ touch hello.c 
[hayuk@localhost qinghua]$ ls 
hello.c 
[hayuk@localhost qinghua]$ vim hello.c
// liangxiaxu@126.com
#include <stdio.h>
int main(void)
{
	printf("hello, gcc!\n");
	return 0;
}
// vim: set tabstop=4 shiftwidth=4 expandtab:
[hayuk@localhost qinghua]$ ls 
hello.c
```
## 预处理 替换宏
```console
[hayuk@localhost qinghua]$ gcc -E hello.c -o hello.i
[hayuk@localhost qinghua]$ ls 
hello.c  hello.i
```
## 编译 检查语法
```console
[hayuk@localhost qinghua]$ gcc -S hello.i -o hello.s 
[hayuk@localhost qinghua]$ ls 
hello.c  hello.i  hello.s
```
## 汇编 生成机器语言
```console
[hayuk@localhost qinghua]$ gcc -c hello.s -o hello.o
[hayuk@localhost qinghua]$ ls 
hello.c  hello.i  hello.o  hello.s
```
## 链接 链接.o文件或和外部链接库，生成可执行文件
```console
[hayuk@localhost qinghua]$ gcc hello.o -o hello
[hayuk@localhost qinghua]$ ls 
hello  hello.c  hello.i  hello.o  hello.s
```
## 执行
```console
[hayuk@localhost qinghua]$ ./hello 
hello, gcc! 
[hayuk@localhost qinghua]$ ls 
hello  hello.c  hello.i  hello.o  hello.s
```