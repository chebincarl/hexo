---
layout: title
title: MySQL主从复制
date: 2019-06-12 14:46:39
tags: MySQL
---
思考并回答以下问题：
1.
2.
<!--more-->

MySQL复制的优点

横向扩展解决方案：在多个从库之间分配负载以提高性能。在此环境中，所有写入和更新都必须在主服务器上进行。但是，读取操作可以在一个或多个从站上进行。该模型可以提高写入性能（因为主设备专用于写操作），同时显著提高了越来越多的从库的读取速度。
数据安全性 - 因为数据被复制到从库，并且从库可以暂停复制过程，所以可以在从站上运行备份服务而不会破坏相应的主数据。
分析 - 可以在主服务器上创建实时数据，而信息的分析可以在从服务器上进行，而不会影响主服务器的性能。
远程数据分发 - 您可以使用复制为远程站点创建数据的本地副本，而无需永久访问主服务器。
MySQL复制的类型
异步复制：主库不会等待从库的执行情况，是默认的复制模式
半同步复制：使用半同步复制，在返回执行事务的会话之前对主块执行提交，直到至少一个从服务器确认已接收并记录事务的事件为止;
MySQL 5.7 还支持延迟复制，使得从属服务器故意滞后于主服务器至少一段指定的时间;
MySQL实现类型格式
有两种核心类型的复制格式：基于语句的复制（SBR），它复制整个SQL语句，以及基于行的复制（RBR），它仅复制已更改的行。

主备通用设置
主库上的设置
开启二进制日志（bin_log）
server-id=1
如果没有设置server_id（或者默认其为0），则住服务器拒绝来自从服务器的任何连接

备库上的设置
server-id=2  设置与主库不同的号，此ID用于标识组内的各个服务器，并且必须是介于1和（2 32）-1 之间的正整数
若是server-id没有设置或是0，则从库会拒绝主库连接

如果这个备库只是一个备库的话，则它无需开启 bin_log 就可以实现功能，但是bin_log的可以让备库进行数据备份和崩溃恢复，还可以作为其他机器的备库。所以，强烈建议开启备库的bin_log。

创建复制用户
每个从库使用MySQL的用户名和密码连接到主库，因此必须使用用户账户，该账户必须被赋予 replication slave 权限

如下此用户，就有复制权限

MySQL [nba]> select user,Repl_slave_priv from mysql.user where user = 'replicate';
+-----------+-----------------+
| user | Repl_slave_priv |
+-----------+-----------------+
| replicate | Y |
+-----------+-----------------+
1 row in set (0.00 sec)
创建用户的过程为

MySQL [nba]> select user,Repl_slave_priv from mysql.user where user = 'stephen';
+---------+-----------------+
| user | Repl_slave_priv |
+---------+-----------------+
| stephen | N |
+---------+-----------------+
1 row in set (0.00 sec)

MySQL [nba]> grant replication slave on *.* to 'stephen'@'%';
Query OK, 0 rows affected (0.00 sec)

MySQL [nba]> select user,Repl_slave_priv from mysql.user where user = 'stephen';
+---------+-----------------+
| user | Repl_slave_priv |
+---------+-----------------+
| stephen | Y |
+---------+-----------------+
1 row in set (0.00 sec)
备库上的数据
要搭建备库，首先备库上需要有现有主库的数据快照，一般使用下面两种方法实现

使用mysqldump
使用mysql自带的mysqldump命令可以将整库备份出来

mysqldump -S /mysqldata/mysql.sock -uroot -pmysql --all-databases --master-data > dbdump.db
除了这个备份，还需要获取主二进制日志坐标，过程如下：

MySQL [nba]> FLUSH TABLES WITH READ LOCK;
Query OK, 0 rows affected (0.01 sec)

MySQL [nba]> show master status\G;
*************************** 1. row ***************************
             File: mysql-bin.000027
         Position: 154
     Binlog_Do_DB: test
 Binlog_Ignore_DB: 
Executed_Gtid_Set: 
1 row in set (0.00 sec)
使用 XtraBackup全备
安装了此功能之后，可以使用这个方法进行数据传输

innobackupex --defaults-file=/etc/my.cnf --user=root --password=mysql --no-timestamp /databackup
在从库上新建备份目录，并把主库的全备传输到从库上

scp -r /databackup/* 192.168.56.11:/databackup
应用一下全备

innobackupex --apply-log /databackup/
在从库上恢复全备

innobackupex --copy-back /databackup
此方法的日志位置，可以在备份目录中的xtrabackup_info文件中查看到

binlog_pos = filename 'mysql-bin.000024', position '154'
搭建主从
建立备库连接
进行连接配置

CHANGE MASTER TO 
 MASTER_HOST='192.168.56.10',
 MASTER_USER='replicate',
 MASTER_PASSWORD='replicate',
 MASTER_PORT=3306,
 MASTER_LOG_FILE='mysql-bin.000024', 
 MASTER_LOG_POS=468,
 MASTER_CONNECT_RETRY=10;
接下来，如果是使用mysqldump

使用 --skip-slave-start 启动备库
导入转储文件：

shell>mysql < fulldb.dump
3. 启动从库线程
mysql>start slave;
如果是使用Xtrabackup全量备份
则在开启数据库后直接开启从库线程即可

半同步复制
概述
MySQL5.7默认的复制是异步复制，主将事物操作写到二进制文件，但是不知道从库是否或何时去应用这些日志，如果主库崩溃，则会有事物未提交到备库，就会导致数据库丢失。为了解决这个问题，我们可以使用半同步复制。

半同步复制有以下性质：
若主库是半同步复制，则在主上执行事物提交的线程等待，直到至少一个半同步设备确认已收到事物的所有内容，或者直到发生超时，这也是半同步复制的'半'字的原因。若是全同步复制，则就需要全部从库收到事务的所有内容

从库只有将事物写到日志直到刷新磁盘后，从库才会确认收到全部事物

安装插件
在使用半同步之前，需要先在主从上分别安装插件
主库上安装插件

MySQL [(none)]> install plugin rpl_semi_sync_master soname 'semisync_master.so';
Query OK, 0 rows affected (0.04 sec)

MySQL [(none)]> set global rpl_semi_sync_master_enabled=on;
Query OK, 0 rows affected (0.00 sec)
查看是否安装成功

MySQL [(none)]> show variables like '%semi%';
+-------------------------------------------+------------+
| Variable_name | Value |
+-------------------------------------------+------------+
| rpl_semi_sync_master_enabled | ON |            #设置为on表示开启了半同步功能
| rpl_semi_sync_master_timeout | 10000 |        #单位是毫秒，表示如果主库等待从库回复消息的时间超过该值，就自动切换为异步复制模式
| rpl_semi_sync_master_trace_level | 32 |        #它控制主库接收多少个从库写事物成功反馈，才返回成功给客户端
| rpl_semi_sync_master_wait_for_slave_count | 1 | #
| rpl_semi_sync_master_wait_no_slave | ON |
| rpl_semi_sync_master_wait_point | AFTER_SYNC |  # 默认值是AFTER_SYNC，含义是主库将每个事物写入binlog,并传递给从库，刷新到中继日志，主库开始等待从库的反馈，接收到从库的回复之后，再提交事物并且返回“commit ok”结果给客户端
+-------------------------------------------+------------+
6 rows in set (0.01 sec)
从库上安装插件

MySQL [(none)]> install plugin rpl_semi_sync_slave soname 'semisync_slave.so';
Query OK, 0 rows affected (0.01 sec)

MySQL [(none)]> set global rpl_semi_sync_slave_enabled=on;
Query OK, 0 rows affected (0.00 sec)
参数完成之后，启动从库线程

MySQL [(none)]> start slave io_thread;
Query OK, 0 rows affected (0.00 sec)
延迟复制
概述
MySQL5.7 支持延迟复制，使得从库可以刻意滞后主库至少一段指定的时间，默认延迟是0秒，使用MASTER_DELAY 选项CHANGE_MASTER_TO 使用

MySQL [nba]> CHANGE MASTER TO MASTER_DELAY = 20;
Query OK, 0 rows affected (0.09 sec)
延迟复制的意义：

防止用户在主库上出错，DBA可以将延迟的从库回滚到误操作前的时间
测试延迟时系统的行为方式
检查数据库很久之前的样子，而不必重新加载备份
信息显示
在show slave statusG 上有几个字段显示了延迟复制的信息

SQL_Delay：默认为0，显示与主库延迟的秒数
SQL_Remaining_Delay：当 Slave_SQL_Running_State是Waiting until MASTER_DELAY seconds after master executed event，则此字段包含一个整数指示左延迟的秒数。在其他时候，这个参数是NULL
Slave_SQL_Running_State：当前的复制的状态，一般的同步完成是Slave has read all relay log; waiting for more updates