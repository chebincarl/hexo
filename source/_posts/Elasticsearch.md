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

全文搜索属于最常见的需求，开源的Elasticsearch（以下简称Elastic）是目前全文搜索引擎的首选。它可以快速地储存、搜索和分析海量数据。维基百科、Stack Overflow、Github都采用它。

Elastic的底层是开源库Lucene。但是，你没法直接用Lucene，必须自己写代码去调用它的接口。Elastic是Lucene 的封装，提供了REST API的操作接口，开箱即用。

# 安装 

不管三七二十一，先安装试用。


