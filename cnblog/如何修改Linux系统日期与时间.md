---
title: 如何修改Linux系统日期与时间
date: 2012-11-03 22:02:06
tags: linux
---

[原文地址](http://www.cnblogs.com/liangxiaxu/archive/2012/11/03/2752254.html)

[TOC]

## 修改时区

> cp /usr/share/zoneinfo/UTC /etc/localtime

```console
[hayuk@localhost qinghua]$ date 
2012年 11月 02日 星期五 08:04:30 CST
[hayuk@localhost qinghua]$ su - root 
口令：
[root@localhost ~]# cp /usr/share/zoneinfo/UTC /etc/localtime 
cp：是否覆盖“/etc/localtime”? y 
[root@localhost ~]# date 
2012年 11月 02日 星期五 00:07:30 UTC
```

<!--more-->

## 修改系统日期与时间

> date -s "2012-11-03 10:25:25"

```console
[root@localhost ~]# date 
2012年 11月 02日 星期五 00:08:27 UTC
[root@localhost ~]# date -s "2012-11-03 10:25:25" 
2012年 11月 03日 星期六 10:25:25 UTC 
[root@localhost ~]# date 
2012年 11月 03日 星期六 10:25:27 UTC
```