---
title: MySQL的索引
date: 2018-11-24 22:34:18
tags: MySQL
---
思考并回答以下问题：
1.MySQL数据库中，数据是如何检索的？
2.什么是前缀索引？什么是复合索引？
3.什么是主索引和聚簇索引？
4.索引越多越好吗？
5.索引与约束有什么关系？
6.最左前缀原则是什么意思？

<!--more-->

创建数据表时，初学者通常仅仅关注该表有哪些字段、字段的数据类型以及约束条件等信息，很容易忽视数据表中的另一个重要的概念“索引”。

# <span style="color:#039BE5;">理解索引</span>

想象一下《现代汉语词典》的使用方法，就可以理解索引的重要性。《现代汉语词典》将近1800页，收录汉字达1.3万多个，如何在众多汉字中找到某个字（例如“祥”）？从现代汉语词典的第一页开始逐页逐字查找，直到查找到含有“祥”字的那一页？相信读者不会这么做。词典提供了“音节表”，“音节表”将汉语拼音“xiang”编入其中，并且“音节表”按“a”到“z”的顺序排序，故而读者可以轻松地在“音节表”中先找到“xiang->1488”，然后再从1488页开始逐字查找，这样可以快速地检索到“祥”字。“音节表”就是《现代汉语词典》的一个“索引”，其中“音节表”中的“xiang”是“索引”的“关键字”，该“关键字”的值必须来自于词典正文中的“xiang”（或者说是词典正文中“xiang”的复制），索引中的“1488”是“数据”所在起始页。数据库表中存储的数据往往比《现代汉语词典》收录的汉字多得多，没有索引的词典对读者而言变得不可想象，同样没有“索引”的数据库表对于数据库用户而言更是“悲催”。

** （1）索引的本质是什么？ **

本质上，索引其实是数据表中字段值的复制，该字段称为索引的关键字。

** （2） MySQL数据库中，数据是如何检索的？ **

简言之， MySQL在检索表中的数据时，先按照索引“关键字”的值在索引中进行查找，如果能够查到，则可以直接定位到数据所在的起始页；如果没有查到，只能全表扫描查找数据了。

** （3）一个数据库表只能创建一个索引吗？ **

当然不是。想象一下《现代汉语词典》，除了将汉语拼音编入“音节表”实现汉字的检索功能外，还将所有汉字的部首编入“部首检字表”实现汉字的检索功能，“部首检字表”是《现代汉语词典）的另一个“索引”。同样对于数据库表而言，一个数据库表可以创建多个索引。

** （4）什么是前缀索引？ **

“部首检字表”的使用方法是：首先确定一个字的部首，结合笔画可以查找到该字所在的起始页。例如部首“礻”，结合“羊”的笔画是6，可以快速地在“部首检字表”中查到“祥->1488”。 “部首检字表”中的部首“礻”仅仅是汉字的一个部分（part），不是整个汉字的拷贝。同样对于数据库表而言，索引中关键字的值可以是索引“关键字”字段值的一个部分，这种索引称为“前缀索引”。例如，可以仅仅对教师姓名（例如“张老师”）中的“姓”（“张”）建立前缀索引。

** （5）索引可以是字段的组合吗？ **

当然可以。《现代汉语词典》中的“部首检字表”中，部首是“索引”的第一关键字（也叫主关键字），部首相同时，“笔画”未必相同，笔画是“索引”的第二关键字（也叫次关键字），同样对于数据库而言，索引可以是字段的组合。数据库表的某个索引如果由多个关键字构成，此时该索引称为“复合索引”。无论索引的关键字是一个字段，还是一个字段的组合，需要注意的是，这些字段必须来自于同一张表，并且关键字的值必须是表中相应字段值的拷贝。另外，数据库为了提高查询“索引”的效率，<span style="color:red">需要对索引的关键字进行排序</span>。

** （6）能跨表创建索引吗？ **

当然不能。这个问题如同在问：是否可以在《牛津高阶英汉双解词典》创建一个“偏旁部首”索引？数据库中同一个索引允许有多个关键字，但每个关键字必须来自同一张表。

** （7）索引数据需要额外的存储空间吗？ **

当然需要。翻开词典后，几十页甚至上百页的内容存放的是“索引”数据（音节表、部首检字表）。对于数据库表的索引而言，索引关键字经排序后存放在外存中。对于MyISAM表而言，索引数据存放在外存MYI索引文件中。对于InnoDB数据库表而言，索引存在InnoDB表空间文件中（可能是共享表空间文件，也可能是独享表空间文件）。就像“音节表”是按照“从a到z”的升序顺序排放，部首检字表是按照笔画的升序顺序排放一样。为了提升数据的检索效率，无论MyISAM表的索引，还是InnoDB表的索引，<span style="color:red">索引关键字经排序（默认为升序排序）后存放在外存文件中</span>。

** （8）表中的哪些字段适合选作表的索引？什么是主索引？什么是聚簇索引？ **

想象一下，单独的笔画能作为《现代汉语词典》的索引吗？显然不能，原因在于同一个笔画的汉字太多。反过来说，由于表的主键值不可能重复，表的主键当作索引最合适不过了。

对于MyISAM表而言，MySQL会自动地将表中所有记录主键值的“备份”及每条记录所在的起始页编入索引中，像“部首检字表”一样形成一张“索引表”，存放在外存，这种索引称为“主索引” （primary index）， MyISAM表的MYI索引文件与MYD数据文件位于两个文件，通过MYI索引文件中的“表记录指针”可以找到MYD数据文件中表记录所在的物理地址。如果teacher表是MyISAM存储引擎， teacher表的主索引如下图所示。

{% asset_img 1.png %}
<center><font color="gray">MyISAM存储引擎teacher表的主索引以及普通索引</font></center>

** 说明 **
图中teacher表的记录，并没有按照教师的工号teacher_no字段进行排序，即主索引关键字的顺序与表记录主键值的顺序无需一致。

InnoDB表的“主索引”与MyISAM表的主索引不同。InnoDB表的“主索引”关键字的顺序必须与InnoDB表记录主键值的顺序一致，严格地说，这种“主索引”称为“聚簇索引”，并且每一张表只能拥有一个聚簇索引，如下图所示。假设一个汉语拼音只对应一个汉字，《现代汉语词典》中的“音节表”就变成了汉语词典的聚簇索引。

{% asset_img 2.png %}
<center><font color="gray">InnoDB存储引擎teacher表的聚簇索引</font></center>

MySQL的聚簇索引与其他数据库管理系统不同之处在于，即便是一个没有主键的MySQL表， MySQL也会为该表自动创建一个“隐式”的主键。对于InnoDB表而言，必须有聚簇索引（有且仅有一个聚簇索引） 。

前面曾经提到，由于InnoDB表记录与索引位于同一个表空间文件中，因此InnoDB表就是聚簇索引，聚簇索引就是InnoDB表。就像一本撕掉音节表、部首检字表的汉语词典一样，读者同样可以通过拼音直接在汉语词典中查找汉字，原因在于，撕掉音节表、部首检字表后的汉语词典本身就是聚簇索引。

对于InnoDB表而言，MySQL的非聚簇索引统称为“辅助索引”（secondary index），辅助索引的“表记录指针”称为“书签”（bookmark），实际上是主键值，如下图所示，可以看到，所有的辅助索引都包含主键列，所有的InnoDB表都是通过主键来聚簇的。

{% asset_img 3.png %}
<center><font color="gray">InnoDB存储引擎teacher表的聚簇索引与辅助索引</font></center>

** 说明 **
* 这里为了更直观地描述索引，图中将表的索引制作成了一个表格。事实上，表的索引往往通过更为复杂的数据结构（例如双向链表、B+树btree、hash等数据结构）实现，从而可以大幅提升数据的检索效率。

* MyISAM存储引擎的表支持主索引，并且还可以采用压缩机制（Packed：Packed的说明如下表所示）存储索引的关键字，比如第一个关键字的值
为“her”，第二关键字的值为“here”，那么第二关键字会被存储为“3, e”。

* InnoDB存储引擎的表支持聚簇索引。由于创建聚簇索引时需要对“索引”中的数据以及表中的数据进行排序，为了避免更新数据（例如插入数据）耗费过多的时间，建议将InnoDB表的主键设置为自增型字段。
	
** （9）索引与数据结构是什么关系？ **

数据库中的索引关键字在索引文件中的存储规则远比词典中的“音节表”复杂得多。为了有效提升数据检索效率，索引通常使用平衡树（btree）或者哈希表等复杂的数据结构进行“编排”。当然在操作数据库的过程中，数据库用户并不会感觉到这些数据结构的存在，原因在于SQL语句（例如select语句等）已经实现了复杂数据结构的“封装”，在执行这些SQL语句时，其底层操作实际上执行的是复杂数据结构的操作。

** （10）索引非常重要，同一个表中，表的索引越多越好吗？ **

如果没有索引， MySQL必须从第1条记录开始，甚至读完整个表才能找出相关的记录，表越大，花费的时间越多。有了索引，索引就可以帮助数据库用户快速地找出相关的记录，并且索引由MySQL自动维护，但这不意味着表的索引越多越好。

索引确实可以提高检索效率，但要记住，索引是冗余数据，冗余数据不仅需要额外的存储空间，而且还需要额外的维护（虽然不需要人为的维护） 。

如果索引过多，在更新数据（添加、修改或者删除）时，除了需要修改表中的数据外，还需要对该表的所有索引进行维护，以维持表字段值和索引关键字值之间的一致性，反而降低了数据的更新速度。实践表明，当修改表记录的操作特别频繁时，过多的索引会导致硬盘I/O次数明显增加，反而会显著地降低服务器性能，甚至可能会导致服务器宕机。不恰当的索引不但于事无补，反而会降低系统性能。因此，索引是把双刃剑，并不是越多越好，哪些字段（或字段组合）更适合选作索引的关键字？

# <span style="color:#039BE5;">索引关键字的选取原则</span>

索引的设计往往需要一定的技巧，掌握了这些技巧，可以确保索引能够大幅地提升数据的检索效率，弥补索引在数据更新方面带来的缺陷。

** 原则1：表的某个字段值的离散度越高，该字段越适合选作索引的关键字。 ** 

考虑现实生活中的场景：学生甲到别的学校找学生乙，但甲只知道乙的性别，那么学生甲要想找到乙，无异于“大海捞针”。原因很简单，性别字段的值要么是男，要么是女，取值离散度较低（Cardinality的值最多为2），因此，性别字段就没有必要选作索引的关键字了。

如果甲知道的是乙的学号，情况就比较乐观了，因为对于一个学校而言，有多少名学生，就会有多少个学号与之相对应。学号的取值特别离散，因此，比较适合选作学生表索引的关键字。

主键字段以及唯一性约束字段适合选作索引的关键字，原因就是这些字段的值非常离散。尤其是在主键字段创建索引时， Cardinality的值就等于该表的行数。MySQL在处理主键约束以及唯一性约束时，考虑得比较周全。数据库用户创建主键约束的同时，MySQL会自动创建主索引（primary index），且索引名称为PRIMARY；数据库用户创建唯一性约束的同时，MySQL会自动地创建唯一性索引（unique index），默认情况下，索引名为唯一性约束的字段名。

** 原则2：占用储存空间少的字段更适合选作索引的关键字。 **

如果索引中关键字的值占用的存储空间较多，那么检索效率势必会造成影响。例如，与字符串字段相比，整数字段占用的存储空间较少，因此，较为适合选作索引的关键字。

** 原则3：储存空间固定的字段更适合选作索引的关键字。 **

与text类型的字段相比， char类型的字段较为适合选作索引的关键字。
	
** 原则4：where子句中经常使用的字段应该创建索引，分组字段或者排序字段应该创建索引，两个表的连接字段应该创建索引。 **

引入索引的目的是提高数据的检索效率，因此索引关键字的选择与select语句息息相关。这句话有两个方面的含义：select语句的设计可以决定索引的设计；索引的设计也同样影响着select语句的设计。例如原则1与原则2，可以影响select语句的设计；而select语句中的where子句、group by子句以及order by子句，又可以影响索引的设计。两个表的连接字段应该创建索引，外键约束一经创建，MySQL会自动地创建与外键相对应的索引，这是由于外键字段通常是两个表的连接字段。
	
** 原则5：更新频繁的字段不适合创建索引，不会出现在where子句中的字段不应该创建索引。 **

** 原则6：最左前缀原则。 **

复合索引还有另外一个优点，它通过被称为“最左前缀”（leftmost prefixing）的概念体现出来。假设向一个表的多个字段（例如firstname、lastname、address）创建复合索引（索引名为fname_lname_address）。当where查询条件是以下各种字段的组合时，MySQL将使用fname_lname_address索引。其他情况将无法使用fname_lname_address索引。

firstname， lastname， address
firstname， lastname
firstname
	
可以这样理解：一个复合索引（firstname、lastname、address）等效于（firstname、lastname、age）、（firstname、lastname）以及（firstname）三个索引。

基于最左前缀原则，应该尽量避免创建重复的索引，例如创建了fname_lname_address索引后，就无需在first_name字段上单独创建一个索引。

** 原则7：尽量使用前缀索引。 **

例如，仅仅在姓名（例如“张三”）中的姓氏部分（“张”）创建索引，从而可以节省索引的存储空间，提高检索效率。

当然，索引的设计技巧还有很多，而且不是千篇一律的，更不是照本宣料的，没有索引的表同样可以完成数据检索任务。索引的设计没有对错之分，只有合适与不合适之分。与数据库的设计一样，索引的设计同样需要数据库开发人员经验的积累以及智慧的沉淀，同时需要依据系统各自的特点设计出更好的索引，在“加快检索效率”与“降低更新速度”之间做好平衡，从而大幅提升数据库的整体性能。

# <span style="color:#039BE5;">索引与约束</span>

MySQL中表的索引与约束之间存在怎样的关系？约束分为主键约束、唯一性约束、默认值约束、检查约束、非空约束以及外键约束。其中，主键约束、唯一性约束以及外键约束与索引的联系较为紧密。

约束主要用于保证业务逻辑操作数据库时数据的完整性；而索引则是将关键字数据以某种数据结构的方式存储到外存，用于提升数据的检索性能。约束是逻辑层面的概念；而索引既有逻辑上的概念，更是一种物理存储方式，且事实存在，需要耗费一定的存储空间。

对于一个MySQL数据库表而言，主键约束、唯一性约束以及外键约束是基于索引实现的。因此，创建主键约束的同时，会自动创建一个主索引，且主索引名与主键约束名相同（PRIMARY）；创建唯一性约束的同时，会自动创建一个唯一性索引，且唯一性索引名与唯一性约束名相同；创建外键约束的同时，会自动创建一个普通索引，且索引名与外键约束名相同。

在MySQL数据库中，删除了唯一性索引，对应的唯一性约束也将自动删除。若不考虑存储空间方面的因素，唯一性索引就是唯一性约束。

# <span style="color:#039BE5;">创建索引</span>

通过前面知识的讲解，我们已经将索引分为聚簇索引、主索引、唯一性索引、普通索引、复合索引等。<span style="color:red">如果数据库表的存储引擎是MyISAM，那么创建主键约束的同时， MySQL会自动地创建主索引。如果数据库表的存储引擎是InnoDB，那么创建主键约束的同时， MySQL会自动地创建聚簇索引。</font>

MySQL还支持全文索引（fulltext），当查询数据量大的字符串信息时，使用全文索引可以大幅提升字符串的检索效率。需要注意的是，全文索引只能创建在char， varchar或者text字符串类型的字段上，且全文索引不支持前缀索引。

** 说明 **
从MySQL3.23.23版本开始，MyISAM存储引擎的表最先支持全文索引；从MySQL5.6版本开始，InnoDB存储引擎的表才支持全文索引。

创建索引的方法有两种：创建表的同时创建索引，在已有表上创建索引。

** 方法一：创建表的同时创建索引。 **

使用这种方法创建索引时，可以一次性地创建一个表的多个索引（例如唯一性索引、普通索引、复合索引等），其语法格式与创建表的语法格式基本相同（注意粗体字部分的代码）
```sql
create table表名 (
字段名1 数据类型[约束条件]，
字段名2 数据类型[约束条件]，
...
[其他约束条件]，
[其他约束条件]，
...
[unique | fulltext ] index [索引名] 
(字段名[（长度）] [asc | desc])
) engine=存储引擎类型 default charset=字符集类型
```
** 说明 **
* “[]”表示可选项，“[]”里面的“|”表示将各选项隔开，“（）”表示必选项。
* 长度表示索引中关键字的字符长度，<span style="color:red">关键字的值可以是数据库表中字段值的一部分，这种索引称为“前缀索引”。</font>
* asc与desc为可选参数，分别表示升序与降序，不过目前这两个可选参数没有实际的作用，索引中所有关键字的值均以升序存储。

使用下面的SQL语句创建了一个存储引擎为MyISAM、默认字符集为gbk的书籍book表，其中定义了主键isbn、书名name、简介brief_introduction、价格price以及出版时间publish_time，并在该表分别定义了唯一性索引isbn_unique、普通索引name_index、全文索引brief_fulltext以及复合索引complex_index。
```sql
create table book(
	isbn char(20) primary key,
	name char(100) not null,
	brief_introduction text not null,
	price decimal(6,2),
	publish_time date not null,
	unique index isbn_unique (isbn),
	index name_index (name (20)),
	fulltext index brief_fulltext (name,brief_introduction),
	index complex_index (price,publish_time) 
) engine=MyISAM default charset=gbk;
```

** 说明 **
从MySQL5.6开始， InnoDB存储引擎的表已经支持全文索引，因此book表的存储引擎也可以设置为InnoDB。

** 方法二：在已有表上创建索引。 **

在已有表上创建索引有两种语法格式，这两种语法格式的共同特征是需要指定在哪个表上创建索引，语法格式分别如下。

语法格式一：
```sql
create [unique | fulltext ] index 索引名 on 表名(字段名[(长度)] [asc | desc ]）
```
语法格式二：
```sql
alter table 表名 add [ unique | fulltext ] index 索引名(字段名[(长度)][asc | desc])
```
例如，向课程course表的课程描述description字段添加全文索引，可以使用下面的SOL语句，执行结果如下。
```sql
alter table course add fulltext index description_fulltext (description);
```
{% asset_img 4.png %}
<center><font color="gray">在已有表上创建索引</font></center>	

该SQL语句等效于：
```sql
create fulltext index description_fulltext on course (description)
```
# <span style="color:#039BE5;">删除索引</span>

如果某些索引降低了数据库的性能，或者根本就没有必要使用该索引，此时可以考虑将该索引删除，删除索引的语法格式如下。
```sql
drop index 索引名 on 表名
```
例如，删除书籍book表的复合索引complex_index，可以使用下面的SQL语句实现该功能。
```sql
drop index complex_index on book;
```