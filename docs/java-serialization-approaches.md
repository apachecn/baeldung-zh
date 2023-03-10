# Java 的不同序列化方法

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-serialization-approaches>

## 1.概观

序列化是将对象转换为字节流的过程。然后，该对象可以保存到数据库或通过网络传输。相反的操作，从一系列字节中提取一个对象，就是反序列化。它们的主要目的是保存对象的状态，以便我们可以在需要时重新创建它。

在本教程中，**我们将探索 Java 对象**的 **不同的序列化方法。**

首先，我们将讨论 Java 的本地序列化 API。接下来，我们将探索支持 JSON 和 YAML 格式的库来做同样的事情。最后，我们将了解一些跨语言协议。

## 2.示例实体类

让我们从介绍一个简单的实体开始，我们将在整个教程中使用它:

```java
public class User {
    private int id;
    private String name;

    //getters and setters
} 
```

在接下来的部分中，我们将讨论最广泛使用的序列化协议。通过例子，我们将学习它们的基本用法。

## 3.Java 的本机序列化

[Java 中的序列化](/web/20221027185710/https://www.baeldung.com/java-serialization)有助于实现多个系统之间的有效和及时的通信。Java 指定了序列化对象的默认方式。Java 类可以覆盖这种默认的序列化，并定义自己序列化对象的方式。

Java 本地序列化的优点是:

*   这是一个简单但可扩展的机制
*   它以序列化形式维护对象类型和安全属性
*   可扩展以支持远程对象所需的封送和解封
*   **这是一个本地 Java 解决方案，所以它不需要任何外部库**

### 3.1。默认机制

根据 [Java 对象序列化规范](https://web.archive.org/web/20221027185710/https://docs.oracle.com/en/java/javase/11/docs/specs/serialization/index.html)，我们可以使用`ObjectOutputStream` 类的`writeObject()`方法来序列化对象。另一方面，我们可以使用属于`ObjectInputStream`类的`readObject()`方法来执行反序列化。

我们将用我们的*用户*类来说明这个基本过程。

首先，我们的类需要**实现`[Serializable](https://web.archive.org/web/20221027185710/https://docs.oracle.com/en/java/javase/15/docs/api/java.base/java/io/Serializable.html)`接口**:

```java
public class User implements Serializable {
    //fields and methods
}
```

接下来，我们需要给**添加 [`serialVersionU` `ID`属性](/web/20221027185710/https://www.baeldung.com/java-serial-version-uid)** :

```java
private static final long serialVersionUID = 1L;
```

现在，让我们创建一个`User`对象:

```java
User user = new User();
user.setId(1);
user.setName("Mark");
```

我们需要提供一个文件路径来保存数据:

```java
String filePath = "src/test/resources/protocols/user.txt"; 
```

现在，是时候将我们的`User`对象序列化为一个文件了:

```java
FileOutputStream fileOutputStream = new FileOutputStream(filePath);
ObjectOutputStream objectOutputStream = new ObjectOutputStream(fileOutputStream);
objectOutputStream.writeObject(user);
```

这里，我们使用`ObjectOutputStream`将`User`对象的状态保存到一个`“user.txt”`文件中。

另一方面，我们可以从同一个文件中读取`User`对象并反序列化它:

```java
FileInputStream fileInputStream = new FileInputStream(filePath);
ObjectInputStream objectInputStream = new ObjectInputStream(fileInputStream);
User deserializedUser = (User) objectInputStream.readObject();
```

最后，我们可以测试加载对象的状态:

```java
assertEquals(1, deserializedUser.getId());
assertEquals("Mark", deserializedUser.getName());
```

这是序列化 Java 对象的默认方式。在下一节中，我们将看到同样的自定义方式。

### 3.2。使用`Externalizable`接口定制序列化

当尝试序列化具有某些不可序列化属性的对象时，自定义序列化可能特别有用。这可以通过实现`[Externalizable](https://web.archive.org/web/20221027185710/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/io/Externalizable.html)` 接口来实现，该接口有两个方法:

```java
public void writeExternal(ObjectOutput out) throws IOException;

public void readExternal(ObjectInput in) throws IOException, ClassNotFoundException;
```

我们可以在想要序列化的类中实现这两个方法。详细的例子可以在我们关于 [`Externalizable`界面](/web/20221027185710/https://www.baeldung.com/java-externalizable)的文章中找到。

### 3.3。 **Java 序列化注意事项**

关于 Java 中的本机序列化，有一些注意事项:

*   **只有标有`Serializable`的对象才能持久化。**`Object`类没有实现`Serializable,`，因此，并不是 Java 中的所有对象都可以自动持久化
*   当一个类实现了 `Serializable`接口时，它的所有子类也是可序列化的。但是，**当一个对象引用另一个对象时，这些对象必须单独实现`Serializable`接口，否则会抛出 *NotSerializableException***
*   **如果我们想要控制版本控制，我们需要提供`serialVersionUID`属性。**该属性用于验证保存和加载的对象是否兼容。因此，我们需要确保它总是相同的，否则`InvalidClassException` 将被抛出
*   Java 序列化大量使用 I/O 流。我们需要在读或写操作后立即关闭流，因为如果我们忘记关闭流，我们将会以[资源泄漏](https://web.archive.org/web/20221027185710/https://en.wikipedia.org/wiki/Resource_leak) 结束。为了防止这样的资源泄露，我们可以使用`[try-with-resources](/web/20221027185710/https://www.baeldung.com/java-try-with-resources) `习语

## 4.Gson 图书馆

Google 的 [Gson](https://web.archive.org/web/20221027185710/https://sites.google.com/site/gson/) 是一个 Java 库，用于在 JSON 表示中序列化和反序列化 Java 对象。

Gson 是一个托管在 [GitHub](https://web.archive.org/web/20221027185710/https://github.com/google/gson) 的开源项目。一般来说，它提供了`toJson()`和`fromJson()`方法将 Java 对象转换成 JSON，反之亦然。

### 4.1.Maven 依赖性

让我们添加对 [Gson 库](https://web.archive.org/web/20221027185710/https://search.maven.org/classic/#search%7Cgav%7C1%7Cg%3A%22com.google.code.gson%22%20AND%20a%3A%22gson%22)的依赖:

```java
<dependency>
    <groupId>com.google.code.gson</groupId>
    <artifactId>gson</artifactId>
    <version>2.8.7</version>
</dependency>
```

### 4.2.Gson 序列化

首先，让我们创建一个`User`对象:

```java
User user = new User();
user.setId(1);
user.setName("Mark"); 
```

接下来，我们需要提供一个文件路径来保存 JSON 数据:

```java
String filePath = "src/test/resources/protocols/gson_user.json";
```

现在，让我们使用来自`Gson`类的`toJson()`方法将`User`对象序列化到“`gson_user.json”`文件中:

```java
Writer writer = new FileWriter(filePath);
Gson gson = new GsonBuilder().setPrettyPrinting().create();
gson.toJson(user, writer); 
```

### 4.3.Gson 反序列化

我们可以使用来自`Gson`类的`fromJson()`方法来反序列化 JSON 数据。

让我们读取 JSON 文件并将数据反序列化到一个`User`对象中:

```java
Gson gson = new GsonBuilder().setPrettyPrinting().create();
User deserializedUser = gson.fromJson(new FileReader(filePath), User.class);
```

最后，我们可以测试反序列化的数据:

```java
assertEquals(1, deserializedUser.getId());
assertEquals("Mark", deserializedUser.getName()); 
```

### 4.4.Gson 功能

Gson 具有许多重要特性，包括:

*   它可以处理集合、泛型类型和嵌套类
*   使用 Gson，我们还可以编写一个定制的序列化程序和/或反序列化程序，这样我们就可以控制整个过程
*   最重要的是，**它允许反序列化源代码不可访问的类的实例**
*   此外，如果我们的类文件在不同版本中被修改，我们可以使用版本控制特性。**我们可以对新添加的字段使用`@Since`注释，然后我们可以使用来自`GsonBuilder`** 的`setVersion()`方法

更多的例子，请查看我们关于 [Gson 序列化](/web/20221027185710/https://www.baeldung.com/gson-serialization-guide)和 [Gson 反序列化](/web/20221027185710/https://www.baeldung.com/gson-deserialization-guide)的食谱。

在本节中，我们使用 Gson API 序列化 JSON 格式的数据。在下一节中，我们将使用 Jackson API 来做同样的事情。

## 5.杰克逊 API

[Jackson](https://web.archive.org/web/20221027185710/https://github.com/FasterXML/jackson) 也被称为“Java JSON 库”或者“Java 最好的 JSON 解析器”。它提供了多种处理 JSON 数据的方法。

要全面了解杰克逊图书馆，我们的[杰克逊教程](/web/20221027185710/https://www.baeldung.com/jackson)是一个很好的起点。

### 5.1.Maven 依赖性

让我们为 [添加依赖杰克逊库](https://web.archive.org/web/20221027185710/https://search.maven.org/classic/#search%7Cga%7C1%7Cg%3A%22com.fasterxml.jackson.core%22) :

```java
<dependency>
    <groupId>com.fasterxml.jackson.core</groupId>
    <artifactId>jackson-core</artifactId>
    <version>2.12.4</version>
</dependency>
<dependency>
    <groupId>com.fasterxml.jackson.core</groupId>
    <artifactId>jackson-annotations</artifactId>
    <version>2.12.4</version>
</dependency>
<dependency>
    <groupId>com.fasterxml.jackson.core</groupId>
    <artifactId>jackson-databind</artifactId>
     <version>2.12.4</version>
</dependency> 
```

### 5.2.JSON 的 Java 对象

**我们可以使用属于`ObjectMapper`类的`writeValue()`方法将任何 Java 对象序列化为 JSON 输出。**

让我们从创建一个`User`对象开始:

```java
User user = new User();
user.setId(1);
user.setName("Mark Jonson"); 
```

之后，让我们提供一个文件路径来存储我们的 JSON 数据:

```java
String filePath = "src/test/resources/protocols/jackson_user.json";
```

现在，我们可以使用`ObjectMapper`类将`User`对象存储到 JSON 文件中:

```java
File file = new File(filePath);
ObjectMapper mapper = new ObjectMapper();
mapper.writeValue(file, user); 
```

这段代码将把我们的数据写到`“jackson_user.json”`文件中。

### 5.3.JSON 到 Java 对象

**简单的`ObjectMapper`方法`readValue()`是一个很好的切入点。**我们可以用它将 JSON 内容反序列化成 Java 对象。

让我们从 JSON 文件中读取`User`对象:

```java
User deserializedUser = mapper.readValue(new File(filePath), User.class);
```

我们总是可以测试加载的数据:

```java
assertEquals(1, deserializedUser.getId());
assertEquals("Mark Jonson", deserializedUser.getName()); 
```

### 5.4.杰克逊特征

*   Jackson 是一个坚实而成熟的 Jackson 序列化库
*   `ObjectMapper`类是序列化过程的入口点，它提供了一种简单的方法来解析和生成 JSON 对象，并且非常灵活
*   Jackson 库最大的优势之一是高度可定制的[序列化](/web/20221027185710/https://www.baeldung.com/jackson-custom-serialization)和[反序列化](/web/20221027185710/https://www.baeldung.com/jackson-deserialization)过程

到目前为止，我们看到了 JSON 格式的数据序列化。在下一节中，我们将探索使用 YAML 的序列化。

## 6.亚姆

[YAML](https://web.archive.org/web/20221027185710/http://yaml.org/) 代表“YAML 不是标记语言”。它是一种人类可读的数据序列化语言。**我们可以将 YAML 用于配置文件，以及我们想要存储或传输数据的应用程序中。**

在上一节中，我们看到了 Jackson API 处理 JSON 文件。我们还可以使用 Jackson APIs 来处理 YAML 文件。一个详细的例子可以在我们关于用 Jackson 解析 YAML 的文章中找到。

现在，让我们看看其他库。

### 6.1.YAML 豆

**[【YAML 豆】](https://web.archive.org/web/20221027185710/https://github.com/EsotericSoftware/yamlbeans)使得与 YAML 之间的 Java 对象图的序列化和反序列化变得容易。**

`YamlWriter`类用于将 Java 对象序列化为 YAML。`write()`方法通过识别公共字段和 bean 的 getter 方法来自动处理这个问题。

相反，我们可以使用`YamlReader`类将 YAML 反序列化为 Java 对象。`read()`方法读取 YAML 文档并将其反序列化为所需的对象。

首先，让我们为[YAML bean](https://web.archive.org/web/20221027185710/https://search.maven.org/classic/#search%7Cga%7C1%7Cg%3A%22com.esotericsoftware.yamlbeans%22%20AND%20a%3A%22yamlbeans%22)添加依赖项:

```java
<dependency>
    <groupId>com.esotericsoftware.yamlbeans</groupId>
    <artifactId>yamlbeans</artifactId>
    <version>1.15</version>
</dependency>
```

现在。让我们创建一个`User`对象的地图:

```java
private Map<String, User> populateUserMap() {
    User user1 = new User();
    user1.setId(1);
    user1.setName("Mark Jonson");
    //.. more user objects

    Map<String, User> users = new LinkedHashMap<>();
    users.put("User1", user1);
    // add more user objects to map

    return users;
}
```

之后，我们需要提供一个文件路径来存储我们的数据:

```java
String filePath = "src/test/resources/protocols/yamlbeans_users.yaml";
```

现在，我们可以使用`YamlWriter`类将地图序列化为 YAML 文件:

```java
YamlWriter writer = new YamlWriter(new FileWriter(filePath));
writer.write(populateUserMap());
writer.close(); 
```

另一方面，我们可以使用`YamlReader`类来反序列化地图:

```java
YamlReader reader = new YamlReader(new FileReader(filePath));
Object object = reader.read();
assertTrue(object instanceof Map); 
```

最后，我们可以测试加载的地图:

```java
Map<String, User> deserializedUsers = (Map<String, User>) object;
assertEquals(4, deserializedUsers.size());
assertEquals("Mark Jonson", (deserializedUsers.get("User1").getName()));
assertEquals(1, (deserializedUsers.get("User1").getId()));
```

### 6.2.SnakeYAML

[SnakeYAML](https://web.archive.org/web/20221027185710/https://bitbucket.org/asomov/snakeyaml/src/master/) 提供了一个高级 API 来将 Java 对象序列化为 YAML 文档，反之亦然。最新版本 1.2 可以与 JDK 1.8 或更高版本的 Java 一起使用。它可以解析 Java 结构，比如`String`、`List`和`Map`。

SnakeYAML 的入口点是`Yaml`类，它包含几个有助于序列化和反序列化的方法。

为了将 YAML 输入反序列化为 Java 对象，我们可以用`load()`方法加载单个文档，用`loadAll()`方法加载多个文档。这些方法接受一个`InputStream`，以及`String`对象。

反过来，我们可以使用 `dump()`方法将 Java 对象序列化为 YAML 文档。

一个详细的例子可以在我们关于用 SnakeYAML 解析 YAML 的文章中找到。

当然，SnakeYAML 可以很好地与 Java 一起工作，但是，它也可以与定制的 Java 对象一起工作。

在本节中，我们看到了将数据序列化为 YAML 格式的不同库。在下一节中，我们将讨论跨平台协议。

## 7.阿帕奇节俭

[Apache Thrift](https://web.archive.org/web/20221027185710/https://thrift.apache.org/) 最初由脸书开发，目前由 Apache 维护。

使用 Thrift 的最大好处是它**以较低的开销**支持跨语言序列化。此外，许多序列化框架只支持一种序列化格式，然而，Apache Thrift 允许我们从多种序列化格式中进行选择。

### 7.1.节俭特征

Thrift 提供了被称为协议的可插拔序列化器。这些协议提供了使用几种序列化格式中的任何一种进行数据交换的灵活性。支持的协议包括:

*   `TBinaryProtocol`使用二进制格式，因此处理速度比文本协议更快
*   `TCompactProtocol`是一种更紧凑的二进制格式，因此处理起来也更有效
*   `TJSONProtocol`使用 JSON 编码数据

Thrift 还支持容器类型的序列化——列表、集合和映射。

### 7.2.Maven 依赖性

为了在我们的应用程序中使用 Apache Thrift 框架，让我们添加 [Thrift 库](https://web.archive.org/web/20221027185710/https://search.maven.org/classic/#search%7Cgav%7C1%7Cg%3A%22org.apache.thrift%22%20AND%20a%3A%22libthrift%22):

```java
<dependency>
    <groupId>org.apache.thrift</groupId>
    <artifactId>libthrift</artifactId>
    <version>0.14.2</version>
</dependency>
```

### 7.3.节俭数据序列化

Apache Thrift 协议和传输被设计为作为一个分层堆栈一起工作。协议将数据序列化为字节流，传输读取和写入字节。

如前所述，Thrift 提供了许多协议。我们将使用二进制协议来说明节俭序列化。

首先，我们需要一个`User`对象:

```java
User user = new User();
user.setId(2);
user.setName("Greg");
```

下一步是创建二进制协议:

```java
TMemoryBuffer trans = new TMemoryBuffer(4096);
TProtocol proto = new TBinaryProtocol(trans);
```

现在，让我们序列化我们的数据`.` 我们可以使用`write`API 来完成:

```java
proto.writeI32(user.getId());
proto.writeString(user.getName());
```

### 7.4.节约数据反序列化

让我们使用`read`API 来反序列化数据:

```java
int userId = proto.readI32();
String userName = proto.readString();
```

最后，我们可以测试加载的数据:

```java
assertEquals(2, userId);
assertEquals("Greg", userName);
```

更多的例子可以在我们关于 [Apache Thrift](/web/20221027185710/https://www.baeldung.com/apache-thrift) 的文章中找到。

## 8.谷歌协议缓冲区

我们将在本教程中介绍的最后一种方法是 [Google 协议缓冲区](https://web.archive.org/web/20221027185710/https://developers.google.com/protocol-buffers/docs/javatutorial) (protobuf)。这是一种众所周知的二进制数据格式。

### 8.1.协议缓冲区的优势

协议缓冲区有几个好处，包括:

*   **它是语言和平台中立的**
*   这是一种二进制传输格式，意味着数据以二进制形式传输。这提高了传输速度，因为它占用更少的空间和带宽
*   **支持向后和向前兼容，以便新版本可以读取旧数据，反之亦然**

### 8.2.Maven 依赖性

让我们从添加对 [Google 协议缓冲库](https://web.archive.org/web/20221027185710/https://search.maven.org/classic/#search%7Cgav%7C1%7Cg%3A%22com.google.protobuf%22%20AND%20a%3A%22protobuf-java%22)的依赖开始:

```java
<dependency>
    <groupId>com.google.protobuf</groupId>
    <artifactId>protobuf-java</artifactId>
    <version>3.17.3</version>
</dependency>
```

### 8.3.定义协议

解决了依赖关系后，我们现在可以定义消息格式了:

```java
syntax = "proto3";
package protobuf;
option java_package = "com.baeldung.serialization.protocols";
option java_outer_classname = "UserProtos";
message User {
    int32 id = 1;
    string name = 2;
}
```

这是一个*用户*类型的简单消息的协议，它有两个字段，分别是`integer`和`string`类型的`id`和`name`。注意，我们将它保存为`“user.proto”`文件。

### 8.4.从 Protobuf 文件生成 Java 代码

一旦我们有了 protobuf 文件，我们就可以使用`protoc`编译器从它生成代码:

```java
protoc -I=. --java_out=. user.proto
```

因此，该命令将生成一个`UserProtos.java`文件。

之后，我们可以创建一个`UserProtos` 类的实例:

```java
UserProtos.User user = UserProtos.User.newBuilder().setId(1234).setName("John Doe").build();
```

### 8.5。序列化和反序列化 Protobuf

首先，我们需要提供一个文件路径来存储数据:

```java
String filePath = "src/test/resources/protocols/usersproto";
```

现在，让我们将数据保存到一个文件中。我们可以使用来自`UserProtos`类的`writeTo()`方法，这个类是我们从 protobuf 文件中生成的:

```java
FileOutputStream fos = new FileOutputStream(filePath);
user.writeTo(fos);
```

执行这段代码后，我们的对象将被序列化为二进制格式，并保存到“`usersproto`”文件中。

相反，我们可以使用`mergeFrom() `方法从文件中加载数据，并将其反序列化回一个`User` 对象:

```java
UserProtos.User deserializedUser = UserProtos.User.newBuilder().mergeFrom(new FileInputStream(filePath)).build(); 
```

最后，我们可以测试加载的数据:

```java
assertEquals(1234, deserializedUser.getId());
assertEquals("John Doe", deserializedUser.getName());
```

## 9.摘要

在本教程中，我们探讨了一些广泛使用的 Java 对象序列化协议。为应用程序选择数据序列化格式取决于各种因素，如数据复杂性、人类可读性需求和速度。

Java 支持易于使用的内置序列化。

JSON 由于可读性和无模式性而更受欢迎。因此，Gson 和 Jackson 都是序列化 JSON 数据的好选择。它们易于使用，并且有据可查。**对于编辑数据来说， YAML 很适合。**

另一方面，二进制格式比文本格式更快。当速度对我们的应用程序很重要时，Apache Thrift 和 Google Protocol Buffers 是序列化数据的绝佳选择。两者都比 XML 或 JSON 格式更紧凑、更快速。

综上所述，在便利性和性能之间经常会有一个权衡，序列化也不例外。当然，还有许多其他的[格式可用于数据序列化](https://web.archive.org/web/20221027185710/https://en.wikipedia.org/wiki/Comparison_of_data-serialization_formats#Syntax_comparison_of_human-readable_formats)。

和往常一样，完整的示例代码在 GitHub 上的[。](https://web.archive.org/web/20221027185710/https://github.com/eugenp/tutorials/tree/master/libraries-data-io)