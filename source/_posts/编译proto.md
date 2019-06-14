---
layout: title
title: 编译proto
date: 2019-06-14 09:55:26
categories: Unity
tags: 大话Unity2018
---
思考并回答以下：
1.把之前的聊天系统的消息写成一个proto文件，然后编译成C#代码放到Unity中。
2.如何编译.proto文件到C#代码，然后导入到Unity工程中？

<!--more-->

写好proto文件之后，就可以使用protobuf提供的工具，将.proto文件编译成对应编程语言的代码文件了。

# 将.proto编译成代码

定义好了proto文件之后，就可以使用protobuf提供的编译器，将proto文件编译成你想要的编程语言的代码。

首先需要下载proto的编译器程序protoc，地址是：
https://github.com/protocolbuffers/protobuf/releases





下载对应操作系统的protoc即可，在这我下载的是protoc win64版本。

下载好之后，我们需要用到压缩包里面bin目录下的protoc.exe。同时压缩包里还包含了protobuf自带的一些常用proto的定义，如果你的proto文件中引用了这些定义，也要解压出来。





对于我们上节课定义的addressbook.proto，将压缩包中bin\protoc.exe放到同一目录，将include\google也放到同一目录，使用以下命令进行编译：

protoc.exe --csharp_out=. Person.proto




完整的编译命令如下：

protoc -I=$SRC_DIR --csharp_out=$DST_DIR $SRC_DIR/addressbook.proto
其中-I等价于--proto_path，指定了编译时包含的目录，用于解析proto文件中的import指令，查找对应.proto文件所在的位置。

其中--csharp_out指定了将proto编译成C#的代码，如果你需要在别的编程语言中使用proto，可以将proto文件编译到对应语言，具体参数可以使用protoc -h查看。

最后一个参数是编译输出的文件的位置。

使用编译出的代码文件
编译完成后我们就得到了一个C#文件，是不是放到Unity工程中就直接能用了呢？





如果直接将C#文件放到Unity工程中，会有报错。根据报错的提示可以看出来，是少引用了google的相关代码或者dll？

google并没有提供这个dll的预编译版本，所以想要解决这个问题，有几个方式：

* 使用protobuf的C#源码，直接放到Unity工程中：[https://github.com/protocolbuffers/protobuf/tree/master/csharp/src/Google.Protobuf](https://github.com/protocolbuffers/protobuf/tree/master/csharp/src/Google.Protobuf) 
)
* 用protobuf的C#源码自行编译dll
* 使用neget中的dll

在这我们直接使用源码：





下载release（https://github.com/protocolbuffers/protobuf/releases）中的这个源码后，将protobuf-3.7.1\csharp\src中的Google.Protobuf目录直接全部放到Unity工程中即可。





在这我们用的是Unity2018.3的版本，默认.Net运行时是4.x，如果你使用了旧版本（2017-2018.2）的Unity，需要手动切换到4.x的运行时或使用支持.Net3.5的protobuf非官方版本（https://github.com/bitcraftCoLtd/protobuf3-for-unity
）。

# proto在Unity中的编译插件

每次修改、编译proto文件还是挺麻烦的，有没有什么好办法？这时候就需要一些额外的工具，工具就是为了提高生产效率而生的。在Github上有这么一个工具，可以在你修改proto文件时自动编译：https://github.com/5argon/protobuf-unity。
