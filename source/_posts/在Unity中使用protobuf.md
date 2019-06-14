---
layout: title
title: 在Unity中使用protobuf
date: 2019-06-14 11:47:53
categories: Unity
tags: 大话Unity2018
---
思考并回答以下问题：
1.把之前的聊天系统的消息写成一个proto文件，然后编译成C#代码放到Unity中，试试如何序列化和反序列化。

<!--more-->

上次我们已经可以把.proto文件编译生成的C#代码放到Unity中，并解决了报错，那么这些生成的代码如何在Unity中使用呢？

# 生成代码解析

上次课我们使用protobuf的编译器生成了一个Addressbook.cs代码，一起来看下这个代码里都有什么东西：

一个静态的 AddressbookReflection 类，包含protobuf消息的元数据。

一个AddressBook类包含一个只读的People属性。

一个Person类，包含Name， Id, Email 和 Phones属性。

一个PhoneNumber类，包含在静态的Person.Types类中。

一个 PhoneType枚举，也包含在Person.Types类中。

建议你浏览一下生成的代码，如果你想知道更多生成代码的细节，可以查看google的protobuf的文档：https://developers.google.com/protocol-buffers/docs/reference/csharp-generated

proto中的大部分类型，你都可以当作是C#原生的类型。唯一需要注意的是，repeated字段是只读的，你可以从集合中添加或者移除元素，但是你不能使用其他的集合替换现在的这个集合。repeated字段的集合类型是RepeatedField<T>，类似List<T>，但是提供了一些额外的便捷方法，比如Add方法的参数支持集合。

# 生成代码的使用

创建一个Person实例的代码示例如下：
```cs
using BigTalkUnity.AddressBook;
using UnityEngine;
using static BigTalkUnity.AddressBook.Person.Types;

public class Test : MonoBehaviour
{
    void Start()
    {
        Person john = new Person
        {
            Id = 1234,
            Name = "John Doe",
            Email = "jdoe@example.com",
            Phones = { new Person.Types.PhoneNumber { Number = "555-4321", Type = PhoneType.Home } }
        };
    }
}
```
注意上面的using static是C# 6引入的一个语法，可以用来引用静态类型。否则在使用时需要写冗长的前缀代码：
```cs
// C# 6之前需要这么写：
Person john = new Person
{
    Id = 1234,
    Name = "John Doe",
    Email = "jdoe@example.com",
    Phones = { new Person.Types.PhoneNumber { Number = "555-4321", Type = Person.Types.PhoneType.HOME } }
};
```

# 序列化

那么如何把一个ProtoBuf的message类序列化成所需要的数据呢？这里面有几种方法。

* 写入Stream
* 转为ByteString二进制串/byteArray二进制数组
* 转为Json字符串
```cs
using BigTalkUnity.AddressBook;
using UnityEngine;
using static BigTalkUnity.AddressBook.Person.Types;
using Google.Protobuf;
using System.IO;

public class Test : MonoBehaviour
{
    void Start()
    {
        Person john = new Person
        {
            Id = 1234,
            Name = "John Doe",
            Email = "jdoe@example.com",
            Phones = { new Person.Types.PhoneNumber { Number = "555-4321", Type = PhoneType.Home } }
        };

        // 写入stream
        using (var output = File.Create("john.dat"))
        {
            john.WriteTo(output);
        }

        // 转为json字符串
        var jsonStr = john.ToString();

        // 转为bytestring
        var byteStr = john.ToByteString();

        // 转为byte数组
        var byteArray = john.ToByteArray();
    }
}
```

# 解析
既然可以把protobuf的对象序列化成字节数组或者json字符串，那么如何反序列化解析出来呢？
```cs
using BigTalkUnity.AddressBook;
using UnityEngine;
using static BigTalkUnity.AddressBook.Person.Types;
using Google.Protobuf;
using System.IO;

public class Test : MonoBehaviour
{
    void Start()
    {
        Person john = new Person
        {
            Id = 1234,
            Name = "John Doe",
            Email = "jdoe@example.com",
            Phones = { new Person.Types.PhoneNumber { Number = "555-4321", Type = PhoneType.Home } }
        };

        // 写入stream
        using (var output = File.Create("john.dat"))
        {
            john.WriteTo(output);
        }

        // 转为json字符串
        var jsonStr = john.ToString();

        // 转为bytestring
        var byteStr = john.ToByteString();

        // 转为byte数组
        var byteArray = john.ToByteArray();

        // 1. 从stream中解析
        using (var input = File.OpenRead("john.dat"))
        {
            john = Person.Parser.ParseFrom(input);
        }

        // 2. 从字节串中解析
        john = Person.Parser.ParseFrom(byteStr);

        // 3. 从字节数组中解析
        john = Person.Parser.ParseFrom(byteArray);

        // 4. 从json字符串解析
        john = Person.Parser.ParseJson(jsonStr);
    }
}
```
# 总结

protobuf的用法有些类似之前使用过的XML或者Json，但是序列化出来的字节数更少，更适合在网络传输中使用。