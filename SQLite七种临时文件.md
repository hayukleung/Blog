---
title: SQLite七种临时文件
date: 2013-02-08 22:14:15
tags: database
---

[原文链接](http://www.cnblogs.com/liangxiaxu/archive/2013/02/08/2909236.html)

[TOC]

## 1.Rollback Journals

> 出现在事务的起始点，消失在事务的结束点
> 若PRAGMA locking_mode=EXCLUSIVE; // 默认为normal
> 则不会在事务结束点删除回滚日志
> 若pragma journal_mode=persist; // 默认为delete
> 也不会在事务结束点删除回滚日志
> 若pragma journal_mode=off;
> 则不会产生回滚日志
> 存储于磁盘上

<!--more-->

## 2.Master Journal Files

> 在一个数据库连接使用attach命令接入N个外部数据库的情况下，事务开始点产生的回滚日志
> 存储于磁盘上

## 3.Statement Journal Files

> 针对单个SQL语句的回滚日志
> 存储于磁盘上

## 4.Temp Databases

> 使用PRAGMA synchronous=OFF
> 使用PRAGMA journal_mode=PERSIST
> 操作速度较快，一般用于临时日志的记录操作

## 5.Materializations Of Views And Subqueries

> 存储视图、子查询等临时表

## 6.Transient Indices

> 瞬时索引，比如
> ORDER BY从句
> GROUP BY从句
> DISTINCT聚合查询关键词
> 复合SELECT从句joined by UNION, EXCEPT或INTERSECT

## 7.Transient Database Used By VACUUM

> 使用VACUUM压缩数据库文件时使用的临时文件

## 总结

- Temp Databases
- Materializations Of Views And Subqueries
- Transient Indices
- Transient Database Used By VACUUM

> 这四类临时文件的存储位置受pragma temp_store和SQLITE_TEMP_STORE影响，如下表所示：

### 表一
|名称|解释|存储位置|
|:----------------------|:---------------------|:-|
|Rollback Journals|回滚日志<br/>存在于事务的始末|始终存储于磁盘上<br/>这三类文件只是出现的场合不一样<br/>而本质是一样的<br/>可以总结为事务的回滚日志文件命名为’dbname.db-journal’|
|Master Journal Files|主数据库日志，连接外部数据库时出现|同上|
|Statement Journal Files|出现在单条SQL语句的事务始末|同上|
|Temp Databases|用于存储临时表（temp table）|见表2|
|Materializations Of Views And Subqueries|用于存储视图、子查询等临时表|同上|
|Transient Indices|用于存储order by、group by等临时索引|同上|
|Transient Database Used By VACUUM|Vacuum命令临时文件|同上|

### 表二
|注：<br/>即使将它们的位置设置为磁盘<br/>SQLite也会将它们存储在内存页中<br/>直到内存页满才写进磁盘<br/>而这个内存页大小由<br/>SQLITE_DEFAULT_TEMP_CACHE_SIZE决定<br/>默认是500页||||
|:----------------|:---------|:-|:-|
|pragma temp_store|0(default)|1 |2 |
|SQLITE_TEMP_STORE|0(default)|1 |2 |
|0                |磁盘|磁盘|磁盘|
|1(default)       |磁盘|磁盘|内存|
|2                |内存|磁盘|内存|
|3                |内存|内存|内存|