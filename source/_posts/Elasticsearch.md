---
layout: title
title: Elasticsearch
date: 2019-04-02 10:15:01
tags: 工具
---
ES的配置

<!--more-->

# 简介

使用的Elasticsearch版本是7.0.1。

全文搜索属于最常见的需求，开源的Elasticsearch是目前全文搜索引擎的首选。它可以快速地储存、搜索和分析海量数据。维基百科、Stack Overflow、Github都采用它。

Elastic的底层是开源库Lucene。但是，你没法直接用Lucene，必须自己写代码去调用它的接口。Elastic是Lucene 的封装，提供了REST API的操作接口，开箱即用。


Elasticsearch是一个实时的分布式搜索和分析引擎。它可以帮助你用前所未有的速度去处理大规模数据。ElasticSearch是一个基于Lucene的搜索服务器。它提供了一个分布式多用户能力的全文搜索引擎，基于RESTful web接口。<span style="color:red;">Elasticsearch是用Java开发的</span>，并作为Apache许可条款下的开放源码发布，是当前流行的企业级搜索引擎。设计用于云计算中，能够达到实时搜索，稳定，可靠，快速，安装使用方便。


# 安装 

## Linux

因为是用Java开发，所以必须要先安装Java。

Elastic 需要 Java 8 环境。如果你的机器还没安装 Java，可以参考这篇文章，注意要保证环境变量JAVA_HOME正确设置。

CentOS7 
```Bash
curl -L -O https://artifacts.elastic.co/downloads/elasticsearch/elasticsearch-7.0.1-linux-x86_64.tar.gz
```

** 开机启动 **

## Windows



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

2.1 Node 与 Cluster
Elastic 本质上是一个分布式数据库，允许多台服务器协同工作，每台服务器可以运行多个 Elastic 实例。

单个 Elastic 实例称为一个节点（node）。一组节点构成一个集群（cluster）。

2.2 Index
Elastic 会索引所有字段，经过处理后写入一个反向索引（Inverted Index）。查找数据的时候，直接查找该索引。

所以，Elastic 数据管理的顶层单位就叫做 Index（索引）。它是单个数据库的同义词。每个 Index （即数据库）的名字必须是小写。

下面的命令可以查看当前节点的所有 Index。

```
$ curl -X GET 'http://localhost:9200/_cat/indices?v'
```
2.3 Document
Index 里面单条的记录称为 Document（文档）。许多条 Document 构成了一个 Index。

Document 使用 JSON 格式表示，下面是一个例子。

```json
{
  "user": "张三",
  "title": "工程师",
  "desc": "数据库管理"
}
```
同一个 Index 里面的 Document，不要求有相同的结构（scheme），但是最好保持相同，这样有利于提高搜索效率。

# 三、新建和删除 Index

新建 Index，可以直接向 Elastic 服务器发出 PUT 请求。下面的例子是新建一个名叫weather的 Index。


$ curl -X PUT 'localhost:9200/weather'
服务器返回一个 JSON 对象，里面的acknowledged字段表示操作成功。


{
  "acknowledged":true,
  "shards_acknowledged":true
}
然后，我们发出 DELETE 请求，删除这个 Index。


$ curl -X DELETE 'localhost:9200/weather'

# 中文分词 
ik-analyzer
java开源中文分词器

IK Analyzer是一个开源的，基于java语言开发的轻量级的中文分词工具包。

## Windows系统中Elasticsearch安装中文分词插件elasticsearch-analysis-ik

前言
系统：Windows10

elasticsearch版本：5.6.6

中文分词版本：5.6.6（需要与elasticsearch版本匹配）

maven版本：3.5.5

安装
step1 官网下载合适的版本
下载页面地址：https://github.com/medcl/elasticsearch-analysis-ik

选择合适的版本，并下载：

 


step2 解压到某个目录下
目录结构如下：



step3 使用maven进行编译打包
打开dos命令行，进入解压后的根目录：



分别执行三个命令（前提是成功安装maven）：

mvn clean

mvn compile

mvn package



step4 解压编译后的zip包
上述3个命令操作之后，多了一个target文件夹：



找到releases下面的zip包



在elasticsearch的plugins文件夹下新建一个文件夹，命名为ik：



把maven打包之后的zip文件解压到ik文件夹里，如下图：



step5 检测是否安装成功
重新启动elasticsearch，可以启动说明安装成功！

OK, GAME OVER !

## Linux
首先，安装中文分词插件。这里使用的是 ik，也可以考虑其他插件（比如 smartcn）。


$ ./bin/elasticsearch-plugin install https://github.com/medcl/elasticsearch-analysis-ik/releases/download/v5.5.1/elasticsearch-analysis-ik-5.5.1.zip
上面代码安装的是5.5.1版的插件，与 Elastic 5.5.1 配合使用。

接着，重新启动 Elastic，就会自动加载这个新安装的插件。

然后，新建一个 Index，指定需要分词的字段。这一步根据数据结构而异，下面的命令只针对本文。基本上，凡是需要搜索的中文字段，都要单独设置一下。


$ curl -X PUT 'localhost:9200/accounts' -d '
{
  "mappings": {
    "person": {
      "properties": {
        "user": {
          "type": "text",
          "analyzer": "ik_max_word",
          "search_analyzer": "ik_max_word"
        },
        "title": {
          "type": "text",
          "analyzer": "ik_max_word",
          "search_analyzer": "ik_max_word"
        },
        "desc": {
          "type": "text",
          "analyzer": "ik_max_word",
          "search_analyzer": "ik_max_word"
        }
      }
    }
  }
}'
上面代码中，首先新建一个名称为accounts的 Index，里面有一个名称为person的 Type。person有三个字段。

user
title
desc
这三个字段都是中文，而且类型都是文本（text），所以需要指定中文分词器，不能使用默认的英文分词器。

Elastic 的分词器称为 analyzer。我们对每个字段指定分词器。


"user": {
  "type": "text",
  "analyzer": "ik_max_word",
  "search_analyzer": "ik_max_word"
}
上面代码中，analyzer是字段文本的分词器，search_analyzer是搜索词的分词器。ik_max_word分词器是插件ik提供的，可以对文本进行最大数量的分词。

# 数据操作

5.1 新增记录
向指定的 /Index/Type 发送 PUT 请求，就可以在 Index 里面新增一条记录。比如，向/accounts/person发送请求，就可以新增一条人员记录。


$ curl -X PUT 'localhost:9200/accounts/person/1' -d '
{
  "user": "张三",
  "title": "工程师",
  "desc": "数据库管理"
}' 
服务器返回的 JSON 对象，会给出 Index、Type、Id、Version 等信息。
```
{
  "_index":"accounts",
  "_type":"person",
  "_id":"1",
  "_version":1,
  "result":"created",
  "_shards":{"total":2,"successful":1,"failed":0},
  "created":true
}
```
如果你仔细看，会发现请求路径是/accounts/person/1，最后的1是该条记录的 Id。它不一定是数字，任意字符串（比如abc）都可以。

新增记录的时候，也可以不指定 Id，这时要改成 POST 请求。

```
$ curl -X POST 'localhost:9200/accounts/person' -d '
{
  "user": "李四",
  "title": "工程师",
  "desc": "系统管理"
}'
```
上面代码中，向/accounts/person发出一个 POST 请求，添加一个记录。这时，服务器返回的 JSON 对象里面，\_id字段就是一个随机字符串。

```
{
  "_index":"accounts",
  "_type":"person",
  "_id":"AV3qGfrC6jMbsbXb6k1p",
  "_version":1,
  "result":"created",
  "_shards":{"total":2,"successful":1,"failed":0},
  "created":true
}
```
注意，如果没有先创建 Index（这个例子是accounts），直接执行上面的命令，Elastic 也不会报错，而是直接生成指定的 Index。所以，打字的时候要小心，不要写错 Index 的名称。

5.2 查看记录
向/Index/Type/Id发出 GET 请求，就可以查看这条记录。


$ curl 'localhost:9200/accounts/person/1?pretty=true'
上面代码请求查看/accounts/person/1这条记录，URL 的参数pretty=true表示以易读的格式返回。

返回的数据中，found字段表示查询成功，\_source字段返回原始记录。
```

{
  "_index" : "accounts",
  "_type" : "person",
  "_id" : "1",
  "_version" : 1,
  "found" : true,
  "_source" : {
    "user" : "张三",
    "title" : "工程师",
    "desc" : "数据库管理"
  }
}
```
如果 Id 不正确，就查不到数据，found字段就是false。

```
$ curl 'localhost:9200/weather/beijing/abc?pretty=true'

{
  "_index" : "accounts",
  "_type" : "person",
  "_id" : "abc",
  "found" : false
}
```
5.3 删除记录
删除记录就是发出 DELETE 请求。
```

$ curl -X DELETE 'localhost:9200/accounts/person/1'
```
这里先不要删除这条记录，后面还要用到。

5.4 更新记录
更新记录就是使用 PUT 请求，重新发送一次数据。
```

$ curl -X PUT 'localhost:9200/accounts/person/1' -d '
{
    "user" : "张三",
    "title" : "工程师",
    "desc" : "数据库管理，软件开发"
}' 

{
  "_index":"accounts",
  "_type":"person",
  "_id":"1",
  "_version":2,
  "result":"updated",
  "_shards":{"total":2,"successful":1,"failed":0},
  "created":false
}
```
上面代码中，我们将原始数据从"数据库管理"改成"数据库管理，软件开发"。 返回结果里面，有几个字段发生了变化。

```
"_version" : 2,
"result" : "updated",
"created" : false
可以看到，记录的 Id 没变，但是版本（version）从1变成2，操作类型（result）从created变成updated，created字段变成false，因为这次不是新建记录。
```

# 数据查询 

6.1 返回所有记录
使用 GET 方法，直接请求/Index/Type/\_search，就会返回所有记录。

```
$ curl 'localhost:9200/accounts/person/_search'

{
  "took":2,
  "timed_out":false,
  "_shards":{"total":5,"successful":5,"failed":0},
  "hits":{
    "total":2,
    "max_score":1.0,
    "hits":[
      {
        "_index":"accounts",
        "_type":"person",
        "_id":"AV3qGfrC6jMbsbXb6k1p",
        "_score":1.0,
        "_source": {
          "user": "李四",
          "title": "工程师",
          "desc": "系统管理"
        }
      },
      {
        "_index":"accounts",
        "_type":"person",
        "_id":"1",
        "_score":1.0,
        "_source": {
          "user" : "张三",
          "title" : "工程师",
          "desc" : "数据库管理，软件开发"
        }
      }
    ]
  }
}
```
上面代码中，返回结果的 took字段表示该操作的耗时（单位为毫秒），timed_out字段表示是否超时，hits字段表示命中的记录，里面子字段的含义如下。

total：返回记录数，本例是2条。
max_score：最高的匹配程度，本例是1.0。
hits：返回的记录组成的数组。
返回的记录中，每条记录都有一个_score字段，表示匹配的程序，默认是按照这个字段降序排列。

6.2 全文搜索
Elastic 的查询非常特别，使用自己的查询语法，要求 GET 请求带有数据体。

```
$ curl 'localhost:9200/accounts/person/_search'  -d '
{
  "query" : { "match" : { "desc" : "软件" }}
}'
```
上面代码使用 Match 查询，指定的匹配条件是desc字段里面包含"软件"这个词。返回结果如下。

```
{
  "took":3,
  "timed_out":false,
  "_shards":{"total":5,"successful":5,"failed":0},
  "hits":{
    "total":1,
    "max_score":0.28582606,
    "hits":[
      {
        "_index":"accounts",
        "_type":"person",
        "_id":"1",
        "_score":0.28582606,
        "_source": {
          "user" : "张三",
          "title" : "工程师",
          "desc" : "数据库管理，软件开发"
        }
      }
    ]
  }
}
```
Elastic 默认一次返回10条结果，可以通过size字段改变这个设置。

```
$ curl 'localhost:9200/accounts/person/_search'  -d '
{
  "query" : { "match" : { "desc" : "管理" }},
  "size": 1
}'
```
上面代码指定，每次只返回一条结果。

还可以通过from字段，指定位移。

```
$ curl 'localhost:9200/accounts/person/_search'  -d '
{
  "query" : { "match" : { "desc" : "管理" }},
  "from": 1,
  "size": 1
}'
```
上面代码指定，从位置1开始（默认是从位置0开始），只返回一条结果。

6.3 逻辑运算
如果有多个搜索关键字， Elastic 认为它们是or关系。

```
$ curl 'localhost:9200/accounts/person/_search'  -d '
{
  "query" : { "match" : { "desc" : "软件 系统" }}
}'
```
上面代码搜索的是软件 or 系统。

如果要执行多个关键词的and搜索，必须使用布尔查询。

```
$ curl 'localhost:9200/accounts/person/_search'  -d '
{
  "query": {
    "bool": {
      "must": [
        { "match": { "desc": "软件" } },
        { "match": { "desc": "系统" } }
      ]
    }
  }
}'
```


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

两个文件： jdbc.conf  jdbc.sql 。名字随便起啦。

一个 mysql 的java 驱动包： mysql-connector-java-5.1.36-bin.jar

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
```sql
INSERT INTO hotel(hotel_name, photo_url) VALUES("马二帅酒店","images/madashuai.img");
```
找到 hotel 的id 

```sql
INSERT INTO hotel_account(hotel_id, finance_person) VALUES(15627, "马二帅");
```
大约30秒的时候查看es

{% asset_img 2.png %}

很好马二帅酒店已经增加进来了。

现在修改下 hotel_account 
```sql
UPDATE hotel_account SET finance_person = "马二帅修改为马小帅" where id = 1601;
```
{% asset_img 3.png %}


## Windows 

Logstash-input-jdbc插件高效又方便，而且可以设置定时任务。

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

报错了，需要把mysql.conf改成ANSI格式。


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

ealsticsearch只是后端提供各种api，那么怎么直观的使用它呢？elasticsearch-head是一款专门针对于elasticsearch的客户端工具 

elasticsearch-head配置包，下载地址：https://github.com/mobz/elasticsearch-head

elasticsearch-head是一个基于node.js的前端工程，启动elasticsearch-head的步骤如下（这里针对的是elasticsearch 5.x以上的版本）：

　　1、进入elasticsearch-head的文件夹，如：D:\xwj_github\elasticsearch-head

　　2、执行 npm install

　　3、执行 npm run start

在浏览器访问http://localhost:9100，可看到如下界面，表示启动成功：



仔细观察，我们会发现客户端默认连接的是我们elasticsearch的默认路径。而此时elasticsearch服务未启动，所以集群健康值是未连接

集群健康值的几种状态如下：

* 绿色，最健康的状态，代表所有的分片包括备份都可用
* 黄色，基本的分片可用，但是备份不可用（也可能是没有备份）
* 红色，部分的分片可用，表明分片有一部分损坏。此时执行查询部分数据仍然可以查到，遇到这种情况，还是赶快解决比较好
* 灰色，未连接到elasticsearch服务

## 配合elasticsearch启动

ElasticSearch-head 和 elasticsearch 是两个功能，如果互相访问，是跨域问题。解决跨域问题，后才可以正常用elasticsearch-head 管理 elasticsearch。

修改config/elasticsearch.yml 文件(增加如下配置,中间为英文符号空格)
```yml
http.cors.enabled: true
http.cors.allow-origin: "*"
```
## 启动服务

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


5、复合查询

在这个页签，可以使用json进行复杂的查询，也可发送put请求新增及跟新索引，使用delete请求删除索引等等。如图所示：

该页签的简单使用可以参考https://blog.csdn.net/bsh_csn/article/details/53908406　　


# logstash-input-jdbc配置说明

Logstash由三个组件构造成，分别是input、filter以及output。我们可以吧Logstash三个组件的工作流理解为：input收集数据，filter处理数据，output输出数据。至于怎么收集、去哪收集、怎么处理、处理什么、怎么发生以及发送到哪等等一些列的问题就是我们接下啦要讨论的一个重点。
我们今天先讨论input组件的功能和基本插件。前面我们意见介绍过了，input组件是Logstash的眼睛和鼻子，负责收集数据的，那么们就不得不思考两个问题，第一个问题要清楚的就是，元数据在哪，当然，这就包含了元数据是什么类型，属于什么业务；第二个问题要清楚怎么去拿到元数据。只要搞明白了这两个问题，那么Logstash的input组件就算是弄明白了。
对于第一个问题，元数据的类型有很多，比如说你的元数据可以是日志、报表、可以是数据库的内容等等。元数据是什么样子的我们不需要关心，我们要关系的是元数据是什么类型的，只要你知道元数据是什么类型的，你才能给他分类，或者说给他一个type，这很重要，type对于你后面的工作处理是非常有帮助的。所以第一个问题的重心元数据在吗，是什么，现在已经是清楚了。那么进行第二个问题。
第二个问题的核心是怎么拿到这些不同类型的原数据？这是一个真个input组件的核心内容了，我们分门别类的来看待这和解决个问题。
首先，我们肯定需要认同的，什么样的数据源，就需要使用什么样的方式去获取数据。
我们列举几种：
1、文件类型：文件类型，顾名思义，文件数据源，我们可以使用input组件的file插件来获取数据。file{}插件有很多的属性参数，我们可以张开讲解一下。具体内容在下面的代码中展示：

```
input{
    file{
        #path属性接受的参数是一个数组，其含义是标明需要读取的文件位置
        path => [‘pathA’，‘pathB’]
        #表示多就去path路径下查看是够有新的文件产生。默认是15秒检查一次。
        discover_interval => 15
        #排除那些文件，也就是不去读取那些文件
        exclude => [‘fileName1’,‘fileNmae2’]
        #被监听的文件多久没更新后断开连接不在监听，默认是一个小时。
        close_older => 3600
        #在每次检查文件列 表的时候， 如果一个文件的最后 修改时间 超过这个值， 就忽略这个文件。 默认一天。
        ignore_older => 86400
        #logstash 每隔多 久检查一次被监听文件状态（ 是否有更新） ， 默认是 1 秒。
        stat_interval => 1
        #sincedb记录数据上一次的读取位置的一个index
        sincedb_path => ’$HOME/. sincedb‘
        #logstash 从什么 位置开始读取文件数据， 默认是结束位置 也可以设置为：beginning 从头开始
        start_position => ‘beginning’
        #注意：这里需要提醒大家的是，如果你需要每次都从同开始读取文件的话，关设置start_position => beginning是没有用的，你可以选择sincedb_path 定义为 /dev/null
    }           

}
```
 

2、数据库类型：数据库类型的数据源，就意味着我们需要去和数据库打交道了是吗？是的！那是必须的啊，不然怎么获取数据呢。input组件如何获取数据库类的数据呢？没错，下面即将隆重登场的是input组件的JDBC插件jdbc{}。同样的，jdbc{}有很多的属性，我们在下面的代码中作出说明；

```
input{
    jdbc{
    #jdbc sql server 驱动,各个数据库都有对应的驱动，需自己下载
    jdbc_driver_library => "/etc/logstash/driver.d/sqljdbc_2.0/enu/sqljdbc4.jar"
    #jdbc class 不同数据库有不同的 class 配置
    jdbc_driver_class => "com.microsoft.sqlserver.jdbc.SQLServerDriver"
    #配置数据库连接 ip 和端口，以及数据库   
    jdbc_connection_string => "jdbc:sqlserver://200.200.0.18:1433;databaseName=test_db"
    #配置数据库用户名
    jdbc_user =>   
    #配置数据库密码
    jdbc_password =>
    #上面这些都不重要，要是这些都看不懂的话，你的老板估计要考虑换人了。重要的是接下来的内容。
    # 定时器 多久执行一次SQL，默认是一分钟
    # schedule => 分 时 天 月 年  
    # schedule =>  22     表示每天22点执行一次
    schedule => "  *"
    #是否清除 last_run_metadata_path 的记录,如果为真那么每次都相当于从头开始查询所有的数据库记录
    clean_run => false
    #是否需要记录某个column 的值,如果 record_last_run 为真,可以自定义我们需要表的字段名称，
    #此时该参数就要为 true. 否则默认 track 的是 timestamp 的值.
    use_column_value => true
    #如果 use_column_value 为真,需配置此参数. 这个参数就是数据库给出的一个字段名称。当然该字段必须是递增的，可以是 数据库的数据时间这类的
    tracking_column => create_time
    #是否记录上次执行结果, 如果为真,将会把上次执行到的 tracking_column 字段的值记录下来,保存到 last_run_metadata_path 指定的文件中
    record_last_run => true
    #们只需要在 SQL 语句中 WHERE MY_ID > :last_sql_value 即可. 其中 :sql_last_value 取得就是该文件中的值
    last_run_metadata_path => "/etc/logstash/run_metadata.d/my_info"
    #是否将字段名称转小写。
    #这里有个小的提示，如果你这前就处理过一次数据，并且在Kibana中有对应的搜索需求的话，还是改为true，
    #因为默认是true，并且Kibana是大小写区分的。准确的说应该是ES大小写区分
    lowercase_column_names => false
    #你的SQL的位置，当然，你的SQL也可以直接写在这里。
    #statement => SELECT * FROM tabeName t WHERE  t.creat_time > :sql_last_value
statement_filepath => "/etc/logstash/statement_file.d/my_info.sql" #数据类型，标明你属于那一方势力。单了ES哪里好给你安排不同的山头。 type => "my_info" } #注意：外载的SQL文件就是一个文本文件就可以了，还有需要注意的是，一个jdbc{}插件就只能处理一个SQL语句， #如果你有多个SQL需要处理的话，只能在重新建立一个jdbc{}插件。 }
```
 

好了，废话不多说了，接着第三种情况：

```
input {
  beats {
    #接受数据端口
    port => 5044
    #数据类型
    type => "logs"
  }
  #这个插件需要和filebeat进行配很这里不做多讲，到时候结合起来一起介绍。
}
```


现在我们基本清楚的知道了input组件需要做的事情和如何去做，当然他还有很多的插件可以进行数据的收集，比如说TCP这类的，还有可以对数据进行encode，这些感兴趣的朋友可以自己去查看，我说的只是我自己使用的。一般情况下我说的三种插件已经足够了。
今天的ELK种的Logstash的input组件就到这。后面还会讲述Logstash的另外另个组件filter和output。

 

   注意：如果看到这样的报错信息 Logstash could not be started because there is already another instance using the configured data directory.  If you wish to run multiple instances, you must change the "path.data" setting. 请执行命令：service logstash stop 然后在执行就可以了。


```
input{
	stdin{
	}
	jdbc{
		jdbc_connection_string => "jdbc:mysql://127.0.0.1:3306/yld"
		jdbc_user => "root"
		jdbc_password => "root"
		jdbc_driver_library => "D:\Downloads\logstash-7.1.0\logstash-7.1.0\lib\mysql-connector-java-5.1.38.jar"
		jdbc_driver_class => "com.mysql.jdbc.Driver"		
		jdbc_paging_enabled => "true"
		jdbc_page_size => "50000"
		statement => "select Ems_kctitle,tags from study_indextag"
		schedule => "* * * * *"
		type => "jdbc"
	}
}

filter {
	json{
		source => "message"
		remove_field => ["message"]
	}
}


output {
	elasticsearch{
		hosts => "127.0.0.1:9200"
		index => "ceshi"
		document_id => "%{id}"
	}
	
	stdout{
		codec => json_lines
	}
}
```

# PHP操作Elasticsearch

