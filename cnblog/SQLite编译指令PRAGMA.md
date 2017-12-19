---
title: SQLite编译指令PRAGMA
date: 2012-09-09 21:31:30
tags: database
---

[原文地址](http://www.cnblogs.com/liangxiaxu/archive/2012/09/09/2677361.html)

[TOC]

## SQLite支持的编译指令(pragma)

> PRAGMA命令是用于修改SQLite库或查询SQLite库内部数据(non-table)的特殊命令。
> PRAGMA命令使用与其它SQLite命令(e.g. SELECT, INSERT)相同的接口，但在如下重要方面与其它命令不同:

> - 在未来的SQLite版本中部分pragma可能被删除或添加，小心使用。

> - 当使用未知的pragma语句时不产生报错。未知的pragma仅仅会被忽略，即是说若是打错了pragma语句SQLite不会提示用户。

> - 一些pragma在SQL编译阶段生效而非执行阶段。即是说若使用C语言的sqlite3_compile(), sqlite3_step(), sqlite3_finalize() API (或类似的封装接口中)，pragma可能在调用sqlite3_compile()期间起作用。

> - pragma命令不与其它SQL引擎兼容。

<!--more-->

> 可用的pragma命令有如下四个基本类型：

> - 用于察看当前数据库的模式。

> - 用于修改SQLite库的操作或查询当前的操作模式。

> - 用于查询或修改两个数据库的版本号，schema-version和user-version.

> - 用于调试库和校验数据库文件。

> PRAGMA命令语法

> sql-statement::=PRAGMA name [= value] | PRAGMA function(arg)

> 使用整数值value的pragma也可以使用符号表示，字符串"on"，"true"，和"yes"等同于1，"off"，"false"和"no"等同于0。这些字符串大小写不敏感且无须进行引用。无法识别的字符串被当作1且不会报错。value返回时是整数。

## 用于修改SQLite库的操作的pragma 

- PRAGMA auto_vacuum;

> PRAGMA auto_vacuum = 0 | 1;

> 查询或设置数据库的auto-vacuum标记。
> 正常情况下，当提交一个从数据库中删除数据的事务时，数据库文件不改变大小。未使用的文件页被标记并在以后的添加操作中再次使用。这种情况下使用VACUUM命令释放删除得到的空间。
> 当开启auto-vacuum，当提交一个从数据库中删除数据的事务时，数据库文件自动收缩， (VACUUM命令在auto-vacuum开启的数据库中不起作用)。数据库会在内部存储一些信息以便支持这一功能，这使得 数据库文件比不开启该选项时稍微大一些。
> 只有在数据库中未建任何表时才能改变auto-vacuum标记。试图在已有表的情况下修改不会导致报错。

- PRAGMA cache_size;

> PRAGMA cache_size = Number-of-pages;

> 查询或修改SQLite一次存储在内存中的数据库文件页数。每页使用约1.5K内存，缺省的缓存大小是2000。若需要使用改变大量多行的UPDATE或DELETE命令，并且不介意SQLite使用更多的内存的话，可以增大缓存以提高性能。
> 当使用pragma cache_size改变缓存大小时，改变仅对当前对话有效，当数据库关闭重新打开时缓存大小恢复到缺省大小。要想永久改变缓存大小，使用pragma default_cache_size = Number-of-pages;

- PRAGMA case_sensitive_like;

> PRAGMA case_sensitive_like = 0 | 1;

> LIKE运算符的缺省行为是忽略latin1字符的大小写。因此在缺省情况下'a' LIKE 'A'的值为真。可以通过打开 case_sensitive_like pragma来改变这一缺省行为。当启用case_sensitive_like，'a' LIKE 'A'为假而 'a' LIKE 'a'依然为真。

- PRAGMA count_changes;

> PRAGMA count_changes = 0 | 1;

> 查询或更改count-changes标记。正常情况下INSERT, UPDATE和DELETE语句不返回数据。 当开启count-changes，以上语句返回一行含一个整数值的数据——该语句插入，修改或删除的行数。 返回的行数不包括由触发器产生的插入，修改或删除等改变的行数。

- PRAGMA default_cache_size;

> PRAGMA default_cache_size = Number-of-pages;

> 查询或修改SQLite一次存储在内存中的数据库文件页数。每页使用约1.5K内存，它与cache_sizepragma类似，只是它永久性地改变缓存大小。 利用该pragma，你可以设定一次缓存大小，并且每次重新打开数据库时都继续使用该值。

- PRAGMA default_synchronous;

> 该语句在2.8版本中可用，但在3.0版中被去掉了。这条pragma很危险且不推荐使用，安全起见在该文档中不涉及此pragma的用法。

- PRAGMA empty_result_callbacks;

> PRAGMA empty_result_callbacks = 0 | 1;

> 查询或更改empty-result-callbacks标记。
> empty-result-callbacks标记仅仅影响sqlite3_exec API函数。正常情况下，empty-result-callbacks标记清空， 则对返回0行数据的命令不调用sqlite3_exec()的回叫函数，当设置了empty-result-callbacks，则调用回叫 函数一次，置第三个参数为0 (NULL).这使得使用sqlite3_exec() API的程序即使在一条查询不返回数据时依然检索字段名。

- PRAGMA encoding;

> PRAGMA encoding = "UTF-8";
> PRAGMA encoding = "UTF-16";
> PRAGMA encoding = "UTF-16le";
> PRAGMA encoding = "UTF-16be";

> 在第一种形式中，若主数据库已创建，这条pragma返回主数据库使用得文本编码格式，为 "UTF-8", "UTF-16le" (little-endian UTF-16 encoding) 或者"UTF-16be" (big-endian UTF-16 encoding)中的一种。 若主数据库未创建，返回值为当前会话创建的主数据库将要使用的文本编码格式。
> 第二种及以后几种形式只在主数据库未创建时有效。这时该pragma设置当前会话创建的主数据库将要使用的文本编码格式。 "UTF-16"表示"使用本机字节顺序的UTF-16编码"。若这些形式在主数据库创建后使用，将被忽略且不产生任何效果。
> 数据库的编码格式设置后不能够被改变。
> ATTACH命令创建的数据库使用与主数据库相同的编码格式。

- PRAGMA full_column_names;

> PRAGMA full_column_names = 0 | 1;

> 查询或更改the full-column-names标记。该标记影响SQLite命名SELECT语句(当字段表达式为表-字段或通配符"*"时) 返回的字段名的方式。正常情况下，当SELECT语句将两个或多个表连接时，这类结果字段的返回名为___，当SELECT语句查询一个单独的表时， 返回字段名为___。当设置了full-column-names标记，返回的字段名将统一为___不管是否对表进行了连接。
> 若short-column-names和full-column-names标记同时被设置，则使用full-column-names方式。

- PRAGMA fullfsync;

> PRAGMA fullfsync = 0 | 1;

> 查询或更改fullfsync标记。该标记决定是否在支持的系统上使用F_FULLFSYNC同步模式。缺省值为off.截至目前(2006-02-10) 只有Mac OS X 系统支持F_FULLFSYNC.

- PRAGMA page_size;

> PRAGMA page_size = bytes;

> 查询或设置page-size值。只有在未创建数据库时才能设置page-size。页面大小必须是2的整数倍且大于等于512小于等于8192。 上限可以通过在编译时修改宏定义SQLITE_MAX_PAGE_SIZE的值来改变。上限的上限是32768.

- PRAGMA read_uncommitted;

> PRAGMA read_uncommitted = 0 | 1;

> 查询，设置或清除READ UNCOMMITTED isolation(读取未授权的分隔符).缺省的SQLite分隔符等级是SERIALIZABLE. 任何线程或进程可选用READ UNCOMMITTED isolation,但除了共享公共页和schema缓存的连接之间以外的地方也会 使用SERIALIZABLE.缓存共享通过sqlite3_enable_shared_cache() API开启，且只在运行同一线程的连接间有效。缺省情况下缓存共享是关闭的。

- PRAGMA short_column_names;

> PRAGMA short_column_names = 0 | 1;

> 查询或更改the short-column-names标记。该标记影响SQLite命名SELECT语句(当字段表达式为表-字段或通配符"*"时) 返回的字段名的方式。正常情况下，当SELECT语句将两个或多个表连接时， 这类结果字段的返回名为 ，当SELECT语句查询一个单独的表时， 返回字段名为。当设置了full-column-names标记，返回的字段名将统一为 不管是否对表进行了连接。
> 若short-column-names和full-column-names标记同时被设置，则使用full-column-names方式。

- PRAGMA synchronous;

> PRAGMA synchronous = FULL; (2)
> PRAGMA synchronous = NORMAL; (1)
> PRAGMA synchronous = OFF; (0)

> 查询或更改"synchronous"标记的设定。第一种形式(查询)返回整数值。 当synchronous设置为FULL (2), SQLite数据库引擎在紧急时刻会暂停以确定数据已经写入磁盘。 这使系统崩溃或电源出问题时能确保数据库在重起后不会损坏。FULL synchronous很安全但很慢。 当synchronous设置为NORMAL, SQLite数据库引擎在大部分紧急时刻会暂停，但不像FULL模式下那么频繁。 NORMAL模式下有很小的几率(但不是不存在)发生电源故障导致数据库损坏的情况。但实际上，在这种情况 下很可能你的硬盘已经不能使用，或者发生了其他的不可恢复的硬件错误。 设置为synchronous OFF (0)时，SQLite在传递数据给系统以后直接继续而不暂停。若运行SQLite的应用程序崩溃， 数据不会损伤，但在系统崩溃或写入数据时意外断电的情况下数据库可能会损坏。另一方面，在synchronous OFF时，一些操作可能会快50倍甚至更多。
> 在SQLite 2中，缺省值为NORMAL.而在3中修改为FULL.

- PRAGMA temp_store;

> PRAGMA temp_store = DEFAULT; (0)
> PRAGMA temp_store = FILE; (1)
> PRAGMA temp_store = MEMORY; (2)

> 查询或更改"temp_store"参数的设置。当temp_store设置为DEFAULT (0),使用编译时的C预处理宏 TEMP_STORE来定义储存临时表和临时索引的位置。当设置为MEMORY (2)临时表和索引存放于内存中。 当设置为FILE (1)则存放于文件中。temp_store_directory pragma 可用于指定存放该文件的目录。当改变temp_store设置，所有已存在的临时表，索引，触发器及视图将被立即删除。
> 库中的编译时C预处理标志TEMP_STORE可以覆盖该pragma设置。下面的表给出TEMP_STORE预处理宏和 temp_store pragma交互作用的总结：

|TEMP_STORE|PRAGMA temp_store|临时表和索引使用的存储方式|
|:--|:--|:--|
|0  |any|文件| 
|1  |0  |文件| 
|1  |1  |文件|
|1  |2  |内存|
|2  |0  |内存|
|2  |1  |文件|
|2  |2  |内存|
|3  |any|内存|

- PRAGMA temp_store_directory;

> PRAGMA temp_store_directory = 'directory-name';

> 查询或更改"temp_store_directory"设置——存储临时表和索引的文件所在的目录。 仅在当前连接有效，在建立新连接时重置为缺省值。
> 当改变了temp_store_directory设置，所有已有的临时表，索引，触发器，视图会被直接删除。 建议在数据库一打开时就设置好temp_store_directory.
> directory-name需用单引号引起来。要想恢复缺省目录，把directory-name设为空字符串。例如 PRAGMA temp_store_directory = ''.若directory-name未找到或不可写会引发错误。
> 临时文件的缺省目录与主机的系统有关，使用Unix/Linux/OSX系统的主机，缺省目录是如下序列之中第一个可写的 /var/tmp, /usr/tmp, /tmp,current-directory.对于Windows NT,缺省目录由Windows决定，一般为 C:\Documents and Settings\user-name\Local Settings\Temp\. SQLite创建的临时文件在使用完毕时就被unlink,所以操作系统可以在SQLite进程进行中自动删除临时文件。 于是，正常情况下不能通过ls 或 dir命令看到临时文件。

## 用于查询数据库的schema的pragma

- PRAGMA database_list;

> 对每个打开的数据库，使用该数据库的信息调用一次回叫函数。使用包括附加的数据库名和索引名在内的参数。第一行用于主数据库，第二行用于存放临时表的临时数据库。

- PRAGMA foreign_key_list(table-name);

> 对于参数表中每个涉及到字段的外键，使用该外键的信息调用一次回叫函数。每个外键中的每个字段都将调用一次回叫函数。

- PRAGMA index_info(index-name);

> 对该索引涉及到的每个字段，使用字段信息(字段名，字段号)调用一次回叫函数。

- PRAGMA index_list(table-name);

> 对表中的每个索引，使用索引信息调用回叫函数。参数包括索引名和一个指示索引是否唯一的标志。

- PRAGMA table_info(table-name);

> 对于表中的每个字段，使用字段信息(字段名，数据类型，可否为空，缺省值)调用回叫函数。

## 用于查询/更改版本信息的pragma

- PRAGMA [database.]schema_version;

> PRAGMA [database.]schema_version = integer ;

- PRAGMA [database.]user_version;

> PRAGMA [database.]user_version = integer ;

> 这两条pragma分别用于设置schema-version和user-version的值。schema-version 和user-version均为32位有符号整数，存放于数据库头中。
> schema-version通常只由SQLite内部操作。每当数据库的schema改变时(创建或撤消表或索引)，SQLite 将这个值增大。schema版本在每一次query被执行时被SQLite所使用，以确定编译SQL query时内部cache的schema与编译后的query实际执行时数据库的schema相匹配。使用"PRAGMA schema_version"更改schema-version会破坏这一机制，有导致程序崩溃或数据库损坏的潜在危险。请小心使用！
> user-version不在SQLite内部使用，任何程序可以用它来做任何事。

## 用于库debug的pragma

- PRAGMA integrity_check;

> 该命令对整个数据库进行完整性检查，查找次序颠倒的记录，丢失的页，残缺的记录以及损坏的索引。若发现任何问题则返回一形容问题所在的字符串，若一切正常返回"ok".

- PRAGMA parser_trace = ON; (1) && PRAGMA parser_trace = OFF; (0)

> 打开或关闭SQLite库中的SQL语法分析追踪，用于debug.只有当SQLite不使用NDEBUG宏进行编译时该pragma才可用。

- PRAGMA vdbe_trace = ON; (1) && PRAGMA vdbe_trace = OFF; (0)

> 打开或关闭SQLite库中的虚拟数据库引擎追踪，用于debug.更多信息，察看 VDBE文档。

- PRAGMA vdbe_listing = ON; (1) && PRAGMA vdbe_listing = OFF; (0)

> 打开或关闭虚拟机程序列表，当开启列表功能，整个程序的内容在执行前被打印出来，就像在每条语句之前自动执行EXPLAIN. 语句在打印列表之后正常执行。用于debug.更多信息，查看VDBE文档。
