---
layout: title
title: Elasticsearch
date: 2019-04-02 10:15:01
tags: 工具
---
公司之前的全文检索引擎是Coreseek+中文分词，下面把转换成Elasticsearch的过程记录下来，包括学习与执行的整个过程。

<!--more-->

参考和感谢写在前头：阮一峰、官方文档。

使用的Elasticsearch版本是7.0。

全文搜索属于最常见的需求，开源的Elasticsearch是目前全文搜索引擎的首选。它可以快速地储存、搜索和分析海量数据。维基百科、Stack Overflow、Github都采用它。

Elastic的底层是开源库Lucene。但是，你没法直接用Lucene，必须自己写代码去调用它的接口。Elastic是Lucene 的封装，提供了REST API的操作接口，开箱即用。


Elasticsearch是一个实时的分布式搜索和分析引擎。它可以帮助你用前所未有的速度去处理大规模数据。ElasticSearch是一个基于Lucene的搜索服务器。它提供了一个分布式多用户能力的全文搜索引擎，基于RESTful web接口。<span style="color:red;">Elasticsearch是用Java开发的</span>，并作为Apache许可条款下的开放源码发布，是当前流行的企业级搜索引擎。设计用于云计算中，能够达到实时搜索，稳定，可靠，快速，安装使用方便。


# 安装 




不管三七二十一，先安装试用。

CentOS7 
```Bash
curl -L -O https://artifacts.elastic.co/downloads/elasticsearch/elasticsearch-7.0.1-linux-x86_64.tar.gz
```

# 基本概念

有一些概念是Elasticsearch的核心。从一开始就理解这些概念将极大地帮助简化学习过程。

## 近实时（NRT）

Elasticsearch是一个近实时搜索平台。这意味着从索引文档到可搜索文档的时间有一点延迟（通常是一秒）。

## 集群

集群是一个或多个节点（服务器）的集合，它们共同保存您的整个数据，并提供跨所有节点的联合索引和搜索功能。集群由唯一名称标识，默认情况下为“elasticsearch”。此名称很重要，因为如果节点设置为按名称加入集群，则该节点只能是此集群的一部分。

确保不要在不同的环境中重用相同的集群名称，否则最终会导致节点加入错误的集群。例如，您可以使用logging-dev，logging-stage以及logging-prod用于开发，升级和生产集群。

请注意，拥有一个只包含单个节点的集群是完全正常的。此外，您还可以拥有多个独立的集群，每个集群都有自己唯一的集群名称。

## 节点

节点是作为群集一部分的单个服务器，存储数据并参与群集的索引和搜索功能。与集群一样，节点由名称标识，默认情况下，该名称是在启动时分配给节点的随机通用唯一标识符（UUID）。如果不需要默认值，可以定义所需的任何节点名称。此名称对于管理目的非常重要，您可以在其中识别网络中的哪些服务器与Elasticsearch集群中的哪些节点相对应。

可以将节点配置为按群集名称加入特定群集。默认情况下，每个节点都设置为加入一个名为cluster的集群elasticsearch，这意味着如果您在网络上启动了许多节点并且假设它们可以相互发现 - 它们将自动形成并加入一个名为的集群elasticsearch。

在单个群集中，您可以拥有任意数量的节点。此外，如果您的网络上当前没有其他Elasticsearch节点正在运行，则默认情况下启动单个节点将形成一个名为的新单节点集群elasticsearch。

## 索引

索引是具有某些类似特征的文档集合。例如，您可以拥有客户数据的索引，产品目录的另一个索引以及订单数据的另一个索引。索引由名称标识（必须全部为小写），并且此名称用于在对其中的文档执行索引，搜索，更新和删除操作时引用索引。

在单个群集中，您可以根据需要定义任意数量的索引。

## 文件

文档是可以编制索引的基本信息单元。例如，您可以为单个客户提供文档，为单个产品提供另一个文档，为单个订单提供另一个文档。该文档以JSON（JavaScript Object Notation）表示，JSON是一种普遍存在的互联网数据交换格式。在索引中，您可以根据需要存储任意数量的文档。

## 碎片和副本

索引可能存储大量可能超过单个节点的硬件限制的数据。例如，占用1TB磁盘空间的十亿个文档的单个索引可能不适合单个节点的磁盘，或者可能太慢而无法单独从单个节点提供搜索请求。

为了解决这个问题，Elasticsearch提供了将索引细分为多个称为分片的功能。创建索引时，只需定义所需的分片数即可。每个分片本身都是一个功能齐全且独立的“索引”，可以托管在集群中的任何节点上。

分片很重要，主要有两个原因：

它允许您水平分割/缩放内容量
它允许您跨分片（可能在多个节点上）分布和并行化操作，从而提高性能/吞吐量
分片的分布方式以及如何将其文档聚合回搜索请求的机制完全由Elasticsearch管理，对用户而言是透明的。

在任何时候都可以预期出现故障的网络/云环境中，非常有用，强烈建议使用故障转移机制，以防分片/节点以某种方式脱机或因任何原因消失。为此，Elasticsearch允许您将索引的分片的一个或多个副本制作成所谓的副本分片或简称副本。

复制很重要，主要有两个原因：

它在碎片/节点出现故障时提供高可用性。因此，请务必注意，副本分片永远不会在与从中复制的原始/主分片相同的节点上分配。
它允许您扩展搜索量/吞吐量，因为可以在所有副本上并行执行搜索。
总而言之，每个索引可以拆分为多个分片。索引也可以复制为零（表示没有副本）或更多次。复制后，每个索引都将具有主分片（从中复制的原始分片）和副本分片（主分片的副本）。

可以在创建索引时为每个索引定义分片和副本的数量。创建索引后，您还可以随时动态更改副本数。您可以使用_shrink和_splitAPI 更改现有索引的分片数，但这不是一项简单的任务，预先计划正确数量的分片是最佳方法。

默认情况下，Elasticsearch中的每个索引都分配了一个主分片和一个副本，这意味着如果群集中至少有两个节点，则索引将具有一个主分片和另一个副本分片（一个完整副本），总共两个每个索引的分片。

# 中文分词 
ik-analyzer
java开源中文分词器

IK Analyzer是一个开源的，基于java语言开发的轻量级的中文分词工具包。


# 数据同步

首先是数据同步，将mysql数据同步到es的方式很多，经过测试，稳定且易用的是 logstash-input-jdbc

全量同步与增量同步

全量同步是指全部将数据同步到es，通常是刚建立es，第一次同步时使用。增量同步是指将后续的更新、插入记录同步到es。（删除记录没有办法同步，只能两边执行自己的删除命令）

# 二、安装logstash-input-jdbc插件

logstash-input-jdbc插件是logstash 的一个插件。

Logstash是一个开源数据收集引擎，具有实时管道功能。Logstash可以动态地将来自不同数据源的数据统一起来，并将数据标准化到你所选择的目的地。logstash是一个数据分析软件，主要目的是分析log日志。首先将数据传给logstash，它将数据进行过滤和格式化（转成JSON格式），然后传给Elasticsearch进行存储、建搜索的索引。logstash和Elasticsearch是用Java写的

使用ruby语言开发。

下载插件过程中最大的坑是下载插件相关的依赖的时候下不动，因为国内网络的原因，访问不到亚马逊的服务器。

解决办法，改成国内的ruby仓库镜像。此镜像托管于淘宝的阿里云服务器上 ：

# 三、实现样例。

## Linux
目的 ： 监听数据表的数据，当我有新增时增加到elasticsearch，当我修改时，update到elasticsearch。

第一 前提：

1, 我有mysql数据库，我有一张hotel 表， hotel_account表（此表里有hotel_id）, 里面无数据。

2，已经启动 elasticsearch . 地址是 http://192.168.0.45 端口：9200.

3，已经安装 logstash, 地址在 /opt/logstash

第二 准备

两个文件： jdbc.conf  

                   jdbc.sql 。名字随便起啦。

  一个 mysql 的java 驱动包  ： mysql-connector-java-5.1.36-bin.jar

jdbc.conf 内容：

注意 statement_filepath => "jdbc.sql" 这个名字要跟下面的sql文件的名字对应上。

参考 logstash-input-jdbc官方参考文档

重点字段说明：

schedule：设置监听间隔。可以设置每隔多久监听一次什么的。具体参考官方文档。

statement_filepath： 执行的sql 文件路径+名称

```conf

input {
    stdin {
    }
    jdbc {
      # mysql jdbc connection string to our backup databse
      jdbc_connection_string => "jdbc:mysql://192.168.0.49:3306/dfb"
      # the user we wish to excute our statement as
      jdbc_user => "test"
      jdbc_password => "test"
      # the path to our downloaded jdbc driver
      jdbc_driver_library => "/opt/logstash/mysql-connector-java-5.1.36-bin.jar"
      # the name of the driver class for mysql
      jdbc_driver_class => "com.mysql.jdbc.Driver"
      jdbc_paging_enabled => "true"
      jdbc_page_size => "50000"
      statement_filepath => "jdbc.sql"
      schedule => "* * * * *"
      type => "jdbc"
    }
}

filter {
    json {
        source => "message"
        remove_field => ["message"]
    }
}

output {
    elasticsearch {
        host => "192.168.0.199"
        port => "9200"
        protocol => "http"
        index => "mysql01"
        document_id => "%{id}"
        cluster => "logstash-elasticsearch"
    }
    stdout {
        codec => json_lines
    }
}
```

jdbc.sql 
```sql
select 
    h.id as id, 
    h.hotel_name as name, 
    h.photo_url as img,
    ha.id as haId, 
    ha.finance_person
from 
    hotel h LEFT JOIN hotel_account ha on h.id = ha.hotel_id
where 
    h.last_modify_time >= :sql_last_start
```
第三： 启动 logstash
```bash
sudo bin/logstash -f jdbc.conf
```
如果一切顺利 应该如图：

{% asset_img 1.png %}

现在 logstash 已经开始监听mysql 的表了。查询哪些表 就在jdbc.sql 写sql语句就行了。

现在 手动的 hotel， hotel_account 分别增加一条数据：

 INSERT INTO hotel(hotel_name, photo_url) VALUES("马二帅酒店","images/madashuai.img");
找到 hotel 的id 

INSERT INTO hotel_account(hotel_id, finance_person) VALUES(15627, "马二帅");
大约30秒的时候查看es

{% asset_img 2.png %}

很好马二帅酒店已经增加进来了。

现在修改下 hotel_account 

UPDATE hotel_account SET finance_person = "马二帅修改为马小帅" where id = 1601;

{% asset_img 3.png %}


## Windows 
项目中用到elasticsearch，初始化数据时时写的程序从数据库里面查询出来，然后多线程往elasticsearch里面写入的。

今天试了一下Logstash-input-jdbc插件，发现高效又方便，而且可以设置定时任务。

1、安装插件

在logstash的bin目录下执行命令： logstash-plugin install logstash-input-jdbc

{% asset_img 13.png %}

2、配置文件和jar包
在bin目录下新建一个config-mysql目录,里面包含mysql.conf,
在H:\software\logstash-6.1.2\logstash-6.1.2\lib 加入mysql的驱动 mysql-connector-java-5.1.38.jar

mysql.conf的内容如下：

```conf
input {
    stdin {
    }
    jdbc {
      # 数据库
      jdbc_connection_string => "jdbc:mysql://39.107.60.74:3306/db_plat3"
      # 用户名密码
      jdbc_user => "root"
      jdbc_password => "letsgo123QWE"
      # jar包的位置
      jdbc_driver_library => "H:\software\logstash-6.1.2\logstash-6.1.2\lib\mysql-connector-java-5.1.38.jar"
      # mysql的Driver
      jdbc_driver_class => "com.mysql.jdbc.Driver"
      jdbc_paging_enabled => "true"
      jdbc_page_size => "50000"
      #statement_filepath => "config-mysql/test02.sql"
      statement => "select * from information"
      schedule => "* * * * *"
      #索引的类型
      type => "information"
    }
}
 
filter {
    json {
        source => "message"
        remove_field => ["message"]
    }
}
 
output {
    elasticsearch {
        hosts => "192.168.1.70:9200"
        # index名
        index => "information"
        # 需要关联的数据库中有有一个id字段，对应索引的id号
        document_id => "%{id}"
    }
    stdout {
        codec => json_lines
    }
}
```

3、启动logstash

然后通过以下命令启动logstash

.\logstash.bat -f  .\config-mysql\mysql.conf


过一会他就会自动的往ES里添加数据。



4、增量索引

但是现在有一个问题是：往elasticsearch里面写入是全量的，需要改成增量。

修改mysql.conf的内容如下


```conf
input {
    stdin {
    }
    jdbc {
      # 数据库
      jdbc_connection_string => "jdbc:mysql://39.107.60.74:3306/db_plat3"
      # 用户名密码
      jdbc_user => "root"
      jdbc_password => "letsgo123QWE"
      # jar包的位置
      jdbc_driver_library => "H:\software\logstash-6.1.2\logstash-6.1.2\lib\mysql-connector-java-5.1.38.jar"
      # mysql的Driver
      jdbc_driver_class => "com.mysql.jdbc.Driver"
 
 
 
     #使用其它字段追踪，而不是用时间
      use_column_value => true
      #追踪的字段
      tracking_column => id
      record_last_run => true
     #上一个sql_last_value值的存放文件路径, 必须要在文件中指定字段的初始值
last_run_metadata_path=>"H:\software\logstash-6.1.2\logstash-6.1.2\bin\config-mysql\station_parameter.txt"
     #开启分页查询
      jdbc_paging_enabled => "true"
      jdbc_page_size => "50000"
      statement_filepath => "config-mysql/information.sql"
      #statement => "select * from information where  id > :sql_last_value "
      schedule => "* * * * *"
      #索引的类型
      type => "information"
    }
}
 
filter {
    json {
        source => "message"
        remove_field => ["message"]
    }
}
 
output {
    elasticsearch {
        hosts => "192.168.1.70:9200"
        # index名
        index => "information"
        # 需要关联的数据库中有有一个id字段，对应索引的id号
        document_id => "%{id}"
    }
    stdout {
        codec => json_lines
    }
}
```
修改的内容见红色。



station_parameter.txt中的内容如下：


{% asset_img 14.png %}

logstash执行时打印出来的sql如下：


{% asset_img 15.png %}


其中遇到的一个问题是用：

statement => "select * from information where  id > :sql_last_value "

时会报:sql_last_value 的错 ，暂时不知道怎么解决。于是改用文件config-mysql/information.sql存在sql语句。



ps : 

 上述问题是因为从人家那里copy过来的时候是错的  把 :last_sql_value 改成 :sql_last_value  就可以了。



5、用时间来实现增量：

```conf
input {
stdin {
}
jdbc {
# 数据库
jdbc_connection_string => "jdbc:mysql://39.107.60.74:3306/db_plat3"
# 用户名密码
jdbc_user => "root"
jdbc_password => "letsgo123QWE"
# jar包的位置
jdbc_driver_library => "H:\software\logstash-6.1.2\logstash-6.1.2\lib\mysql-connector-java-5.1.38.jar"
# mysql的Driver
jdbc_driver_class => "com.mysql.jdbc.Driver"
#开启分页查询
jdbc_paging_enabled => "true"
jdbc_page_size => "50000"
#statement_filepath => "config-mysql/information.sql"
statement => "select * from information where modify_time > :sql_last_value"
schedule => "* * * * *"
#索引的类型
type => "information"
}
}
filter {
json {
source => "message"
remove_field => ["message"]
}
}
output {
elasticsearch {
hosts => "192.168.1.70:9200"
# index名
index => "information"
# 需要关联的数据库中有有一个id字段，对应索引的id号
document_id => "%{id}"
}
stdout {
codec => json_lines
}
}
```
modify_time 是我表里的最后修改时间字段。
而 :sql_last_value如果input里面use_column_value => true， 即如果设置为true的话，可以是我们设定的字段的上一次的值。 
默认 use_column_value => false， 这样 :sql_last_value为上一次更新的最后时刻值。 
也就是说，对于新增的值，才会更新。这样就实现了增量更新的目的。

{% asset_img 16.png %}

# 管理工具 

可视化

ealsticsearch只是后端提供各种api，那么怎么直观的使用它呢？elasticsearch-head将是一款专门针对于elasticsearch的客户端工具 

elasticsearch-head配置包，下载地址：https://github.com/mobz/elasticsearch-head

elasticsearch-head是一个基于node.js的前端工程，启动elasticsearch-head的步骤如下（这里针对的是elasticsearch 5.x以上的版本）：

　　1、进入elasticsearch-head的文件夹，如：D:\xwj_github\elasticsearch-head

　　2、执行 npm install

　　3、执行 npm run start

　在浏览器访问http://localhost:9100，可看到如下界面，表示启动成功：

仔细观察，我们会发现客户端默认连接的是我们elasticsearch的默认路径。而此时elasticsearch服务未启动，所以集群健康值是未连接

　　集群健康值的几种状态如下：

　　 　　绿色，最健康的状态，代表所有的分片包括备份都可用

　　　　黄色，基本的分片可用，但是备份不可用（也可能是没有备份）

　    　　红色，部分的分片可用，表明分片有一部分损坏。此时执行查询部分数据仍然可以查到，遇到这种情况，还是赶快解决比较好

       　　灰色，未连接到elasticsearch服务

 配合elasticsearch启动
ElasticSearch-head 和 elasticsearch 是两个功能，如果互相访问，是跨域问题。解决跨域问题，后才可以正常用elasticsearch-head 管理 elasticsearch。

修改config/elasticsearch.yml 文件(增加如下配置,中间为英文符号空格)
```yml
http.cors.enabled: true
http.cors.allow-origin: "*"
```
启动服务
重启elasticsearch-head 和 elasticsearch。访问elasticsearch-head

　此时，我们启动elasticsearch服务，重新刷新浏览器，发现集群健康值变成了黄色，如下：

1、概览

　　通过上图可以看到我们的节点名称为elasticsearch，并且该节点下有两个索引test_index1、test_index2

　　在test_index2下，选择信息-->索引信息，可以查看该索引的所有信息，包括mappings、setting等等

在test_index2下，选择动作-->关闭/开启，可以关闭/开启该索引，关闭后的索引如图：

在该界面也可以模糊查询索引、设置刷新频率等操作。如下图：

　　

 

2、索引

　　在这里，可以查看到所以的索引，并且还可以创建一个新的索引，如下图：

3、数据浏览

　　这里可看到索引、类型、字段、数据信息，如下图所示：

　　

　　关于这些名词表示的意思，可以参考https://www.cnblogs.com/luxiaoxun/p/4869509.html

4、基本查询

　　在这个页签，可以做数据进项简单的查询
　　　

　　选择一个索引，然后再选择不同的查询条件，勾选“显示查询语句”，最后点击搜索，可以看到具体的查询json和查询结果

　　至于不同组合的查询条件表示的意思，可以参考https://www.cnblogs.com/ljhdo/p/5040252.html
D:\Install\NetSarang\Xshell 6\;D:\Install\NetSarang\Xftp 6\;%SystemRoot%\system32;%SystemRoot%;%SystemRoot%\System32\Wbem;%SYSTEMROOT%\System32\WindowsPowerShell\v1.0\;C:\Program Files\Microsoft\Web Platform Installer\;D:\Soft\TortoiseSVN\bin;D:\Soft\VisualSVN Server\bin;C:\Program Files\TortoiseGit\bin;C:\Program Files\Git\cmd;E:\phpStudy\php\php-7.0.12-nts;C:\ProgramData\ComposerSetup\bin;D:\Install\Node.js\;D:\Soft\nodejs\node_global;D:\Install\phpStudy\PHPTutorial\php\php-7.2.1-nts;C:\Windows\Microsoft.NET\Framework\v4.0.30319

5、复合查询

　　在这个页签，可以使用json进行复杂的查询，也可发送put请求新增及跟新索引，使用delete请求删除索引等等。如图所示：

　

　　该页签的简单使用可以参考https://blog.csdn.net/bsh_csn/article/details/53908406　　

 
 

