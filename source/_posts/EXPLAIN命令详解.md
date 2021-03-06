---
layout: title
title: EXPLAIN命令详解
date: 2019-06-05 09:38:05
tags: MySQL
---
思考并回答以下问题：
1.explain是干嘛用的？如何使用？

<!--more-->

有的应用对select语句的性能要求较高，此时单纯依靠开发者的直觉设计select语句，可能能导致“理想”与“现实”出现偏差。可以使用explain命令或desc命令分析select语句的执行计划，从而了解select语句的执行情况，进而分析select语句的性能瓶颈，然后对select语句进行重新设计或者对表结构进行重新设计（例如添加索引或者拆分表），让查询优化器能够更好地工作，提升查询性能。explain的语法格式非常简单（desc语法格式与explain的语法格式相同），如下所示：

explain select语句说明

desc命令通常用于获取表结构的相关信息。而explain命令通常用于获取查询的执行计划例如，多个表进行 join 连接运算时，这些表如何连接、连接顺序如何等信息都可以通过explain命令获取。在MySQL 5.7中，使用explain命令还可以获取select、delete、insert、replace以及update等语句的执行计划。然而，一般而言，由于select语句对MySQL的性能影响较大，通常使用explain命令获取select语句的执行计划。explain命令返回了一行或者多行记录，包括了select语句中用到的各个表的信息。
例如，查询姓“张”学生的信息，可以使用下面的SQL语句。使用explain对该SQL语句进行分析，如图14-68所示。
```sql
explain select * from student where student_name like '张_';
```
explain命令的返回信息说明如下。
● id：查询的序列号。
● select_type：查询语句的类型，可以为以下任何一种。
simple：简单查询语句（不使用union或子查询的查询）。
primary：主查询语句

union：union中的第二个或后面的select语句。
dependent union：相关子查询中的union语句，union中的第二个或后面的select语句。
union result：union 的合并结果。
subquery：非相关子查询中的第一个select语句。
uncacheable subquery：结果集无法缓存的子查询。
dependent subquery：相关子查询中的第一个select语句。
derived：派生表的select语句。
● table：执行该查询时所访问的数据库表。
● type：表数据的访问类型。下面给出各种访问类型，按照性能从最佳类型到最坏类型进行排序。
system：结果集中仅有一条记录。这是const连接类型的一个特例。
const：表中有多条记录，但结果集只包含一条记录。例如比较运算符中含有主键字段或者唯一性约束字段，只查询出表的一条记录。
eq_ref：最多只会有一条匹配结果。两个表进行连接运算时，一个表使用主键字段或者唯一性约束字段与另一个表连接，查询出若干条记录。除了const类型，是比较好的连接类型。

ref：两个表进行连接运算时，一个表使用普通索引与另一个表连接，查询出若干条记录。
ref_or_null：两个表进行连接运算时，一个表使用普通索引与另一个表连接（这与ref类似），不同之处在于，检索时额外搜索包含null值的记录。
index_merge：查询中同时使用两个（或更多）索引，然后对索引结果进行merge之后再读取表数据。在这种情况下，key字段表示使用了哪个索引，key_len字段表示使用索引时关键字的最长长度（字节数）。
unique_subquery：使用了子查询，且子查询的返回结果包括主键字段或者唯一性约束字段。
index_subquery：使用了子查询，且子查询使用了普通索引（不是主索引或唯一索引）。
range：使用索引字段，检索给定范围的记录。在这种情况下，key字段表示使用了哪个索引。key_len字段表示使用索引时关键字的最长长度（字节数）。在该类型中，ref字段值为NULL。
index：从第一个关键字开始，对索引进行顺序扫描。即便如此，index通常比ALL快，因为索引文件比数据文件小。
all：对表进行全表扫描。说明检索数据时MySQL可能没有使用索引，效率会受到重大影响，应尽可能地优化select语句或者添加索引以避免此类情况的发生。
fulltext：全文索引。说明
从全表扫描（full table scan）、索引扫描（index scans）、范围扫描(range scans)、唯一索引查找（unique index lookups）到常量（constants）扫描，访问速度依次递增，访问的数据越来越少。
● possible_keys：检索数据时可能使用到的索引，这就意味着possible_keys里面所包含的索引可能在 select 语句实际运行过程中根本没有用到。如果这个字段的值是 null，就表示没有索引被用到。这种情况下，可以检查where子句中哪些字段适合增加索引以提高查询的性能。
● key：实际使用的索引。当没有任何索引被用到的时候，这个字段的值就是NULL。想让MySQL强行使用或者忽略在possible_keys 字段中的索引列表，可以在select 语句中使用关键字force index，use index或ignore index。
● key_len：使用的索引关键字的长度（字节数）。当key字段的值为NULL时，索引的长度就是NULL。
● ref：显示了哪些字段或者常量被用来和索引关键字匹配以从表中查询记录。
● rows：返回MySQL认为在查询中应该检索的记录数。

extra：显示了查询中MySQL的附加信息。以下是这个字段的几个不同值的解释。
distinct：当MySQL找到第一个匹配记录后，就不再搜索其他记录了。
not exists：MySQL 能够对查询进行left join 优化，当在当前表中找到一条记录符合left join匹配标准时，就不再搜索更多的记录了。
range checked for each record (index map：#)：MySQL 没找到合适的可用的索引，但是发现来自前面表的字段值已知，部分索引可以使用。
using filesort：当查询中包含order by子句，而且无法利用索引完成排序操作的时候，MySQL查询优化器不得不选择相应的排序算法，在内存或者硬盘上进行排序。应尽可能地优化select语句或者添加索引以避免此类情况的发生。
using index：直接从索引中取得信息，不需要从表中获取数据。这就意味着查询时的字段是索引的关键字。
using temporary：MySQL需要创建临时表存储结果集以完成查询。在group by以及order by查询中比较常见。
using where：如果查询不是读取表的所有数据，也不是仅仅通过索引就可以获取所有需要的数据，则会出现using where信息。


using sort_union(……), using union(……), using intersect(……)：说明如何为index_merge连接类型合并索引扫描。
using index for group-by：类似于访问表的using index 方式，using index for group-by表示在进行group by或distinct查询时，分组字段也在索引中。
Full scan on NULL key：子查询中的一种优化方式，主要在遇到无法通过索引访问NULL 值的时候使用。
Impossible WHERE noticed after reading const tables：MySQL 查询优化器通过收集到的统计信息判断出不可能存在的结果。
No tables：查询语句中使用不包含任何FROM的子句。
Select tables optimized away：当使用某些聚合函数来访问某个索引字段的时候，MySQL 查询优化器会通过索引直接一次定位到所需的数据行，完成整个查询。当然，前提是在查询语句中不能有group by操作。
使用下面的SQL语句在student表的student_name字段创建两个普通索引，其中name_index1索引仅仅对学生的“姓”创建了索引。然后重新使用explain命令分析两条select语句，如图14-69所示。
create index name_index on student(student_name);
create index name_index1 on student (student_name(1));
explain select * from student where student_name like '张％'\G
explain select * from student use index (name_index1) where student_name like '张％'\G

从图中可以分析，第一条select语句执行时使用了name_index索引检索了student表，第二条select语句由于指定使用索引name_index1，执行时使用了name_index1索引检索了student表（注意两个索引的长度key_len的值不相同）。

https://www.cnblogs.com/gomysql/p/3720123.html

https://blog.csdn.net/jiadajing267/article/details/81269067

https://blog.csdn.net/wuseyukui/article/details/71512793

explain显示了mysql如何使用索引来处理select语句以及连接表。可以帮助选择更好的索引和写出更优化的查询语句。

使用方法，在select语句前加上explain就可以了：

如：
```sql
explain select surname,first_name from a,b where a.id=b.id;
```
# EXPLAIN列的解释
table：显示这一行的数据是关于哪张表的

type：这是重要的列，显示连接使用了何种类型。从最好到最差的连接类型为const、eq_reg、ref、range、index和ALL

         type显示的是访问类型，是较为重要的一个指标，结果值从好到坏依次是：system > const > eq_ref > ref > fulltext > ref_or_null > index_merge > unique_subquery > index_subquery > range > index > ALL
         一般来说，得保证查询至少达到range级别，最好能达到ref。

possible_keys：显示可能应用在这张表中的索引。如果为空，没有可能的索引。可以为相关的域从WHERE语句中选择一个合适的语句

key： 实际使用的索引。如果为NULL，则没有使用索引。很少的情况下，MYSQL会选择优化不足的索引。这种情况下，可以在SELECT语句中使用USE INDEX（indexname）来强制使用一个索引或者用IGNORE INDEX（indexname）来强制MYSQL忽略索引

key_len：使用的索引的长度。在不损失精确性的情况下，长度越短越好

ref：显示索引的哪一列被使用了，如果可能的话，是一个常数

rows：MYSQL认为必须检查的用来返回请求数据的行数

Extra：关于MYSQL如何解析查询的额外信息。将在表4.3中讨论，但这里可以看到的坏的例子是Using temporary和Using filesort，意思MYSQL根本不能使用索引，结果是检索会很慢

extra列返回的描述的意义

 Distinct:一旦MYSQL找到了与行相联合匹配的行，就不再搜索了

 Not exists: MYSQL优化了LEFT JOIN，一旦它找到了匹配LEFT JOIN标准的行，就不再搜索了

 Range checked for each Record（index map:#）:没有找到理想的索引，因此对于从前面表中来的每一个行组合，MYSQL检查使用哪个索引，并用它来从表中返回行。这是使用索引的最慢的连接之一

 Using filesort: 看到这个的时候，查询就需要优化了。MYSQL需要进行额外的步骤来发现如何对返回的行排序。它根据连接类型以及存储排序键值和匹配条件的全部行的行指针来排序全部行

 Using index: 列数据是从仅仅使用了索引中的信息而没有读取实际的行动的表返回的，这发生在对表的全部的请求列都是同一个索引的部分的时候

 Using temporary 看到这个的时候，查询需要优化了。这里，MYSQL需要创建一个临时表来存储结果，这通常发生在对不同的列集进行ORDER BY上，而不是GROUP BY上

 Where used 使用了WHERE从句来限制哪些行将与下一张表匹配或者是返回给用户。如果不想返回表中的全部行，并且连接类型ALL或index，这就会发生，或者是查询有问题不同连接类型的解释（按照效率高低的顺序排序）

 system 表只有一行：system表。这是const连接类型的特殊情况

 const:表中的一个记录的最大值能够匹配这个查询（索引可以是主键或惟一索引）。因为只有一行，这个值实际就是常数，因为MYSQL先读这个值然后把它当做常数来对待

 eq_ref:在连接中，MYSQL在查询时，从前面的表中，对每一个记录的联合都从表中读取一个记录，它在查询使用了索引为主键或惟一键的全部时使用

 ref:这个连接类型只有在查询使用了不是惟一或主键的键或者是这些类型的部分（比如，利用最左边前缀）时发生。对于之前的表的每一个行联合，全部记录都将从表中读出。这个类型严重依赖于根据索引匹配的记录多少—越少越好

 range:这个连接类型使用索引返回一个范围中的行，比如使用>或<查找东西时发生的情况

 index: 这个连接类型对前面的表中的每一个记录联合进行完全扫描（比ALL更好，因为索引一般小于表数据）

 ALL:这个连接类型对于前面的每一个记录联合进行完全扫描，这一般比较糟糕，应该尽量避免

  

先看一个例子：

mysql> explain select * from t_order; 
+----+-------------+---------+------+---------------+------+---------+------+--------+-------+ 
| id | select_type | table   | type | possible_keys | key  | key_len | ref  | rows   | Extra | 
+----+-------------+---------+------+---------------+------+---------+------+--------+-------+ 
|  1 | SIMPLE      | t_order | ALL  | NULL          | NULL | NULL    | NULL | 100453 |       | 
+----+-------------+---------+------+---------------+------+---------+------+--------+-------+ 
1 row in set (0.03 sec) 
加上extended后之后：

mysql> explain extended select * from t_order; 
+----+-------------+---------+------+---------------+------+---------+------+--------+----------+-------+ 
| id | select_type | table   | type | possible_keys | key  | key_len | ref  | rows   | filtered | Extra | 
+----+-------------+---------+------+---------------+------+---------+------+--------+----------+-------+ 
|  1 | SIMPLE      | t_order | ALL  | NULL          | NULL | NULL    | NULL | 100453 |   100.00 |       | 
+----+-------------+---------+------+---------------+------+---------+------+--------+----------+-------+ 
1 row in set, 1 warning (0.00 sec) 
有必要解释一下这个长长的表格里每一列的含义：

id	SELECT识别符。这是SELECT的查询序列号
select_type	
SELECT类型,可以为以下任何一种:

SIMPLE:简单SELECT(不使用UNION或子查询)
PRIMARY:最外面的SELECT
UNION:UNION中的第二个或后面的SELECT语句
DEPENDENT UNION:UNION中的第二个或后面的SELECT语句,取决于外面的查询
UNION RESULT:UNION 的结果
SUBQUERY:子查询中的第一个SELECT
DEPENDENT SUBQUERY:子查询中的第一个SELECT,取决于外面的查询
DERIVED:导出表的SELECT(FROM子句的子查询)
table	
输出的行所引用的表

type	
联接类型。下面给出各种联接类型,按照从最佳类型到最坏类型进行排序:

system:表仅有一行(=系统表)。这是const联接类型的一个特例。
const:表最多有一个匹配行,它将在查询开始时被读取。因为仅有一行,在这行的列值可被优化器剩余部分认为是常数。const表很快,因为它们只读取一次!
eq_ref:对于每个来自于前面的表的行组合,从该表中读取一行。这可能是最好的联接类型,除了const类型。
ref:对于每个来自于前面的表的行组合,所有有匹配索引值的行将从这张表中读取。
ref_or_null:该联接类型如同ref,但是添加了MySQL可以专门搜索包含NULL值的行。
index_merge:该联接类型表示使用了索引合并优化方法。
unique_subquery:该类型替换了下面形式的IN子查询的ref: value IN (SELECT primary_key FROM single_table WHERE some_expr) unique_subquery是一个索引查找函数,可以完全替换子查询,效率更高。
index_subquery:该联接类型类似于unique_subquery。可以替换IN子查询,但只适合下列形式的子查询中的非唯一索引: value IN (SELECT key_column FROM single_table WHERE some_expr)
range:只检索给定范围的行,使用一个索引来选择行。
index:该联接类型与ALL相同,除了只有索引树被扫描。这通常比ALL快,因为索引文件通常比数据文件小。
ALL:对于每个来自于先前的表的行组合,进行完整的表扫描。
possible_keys	
指出MySQL能使用哪个索引在该表中找到行

key	显示MySQL实际决定使用的键(索引)。如果没有选择索引,键是NULL。
key_len	显示MySQL决定使用的键长度。如果键是NULL,则长度为NULL。
ref	显示使用哪个列或常数与key一起从表中选择行。
rows	显示MySQL认为它执行查询时必须检查的行数。多行之间的数据相乘可以估算要处理的行数。
filtered	显示了通过条件过滤出的行数的百分比估计值。
Extra	
该列包含MySQL解决查询的详细信息

Distinct:MySQL发现第1个匹配行后,停止为当前的行组合搜索更多的行。
Not exists:MySQL能够对查询进行LEFT JOIN优化,发现1个匹配LEFT JOIN标准的行后,不再为前面的的行组合在该表内检查更多的行。
range checked for each record (index map: #):MySQL没有发现好的可以使用的索引,但发现如果来自前面的表的列值已知,可能部分索引可以使用。
Using filesort:MySQL需要额外的一次传递,以找出如何按排序顺序检索行。
Using index:从只使用索引树中的信息而不需要进一步搜索读取实际的行来检索表中的列信息。
Using temporary:为了解决查询,MySQL需要创建一个临时表来容纳结果。
Using where:WHERE 子句用于限制哪一个行匹配下一个表或发送到客户。
Using sort_union(...), Using union(...), Using intersect(...):这些函数说明如何为index_merge联接类型合并索引扫描。
Using index for group-by:类似于访问表的Using index方式,Using index for group-by表示MySQL发现了一个索引,可以用来查 询GROUP BY或DISTINCT查询的所有列,而不要额外搜索硬盘访问实际的表。
 

一.select_type的说明

1.UNION:

当通过union来连接多个查询结果时，第二个之后的select其select_type为UNION。

mysql> explain select * from t_order where order_id=100 union select * from t_order where order_id=200; 
+----+--------------+------------+-------+---------------+---------+---------+-------+------+-------+ 
| id | select_type  | table      | type  | possible_keys | key     | key_len | ref   | rows | Extra | 
+----+--------------+------------+-------+---------------+---------+---------+-------+------+-------+ 
|  1 | PRIMARY      | t_order    | const | PRIMARY       | PRIMARY | 4       | const |    1 |       | 
|  2 | UNION        | t_order    | const | PRIMARY       | PRIMARY | 4       | const |    1 |       | 
| NULL | UNION RESULT | <union1,2> | ALL   | NULL          | NULL    | NULL    | NULL  | NULL |       | 
+----+--------------+------------+-------+---------------+---------+---------+-------+------+-------+ 
3 rows in set (0.34 sec) 
2.DEPENDENT UNION与DEPENDENT SUBQUERY:

当union作为子查询时，其中第二个union的select_type就是DEPENDENT UNION。
第一个子查询的select_type则是DEPENDENT SUBQUERY。

mysql> explain select * from t_order where order_id in (select order_id from t_order where order_id=100 union select order_id from t_order where order_id=200); 
+----+--------------------+------------+-------+---------------+---------+---------+-------+--------+-------------+ 
| id | select_type        | table      | type  | possible_keys | key     | key_len | ref   | rows   | Extra       | 
+----+--------------------+------------+-------+---------------+---------+---------+-------+--------+-------------+ 
|  1 | PRIMARY            | t_order    | ALL   | NULL          | NULL    | NULL    | NULL  | 100453 | Using where | 
|  2 | DEPENDENT SUBQUERY | t_order    | const | PRIMARY       | PRIMARY | 4       | const |      1 | Using index | 
|  3 | DEPENDENT UNION    | t_order    | const | PRIMARY       | PRIMARY | 4       | const |      1 | Using index | 
| NULL | UNION RESULT       | <union2,3> | ALL   | NULL          | NULL    | NULL    | NULL  |   NULL |             | 
+----+--------------------+------------+-------+---------------+---------+---------+-------+--------+-------------+ 
4 rows in set (0.03 sec) 
3.SUBQUERY:

子查询中的第一个select其select_type为SUBQUERY。

mysql> explain select * from t_order where order_id=(select order_id from t_order where order_id=100); 
+----+-------------+---------+-------+---------------+---------+---------+-------+------+-------------+ 
| id | select_type | table   | type  | possible_keys | key     | key_len | ref   | rows | Extra       | 
+----+-------------+---------+-------+---------------+---------+---------+-------+------+-------------+ 
|  1 | PRIMARY     | t_order | const | PRIMARY       | PRIMARY | 4       | const |    1 |             | 
|  2 | SUBQUERY    | t_order | const | PRIMARY       | PRIMARY | 4       |       |    1 | Using index | 
+----+-------------+---------+-------+---------------+---------+---------+-------+------+-------------+ 
2 rows in set (0.03 sec) 
4.DERIVED:

当子查询是from子句时，其select_type为DERIVED。

mysql> explain select * from (select order_id from t_order where order_id=100) a; 
+----+-------------+------------+--------+---------------+---------+---------+------+------+-------------+ 
| id | select_type | table      | type   | possible_keys | key     | key_len | ref  | rows | Extra       | 
+----+-------------+------------+--------+---------------+---------+---------+------+------+-------------+ 
|  1 | PRIMARY     | <derived2> | system | NULL          | NULL    | NULL    | NULL |    1 |             | 
|  2 | DERIVED     | t_order    | const  | PRIMARY       | PRIMARY | 4       |      |    1 | Using index | 
+----+-------------+------------+--------+---------------+---------+---------+------+------+-------------+ 
2 rows in set (0.03 sec) 
二.type的说明

1.system，const

见上面4.DERIVED的例子。其中第一行的type就是为system，第二行是const，这两种联接类型是最快的。

2.eq_ref

在t_order表中的order_id是主键，t_order_ext表中的order_id也是主键，该表可以认为是订单表的补充信息表，他们的关系是1对1，在下面的例子中可以看到b表的连接类型是eq_ref，这是极快的联接类型。

mysql> explain select * from t_order a,t_order_ext b where a.order_id=b.order_id; 
+----+-------------+-------+--------+---------------+---------+---------+-----------------+------+-------------+ 
| id | select_type | table | type   | possible_keys | key     | key_len | ref             | rows | Extra       | 
+----+-------------+-------+--------+---------------+---------+---------+-----------------+------+-------------+ 
|  1 | SIMPLE      | b     | ALL    | order_id      | NULL    | NULL    | NULL            |    1 |             | 
|  1 | SIMPLE      | a     | eq_ref | PRIMARY       | PRIMARY | 4       | test.b.order_id |    1 | Using where | 
+----+-------------+-------+--------+---------------+---------+---------+-----------------+------+-------------+ 
2 rows in set (0.00 sec) 
3.ref

下面的例子在上面的例子上略作了修改，加上了条件。此时b表的联接类型变成了ref。因为所有与a表中order_id=100的匹配记录都将会从b表获取。这是比较常见的联接类型。

mysql> explain select * from t_order a,t_order_ext b where a.order_id=b.order_id and a.order_id=100; 
+----+-------------+-------+-------+---------------+----------+---------+-------+------+-------+ 
| id | select_type | table | type  | possible_keys | key      | key_len | ref   | rows | Extra | 
+----+-------------+-------+-------+---------------+----------+---------+-------+------+-------+ 
|  1 | SIMPLE      | a     | const | PRIMARY       | PRIMARY  | 4       | const |    1 |       | 
|  1 | SIMPLE      | b     | ref   | order_id      | order_id | 4       | const |    1 |       | 
+----+-------------+-------+-------+---------------+----------+---------+-------+------+-------+ 
2 rows in set (0.00 sec) 
4.ref_or_null

user_id字段是一个可以为空的字段，并对该字段创建了一个索引。在下面的查询中可以看到联接类型为ref_or_null，这是mysql为含有null的字段专门做的处理。在我们的表设计中应当尽量避免索引字段为NULL，因为这会额外的耗费mysql的处理时间来做优化。

mysql> explain select * from t_order where user_id=100 or user_id is null; 
+----+-------------+---------+-------------+---------------+---------+---------+-------+-------+-------------+ 
| id | select_type | table   | type        | possible_keys | key     | key_len | ref   | rows  | Extra       | 
+----+-------------+---------+-------------+---------------+---------+---------+-------+-------+-------------+ 
|  1 | SIMPLE      | t_order | ref_or_null | user_id       | user_id | 5       | const | 50325 | Using where | 
+----+-------------+---------+-------------+---------------+---------+---------+-------+-------+-------------+ 
1 row in set (0.00 sec) 
5.index_merge

经常出现在使用一张表中的多个索引时。mysql会将多个索引合并在一起，如下例:

mysql> explain select * from t_order where order_id=100 or user_id=10; 
+----+-------------+---------+-------------+-----------------+-----------------+---------+------+------+-------------------------------------------+ 
| id | select_type | table   | type        | possible_keys   | key             | key_len | ref  | rows | Extra                                     | 
+----+-------------+---------+-------------+-----------------+-----------------+---------+------+------+-------------------------------------------+ 
|  1 | SIMPLE      | t_order | index_merge | PRIMARY,user_id | PRIMARY,user_id | 4,5     | NULL |    2 | Using union(PRIMARY,user_id); Using where | 
+----+-------------+---------+-------------+-----------------+-----------------+---------+------+------+-------------------------------------------+ 
1 row in set (0.09 sec) 
6.unique_subquery

该联接类型用于替换value IN (SELECT primary_key FROM single_table WHERE some_expr)这样的子查询的ref。注意ref列，其中第二行显示的是func，表明unique_subquery是一个函数，而不是一个普通的ref。

mysql> explain select * from t_order where order_id in (select order_id from t_order where user_id=10); 
+----+--------------------+---------+-----------------+-----------------+---------+---------+------+--------+-------------+ 
| id | select_type        | table   | type            | possible_keys   | key     | key_len | ref  | rows   | Extra       | 
+----+--------------------+---------+-----------------+-----------------+---------+---------+------+--------+-------------+ 
|  1 | PRIMARY            | t_order | ALL             | NULL            | NULL    | NULL    | NULL | 100649 | Using where | 
|  2 | DEPENDENT SUBQUERY | t_order | unique_subquery | PRIMARY,user_id | PRIMARY | 4       | func |      1 | Using where | 
+----+--------------------+---------+-----------------+-----------------+---------+---------+------+--------+-------------+ 
2 rows in set (0.00 sec) 
7.index_subquery

该联接类型与上面的太像了，唯一的差别就是子查询查的不是主键而是非唯一索引。

mysql> explain select * from t_order where user_id in (select user_id from t_order where order_id>10); 
+----+--------------------+---------+----------------+-----------------+---------+---------+------+--------+--------------------------+ 
| id | select_type        | table   | type           | possible_keys   | key     | key_len | ref  | rows   | Extra                    | 
+----+--------------------+---------+----------------+-----------------+---------+---------+------+--------+--------------------------+ 
|  1 | PRIMARY            | t_order | ALL            | NULL            | NULL    | NULL    | NULL | 100649 | Using where              | 
|  2 | DEPENDENT SUBQUERY | t_order | index_subquery | PRIMARY,user_id | user_id | 5       | func |  50324 | Using index; Using where | 
+----+--------------------+---------+----------------+-----------------+---------+---------+------+--------+--------------------------+ 
2 rows in set (0.00 sec) 
8.range

按指定的范围进行检索，很常见。

mysql> explain select * from t_order where user_id in (100,200,300); 
+----+-------------+---------+-------+---------------+---------+---------+------+------+-------------+ 
| id | select_type | table   | type  | possible_keys | key     | key_len | ref  | rows | Extra       | 
+----+-------------+---------+-------+---------------+---------+---------+------+------+-------------+ 
|  1 | SIMPLE      | t_order | range | user_id       | user_id | 5       | NULL |    3 | Using where | 
+----+-------------+---------+-------+---------------+---------+---------+------+------+-------------+ 
1 row in set (0.00 sec) 
9.index

在进行统计时非常常见，此联接类型实际上会扫描索引树，仅比ALL快些。
```sql
mysql> explain select count(*) from t_order; 
+----+-------------+---------+-------+---------------+---------+---------+------+--------+-------------+ 
| id | select_type | table   | type  | possible_keys | key     | key_len | ref  | rows   | Extra       | 
+----+-------------+---------+-------+---------------+---------+---------+------+--------+-------------+ 
|  1 | SIMPLE      | t_order | index | NULL          | user_id | 5       | NULL | 100649 | Using index | 
+----+-------------+---------+-------+---------------+---------+---------+------+--------+-------------+ 
1 row in set (0.00 sec) 
10.ALL
```
完整的扫描全表，最慢的联接类型，尽可能的避免。

mysql> explain select * from t_order; 
+----+-------------+---------+------+---------------+------+---------+------+--------+-------+ 
| id | select_type | table   | type | possible_keys | key  | key_len | ref  | rows   | Extra | 
+----+-------------+---------+------+---------------+------+---------+------+--------+-------+ 
|  1 | SIMPLE      | t_order | ALL  | NULL          | NULL | NULL    | NULL | 100649 |       | 
+----+-------------+---------+------+---------------+------+---------+------+--------+-------+ 
1 row in set (0.00 sec) 
三.extra的说明

1.Distinct

MySQL发现第1个匹配行后,停止为当前的行组合搜索更多的行。对于此项没有找到合适的例子，求指点。

2.Not exists

因为b表中的order_id是主键，不可能为NULL，所以mysql在用a表的order_id扫描t_order表，并查找b表的行时，如果在b表发现一个匹配的行就不再继续扫描b了，因为b表中的order_id字段不可能为NULL。这样避免了对b表的多次扫描。

mysql> explain select count(1) from t_order a left join t_order_ext b on a.order_id=b.order_id where b.order_id is null;  
+----+-------------+-------+-------+---------------+--------------+---------+-----------------+--------+--------------------------------------+ 
| id | select_type | table | type  | possible_keys | key          | key_len | ref             | rows   | Extra                                | 
+----+-------------+-------+-------+---------------+--------------+---------+-----------------+--------+--------------------------------------+ 
|  1 | SIMPLE      | a     | index | NULL          | express_type | 1       | NULL            | 100395 | Using index                          | 
|  1 | SIMPLE      | b     | ref   | order_id      | order_id     | 4       | test.a.order_id |      1 | Using where; Using index; Not exists | 
+----+-------------+-------+-------+---------------+--------------+---------+-----------------+--------+--------------------------------------+ 
2 rows in set (0.01 sec) 
3.Range checked for each record

这种情况是mysql没有发现好的索引可用，速度比没有索引要快得多。

mysql> explain select * from t_order t, t_order_ext s where s.order_id>=t.order_id and s.order_id<=t.order_id and t.express_type>5; 
+----+-------------+-------+-------+----------------------+--------------+---------+------+------+------------------------------------------------+ 
| id | select_type | table | type  | possible_keys        | key          | key_len | ref  | rows | Extra                                          | 
+----+-------------+-------+-------+----------------------+--------------+---------+------+------+------------------------------------------------+ 
|  1 | SIMPLE      | t     | range | PRIMARY,express_type | express_type | 1       | NULL |    1 | Using where                                    | 
|  1 | SIMPLE      | s     | ALL   | order_id             | NULL         | NULL    | NULL |    1 | Range checked for each record (index map: 0x1) | 
+----+-------------+-------+-------+----------------------+--------------+---------+------+------+------------------------------------------------+ 
2 rows in set (0.00 sec)
4.Using filesort

在有排序子句的情况下很常见的一种情况。此时mysql会根据联接类型浏览所有符合条件的记录，并保存排序关键字和行指针，然后排序关键字并按顺序检索行。

mysql> explain select * from t_order order by express_type; 
+----+-------------+---------+------+---------------+------+---------+------+--------+----------------+ 
| id | select_type | table   | type | possible_keys | key  | key_len | ref  | rows   | Extra          | 
+----+-------------+---------+------+---------------+------+---------+------+--------+----------------+ 
|  1 | SIMPLE      | t_order | ALL  | NULL          | NULL | NULL    | NULL | 100395 | Using filesort | 
+----+-------------+---------+------+---------------+------+---------+------+--------+----------------+ 
1 row in set (0.00 sec) 
5.Using index

这是性能很高的一种情况。当查询所需的数据可以直接从索引树中检索到时，就会出现。上面的例子中有很多这样的例子，不再多举例了。

6.Using temporary

发生这种情况一般都是需要进行优化的。mysql需要创建一张临时表用来处理此类查询。

mysql> explain select * from t_order a left join t_order_ext b on a.order_id=b.order_id group by b.order_id; 
+----+-------------+-------+------+---------------+----------+---------+-----------------+--------+---------------------------------+ 
| id | select_type | table | type | possible_keys | key      | key_len | ref             | rows   | Extra                           | 
+----+-------------+-------+------+---------------+----------+---------+-----------------+--------+---------------------------------+ 
|  1 | SIMPLE      | a     | ALL  | NULL          | NULL     | NULL    | NULL            | 100395 | Using temporary; Using filesort | 
|  1 | SIMPLE      | b     | ref  | order_id      | order_id | 4       | test.a.order_id |      1 |                                 | 
+----+-------------+-------+------+---------------+----------+---------+-----------------+--------+---------------------------------+ 
2 rows in set (0.00 sec) 
7.Using where

当有where子句时，extra都会有说明。

8.Using sort_union(...)/Using union(...)/Using intersect(...)

下面的例子中user_id是一个检索范围，此时mysql会使用sort_union函数来进行索引的合并。而当user_id是一个固定值时，请参看上面type说明5.index_merge的例子，此时会使用union函数进行索引合并。

mysql> explain select * from t_order where order_id=100 or user_id>10; 
+----+-------------+---------+-------------+-----------------+-----------------+---------+------+------+------------------------------------------------+ 
| id | select_type | table   | type        | possible_keys   | key             | key_len | ref  | rows | Extra                                          | 
+----+-------------+---------+-------------+-----------------+-----------------+---------+------+------+------------------------------------------------+ 
|  1 | SIMPLE      | t_order | index_merge | PRIMARY,user_id | user_id,PRIMARY | 5,4     | NULL |    2 | Using sort_union(user_id,PRIMARY); Using where | 
+----+-------------+---------+-------------+-----------------+-----------------+---------+------+------+------------------------------------------------+ 
1 row in set (0.00 sec) 
对于Using intersect的例子可以参看下例，user_id与express_type发生了索引交叉合并。

mysql> explain select * from t_order where express_type=1 and user_id=100; 
+----+-------------+---------+-------------+----------------------+----------------------+---------+------+------+----------------------------------------------------+ 
| id | select_type | table   | type        | possible_keys        | key                  | key_len | ref  | rows | Extra                                              | 
+----+-------------+---------+-------------+----------------------+----------------------+---------+------+------+----------------------------------------------------+ 
|  1 | SIMPLE      | t_order | index_merge | user_id,express_type | user_id,express_type | 5,1     | NULL |    1 | Using intersect(user_id,express_type); Using where | 
+----+-------------+---------+-------------+----------------------+----------------------+---------+------+------+----------------------------------------------------+ 
1 row in set (0.00 sec) 
9.Using index for group-by

表明可以在索引中找到分组所需的所有数据，不需要查询实际的表。

mysql> explain select user_id from t_order group by user_id; 
+----+-------------+---------+-------+---------------+---------+---------+------+------+--------------------------+ 
| id | select_type | table   | type  | possible_keys | key     | key_len | ref  | rows | Extra                    | 
+----+-------------+---------+-------+---------------+---------+---------+------+------+--------------------------+ 
|  1 | SIMPLE      | t_order | range | NULL          | user_id | 5       | NULL |    3 | Using index for group-by | 
+----+-------------+---------+-------+---------------+---------+---------+------+------+--------------------------+ 
1 row in set (0.00 sec) 
除了上面的三个说明，还需要注意rows的数值，多行之间的数值是乘积的关系，可以估算大概要处理的行数，如果乘积很大，那就很有优化的必要了。

# 用SQL命令查看Mysql数据库大小

要想知道每个数据库的大小的话，步骤如下：

1、进入information_schema 数据库（存放了其他的数据库的信息）

use information_schema;

 

2、查询所有数据的大小：

select concat(round(sum(data_length/1024/1024),2),'MB') as data from tables;
 

3、查看指定数据库的大小：

比如查看数据库home的大小

select concat(round(sum(data_length/1024/1024),2),'MB') as data from tables where table_schema='home';
 

4、查看指定数据库的某个表的大小

比如查看数据库home中 members 表的大小

select concat(round(sum(data_length/1024/1024),2),'MB') as data from tables where table_schema='home' and table_name='members';