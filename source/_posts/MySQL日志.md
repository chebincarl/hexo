---
title: MySQL日志
date: 2018-11-28 22:31:13
tags: MySQL
---

MySQL日志记录了MySQL服务实例的运行轨迹以及数据库用户使用、操作数据库的历史信息。MySQL事件（event）是能够自动执行或者定期执行的任务。本章首先详细讲解 MySQL 日志的功能以及使用方法，然后讲解MySQL事件的用法。通过本章的学习，读者可以掌握利用MySQL日志与事件维护、优化数据库的相关技术。

<!--more-->
### MySQL日志
MySQL提供了表11-1所示的几种日志，供数据库管理员深入了解MySQL服务实例的历史状态，帮助数据库管理员追踪数据库曾经发生过的各种事件。默认情况下，所有的MySQL日志以文件方式存储，存放在数据库根目录下。

{% asset_img 1.png %}

#### 数据皆需要缓存
 
众所周知，系统读取数据时，从内存中读取的速度比从外存（例如硬盘）中读取的速度要快百倍。故现在绝大部分系统软件都会最大程度地使用缓存机制（缓存是内存中的一段存储区域），提高数据的访问效率，MySQL也不例外。MySQL会最大限度地利用缓存，减少I/O请求次数，缩短寻道时间，提高数据访问效率。

{% asset_img 2.png %}

MySQL在执行数据查询语句以及数据更新语句时，会将硬盘的数据、索引载入到内存中的缓存中，所有记录的修改、查询在内存中的缓存中完成，如图11-1所示。记录修改完成后，再将修改结果更新到硬盘，这样不仅可以减少I/O请求次数，同时也可以解决硬盘磁头频繁定位导致硬盘I/O效率低下等问题。图中假设某个应用程序执行若干条update语句，这些update语句影响的是硬盘中第9、10、1、4、200以及5数据块中的记录，并需要将修改结果写入硬盘中，通过缓存机制，可以将这些块排序后再写入硬盘，这样既可以大幅减少硬盘I/O请求次数，又可以解决磁头频繁定位导致I/O效率低下等问题，从而提高数据访问效率。

MySQL日志处理也使用了缓存机制，MySQL日志最初存放在MySQL服务器的内存缓存中，若超过指定的存储容量，内存缓存中的日志则写（或者刷新）到外存，以数据库表或者以文件的方式永久地保存到服务器硬盘中。

#### MySQL错误日志

MySQL错误日志主要用于记录MySQL服务实例每次启动、停止的详细信息，以及MySQL服务实例运行过程中产生的警告或者错误信息。与其他日志不同，MySQL错误日志必须开启，无法关闭。与MySQL错误日志有关的参数包括1个。

log_error：设置了错误日志文件的物理位置（日志所在目录以及日志文件名）。
使用MySQL命令“show variables like 'log_error';”可以查看错误日志文件的物理位置，默认情况下，错误日志文件所在的目录为数据库根目录，错误日志文件名为“主机名.err”（本书使用的主机名为mysql），如下所示。默认情况下，错误日志文件的扩展名为err。
+---------------+--------------------------------------------------------------------+
| Variable_name | Value                                                              |
+---------------+--------------------------------------------------------------------+
| log_error     | D:\Install\phpStudy2018\PHPTutorial\MySQL\data\DESKTOP-G997OST.err |
+---------------+--------------------------------------------------------------------+

当MySQL服务实例意外停止或者无法启动时，可以通过错误日志文件的内容分析产生故障的原因。

注意：MySQL错误日志不会记录所有的错误信息，只有MySQL服务实例运行过程中发生的关键（critical）错误信息才会被记录。

#### MySQL普通查询日志

MySQL 普通查询日志记录了 MySQL 服务实例的所有操作（例如查询语句、更新语句等 SQL语句），无论这些操作是否成功执行。另外，还包含一些其他事件，例如，MySQL客户机与MySQL服务器连接和断开连接的相关信息，无论连接成功还是失败。与 MySQL 普通查询日志有关的参数包括3个。

general_log：设置了普通查询日志是否开启。使用MySQL命令“show variables like 'general_log';”可以查看普通查询日志是否开启，使用 MySQL 命令“set @@global.general_log=1;”可以开启普通查询日志。关闭普通查询日志的方法以此类推，不再赘述。

general_log_file：普通查询日志一旦开启，MySQL服务实例将自动创建普通查询日志文件，general_log_file参数设置了普通查询日志文件的物理位置（日志文件所在目录以及日志文件名）。使用MySQL命令“show variables like 'general_log_file';”可以查看普通查询日志文件的物理位置，如下所示。默认情况下，普通查询日志文件所在的目录为数据库根目录，普通查询日志文件名为“主机名.log”（默认情况下，普通查询日志文件的扩展名为log）。
mysql> show variables like 'general_log_file';
+------------------+--------------------------------------------------------------------+
| Variable_name    | Value                                                              |
+------------------+--------------------------------------------------------------------+
| general_log_file | D:\Install\phpStudy2018\PHPTutorial\MySQL\data\DESKTOP-G997OST.log |
+------------------+--------------------------------------------------------------------+

注意：由于普通查询日志几乎记录了MySQL的所有操作，对于数据访问频繁的数据库服务器而言，如果开启MySQL普通查询日志将大幅降低数据库服务器的性能，因此建议关闭普通查询日志。在特定时期，如果需要跟踪某些特殊的查询语句，可以临时打开普通查询日志。

log_output：设置了普通查询日志以及慢查询日志（慢查询日志稍后介绍）的输出格式，默认值为FILE，表示以文件的形式保存普通查询日志以及慢查询日志的内容，使用命令“show variables like 'log_output';”可以查看日志的输出格式，如下所示。
mysql> show variables like 'log_output';
+---------------+-------+
| Variable_name | Value |
+---------------+-------+
| log_output    | FILE  |
+---------------+-------+

MySQL还允许将普通查询日志以及慢查询日志的内容存储到数据库表中。例如，使用MySQL命令“set @@global.log_output='table';”可以将普通查询日志以及慢查询日志分别存储到mysql系统数据库中的general_log表以及slow_log表中（这两个表的存储引擎为CSV），此后查看新的普通查询日志内容时便可以使用SQL语句“select * from mysql.general_log;”。

#### MySQL慢查询日志
使用MySQL慢查询日志可以有效跟踪“执行时间过长”或者“没有使用索引”的查询语句（包括select语句、update语句、delete语句以及insert语句等），为优化查询语句提供帮助。与普通查询日志的另一个区别在于，慢查询日志只包含成功执行过的查询语句。与MySQL慢查询日志有关的参数包括5个。

slow_query_log：设置了慢查询日志是否开启。使用MySQL命令“show variables like 'slow_query_log';”可以查看慢查询日志是否开启，使用 MySQL 命令“set @@global.slow_query_log=1;”可以开启慢查询日志。关闭慢查询日志的方法以此类推，这里不再赘述。

slow_query_log_file：慢查询日志一旦开启，MySQL 服务实例将自动创建慢查询日志文件，slow_query_log_file参数设置了慢查询日志文件的物理位置（日志所在目录以及日志文件名）。使用MySQL命令“show variables like 'slow_query_log_file';”可以查看慢查询日志文件的物理位置，默认情况下，慢查询日志文件所在的目录为数据库根目录，慢查询日志文件名为“主机名-slow.log”，如下所示。默认情况下，慢查询日志文件与普通查询日志文件的扩展名相同，都是log。
mysql> show variables like 'slow_query_log_file';
+---------------------+-------------------------------------------------------------------------+
| Variable_name       | Value                                                                   |
+---------------------+-------------------------------------------------------------------------+
| slow_query_log_file | D:\Install\phpStudy2018\PHPTutorial\MySQL\data\DESKTOP-G997OST-slow.log |
+---------------------+-------------------------------------------------------------------------+

long_query_time：设置了慢查询的时间阈值，默认值是10秒。数据库管理员可以设置一个阈值，将运行时间大于（不包括等于）该值的所有查询语句记录到慢查询日志文件中。使用MySQL命令“show variables like 'long_query_time';”可以查看该阈值（单位为秒，精确到微秒），如下所示。
mysql> show variables like 'long_query_time';
+-----------------+-----------+
| Variable_name   | Value     |
+-----------------+-----------+
| long_query_time | 10.000000 |
+-----------------+-----------+

说明

@@long_query_time既是全局变量，又是会话变量。也可以通过修改my.ini配置文件的方法设置慢查询的时间阈值，例如，可以在my.ini配置文件的[mysqld]选项组中通过添加“long_query_time=1.000000”将该阈值设置为1秒。

log_queries_not_using_indexes：是否将“没有使用索引的查询语句”记录到慢查询日志中，无论它的执行速度有多快。使用MySQL 命令“show variables like 'log_queries_not_using_indexes';”可以查看该参数的值。默认情况下，log_queries_not_using_indexes的值为OFF（关闭），使用MySQL 命令“set @@global.log_queries_not_using_indexes=1;”可以将该参数的值设置为ON（开启）。开启该参数后，“没有使用索引的查询语句”将被记录到慢查询日志文件中，无论它们的执行速度有多快。

log_output：设置了普通查询日志以及慢查询日志的输出形式，默认值为FILE，有关该参数的其他知识请参看MySQL普通查询日志章节的内容。

##### MySQL慢查询日志的查看
MySQL慢查询日志的查看可以分为两种情形：慢查询日志的输出形式设置为table以及慢查询日志的输出形式设置为file。

情形一：慢查询日志的输出形式设置为table。