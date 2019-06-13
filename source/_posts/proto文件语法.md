---
layout: title
title: proto文件语法
date: 2019-06-13 11:24:54
categories: Unity
tags: 大话Unity2018
---
思考题
1.把之前的聊天系统的消息写成一个proto文件试试。

<!--more-->

之前我们简要了解了了protobuf的工作流，第一步就是要定义.proto文件，今天我们来看看这个.proto的语法具体是什么。

# <span style="color:#039BE5;">.proto语法</span>

举个简单的地址簿的例子，从文件读写联系人的信息。每一个联系人有姓名、ID、email和电话。

比如下面这个地址簿的<span style="color:red;">addressbook.proto</span>文件：

## <span style="color:red;">.proto文件头</span>

<span style="color:red;">首先需要定义一个指定是proto3版本的标志位。如果没有定义这个标志，默认认为你使用proto2版本。
然后可选使用一个package声明，避免不同工程间的名称冲突。</span>
```cs
syntax = "proto3";
package bigtalkunity;

import "google/protobuf/timestamp.proto";
```
同时也<span style="color:red;">可以引用其他.proto中定义的message</span>。上面我们就是引用了protobuf中带的timestamp类型。注意引用的路径。

将.proto生成C#代码时，如果没有指定<span style="color:red;">csharp_namespace</span>字段，自动生成的类的命名空间就是package的名称。但是如果指定了下面的字段，就会生成到对应的命名空间中。
```cs
option csharp_namespace = "BigTalkUnity.AddressBook";
```
## <span style="color:red;">定义message类型</span>

下一步就可以定义具体的message了。

一个.proto文件中可以包含多个message。一个message包含了一个或多个包含类型的字段。简单数据类型包括bool，int32，float，double，string等，同时也可以嵌套其他message类型作为字段的类型。
```cs
message Person 
{
  string name = 1;
  int32 id = 2;  // Unique ID number for this person.
  string email = 3;

  enum PhoneType 
  {
    MOBILE = 0;
    HOME = 1;
    WORK = 2;
  }

  message PhoneNumber 
  {
    string number = 1;
    PhoneType type = 2;
  }

  repeated PhoneNumber phones = 4;

  google.protobuf.Timestamp last_updated = 5;
}

// Our address book file is just one of these.
message AddressBook 
{
  repeated Person people = 1;
}
```
上面的消息中，<span style="color:red;">Person</span>消息包含了<span style="color:red;">PhoneNumber</span>消息，<span style="color:red;">AddressBook</span>又包含了<span style="color:red;">Person消息，消息可以嵌套消息。

你也可以设置枚举<span style="color:red;">enum</span>类型，用于提前定义的数值列表。

** 字段规则 **

.proto中可以包含的数据类型以及和C#的对应关系如下：

| <center>** .proto Type ** </center>  | <center>** 备注 ** </center>  | <center>** C# Type ** </center>  |
| :-| :- | :- |
| double  |   | double  |
| float  |   | float  |
| int32  | 使用变长编码。对于负数效率不高，如果字段可能有负值，使用sint32  | int  |
| int64  | 使用变长编码。对于负数效率不高，如果字段可能有负值，使用sint64  | long  |
| uint32  | 使用变长编码  | uint  |
| uint64  | 使用变长编码  | ulong   |
| sint32  | 使用变长编码，有符号的整形数值。编码负数时比int32效率高  | int  |
| sint64  | 使用变长编码，有符号的整形数值。编码负数时比int64效率高  | long  |
| fixed32  | 总是4字节。如果值经常大于228，效率比uint32高。  | uint  |
| fixed64  | 总是8字节。如果值经常大于256，效率比uint32高。  | ulong  |
| sfixed32  | 总是4字节。  | int   |
| sfixed64  | 总是8字节。  | long   |
| bool  |   | bool  |
| string | 必须是UTF-8或7-bit ASCII编码的文本。  | string  |
| bytes  | 可以包含任意长度的字节  | ByteString  |
    

消息字段可以是以下一种：
* 可选字段：一个消息中可以有0个或1个这个字段的数据，不能超过1个
* <span style="color:red;">repeated</span>：重复字段，这个字段可以把包含任意数量（包含0）的数据，并且顺序会保留。

** 字段编号 **

每个字段后面的“=1”，“=2”是字段的唯一编号，用于二进制的编码，一旦消息投入使用，这个编号就不应该再修改。1-15编号使用一个字节（这个字节包含编号和数据类型），16-2047的会使用2个字节。所以基于优化的角度考虑，常用字段尽量放到1-15标记中，并且为后续的扩展预留。重复字段中的每个元素都需要重新编码标记号，因此重复字段特别适合此优化。

可以使用的编号最小是1，最大是2^29-1，也就是536,870,911。19000-19999之间的编号不能使用，因为是protobuf内部保留使用。

** 注释 **
在.proto文件中使用注释和C#中类似，单行注释使用//，多行注释使用/\* ... \*/。

** 枚举 **
当你定义一个message类型时，你可能想让一个字段只能是预设的某些值，这时候可以用枚举类型。
```cs
enum PhoneType
{
    MOBILE = 0;
    HOME = 1;
    WORK = 2;
}
```
枚举中第一个值必须是0，这是因为：
\*\* 必须有一个值为0，作为enum的默认值
\*\* 0值必须是枚举中的第一个元素，为了兼容proto2。

enum的值必须是32bit的整数，不建议使用负数。
enum既可以定义在message中，也可以单独定义。

** 使用其他message类型 ** 

你可以使用其他消息类型作为字段的类型，类似C#中的类或结构体。
```cs
message PhoneNumber
{
    string number = 1;
    PhoneType type = 2;
}
repeated PhoneNumber phones = 4;
```
## <span style="color:red;">更新.proto文件注意事项</span>

如果现有的消息类型不再满足你的所有需求 - 例如，你希望消息格式具有额外的字段 - 但你仍然希望使用使用旧格式创建的代码，请不要担心！在不破坏任何现有代码的情况下更新消息类型非常简单。请记住以下规则：

* 请勿更改任何现有字段的字段编号。

* 如果添加新字段，则使用“旧”消息格式按代码序列化的任何消息仍可由新生成的代码进行解析。你应该记住这些元素的默认值，以便新代码可以正确地与旧代码生成的消息进行交互。同样，你的新代码创建的消息可以由旧代码解析：旧的二进制文件在解析时只是忽略新字段。

* 只要在更新的消息类型中不再使用字段编号，就可以删除字段。你可能希望重命名该字段，可能添加前缀“OBSOLETE_”，或者保留字段编号，以便你的未来用户.proto不会意外地重复使用该号码。

* int32，uint32，int64，uint64，和bool都是兼容的-这意味着你可以改变这些类型到另一个类型而不破坏向前或向后兼容。如果从网络中解析出一个不适合相应类型的数字，你将获得与在C #中将该数字强制转换为该类型相同的效果（例如，如果将64位数字作为int32读取，它将被截断为32位）。

* sint32并且sint64彼此兼容但与其他整数类型不兼容。

* string bytes中只要是有效的UTF-8字节，它们是兼容的。

* bytes如果字节包含消息的编码版本，则兼容嵌入消息。

* fixed32与sfixed32兼容，fixed64与sfixed64兼容。

* enum与int32，uint32，int64，和uint64兼容（注意，如果他们不符合类型将被截断）。但请注意，在反序列化消息时，客户端代码可能会以不同方式对待它们：例如，enum消息中将保留未识别的proto3 类型，但在反序列化出来的消息如何表示它，是依赖于编程语言的。Int字段总是保留它们的值。

# 总结

编写.proto文件其实和C#中定义类或结构体很相似。