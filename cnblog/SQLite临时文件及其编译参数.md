---
title: SQLite临时文件及其编译参数
date: 2012-09-09 21:23:50
tags: database
---

[原文地址](http://www.cnblogs.com/liangxiaxu/archive/2012/09/09/2677339.html)

[TOC]

## 一、临时文件

> 尽管SQLite的数据库是由单一文件构成，然而事实上在SQLite运行时却存在着一些隐含的临时文件，这些临时文件是出于不同的目的而存在的，对于开发者而言，它们是透明的，因此在开发的过程中我们并不需要关注它们的存在。尽管如此，如果能对这些临时文件的产生机制和应用场景有着很好的理解，那么对我们今后应用程序的优化和维护都是极有帮助的。在SQLite中主要产生以下七种临时文件，如：
- 回滚日志
- 主数据库日志
- SQL语句日志
- 临时数据库文件
- 视图和子查询的临时持久化文件
- 临时索引文件
- VACUUM命令使用的临时数据库文件

<!--more-->

1. 回滚日志
> SQLite为了保证事务的原子性提交和回滚，在事务开始时创建了该临时文件。此文件始终位于和数据库文件相同的目录下，其文件名格式为: 数据库文件名+ "-journal"。换句话说，如果没有该临时文件的存在，当程序运行的系统出现任何故障时，SQLite将无法保证事务的完整性，以及数据状态的一致性。该文件在事务提交或回滚后将被立刻删除。在事务运行期间，如果当前主机因电源故障而宕机，而此时由于回滚日志文件已经保存在磁盘上，那么当下一次程序启动时，SQLite在打开数据库文件的过程中将会发现该临时文件的存在，我们称这种日志文件为"Hot Journal"。SQLite会在成功打开数据库之前先基于该文件完成数据库的恢复工作，以保证数据库的数据回复到上一个事务开始之前的状态。在SQLite中我们可以通过修改pragma journal_mode，而使SQLite对维护该文件采用不同的策略。缺省情况下该值为DELETE，即在事务结束后删除日志文件。而PERSIST选项值将不会删除日志文件，而是将回滚日志文件的头部清零，从而避免了文件删除所带的磁盘开销。再有就是OFF选项值，该值将指示SQLite在开始事务时不产生回滚日志文件，这样一旦出现系统故障，SQLite也无法再保障数据库数据的一致性。

2. 主数据库日志
> 在SQLite中，如果事务的操作作用于多个数据库，即通过ATTACH命令附加到当前连接中的数据库，那么SQLite将生成主数据库日志文件以保证事务产生的改变在多个数据库之间保持原子性。和回滚日志文件一样，主数据库日志文件也位于当前连接中主数据库文件所处的目录内，其文件名格式为：主数据库文件名+ 随机的后缀。在该文件中，将包含所有当前事务将会改变的Attached数据库的名字。在事务被提交之后，此文件亦被SQLite随之删除。主数据库日志文件只有在某一事务同时操作多个数据库时(主数据库和Attached数据库)才有可能被创建。通过该文件，SQLite可以实现跨多个数据库的事务原子性，否则，只能简单的保证每个单一的数据库内的状态一致性。换句话说，如果该事务在执行的过程中出现系统崩溃或主机宕机的现象，在进行数据恢复时，若没有该文件的存在，将会导致部分SQLite数据库处于提交状态，而另外一部分则处于回滚状态，因此该事务的一致性将被打破。

3. SQL语句日志
> 在一个较大的事务中，SQLite为了保证部分数据在出现错误时可以被正常回滚，所以在事务开始时创建了SQL语句日志文件。比如，update语句修改了前50条数据，然而在修改第51条数据时发现该操作将会破坏某字段的唯一性约束，最终SQLite将不得不通过该日志文件回滚已经修改的前50条数据。SQL语句日志文件只有在INSERT或UPDATE语句修改多行记录时才有可能被创建，与此同时，这些操作还极有可能会打破某些约束并引发异常。但是如果INSERT或UPDATE语句没有被包含在BEGIN...COMMIT中，同时也没有任何其它的SQL语句正在当前的连接上运行，在这种情况下，SQLite将不会创建SQL语句日志文件，而是简单的通过回滚日志来完成部分数据的UNDO操作。和上面两种临时文件不同的是，SQL语句日志文件并不一定要存储在和数据库文件相同的目录下，其文件名也是随机生成。该文件所占用的磁盘空间需要视UPDATE或INSERT语句将要修改的记录数量而定。在事务结束后，该文件将被自动删除。

4. 临时数据库文件
> 当使用"CREATE TEMP TABLE"语法创建临时数据表时，该数据表仅在当前连接内可见，在当前连接被关闭后，临时表也随之消失。然而在生命期内，临时表将连同其相关的索引和视图均会被存储在一个临时的数据库文件之内。该临时文件是在第一次执行"CREATE TEMP TABLE"时即被创建的，在当前连接被关闭后，该文件亦将被自动删除。最后需要说明的是，临时数据库不能被执行DETACH命令，同时也不能被其它进程执行ATTACH命令。

5. 视图和子查询的临时持久化文件
> 在很多包含子查询的查询中，SQLite的执行器会将该查询语句拆分为多个独立的SQL语句，同时将子查询的结果持久化到临时文件中，之后在基于该临时文件中的数据与外部查询进行关联，因此我们可以称其为物化子查询。通常而言，SQLite的优化器会尽力避免子查询的物化行为，但是在有些时候该操作是无法避免的。该临时文件所占用的磁盘空间需要依赖子查询检索出的数据数量，在查询结束后，该文件将被自动删除。见如下示例：
SELECT * FROM ex1 WHERE ex1.a IN (SELECT b FROM ex2);
在上面的查询语句中，子查询SELECT b FROM ex2的结果将会被持久化到临时文件中，外部查询在运行时将会为每一条记录去检查该临时文件，以判断当前记录是否出现在临时文件中，如果是则输出当前记录。显而易见的是，以上的行为将会产生大量的IO操作，从而显著的降低了查询的执行效率，为了避免临时文件的生成，我们可以将上面的查询语句改为：
SELECT * FROM ex1 WHERE EXISTS(SELECT 1 FROM ex2 WHERE ex2.b=ex1.a);
对于如下查询语句，如果SQLite不做任何智能的rewrite操作，该查询中的子查询也将会被持久化到临时文件中，如：
SELECT * FROM ex1 JOIN (SELECT b FROM ex2) AS t ON t.b=ex1.a;
在SQLite自动将其修改为下面的写法后，将不会再生成临时文件了，如：
SELECT ex1.*, ex2.b FROM ex1 JOIN ex2 ON ex2.b=ex1.a;

6. 临时索引文件
> 当查询语句包含以下SQL从句时，SQLite为存储中间结果而创建了临时索引文件，如：
> - ORDER BY或GROUP BY从句。
> - 聚集查询中的DISTINCT关键字。
> - 由UNION、EXCEPT和INTERSECT连接的多个SELECT查询语句。
> 需要说明的是，如果在指定的字段上已经存在了索引，那么SQLite将不会再创建该临时索引文件，而是通过直接遍历索引来访问数据并提取有用信息。如果没有索引，则需要将排序的结果存储在临时索引文件中以供后用。该临时文件所占用的磁盘空间需要依赖排序数据的数量，在查询结束后，该文件被自动删除。

7. VACUUM命令使用的临时数据库文件
> VACUUM命令在工作时将会先创建一个临时文件，然后再将重建的整个数据库写入到该临时文件中。之后再将临时文件中的内容拷贝回原有的数据库文件中，最后删除该临时文件。该临时文件所占用的磁盘空间不会超过原有文件的尺寸。

## 二、相关的编译时参数和指令

> 对于SQLite来说，回滚日志、主数据库日志和SQL语句日志文件在需要的时候SQLite都会将它们写入磁盘文件，但是对于其它类型的临时文件，SQLite是可以将它们存放在内存中以取代磁盘文件的，这样在执行的过程中就可以减少大量的IO操作了。要完成该优化主要依赖于以下三个因素：

1. 编译时参数SQLITE_TEMP_STORE
> 该参数是源代码中的宏定义(#define)，其取值范围是0到3(缺省值为1)，见如下说明：
> - 等于0时，临时文件总是存储在磁盘上，而不会考虑pragma temp_store指令的设置。
> - 等于1时，临时文件缺省存储在磁盘上，但是该值可以被pragma temp_store指令覆盖。
> - 等于2时，临时文件缺省存储在内存中，但是该值可以被pragma temp_store指令覆盖。
> - 等于3时，临时文件总是存储在内存中，而不会考虑pragma temp_store指令的设置。

2. 运行时指令pragma temp_store
> 该指令的取值范围是0到2(缺省值为0)，在程序运行时该指令可以被动态的设置，见如下说明：
> - 等于0时，临时文件的存储行为完全由SQLITE_TEMP_STORE编译期参数确定。
> - 等于1时，如果编译期参数SQLITE_TEMP_STORE指定使用内存存储临时文件，那么该指令将覆盖这一行为，使用磁盘存储。
> - 等于2时，如果编译期参数SQLITE_TEMP_STORE指定使用磁盘存储临时文件，那么该指令将覆盖这一行为，使用内存存储。
> pragma temp_store_directory用于更改磁盘日志位置。
> 当改变temp_store设置，所有已存在的临时表、索引、触发器及视图将被立即删除。

3. 临时文件的大小
> 对于以上两个参数，都有参数值表示缺省情况是存储在内存中的，只有当临时文件的大小超过一定的阈值后才会根据一定的算法，将部分数据写入到磁盘中，以免临时文件占用过多的内存而影响其它程序的执行效率。最后在重新赘述一遍，SQLITE_TEMP_STORE编译期参数和pragma temp_store运行时指令只会影响除回滚日志和主数据库日志之外的其它临时文件的存储策略。换句话说，回滚日志和主数据库日志将总是将数据写入磁盘，而不会关注以上两个参数的值。

## 三、其它优化策略

> 在SQLite中由于采用了Page Cache的缓冲优化机制，因此即便临时文件被指定存储在磁盘上，也只有当该文件的大小增长到一定的尺寸后才有可能被SQLite刷新到磁盘文件上，在此之前它们仍将驻留在内存中。这就意味着对于大多数场景，如果临时表和临时索引的数据量相对较少，那么它们是不会被写到磁盘中的，当然也就不会有IO事件发生。只有当它们增长到内存不能容纳的时候才会被刷新到磁盘文件中的。其中SQLITE_DEFAULT_TEMP_CACHE_SIZE编译期参数可以用于指定临时表和索引在占用多少Cache Page时才需要被刷新到磁盘文件，该参数的缺省值为500页。