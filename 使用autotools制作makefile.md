---
title: 使用autotools制作makefile
date: 2012-05-05 22:24:25
tags: c
---
[原文地址](http://www.cnblogs.com/liangxiaxu/archive/2012/05/05/2485099.html)

[TOC]

## flat模式

- autoscan
> 生成configure.scan

- 改写configure.scan
```makefile
AC_INIT(最终可执行文件名, 版本号)
AM_INIT_AUTOMAKE
AC_CONFIG_SRCDIR(源文件所在文件夹中的一个文件名，用于检测路径)
AC_CONFIG_HEADER(config.h)
AC_OUTPUT(Makefile)
AC_PROG_RANLIB
```
> 改写完毕后另存为configure.in

- aclocal
- autoconf
- autoheader
- 编辑Makefile.am
```makefile
AUTOMAKE_OPTIONS=foreign
bin_PROGRAMS=可执行文件名1可执行文件名2 ......可执行文件名n
可执行文件名1_SOURCES=该可执行文件生成所必须的源文件、头文件等，以空格分开
可执行文件名2_SOURCES=该可执行文件生成所必须的源文件、头文件等，以空格分开
......
可执行文件名n_SOURCES=该可执行文件生成所必须的源文件、头文件等，以空格分开
可执行文件名1_LDADD=-l库名
......
可执行文件名n_LDADD=-l库名
```
- automake --add-missing
- ./configure
- make
- 生成可执行文件，运行。

***

## deep模式以例子说明

### 环境
> Linux Redhat Enterprise

### 文件结构
```
|--rtcp
   |--src
   |  |--server.c
   |  |--client.c
   |--include
      |--rtcp.c
      |--rtcp.h
```

### 目标
> 在根目录/rtcp下建立Makefile

### 步骤
- 进入/rtcp
- autoscan
- 产生configure.scan
- 修改该文件：
```makefile
AM_INIT_AUTOMAKE
# Checks for programs.
AC_PROG_CC 
AM_PROG_CC_C_O
# Checks for libraries. 
AC_PROG_RANLIB
AC_OUTPUT(Makefile)
```
- 保存为configure.in
- aclocal
- autoconf
- autoheader
- gedit Makefile.am，编辑Makefile.am
```makefile
AUTOMAKE_OPTIONS=subdir-objects 
bin_PROGRAMS=server client 
server_SOURCES=src/server.c include/rtcp.c include/rtcp.h 
client_SOURCES=src/client.c include/rtcp.c include/rtcp.h 
server_LDADD=-lpthread 
client_LDADD=-lpthread
server_LDFLAGS=-L/path/to/lib/pthread/
client_LDFLAGS=-L/path/to/lib/pthread/
```
- 保存
- automake --add-missing
- ./configure
- make
