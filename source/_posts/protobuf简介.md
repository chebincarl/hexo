---
layout: title
title: protobuf简介
date: 2019-06-13 10:25:05
categories: Unity
tags: 大话Unity2018
---
思考并回答以下问题：
1.protobuf更适合用在什么场景？
2.protobuf的工作流是什么样的？
3.把之前的聊天系统的消息写成一个proto文件试试。

<!--more-->

上次我们在学习如何进行数据包封包的时候说到：在网络传输时数据包需要定义好格式，这个格式就是数据在字节流中是如何排列的，有时候这种数据排列的方式也称作协议（Protocol）。自定义协议灵活性很大，但是相应的复杂度也很高，如果客户端和服务器是两种不同的语言开发的，就需要分别进行协议的编码和解码，工作量会很大。不过，还有一个Protocol Buffers可以解救我们。

# <span style="color:#039BE5;">Protocol Buffers简介</span>

Protocol Buffers (通常简称为protobuf) 是Google开发的一种格式，这种格式与开发语言无关、与运行平台无关，用于序列化结构数据，并且很容易扩展。这种格式可以用于通信协议、数据存储等等。你可以想一下之前学过的xml或json，和他们有些类似，但是protobuf更小、更快、更简单。

protobuf是开源的，地址如下：[https://github.com/protocolbuffers/protobuf](https://github.com/protocolbuffers/protobuf) 这个是编译器的源码，用C++写的。

使用protobuf时，你只需要按照protobuf的格式要求将数据结构定义在.proto文件中一次，就可以通过protobuf的编译工具生成不同开发语言的数据结构<span color="red">代码</span>（就像我们在C#中序列化/反序列化xml或json时需要定义的class类），生成的代码中还封装了数据结构的读写和序列化。

# <span style="color:#039BE5;">为什么要用protobuf呢？</span>

举个简单的地址簿的例子，从文件读写联系人的信息。每一个联系人有姓名、ID、email和电话。

如果使用你已经学过的知识，如何序列化和反序列化这样的数据呢？有几种方式可以实现：

1、第一种，使用我们之前学过的XML或JSON。XML和JSON的好处就是它们具有自描述性，是人类可读的，几乎所有编程语言都有成熟的开源库来序列化/反序列化它们。如果你想要和其他程序进行数据交换，XML/JSON是很好的选择。但是，XML/JSON数据所占的空间比较大（因为有很多冗余的标签、字符），同时解析速度比较慢。后面我们会具体比较XML/JSON和protobuf的区别。

2、第二种，使用.NET的二进制序列化类：<span style="color:red;">System.Runtime.Serialization.Formatters.Binary.BinaryFormatter</span>。用这种方式，一旦改变数据结构会非常麻烦，之前已有的数据可能都无法使用了，并且与其他程序进行数据交换时很麻烦。

3、第三种，使用自定义的数据格式，比如上次我们学习到的格式。这种方式简单、快速，但是不方便的就是如果在多种语言之间传输，每种语言都要写解码、编码的代码。

protocol buffers正是google开发来解决上面提到的这些问题的。protobuf拥有灵活性、高效率、自动化的特性。使用protobuf时，你只需要写一个<span style="color:red;">.proto</span>的数据结构描述文件，protobuf提供的编译器就可以自动将.proto文件编译成你想要的编程语言的代码，<span style="color:red;">这些自动生成的代码里实现了编码、解码、读写这些格式数据的功能</span>。更重要的是，protobuf格式支持后续扩展，扩展后的格式仍然可以兼容旧格式。

# <span style="color:#039BE5;">详细对比XML/JSON</span>

上面我们多次提到了XML/JSON，它们有类似的地方，但是也有很多不同。

相对于XML或JSON，Protocol buffers有许多优点：
* 序列化出来的数据更精炼
* 序列化出来的字节数缩小3-10倍
* 序列化的性能快20-100倍
* 数据歧义更少
* 可以自动生成用于访问数据结构的代码

但同样，有一些情况下proto不如XML或JSON：
* XML/JSON拥有人类可读性、可编辑性，但是protobuf不可以（至少原生protobuf序列化的数据不可以）
* XML/JSON有自描述性，但是proto的数据没有意义，除非你能拿到定义文件（.proto）

所以通常protobuf更适合使用在对数据字节数比较敏感的场景，对序列化/反序列化性能要求高的场景。比如游戏开发这种高并发、数据包的频次非常高、手游对流量敏感就是一个很适合的场景。

# <span style="color:#039BE5;">protobuf的工作流</span>

protobuf有两个版本，分别是proto2和proto3，<span style="color:red;">后面我们学习的时候使用的都是新版的proto3</span>，如果你后续在查找资料时发现不相同的地方，一定要确认下是哪个版本。

** <span style="color:red;">1. 定义.proto文件</span> **
首先你要在<span style="color:red;">.ptoto</span>文件中定义你想要传输的数据结构。.proto文件很简单：使用message来定义每一个需要序列化的数据结构，每个message里面可以定义类型和名称。

** <span style="color:red;">2. 编译.proto文件</span> ** 
使用protobuf提供的编译工具，将.proto文件编译为对应开发语言的数据结构代码。

** <span style="color:red;">3. 在代码中序列化和解析</span> **
在代码中将数据结构序列化成字节数组<span style="color:red;">发送</span>出去，或者将<span style="color:red;">接收</span>到的字节数组反序列化成内存中的数据。
