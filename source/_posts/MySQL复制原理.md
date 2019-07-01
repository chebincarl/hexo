---
layout: title
title: MySQL复制原理
date: 2019-06-30 21:22:53
tags: MySQL
---
思考并回答以下问题：
1.如何创建一个空表？

<!--more-->

如果正确使用 MySQL 的复制，那么它就是一个非常有用的工具 ；但如果复制出现故障，
或配置和使用不当，就会相当令人头疼。本章从一个简单的配置开始，涵盖 MySQL 复
制的基础内容，然后介绍一些基本技巧以丰富你的“复制工具箱”。
本章包括以下复制用例。
通过热备份达到高可用性
如果服务器宕机，一切都将停止 ：不能执行（可能很关键的）事务， 无法得到用户
信息，也不能检索其他重要数据。要不惜（几乎）一切代价避免这种情况发生，因
为它会严重破坏业务。最简单的方法就是配置一个额外的服务器专门作为热备份（hot
standby），在主服务器宕机的时候随时接管业务。
23
24
复制的基本步骤 ｜ 19
产生报表
直接用服务器上的数据创建报表将大大降低服务器的性能，在某些情况下尤其显著。
如果产生报表需要大量的后台作业，最好创建一个额外的服务器来运行这些作业。
停止报表数据库上的复制，然后在不影响主要业务服务器的情况下运行大量查询，
从而得到数据库在某一特定时间的快照。例如，如果在每天最后一个事务处理完毕
后停止复制，可以提取日报表而其他业务仍正常运转。
调试和审计
还可以审查服务器上的查询。例如，查看某些查询是否有性能问题，以及服务器是
否由于某个糟糕的查询而不同步。
复制的基本步骤
本章将介绍一些最大化复制的效率和价值的尖端技术。首先我们要搭建一个如图 3-1 所
示的简单复制，即单一的主从复制实例。这里不需要了解内部架构或复制过程的执行细
节（在介绍更复杂的方案之前，我们先探讨这些内容）。
图3-1：简单的复制
建立基本的复制可以总结为以下三个简单步骤 ：
1. 配置一个服务器作为 master。
2. 配置一个服务器作为 slave。
3. 将 slave 连接到 master。
除非你从一开始就规划了复制且正确配置了 my.cnf 文件，否则步骤 1 和步骤 2 要求必须
重启每个服务器。
25
20 ｜ 第3章 MySQL复制原理
执行这些步骤最简单的方法是拥有一个可以修改 my.cnf 文件权限的 shell 账号
和一个具有 ALL 权限的服务器账号。 注 1
必须严格限制在生产环境中授予权限。准确的指导方法请查阅本章后面的“配
置复制的用户权限”。
配置 master
将服务器配置为 master，要确保该服务器有一个活动的二进制日志（binary log）和唯
一的服务器 ID。我们后面再仔细研究二进制日志，现在只要知道二进制日志上保存了
master 上的所有变更记录，并且可以在 slave 上重新执行。服务器 ID 用于区分服务器。
要创建二进制日志和服务器 ID，你需要停掉服务器，然后按示例 3-1 所示将 log-bin 、
log-bin-index 和 server-id 选项添加到 my.cnf 配置文件。加粗部分为添加的配置选项。
示例3-1：向my.cnf添加选项以配置master
[mysqld]
user = mysql
pid-file = /var/run/mysqld/mysqld.pid
socket = /var/run/mysqld/mysqld.sock
port = 3306
basedir = /usr
datadir = /var/lib/mysql
tmpdir = /tmp
log-bin = master-bin
log-bin-index = master-bin.index
server-id = 1
log-bin 选项给出了二进制日志产生的所有文件的基本名（稍后你将看到，二进制日志
包含多个文件）。如果你创建了一个以 log-bin 为基本名的扩展文件名，该扩展名将被忽
略，而只使用文件的基本名（也就是没有扩展的文件名）。
log-bin-index 选项给出了二进制索引文件的文件名，这个索引文件保存了所有 binlog
文件的列表。
严格地说，不需要为 log-bin 选项提供值，其默认值是 hostname-bin 。 hostname 的值来
自 pid-file 选项，默认值是主机名（可以通过 gethostname(2) 系统调用得到）。如果管
理员后来修改了主机名，binlog 文件名也会随之改变，但是索引文件仍可以获取正确的值。
最好为 MySQL 服务器创建一个机器无关的唯一的服务器名，因 binlog 文件序列中途改
名可能会很混乱。
如果没有为 log-bin-index 赋予任何值，其默认值与 binlog 文件的基本名相同（如果没
有为 log-bin 提供值，则默认为 hostname-bin ）。也就是说，如果你不赋值给 log-bin-
注 1　在 Windows 中，命令行工具（CMD）或 PowerShell 就相当于 UNIX 中的 shell。
26
复制的基本步骤 ｜ 21
index ，索引文件名会随主机名的改变而改变。所以，如果你改变主机名然后重启服务器，
将找不到索引文件，从而认为索引文件不存在，导致二进制日志为空。
每个服务器都有一个唯一的服务器 ID，所以如果一个 slave 连接了 master，并且其
server-id 的参数值与 master 相同，则会报 master 和 slave 服务器 ID 相同的错误。
设置完配置文件后重启服务器，然后添加一个复制用户，这样便完成了配置。
修改 master 的配置文件以后，重启 master，使配置生效。
slave 启动一个标准的客户端连到 master，并请求 master 将所有的变更发送给它。slave
连接时要求 master 上有一个特殊复制权限的用户。示例 3-2 展示了 master 上的一个标准
的 mysql 客户端会话，通过命令创建新用户并赋予适当的权限。
示例3-2：在master上创建一个复制用户
master> CREATE USER repl_user;
Query OK, 0 rows affected (0.00 sec)
master> GRANT REPLICATION SLAVE ON *.*
-> TO repl_user IDENTIFIED BY 'xyzzy';
Query OK, 0 rows affected (0.00 sec)
REPLICATION SLAVE 权限并没有什么特别之处，不过拥有这个权限的用户能
够获取 master 上的二进制日志。完全可以给一个常规用户赋予 REPLICATION
SLAVE 权限，但最好还是将复制 slave 用户与其他用户区别开来。这样的话，
以后如果想禁止某些 slave 的连接，只要删除该用户就可以了。
配置 slave
配置完 master 后，还需要配置 slave。与 master 一样，需要为每个 slave 分配一个唯一的
服务器 ID。分别使用 relay-log 和 relay-log-index 选项向 my.cnf 文件添加中继日志文
件（relay log file）和中继日志索引文件（relay log index file）的文件名（我们将在第 8
章的“复制架构基础”小节中详细讨论中继日志）。示例 3-3 给出了推荐的配置选项，其
中新添加的选项加粗表示。
示例3-3：向my.cnf文件添加选项来配置slave
[mysqld]
user = mysql
pid-file = /var/run/mysqld/mysqld.pid
socket = /var/run/mysqld/mysqld.sock
port = 3306
27
22 ｜ 第3章 MySQL复制原理
basedir = /usr
datadir = /var/lib/mysql
tmpdir = /tmp
server-id = 2
relay-log-index = slave-relay-bin.index
relay-log = slave-relay-bin
与 log-bin 和 log-bin-index 选项一样， relay-log 和 relay-log-index 选项的默认值
取决于 hostname。 relay-log 的默认值是 hostname-relay-bin ， relay-log-index 的默
认值是 hostname-relay-bin.index 。使用默认值有一个问题，即一旦服务器的主机名改
变，将会因为无法找到中继日志索引文件而认为中继日志文件为空。
修改 my.cnf 文件以后，重启 slave，使配置生效。
配置复制的用户权限
要把 slave 连接到 master 做复制，除了需要一个能访问关键文件的 shell 账户，还
要有一个具有特定权限的账户。出于安全原因，通常要严格控制这个账号只具备
配置 master 和 slave 所需的必要权限。为了创建和删除用户，这个账号需要具备
CREATE USER 权限。为了将 REPLICATION SLAVE 权限赋予复制账号，还需要具有
GRANT OPTION 的 REPLICATION SLAVE 权限。
要进一步执行复制相关的工作（本章稍后将介绍），还需要更多选项 ：
y 执行 FLUSH LOGS 命令（或任何 FLUSH 命令）需要 RELOAD 权限。
y 执行 SHOW MASTER STATUS 和 SHOW SLAVE STATUS 命令需要 SUPER 或 REPLICATION
CLIENT 权限。
y 执行 CHANGE MASTER TO 命令需要 SUPER 权限。
举个例子，向用户 mats 赋予足够的权限以完成本章中的所有工作，使用如下命令：
server> GRANT REPLICATION SLAVE, RELOAD, CREATE USER, SUPER
-> ON *.*
-> TO mats@'192.168.2.%'
-> WITH GRANT OPTION;
连接 master 和 slave
现在创建基本的复制只剩下最后一步了 ：将 slave 定向到 master，让它知道从哪里进行
复制。为此，我们需要知道 master 的 4 部分信息 ：
28
二进制日志简介 ｜ 23
y 主机名
y 端口号
y 具备 REPLICATION SLAVE 权限的用户账号
y 该用户账号的密码
在配置 master 的时候，我们已经创建了一个具备适当权限的用户账号及密码。主机名由
操作系统确定，无法在 my.cnf 文件中配置，但端口号可以在 my.cnf 中分配（如果没有指
定端口号，将使用默认值 3306）。创建和运行复制的最后两步是 ：使用 CHANGE MASTER
TO 命令将 slave 定向到 master，然后用 START SLAVE 启动复制 ：
slave> CHANGE MASTER TO
-> MASTER_HOST = 'master-1',
-> MASTER_PORT = 3306,
-> MASTER_USER = 'repl_user',
-> MASTER_PASSWORD = 'xyzzy';
Query OK, 0 rows affected (0.00 sec)
slave> START SLAVE;
Query OK, 0 rows affected (0.15 sec)
恭喜！你已经建立了第一个 master 与 slave 之间的复制。如果你在 master 的数据库上做
一些改动，例如创建新表并填充数据，将发现这些变更都会复制到 slave。试试吧！建立
一个测试数据库（如果还没有的话），再创建一些表，然后添加一些数据到表中，看看
这些变更是否会复制到 slave 中。
注意， MASTER_HOST 参数的值可以是主机名或 IP 地址。如果是主机名，则可以通过调用
gethostname(3) 得到对应的 IP 地址，即通过域名查找来解析主机名，其结果与配置有关。
配置域名解析的步骤不在本书讨论的范围内。
二进制日志简介
复制过程需要使用二进制日志（binary log，或者 binlog），它记录了服务器数据库上的
所有变更。你需要理解二进制日志是如何控制复制过程或解决问题的，所以本节我们将
介绍一些背景知识。
图 3-2 展示了复制架构示意图，图中包括 master、二进制日志和 slave，其中 slave 通过
二进制日志获取 master 上的变更。我们将在第 8 章详细阐述复制架构。当某条语句即将
执行结束时，将在二进制日志的末尾写入一条记录，同时通知语句解析器语句已经执行
完毕。
29
24 ｜ 第3章 MySQL复制原理
图3-2：二进制日志在复制中的作用
通常只有即将执行完毕的语句才会被写入二进制日志，但是在一些特殊情况下可能会写
入其他信息，包括语句的附加信息或者直接代替语句被写入。不久你就会知道这么做的
原因，但是目前我们假设只有执行的语句才会写入二进制日志。
二进制日志记录了什么
二进制日志的作用是记录数据库中表的更改，然后用于复制和 PITR（即时恢复，将在第
15 章中讨论），另外少数审计情况下也会用到。
注意，二进制日志只包括数据库的改动，所以对那些不改变数据的语句则不会写入二进
制日志。
从传统意义上说，MySQL 复制记录了产生变化的 SQL 语句，称为基于语句的复制
（statement-based replication）。由于基于语句的复制会在 slave 上重新执行语句，如果
master 和 slave 的上下文环境不完全一致的话，可能导致 slave 上的结果与 master 不同。
所以在 5.1 版本中，MySQL 还提供了基于行的复制（row-based replication）。相比基于
语句的复制，基于行的复制将每一条记录记为二进制日志中的一行。基于行的复制不仅
更加方便，而且有时候速度更快。
考虑一个含有多表连接或 WHERE 条件的复杂更新，看看这两种复制有什么不同。你真正
需要知道的是更新后数据的状态，而不是像基于语句的复制那样把所有的逻辑都在 slave
上重新执行。另一方面，如果某个更新改变了 10 000 行，你可能宁愿只记录这个语句，
而不是像基于行的复制那样去记录 10 000 个单独的改动。
我们将在第 8 章涉及基于行的复制，介绍它的实现和使用。下面的例子重点讲解基于语
句的复制，这样更容易理解数据库的执行。
30
二进制日志简介 ｜ 25
观察复制的动作
用上一节中的复制例子，我们来看一些简单语句的 binlog 事件。先使用命令行客户端连
接 master，然后执行一些命令获取二进制日志 ：
master> CREATE TABLE tbl (text TEXT);
Query OK, 0 rows affected (0.04 sec)
master> INSERT INTO tbl VALUES ("Yeah! Replication!");
Query OK, 1 row affected (0.00 sec)
master> SELECT * FROM tbl;
+--------------------+
| text |
+--------------------+
| Yeah! Replication! |
+--------------------+
1 row in set (0.00 sec)
master> FLUSH LOGS;
Query OK, 0 rows affected (0.28 sec)
FLUSH LOGS 命令强制轮换（rotate）二进制日志，从而得到一个“完整的”二进制日志文件。
使用 SHOW BINGLOG EVENTS 命令进一步查看该文件，如示例 3-4 所示。
示例3-4：检查二进制日志中有哪些事件
master> SHOW BINLOG EVENTS\G
*************************** 1. row ***************************
Log_name: mysql-bin.000001
Pos: 4
Event_type: Format_desc
Server_id: 1
End_log_pos: 107
Info: Server ver: 5.5.34-0ubuntu0.12.04.1-log, Binlog ver: 4
*************************** 2. row ***************************
Log_name: mysql-bin.000001
Pos: 107
Event_type: Query
Server_id: 1
End_log_pos: 198
Info: use `test`; CREATE TABLE tbl (text TEXT)
*************************** 3. row ***************************
Log_name: mysql-bin.000001
Pos: 198
Event_type: Query
31
26 ｜ 第3章 MySQL复制原理
Server_id: 1
End_log_pos: 266
Info: BEGIN
*************************** 4. row ***************************
Log_name: mysql-bin.000001
Pos: 266
Event_type: Query
Server_id: 1
End_log_pos: 374
Info: use `test`; INSERT INTO tbl VALUES ("Yeah! Replication!")
*************************** 5. row ***************************
Log_name: mysql-bin.000001
Pos: 374
Event_type: Xid
Server_id: 1
End_log_pos: 401
Info: COMMIT /* xid=188 */
*************************** 6. row ***************************
Log_name: mysql-bin.000001
Pos: 401
Event_type: Rotate
Server_id: 1
End_log_pos: 444
Info: mysql-bin.000002;pos=4
6 rows in set (0.00 sec)
这个二进制日志包含 6 个事件 ：一个格式描述事件、三个查询事件，一个 XID 事件，以
及一个日志轮换（rotate）事件。查询事件用于描述如何将数据库上执行的语句写入二进
制日志，XID 事件用于事务管理，而格式描述事件和日志轮换事件则用于在服务器内部
管理二进制日志。第 8 章将详细讨论这些事件，这里我们先简单看一下每个事件所包含
的字段。
Event_type
这是事件的类型。这里我们已经看到了三种类型，但还有很多类型。事件的类型表明
什么样的信息会被传送给 slave。目前在 MySQL 5.1.18 到 5.5.33 版本中，一共有 27
种事件类型（其中有些事件是不使用的，但是为了向后兼容而保留了），到 5.6.12 版
本已经有 35 种事件类型了。这个范围是可扩展的，需要的话可以添加新的事件类型。
Server_id
这是创建事件的服务器的 ID。
Log_name
这是用来存储事件的文件名。一个事件只能存储在一个文件中，永远不能跨两个文件。
32
二进制日志简介 ｜ 27
Pos
这是事件在文件中的开始位置，即事件的第一个字节。
End_log_pos
这是事件在文件中的结束位置，也是下一个事件的开始位置。这个位置比事件的
最后一个字节高一位，因此事件的字节范围为 Pos 到 End_log_pos － 1 。通过计算
End_log_pos － Pos 可以得到这个事件的长度。
Info
这是关于事件信息的可读文本。不同的事件会显示不同的信息，不过至少可以通过
查询事件来知道它所包含的语句。
前两个字段 Log_name 和 Pos 组成了事件的二进制日志位置（binlog position），用于标识
事件的地点或位置。除了以上字段以外，每个事件还包括很多其他信息，例如时间戳，
即从纪元（用经典的 UNIX 时间格式表示，例如 1970-01-01 00:00:00 UTC）开始的秒数。
二进制日志的结构和内容
前面讲过，二进制日志并不是一个单独的文件，而是由一系列易于管理的（例如在不影
响新日志的情况下移除旧的日志）文件组成的。二进制日志包括一组存储实际内容的二
进制日志文件和一个用来跟踪二进制日志文件存储位置的二进制日志索引文件。图 3-3
给出了二进制日志的组织结构。
图3-3：二进制日志的结构
有一个二进制日志文件是活动二进制日志文件（active binlog file），即当前正在被写入
的文件（通常也从这个文件读取）。
每个二进制日志文件都以格式描述事件（format description event）开始，以日志轮换事
33
28 ｜ 第3章 MySQL复制原理
件（rotate event）结束。格式描述日志事件包括产生该文件的服务器版本号、服务器及
二进制日志的信息等。日志轮换事件包含下一个二进制日志文件的名称，以告知二进制
日志继续写入哪个文件。
每个二进制日志文件中有多个二进制日志事件，各个事件之间相互独立，同时也是构成
二进制日志的基本单位。格式描述日志事件还有一个标记，标记二进制日志文件是否正
常关闭。如果正在写入二进制日志文件，则设置该标记 ；如果文件关闭，则清除该标记。
这样就可以检测出在崩溃事件中损坏的二进制日志文件，并允许通过复制进行恢复。
如果在 master 上执行其他语句，你会发现一件奇怪的事情——看不到二进制日志中的变
化 ：
master> INSERT INTO tbl VALUES ("What's up?");
Query OK, 1 row affected (0.00 sec)
master> SELECT * FROM tbl;
+--------------------+
| text |
+--------------------+
| Yeah! Replication! |
| What's up? |
+--------------------+
1 row in set (0.00 sec)
master> SHOW BINLOG EVENTS\G
same as before
新事件哪去了？我们已经知道，二进制日志由若干文件组成，而 SHOW BINLOG EVENTS 语
句只显示第一个二进制日志文件的内容。这与大多数用户的期望相反，他们想看的是活
动二进制日志文件的内容。如果第一个二进制日志的文件名是 master-bin.000001（这个
文件包含前面显示的事件），你可以使用下面的命令查看下一个二进制日志文件（这里
即文件 master-bin.000002）中的事件 ：
master> SHOW BINLOG EVENTS IN 'master-bin.000002'\G
*************************** 1. row ***************************
Log_name: mysql-bin.000002
Pos: 4
Event_type: Format_desc
Server_id: 1
End_log_pos: 107
Info: Server ver: 5.5.34-0ubuntu0.12.04.1-log, Binlog ver: 4
34
二进制日志简介 ｜ 29
*************************** 2. row ***************************
Log_name: mysql-bin.000002
Pos: 107
Event_type: Query
Server_id: 1
End_log_pos: 175
Info: BEGIN
*************************** 3. row ***************************
Log_name: mysql-bin.000002
Pos: 175
Event_type: Query
Server_id: 1
End_log_pos: 275
Info: use `test`; INSERT INTO tbl VALUES ("What's up?")
*************************** 4. row ***************************
Log_name: mysql-bin.000002
Pos: 275
Event_type: Xid
Server_id: 1
End_log_pos: 302
Info: COMMIT /* xid=196 */
4 rows in set (0.00 sec)
你可能已经注意到在示例 3-4 中，二进制日志以日志轮换事件结尾， Info 字段包含下一
个二进制日志文件名和事件的开始位置。使用 SHOW MASTER STATUS 命令查看当前正在写
入的是哪个二进制日志文件 ：
master> SHOW MASTER STATUS\G
*************************** 1. row ***************************
File: master-bin.000002
Position: 205
Binlog_Do_DB:
Binlog_Ignore_DB:
1 row in set (0.00 sec)
查看二进制日志以后，停止并重置 slave，然后删除表 ：
master> DROP TABLE tbl;
Query OK, 0 rows affected (0.00 sec)
slave> STOP SLAVE;
Query OK, 0 rows affected (0.08 sec)
slave> RESET SLAVE;
Query OK, 0 rows affected (0.00 sec)
35
30 ｜ 第3章 MySQL复制原理
接着，删除表，然后重置 master 刷新 ：
master> DROP TABLE tbl;
Query OK, 0 rows affected (0.00 sec)
master> RESET MASTER;
Query OK, 0 rows affected (0.04 sec)
RESET MASTER 命令删除了所有二进制日志文件并清空了二进制日志索引文件。 RESET
SLAVE 命令删除了 slave 上复制用的所有文件，重新开始。
无论是 RESET MASTER 还是 RESET SLAVE 都要求没有活动的复制正在运行。
因此 ：
y 执行 RESET MASTER 命令（在 master 上）时，确保没有 slave 连接
到该 master。
y 执行 RESET SLAVE 命令（在 slave 上）时，先执行 STOP SLAVE 命令，
确保 slave 上没有活动的复制。
本章涵盖了大多数基本事件，而完整详细的事件列表请参考 MySQL 内部手册（http://
bit.ly/mysql-manual）。
建立新 slave
既然你已经对二进制日志有所了解，下面我们要解决之前创建 slave 的过程中存在的一
个基本问题。前面配置 slave 时，我们并没有说明复制从哪里开始，所以 slave 将从头开
始读取 master 上的二进制日志。如果 master 已经运行了一段时间，这显然不是个好方法：
不仅会在 slave 上大量重放事件，还可能导致有些日志无法获取，因为它们可能由于安
全原因存储在其他地方，master 上也没有这些日志信息（第 15 章介绍备份和 PITR 时将
进一步讨论这个）。
所以我们需要另一种方法来建立新的 slave（又称自举 slave），而不是从头开始复制。
这里 CHANGE MASTER TO 命令有两个有用的参数，即： MASTER_LOG_FILE 和 MASTER_LOG_
POS 。（从 MySQL 5.6 开始，又出现了另一种更简便的方式来指定这些位置，包括全局事
务标识符，GTID 等，更多内容参见第 8 章。）使用这些参数指定 master 开始发送事件的
binlog 位置，而不是从头开始。
向 CHANGE MASTER TO 命令添加上述参数，按照以下步骤建立新的 slave ：
1． 配置新的 slave。
36
建立新slave ｜ 31
2． 备份 master （或者备份已经复制了 master 的 slave）。参见第 15 章中的常用备份技术。
3． 记下该备份相应的binlog位置（即产生master当前状态的最后一个事件所在的位置）。
4． 在新 slave 上恢复备份。参见第 15 章中的常用恢复技术。
5． 配置 slave 从这个 binlog 位置开始复制。
根据第 2 步使用的是 master 还是 slave，处理过程略有差异。我们先看只有一个服务器
且作为 master 运行时，如何引导新 slave，这个过程称为克隆 master。
克隆 master 是指获取服务器的快照，通常通过创建备份来完成。服务器的备份技术有很
多，但本章仅使用较为简单的技术，即运行 mysqldump 来创建逻辑备份。其他备份技术
包括 ：通过复制数据库文件创建物理备份，诸如 MySQL 企业备份工具的在线备份技术，
以及使用 Linux 的 LVM（Logical Volume Manager，逻辑卷管理器）进行卷快照等。这
些技术将在第 15 章进行详细介绍，并讨论它们各自的优点。
克隆 master
虽然 mysqldump 工具可以一步完成所有步骤，但为了解释必要的操作，这里我们将单独
执行每个步骤。本节后面会提供一个更紧凑的版本。
如图 3-4 所示，克隆 master 首先要创建 master 的备份。由于 master 可能正在运行，而
且缓存中有很多表，所以需要刷新（flush）所有表并锁定数据库，防止在检查 binlog 位
置之前数据库发生改变。使用 FLUSH TABLES WITH READ LOCK 命令来完成 ：
master> FLUSH TABLES WITH READ LOCK;
Query OK, 0 rows affected (0.02 sec)
图3-4：克隆master来创建新的slave
一旦数据库被锁定，就可以创建备份，并记录 binlog 位置了。注意，这时不应该断开服
37
32 ｜ 第3章 MySQL复制原理
务器连接，因为那会导致锁被释放。由于 master 没有任何改动， SHOW MASTER STATUS
命令将正确返回当前二进制日志文件及其 binlog 位置。我们将在第 8 章详细讨论 SHOW
MASTER STATUS 和 SHOW MASTER LOGS 命令。
master> SHOW MASTER STATUS\G
*************************** 1. row ***************************
File: master-bin.000042
Position: 456552
Binlog_Do_DB:
Binlog_Ignore_DB:
1 row in set (0.00 sec)
下一个事件的写入位置是 master-bin.000042 : 456552 ，这就是复制的起点，这个位置点
之前的所有东西都在备份里。记下 binlog 位置后就可以创建备份了。创建数据库备份最
简单的方法是用 mysqldump ：
$ mysqldump --all-databases --host=master-1 >backup.sql
有了可靠的 master 副本，就可以为数据库中的表解锁，允许数据库继续处理查询。
master> UNLOCK TABLES;
Query OK, 0 rows affected (0.23 sec)
接下来，在 slave 上使用 mysql 实用工具恢复备份 ：
$ mysql --host=slave-1 <backup.sql
已经在 slave 上恢复了 master 的备份，现在可以启动 slave 了。利用前面记下的 master
的 binlog 位置，使用 CHANGE MASTER TO 命令配置 slave，然后启动 slave ：
slave> CHANGE MASTER TO
-> MASTER_HOST = 'master-1',
-> MASTER_PORT = 3306,
-> MASTER_USER = 'slave-1',
-> MASTER_PASSWORD = 'xyzzy',
-> MASTER_LOG_FILE = 'master-bin.000042',
-> MASTER_LOG_POS = 456552;
Query OK, 0 rows affected (0.00 sec)
slave> START SLAVE;
Query OK, 0 rows affected (0.25 sec)
mysqldump 可以自动执行前面的所有步骤。要对 master 服务器上的所有数据库进行逻辑
备份，输入 ：
38
建立新slave ｜ 33
$ mysqldump --host=master -all-databases \
> --master-data=1 >backup-source.sql
--master-data=1 选项使 mysqldump 产生 CHANGE MASTER TO 语句，其参数为二进制日志
文件及其位置（可通过 SHOW MASTER STATUS 得到）。
然后就可以在 slave 上恢复备份了 ：
$ mysql --host=slave-1 <backup-source.sql
注意，只有克隆 master 的时候能用 --master-data=1 自动执行 CHANGE MASTER TO 语句。
后面克隆 slave 的时候，这些步骤都要分开执行。
恭喜！现在你已经克隆了 master，并建立和运行了一个新 slave。根据 master 的负载情况，
你可能需要 slave 从前面记录的位置开始同步，这比从头开始容易得多。
根据备份需要的时间长短不同，可能有大量的数据需要同步，所以，在将 slave 联机
（online）之前，首先要仔细阅读第 6 章的“数据的一致性管理”一节。
克隆 slave
只要有一个 slave 连在 master 上，就可以使用这个 slave 创建新的 slave，而不需要再离
线（offline）master 了。如果数据库很大或访问量较高，那么停机时间可能会相当长，
因为既要考虑创建备份的时间又要考虑 slave 同步的时间。
克隆 slave 的过程如图 3-5 所示，与克隆 master 基本相同，区别在于如何找到 binlog 位置。
另外，需要注意你克隆的那个 slave 同时还在执行从 master 的复制。
图3-5：克隆slave以创建一个新的slave
首先，必须在备份前停止 slave，保证 slave 上不再有变化发生。如果创建备份时复制仍
39
34 ｜ 第3章 MySQL复制原理
在进行，而在备份期间数据库又发生了变化，这时就会得到不一致的备份映像。但是，
如果使用某种在线备份方法，如 MySQL 企业备份工具，则不需要在创建备份前停止
slave。停止 slave 是这样的 ：
original-slave> STOP SLAVE;
Query OK, 0 rows affected (0.20 sec)
slave 停止以后，就可以像从前一样刷新数据表，然后创建备份。创建了 slave 的备份（而
不是 master 的备份）后，使用 SHOW SLAVE STATUS 命令（而不是 SHOW MASTER STATUS ）
来确定从哪里开始复制。该命令会输出很多内容，第 8 章将详细介绍。如果要获得
master 二进制日志中 slave 即将执行的下一个事件的位置，请注意 Relay_Master_Log_
File 和 Exec_Master_Log_Pos 字段的值。
original-slave> SHOW SLAVE STATUS\G
...
Relay_Master_Log_File: master-bin.000042
...
Exec_Master_Log_Pos: 546632
创建备份，然后在新 slave 上恢复备份以后，将复制的起点配置为从这个位置开始，然
后启动新的 slave ：
new-slave> CHANGE MASTER TO
-> MASTER_HOST = 'master-1',
-> MASTER_PORT = 3306,
-> MASTER_USER = 'slave-1',
-> MASTER_PASSWORD = 'xyzzy',
-> MASTER_LOG_FILE = 'master-bin.000042',
-> MASTER_LOG_POS = 546632;
Query OK, 0 rows affected (0.19 sec)
new-slave> START SLAVE;
Query OK, 0 rows affected (0.24 sec)
克隆 master 和克隆 slave 只有一些细节上的区别，这意味着我们的 Python 程序库可以将
这两者合并成一个过程，即在源服务器上创建备份从而创建新的 slave，然后将这个新
slave 连接到 master。
创建备份的常见方法是调用 FLUSH TABLES WITH READ LOCK ，然后在 MySQL 服务器
被读锁锁住的时候创建一份数据库文件的副本。这通常比 mysqldump 快很多，但是在
InnoDB 中使用 FLUSH TABLES WITH READ LOCK 是不安全的！
40
建立新slave ｜ 35
FLUSH TABLES WITH READ LOCK 会锁住表，阻止新事务发生，但后台仍然有一些 FLUSH
TABLES WITH READ LOCK 阻止不了的活动在继续进行。
使用下面的方法可安全地创建 InnoDB 数据表的备份 ：
y 关闭服务器，然后复制文件。如果数据库很大，最好采取这种方法，因为这时使
用 mysqldump 进行数据恢复会很慢。
y 执行 FLUSH TABLES WITH READ LOCK （前面已经介绍过）命令后，使用
mysqldump 工具。读锁保证读取数据的时候不会产生变更。如果有大量数据要读
的话，数据库可能长时间处于被锁的状态。注意，使用 --single-transaction
选项也可以获得一致的快照，但只对 InnoDB 表有效。更多内容参见第 15 章的
“mysqldump 工具”一节。
y 执行 FLUSH TABLES WITH READ LOCK 锁定数据库后，执行某种快照方案，例如
LVM（Linux 平台）或者 ZFS（Zettabyte File System）。
y 使用 MySQL 企业备份工具（或 XtraBackup）做 MySQL 联机备份。
克隆操作的脚本
Python 程序库实现克隆 master 的方法很简单，使用一个 Server 对象代表 master，然后
从这个 master 上复制数据库即可。使用 clone 函数即可实现，见示例 3-6。
克隆 slave 的过程类似，但它是在一台服务器上创建备份，然后新 slave 连接到另一台
服务器进行复制。同时支持 master 和 slave 的克隆操作很容易，只需要使用两个参数 ：
source 参数指定从哪里创建备份， use_master 参数指出备份恢复后 slave 需要连接的服
务器。 clone 方法的调用格式是这样的 ：
clone(slave = slave[1], source = slave[0], use_master = master)
下面要写一些实用工具函数来实现克隆功能，这些函数在其他地方也可以派上用场。示
例 3-5 用到了如下函数 ：
fetch_master_pos
从 master 获取 binlog 位置（即 master 即将写入二进制日志的下一个事件的位置）。
fetch_slave_pos
从 slave 获取 binlog 位置（即从 master 读取的下一个事件的位置）。
replicate_from
需要提供的参数包括 slave、master 及 binlog 位置，将 slave 定向到 master，并从给
定的 binlog 位置开始复制。
41
36 ｜ 第3章 MySQL复制原理
replicate_from 函数从 master 上读取 repl_user 字段获取复制用户的用户名和密码。但
Server 类的定义中并没有该字段，它是服务器注入 Master 角色时添加的字段。
示例3-5：获取服务器的master和slave位置的功能函数
_CHANGE_MASTER_TO = """CHANGE MASTER TO
MASTER_HOST=%s, MASTER_PORT=%s,
MASTER_USER=%s, MASTER_PASSWORD=%s,
MASTER_LOG_FILE=%s, MASTER_LOG_POS=%s"""
def replicate_from(slave, master, position):
slave.sql(_CHANGE_MASTER_TO, (master.host, master.port,
master.repl_user.name,
master.repl_user.passwd,
position.file, position.pos))
def fetch_master_pos(server):
result = server.sql("SHOW MASTER STATUS")
return Position(server.server_id, result["File"], result["Position"])
def fetch_slave_pos(server):
result = server.sql("SHOW SLAVE STATUS")
return Position(server.server_id, result["Relay_Master_Log_File"],
result["Exec_Master_Log_Pos"])
这些都是创建 clone 函数所需要的函数。如果要克隆 slave，应用程序传递一个单独的
use_master 参数，让 clone 函数将新 slave 定向到该 master 做复制。如果要克隆 master，
应用程序则忽略这个 use_master 参数，让 clone 函数使用“source”服务器作为 master。
创建服务器备份的方法有很多，示例 3-6 仅选择了一种方法，即使用 mysqldump 创建服
务器的逻辑备份。稍后我们将演示如何将这个备份过程通用化，使得无论采用任何备份
方法都可以使用相同的基本代码建立新 slave。
示例3-6：克隆master或Slave的函数
def clone(slave, source, use_master = None):
from subprocess import call
backup_file = open(server.host + "-backup.sql", "w+")
if master is not None:
source.sql("STOP SLAVE")
lock_database(source)
if master is None:
position = fetch_master_position(source)
else:
position = fetch_slave_position(source)
call(["mysqldump", "--all-databases", "--host='%s'" % source.host],
42
执行常见的复制任务 ｜ 37
stdout=backup_file)
if master is not None:
start_slave(source)
backup_file.seek() # Rewind to beginning
call(["mysql", "--host='%s'" % slave.host], stdin=backup_file)
if master is None:
replicate_from(slave, source, position)
else:
replicate_from(slave, master, position)
start_slave(slave)
执行常见的复制任务
每个常见的复制案例，包括横向扩展（scale out）、热备份（hot standby）等，都有其实
现细节和可能的陷阱。下面将向读者展示如何执行某些任务，以及如何使 Python 程序库
支持这些任务。
本节中的例子省略了密码。配置某些控制服务器的账户时，可以设置只允许某
些控制部署的特定主机访问（通过创建类似 mats@'192.168.2.136' 这样的
账户），或者也可以给这些命令设置密码。
报表
大多数企业需要大量的例行报告，包括已售物品的周报表，开支和收入的月报表，以及
大量的数据挖掘报表，以发现某些趋势或为营销部门识别重点群体等。
在 master 上运行这些查询会很麻烦。数据挖掘查询可能需要大量的计算资源，拖慢正常
操作，而结果却发现不值得那么做，比如把左撇子作为重点群体。此外，这些报表通常
不是很紧急（与处理日常事务相比），所以没有必要尽快创建报表。换句话说，这些报
表对时间要求不严格，即使一个小时不够，花两个小时完成也没什么关系。
更好的办法是，使用一个空闲的服务器（或者两个，如果真有那么多报表需求的话），搭
建一个到 master 的复制。如果需要做报表的话，就停止复制，执行报表程序，然后再重
启复制，这样完全不会干扰 master。
报表往往需要精确的间隔时间，如汇总当天所有的销售额，需要在适当的时候停止复制，
而不至于将第二天的销售额也计入前一天的报表。然后，没有特定日期或时间的事件发
生，就无法停止 slave，所以必须得想其他的方法。
43
38 ｜ 第3章 MySQL复制原理
假设每天做一次报表，包括所有从午夜 12 点到下一个午夜 12 点的交易。需要在午夜 12
点停止报表 slave，确保 12 点以后 slave 上不再有新的事件执行而 12 点以前的所有事件
都已经执行完毕。我们并不想手动执行此操作，所以考虑如何使这个过程自动化。可按
照以下步骤执行 ：
1． 12 点前，也可能是 12 点前 5 分钟，停止报表 slave，确保不会再从 master 接收事件。
2． 12 点后，检查 master 上的二进制日志，找到 12 点前记录的最后一个事件。显然，
如果在 12 点之前进行这一步操作，可能无法得到当天的所有事件。
3． 记录该事件的 binlog 位置，然后启动 slave。
4． 直到 slave 同步到这个位置后停止。
第一个问题是如何正确地调度任务。调度任务的方法很多，与操作系统有关。这里我们
不会详细讨论这个问题，但你可以在本章后面的“在 UNIX 上任务调度”一节中看到如
何在类 UNIX 操作系统（如 Linux）中调度任务。
停止 slave 很简单，执行 STOP SLAVE 命令，然后在 slave 停止后记录 binlog 位置 ：
slave> STOP SLAVE;
Query OK, 0 rows affected (0.25 sec)
slave> SHOW SLAVE STATUS\G
...
Relay_Master_Log_File: capulet-bin.000004
...
Exec_Master_Log_Pos: 2456
1 row in set (0.00 sec)
其他三个步骤要在实际报表开始前执行，通常作为实际报表脚本的一部分。在描述脚本
之前，我们先考虑每个步骤是怎样执行的。
调用 mysqlbinlog 命令读取二进制日志的内容。这一点后面会详细介绍，第二步中使用了
这个工具。mysqlbinlog 命令提供了两个选项，用于部分读取二进制日志，即 --start-
datetime 和 --stop-datetime 。因此，要获得从停止 slave 时刻到 12 点期间的所有事件，
可以使用以下命令 ：
$ mysqlbinlog --force --read-from-remote-server --host=reporting.bigcorp.com \
> --start-datetime='2009-09-25 23:55:00'
> --stop-datetime='2009-09-25 23:59:59' \
> binlog files
事件中存储的时间戳是指语句开始执行的时间戳，而不是它写入二进制日志的时间戳。
44
执行常见的复制任务 ｜ 39
--stop-datetime 选项说明在指定 date/time 后的第一个时间戳时刻停止产生事件。可能
有些事件，它们在 date/time 之前开始执行，但却在 date/time 后才写入二进制日志，这
样的事件不包括在内。
由于这时 master 正在写二进制日志，需要提供 --force 选项。否则，mysqlbinlog 无
法读取已打开的二进制日志。这个命令需要提供一组 binlog 文件作为参数。由于这些
文件的名称取决于配置选项，所以只能从服务器上获取这些文件名。然后，要搞清楚
mysqlbinlog 命令需要哪些 binlog 文件。使用 SHOW BINARY LOGS 命令可以很容易地获取
binlog 文件名列表 ：
master> SHOW BINARY LOGS;
+--------------------+-----------+
| Log_name|File_size |
+--------------------+-----------+
| capulet-bin.000001 | 24316 |
| capulet-bin.000002 | 1565 |
| capulet-bin.000003 | 125 |
| capulet-bin.000004 | 2749 |
+--------------------+-----------+
4 rows in set (0.00 sec)
本例只有 4 个 binlog 文件，但其实 binlog 文件可以有很多。在停止 slave 前扫描这么多
文件只是浪费时间，所以最好能减少需要读取的文件数量，以便确定 slave 停止的正确
位置。由于第 1 步中已经记下了 binlog 位置，由此可知 slave 停止运行时的 binlog 文件名，
然后将该文件名及其后面所有的文件名作为 mysqlbinlog 实用工具的输入。通常输入参
数只有一个文件（或者有两个，比如在关闭 slave 之后、启动报表事件之前的二进制日
志轮换时）。
当使用 mysqlbinlog 命令解析少数几个 binlog 文件时，将输出事件相关的文本信息。
$ mysqlbinlog --force --read-from-remote-server --host=reporting.bigcorp.com \
> --start-datetime='2009-09-25 23:55:00'
> --stop-datetime='2009-09-25 23:59:59' \
> capulet-bin.000004
/*!40019 SET @@session.max_insert_delayed_threads=0*/;
/*!50003 SET @OLD_COMPLETION_TYPE=@@COMPLETION_TYPE,COMPLETION_TYPE=0*/;
DELIMITER /*!*/;
# at 4
#090909 22:16:25 server id 1 end_log_pos 106 Start: binlog v 4, server v...
ROLLBACK/*!*/;
.
.
.
45
40 ｜ 第3章 MySQL复制原理
# at 2495
#090929 23:58:36 server id 1 end_log_pos 2650 Query thread_id=27 exe...
SET TIMESTAMP=1254213690/*!*/;
SET /*!*/;
INSERT INTO message_board(user, message)
VALUES ('mats@sun.com', 'Midnight, and I'm bored')
/*!*/;
有趣的是，最后一个事件的 end_log_pos 参数（本例中该值为 2650），就是 12 点后的下
一个事件即将被写入的位置。
如果注意观察前面的命令输出结果，你会发现我们并不知道这个字节位置指向哪个
binlog 文件，而 mysqlbinlog 需要指定一个文件来查找这个事件。如果 mysqlbinlog 命令
的参数是单个文件，显然就用这个文件查找 ；但如果参数是两个文件，则必须确定当天
最后一个事件在哪个文件里。
观察含有 end_log_pos 的那行，还会看到事件类型信息。每个 binlog 文件都是以格式描
述事件开始，这些事件会出现在前面的命令输出结果中。检查这些事件，就可以确定要
查找的事件的位置。如果有两个格式描述事件，则该事件在第二个文件中 ；如果只有一
个格式描述事件，则在第一个文件中。
报表工作开始前的最后一步是启动复制，然后在正确的位置停止它，即午夜 12 点以后
的事件写入位置（即将写入或已被写入）。可以使用不太常用的命令 START SLAVE UNTIL
来完成。该命令的参数包括 master 日志文件，以及 slave 停止时对应的 master 日志位置。
一旦 slave 到达指定位置，就会自动停止。
report> START SLAVE UNTIL
-> MASTER_LOG_POS='capulet-bin.000004',
-> MASTER_LOG_POS=2650;
Query OK, 0 rows affected (0.18 sec)
与 STOP SLAVE 命令（不含 UNTIL ）一样， START SLAVE UNTIL 命令也会立即返回，但
是如果 slave 已到达停止位置则不再返回。所以，只要 slave 还在运行，在 STOP SLAVE
UNTIL 之后发出的命令仍会继续执行。使用 MASTER_POS_WAIT 函数等待 slave 到达停止位
置，该函数会阻塞执行直到 slave 到达指定位置。
report> SELECT MASTER_POS_WAIT('capulet-bin.000004', 2650);
Query OK, 0 rows affected (231.15 sec)
现在 slave 在当天最后一个事件处停止了，报表过程可以开始分析数据和生成报告了。
46
执行常见的复制任务 ｜ 41
使用 Python 处理报表
使用 Python 进行自动化报表处理是很简单的。示例 3-7 给出了在适当的时候停止报表的
代码。
fetch_remote_binlog 函数通过 mysqlbinlog 命令从远程服务器读取二进制日志。将文
件内容作为迭代器返回，遍历文件中的各行。还可以提供一个文件列表以优化读取。还
可以指定开始的日期 / 时间和结束的日期 / 时间，以限制返回结果的日期 / 时间范围。这
些都会作为参数传递给 mysqlbinlog 程序。
find_datetime_position 函数逐行扫描 binlog 文件直到找到 end_log_pos ，同时记录观
察到的开始事件个数。该函数还联系报表服务器，以确定何时停止读取 binlog 文件，然
后联系 master 获取 binlog 文件，并确定从哪个 binlog 文件开始扫描。
示例3-7：在指定时间前运行复制的Python代码
def fetch_remote_binlog(server, binlog_files=None,
start_datetime=None, stop_datetime=None):
from subprocess import Popen, PIPE
if not binlog_files:
binlog_files = [
row["Log_name"] for row in server.sql("SHOW BINARY LOGS")]
command = ["mysqlbinlog",
"--read-from-remote-server",
"--force",
"--host=%s" % (server.host),
"--user=%s" % (server.sql_user.name)]
if server.sql_user.passwd:
command.append("--password=%s" % (server.sql_user.passwd))
if start_datetime:
command.append("--start-datetime=%s" % (start_datetime))
if stop_datetime:
command.append("--stop-datetime=%s" % (stop_datetime))
return iter(Popen(command + binlog_files, stdout=PIPE).stdout)
def find_datetime_position(master, report, start_datetime, stop_datetime):
from itertools import dropwhile
from mysql.replicant import Position
import re
all_files = [row["Log_name"] for row in master.sql("SHOW BINARY LOGS")]
stop_file = report.sql("SHOW SLAVE STATUS")["Relay_Master_Log_File"]
files = list(dropwhile(lambda file: file != stop_file, all_files))
lines = fetch_remote_binlog(server, binlog_files=files,
47
42 ｜ 第3章 MySQL复制原理
start_datetime=start_datetime,
stop_datetime=stop_datetime)
binlog_files = 0
last_epos = None
for line in lines:
m = re.match(r"#\d{6}\s+\d?\d:\d\d:\d\d\s+"
r"server id\s+(?P<sid>\d+)\s+"
r"end_log_pos\s+(?P<epos>\d+)\s+"
r"(?P<type>\w+)", line)
if m:
if m.group("type") == "Start":
binlog_files += 1
if m.group("type") == "Query":
last_epos = m.group("epos")
return Position(files[binlog_files-1], last_epos)
现在你可以使用这些函数在真正执行报表任务之前同步报表服务器。
master.connect()
report.connect()
pos = find_datetime_position(master, report,
start_datetime="2009-09-14 23:55:00",
stop_datetime="2009-09-14 23:59:59")
report.sql("START SLAVE UNTIL MASTER_LOG_FILE=%s, MASTER_LOG_POS=%s",
(pos.file, pos.pos))
report.sql("DO MASTER_POS_WAIT(%s,%s)", (pos.file, pos.pos))
.
.
code for reporting
.
.
可见，复制工作是非常简单的。这个例子涉及了一些关键概念，我们将在后面讨论横向
扩展的时候用到它们，包括如何在正确的时间启动和停止 slave，如何获取 binlog 位置
信息或怎样使用标准工具实现，以及如何整合得出自动化解决方案以满足特定需要。
在 UNIX 上调度任务
为了确保在午夜 12 点前停止 slave，然后在 12 点后启动报表，最简单的方法就是建立一
个 cron(8) 作业，向 slave 发送停止运行的命令然后启动报表脚本。
例如，下面的 crontab(5) 作业可以保证 slave 在午夜 12 点前停止，12 点后运行报表脚本
让 slave 前滚（roll forward），比如说，12 点 05 分。这里我们假设 stop_slave 脚本用于
停止 slave，  daily_report 脚本用于运行日报表（首先要同步报表服务器，如前所述）。
48
小结 ｜ 43
# stop reporting slave five minutes before midnight, every day
55 23 * * * $HOME/mysql_control/stop_slave
# Run reporting script five minutes after midnight, every day
5 0 * * * $HOME/mysql_control/daily_report
这些脚本默认放在一个名为 reporttab 的 crontab 文件中，使用 crontab reporttab 命令
可以安装这个 crontab 文件。
在 Windows 上调度任务
打开“运行”（Windows 键＋ R）对话框，输入 taskschd.msc，打开任务计划程序（Task
Scheduler）。根据安全设置及 Windows 版本不同，可能会弹出用户账户控制（UAC）对
话框，单击“继续”按钮。在“操作”菜单中选择“创建基本任务”，将会创建一个由时
间触发的任务。该指令会打开“创建基本任务向导”，其中包含一些创建简单任务的步骤。
第一步是为任务命名，并提供一个可选的描述。然后单击“下一步”按钮。
第二个页面指定任务的频率。有很多控制任务何时运行的选项，包括：一次、每天、每周、
当前用户登录时，或者当指定事件被记录时。选择完后单击“下一步”按钮。根据选择
的频率不同，第三个页面继续指定相应的触发细节（例如日期和时间）。设置完触发器
后单击“下一步”按钮。
第四个页面设置任务被触发时执行什么操作，包括“启动程序”、“发送电子邮件”或者
“显示消息”。选择其中一项，单击“下一步”按钮进入下一个页面。根据刚才的选择出
现相应的页面，以设置具体的操作。例如，如果选择“启动程序”项，你可能要输入程
序或脚本的文件名，添加参数及该任务从哪里开始等。
一旦所有信息填写完毕，单击“下一步”按钮，最后一个页面是对任务的重新检查。如
果一切都没问题，单击“完成”按钮，任务就被创建了。如果需要返回前面的页面更改
设置，则单击“上一步”按钮。如果选中“当单击完成时，打开此任务属性的对话框”项，
可更改任务属性。
小结
本章介绍了 MySQL 复制，包括为什么要使用复制，以及如何建立复制。我们还简单介
绍了二进制日志。下一章将进一步研究二进制日志。