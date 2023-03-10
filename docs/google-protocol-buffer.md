# Google 协议缓冲区介绍

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/google-protocol-buffer>

## 1。概述

在本文中，我们将关注 [Google 协议缓冲区](https://web.archive.org/web/20220626074844/https://developers.google.com/protocol-buffers/)(proto buf)——一种众所周知的语言无关的二进制数据格式。我们可以用一个协议定义一个文件，然后使用这个协议，我们可以用 Java、C++、C#、Go 或 Python 等语言生成代码。

这是一篇介绍格式本身的文章；如果您想了解如何在 Spring web 应用程序中使用这种格式，可以看看这篇文章。

## 2。定义 Maven 依赖关系

要使用协议缓冲区是 Java，我们需要添加一个 Maven 依赖到一个 [protobuf-java](https://web.archive.org/web/20220626074844/https://search.maven.org/classic/#search%7Cgav%7C1%7Cg%3A%22com.google.protobuf%22%20AND%20a%3A%22protobuf-java%22) :

```java
<dependency>
    <groupId>com.google.protobuf</groupId>
    <artifactId>protobuf-java</artifactId>
    <version>${protobuf.version}</version>
</dependency>

<properties>
    <protobuf.version>3.2.0</protobuf.version>
</properties>
```

## 3。定义协议

先说个例子。我们可以用 protobuf 格式定义一个非常简单的协议:

```java
message Person {
    required string name = 1;
}
```

这是一个简单的`Person`类型消息的协议，它只有一个必填字段——name，其类型为`string` 。

让我们来看一个更复杂的定义协议的例子。假设我们需要以 protobuf 格式存储个人信息:

包 protobuf

```java
package protobuf;

option java_package = "com.baeldung.protobuf";
option java_outer_classname = "AddressBookProtos";

message Person {
    required string name = 1;
    required int32 id = 2;
    optional string email = 3;

    repeated string numbers = 4;
}

message AddressBook {
    repeated Person people = 1;
}
```

我们的协议由两种类型的数据组成:生成代码后的一个`Person` 和一个`AddressBook.`(在后面的部分会有更多的介绍)，这些类将是`AddressBookProtos` 类中的内部类。

当我们想要定义一个必需的字段时——这意味着创建一个没有这个字段的对象将会导致一个`Exception`,我们需要使用一个`required` 关键字。

用`optional` 关键字创建一个字段意味着这个字段不需要被设置。`repeated` 关键字是可变大小的数组类型。

所有字段都被编入索引–标有数字 1 的字段将被保存为二进制文件中的第一个字段。标有 2 的字段将被保存，依此类推。这让我们可以更好地控制字段在内存中的布局。

## 4。从 Protobuf 文件生成 Java 代码

一旦我们定义了一个文件，我们就可以从中生成代码。

首先，我们需要在我们的机器上安装 protobuf 。一旦我们这样做了，我们就可以通过执行一个`protoc`命令来生成代码:

```java
protoc -I=. --java_out=. addressbook.proto
```

`protoc`命令将从我们的`addressbook.proto` 文件*生成 Java 输出文件。*`-I`选项指定了一个`proto` 文件所在的目录。`java-out` 指定了生成的类将被创建的目录。

生成的类将为我们定义的消息提供 setters、getters、constructors 和 builders。它还会有一些 util 方法来保存 protobuf 文件，并将它们从二进制格式反序列化为 Java 类。

## 5。创建 Protobuf 定义的消息的实例

我们可以很容易地使用生成的代码来创建一个`Person` 类的 Java 实例:

```java
String email = "[[email protected]](/web/20220626074844/https://www.baeldung.com/cdn-cgi/l/email-protection)";
int id = new Random().nextInt();
String name = "Michael Program";
String number = "01234567890";
AddressBookProtos.Person person =
  AddressBookProtos.Person.newBuilder()
    .setId(id)
    .setName(name)
    .setEmail(email)
    .addNumbers(number)
    .build();

assertEquals(person.getEmail(), email);
assertEquals(person.getId(), id);
assertEquals(person.getName(), name);
assertEquals(person.getNumbers(0), number);
```

我们可以通过对所需的消息类型使用`newBuilder()` 方法来创建一个 fluent 构建器。设置完所有必填字段后，我们可以调用一个`build()` 方法来创建一个`Person` 类的实例。

## 6。序列化和反序列化 Protobuf

一旦我们创建了我们的`Person` 类的一个实例，我们希望以与创建的协议兼容的二进制格式保存在磁盘上。假设我们想要创建一个`AddressBook` 类的实例，并向该对象添加一个人。

接下来，我们希望将该文件保存到光盘上——我们可以使用自动生成代码中的一个`writeTo()` util 方法:

```java
AddressBookProtos.AddressBook addressBook 
  = AddressBookProtos.AddressBook.newBuilder().addPeople(person).build();
FileOutputStream fos = new FileOutputStream(filePath);
addressBook.writeTo(fos);
```

执行该方法后，我们的对象将被序列化为二进制格式并保存在磁盘上。要从磁盘加载数据并将其反序列化回`AddressBook` 对象，我们可以使用一个`mergeFrom()` 方法:

```java
AddressBookProtos.AddressBook deserialized
  = AddressBookProtos.AddressBook.newBuilder()
    .mergeFrom(new FileInputStream(filePath)).build();

assertEquals(deserialized.getPeople(0).getEmail(), email);
assertEquals(deserialized.getPeople(0).getId(), id);
assertEquals(deserialized.getPeople(0).getName(), name);
assertEquals(deserialized.getPeople(0).getNumbers(0), number);
```

## 7。结论

在这篇简短的文章中，我们介绍了以二进制格式描述和存储数据的标准——Google 协议缓冲区。

我们创建了一个简单的协议，创建了符合已定义协议的 Java 实例。接下来，我们看到了如何使用 protobuf 序列化和反序列化对象。

所有这些示例和代码片段的实现都可以在 [GitHub 项目](https://web.archive.org/web/20220626074844/https://github.com/eugenp/tutorials/tree/master/protobuffer)中找到——这是一个 Maven 项目，因此它应该很容易导入和运行。