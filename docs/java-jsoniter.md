# Jsoniter 简介

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-jsoniter>

## 1.介绍

JavaScript Object Notation，或 JSON，作为一种数据交换格式，近年来受到了广泛的欢迎。Jsoniter 是一个新的 JSON 解析库，旨在提供比其他可用解析器更灵活、更高效的 JSON 解析。

在本教程中，我们将看到如何使用 Java 的 Jsoniter 库解析 JSON 对象。

## 2.属国

Jsoniter 的最新版本可以在 Maven 中央存储库中找到。

让我们从将依赖项添加到`pom.xml`开始:

```java
<dependency>
    <groupId>com.jsoniter<groupId> 
    <artifactId>jsoniter</artifactId>
    <version>0.9.23</version>
</dependency> 
```

类似地，我们可以将依赖项添加到我们的`build.gradle`文件中:

```java
compile group: 'com.jsoniter', name: 'jsoniter', version: '0.9.23' 
```

## 3.使用 Jsoniter 进行 JSON 解析

Jsoniter 提供了 3 个 API 来解析 JSON 文档:

*   绑定 API
*   任何 API
*   迭代器 API

让我们来看看上面的每一个 API。

### 3.1.使用绑定 API 进行 JSON 解析

bind API 使用传统的方式将 JSON 文档绑定到 Java 类。

让我们考虑带有学生详细信息的 JSON 文档:

```java
{"id":1,"name":{"firstName":"Joe","surname":"Blogg"}}
```

现在让我们定义`Student` 和`Name` 模式类来表示上面的 JSON:

```java
public class Student {
    private int id;
    private Name name;

    // standard setters and getters
}
```

```java
public class Name {
    private String firstName;
    private String surname;

    // standard setters and getters
}
```

使用 bind API 将 JSON 反序列化为 Java 对象非常简单。我们使用`JsonIterator`的`deserialize`方法:

```java
@Test
public void whenParsedUsingBindAPI_thenConvertedToJavaObjectCorrectly() {
    String input = "{\"id\":1,\"name\":{\"firstName\":\"Joe\",\"surname\":\"Blogg\"}}";

    Student student = JsonIterator.deserialize(input, Student.class);

    assertThat(student.getId()).isEqualTo(1);
    assertThat(student.getName().getFirstName()).isEqualTo("Joe");
    assertThat(student.getName().getSurname()).isEqualTo("Blogg");
}
```

`Student`模式类声明`id`的数据类型为`int` 数据类型`.` ，但是，如果我们收到的 JSON 包含`id`的`String`值而不是一个数字呢？例如:

```java
{"id":"1","name":{"firstName":"Joe","surname":"Blogg"}}
```

注意 JSON 中的`id` 这次是一个字符串值`“1”` 。Jsoniter 提供了`Maybe` 解码器来处理这种情况。

### 3.2.`Maybe`解码器

当 JSON 元素的数据类型模糊时，Jsoniter 的`Maybe`解码器就派上了用场。`student.id`字段的数据类型是模糊的——它可以是`String`或`int`。为了处理这个问题，我们需要使用`MaybeStringIntDecoder`注释我们的模式类中的`id` 字段:

```java
public class Student {
    @JsonProperty(decoder = MaybeStringIntDecoder.class)
    private int id;
    private Name name;

    // standard setters and getters
} 
```

我们现在甚至可以在`id`值为`String`时解析 JSON:

```java
@Test
public void givenTypeInJsonFuzzy_whenFieldIsMaybeDecoded_thenFieldParsedCorrectly() {
    String input = "{\"id\":\"1\",\"name\":{\"firstName\":\"Joe\",\"surname\":\"Blogg\"}}";

    Student student = JsonIterator.deserialize(input, Student.class);

    assertThat(student.getId()).isEqualTo(1); 
}
```

**类似地，Jsoniter 提供了其他解码器，比如`MaybeStringLongDecoder`和`MaybeEmptyArrayDecoder`。**

现在让我们假设我们期望收到一个带有`Student`细节的 JSON 文档，但是我们收到了下面的文档:

```java
{"error":404,"description":"Student record not found"} 
```

这里发生了什么？我们期待使用`Student`数据的成功响应，但我们收到了`error`响应。这是一个非常常见的场景，但是我们该如何处理呢？

一种方法是在提取`Student`数据之前执行`null`检查，看看我们是否收到了错误响应。然而，`null`检查会导致一些难以阅读的代码，如果我们有一个多层嵌套的 JSON，这个问题会变得更糟。

Jsoniter 使用`Any` API 进行解析来解决这个问题。

### 3.3.使用 Any API 进行 JSON 解析

**当 JSON 结构本身是动态的时，我们可以使用 Jsoniter 的`Any` API 来提供 JSON** 的无模式解析。这类似于将 JSON 解析成一个`Map<String, Object>`。

让我们像以前一样解析`Student` JSON，但是这次使用`Any` API:

```java
@Test
public void whenParsedUsingAnyAPI_thenFieldValueCanBeExtractedUsingTheFieldName() {
    String input = "{\"id\":1,\"name\":{\"firstName\":\"Joe\",\"surname\":\"Blogg\"}}";

    Any any = JsonIterator.deserialize(input);

    assertThat(any.toInt("id")).isEqualTo(1);
    assertThat(any.toString("name", "firstName")).isEqualTo("Joe");
    assertThat(any.toString("name", "surname")).isEqualTo("Blogg"); 
}
```

让我们来理解这个例子。首先，我们使用`JsonIterator.deserialize(..)`来解析 JSON。但是，在这个实例中，我们没有指定模式类。结果是类型`Any.`

接下来，我们使用字段名读取字段值。我们使用`Any.toInt`方法读取“id”字段值。`toInt`方法将“id”值转换成一个整数。类似地，我们使用`toString`方法将“名字”和“姓氏”字段值作为字符串值读取。

**使用`Any` API，我们还可以检查 JSON 中是否存在某个元素。**我们可以通过查找元素，然后检查查找结果的`valueType`来做到这一点。当元素不在 JSON 中时，`valueType`将会是`INVALID`。

例如:

```java
@Test
public void whenParsedUsingAnyAPI_thenFieldValueTypeIsCorrect() {
    String input = "{\"id\":1,\"name\":{\"firstName\":\"Joe\",\"surname\":\"Blogg\"}}";

    Any any = JsonIterator.deserialize(input);

    assertThat(any.get("id").valueType()).isEqualTo(ValueType.NUMBER);
    assertThat(any.get("name").valueType()).isEqualTo(ValueType.OBJECT);
    assertThat(any.get("error").valueType()).isEqualTo(ValueType.INVALID);
}
```

“id”和“name”字段出现在 JSON 中，因此它们的`valueType`分别是`NUMBER`和`OBJECT`。然而，JSON 输入没有名为“error”的元素，因此`valueType`是`INVALID`。

回到上一节末尾提到的场景，我们需要检测收到的 JSON 输入是成功响应还是错误响应。我们可以通过检查“error”元素的`valueType`来检查是否收到了错误响应:

```java
String input = "{\"error\":404,\"description\":\"Student record not found\"}";
Any response = JsonIterator.deserialize(input);

if (response.get("error").valueType() != ValueType.INVALID) {
    return "Error!! Error code is " + response.toInt("error");
}
return "Success!! Student id is " + response.toInt("id");
```

运行时，上述代码将返回`“Error!! Error code is 404”`。

接下来，我们将看看如何使用迭代器 API 来解析 JSON 文档。

### 3.4.使用迭代器 API 进行 JSON 解析

**如果我们希望手动执行绑定，我们可以使用 Jsoniter 的`Iterator` API。**让我们考虑一下 JSON:

```java
{"firstName":"Joe","surname":"Blogg"}
```

我们将使用前面使用的`Name` schema 类来解析使用`Iterator` API 的 JSON:

```java
@Test
public void whenParsedUsingIteratorAPI_thenFieldValuesExtractedCorrectly() throws Exception {
    Name name = new Name();    
    String input = "{\"firstName\":\"Joe\",\"surname\":\"Blogg\"}";
    JsonIterator iterator = JsonIterator.parse(input);

    for (String field = iterator.readObject(); field != null; field = iterator.readObject()) {
        switch (field) {
            case "firstName":
                if (iterator.whatIsNext() == ValueType.STRING) {
                    name.setFirstName(iterator.readString());
                }
                continue;
            case "surname":
                if (iterator.whatIsNext() == ValueType.STRING) {
                    name.setSurname(iterator.readString());
                }
                continue;
            default:
                iterator.skip();
        }
    }

    assertThat(name.getFirstName()).isEqualTo("Joe");
    assertThat(name.getSurname()).isEqualTo("Blogg");
}
```

让我们来理解上面的例子。首先，我们将 JSON 文档作为一个迭代器。我们使用产生的`JsonIterator`实例迭代 JSON 元素:

1.  我们首先调用返回下一个字段名的`readObject`方法(或者如果已经到达文档的末尾，则调用`null`)。
2.  如果我们对字段名不感兴趣，我们通过使用`skip`方法跳过 JSON 元素。否则，我们使用`whatIsNext` 方法检查元素的数据类型。调用`whatIsNext` 方法并不是强制性的，但是当字段的数据类型对我们来说是未知的时候会很有用。
3.  最后，我们使用`readString`方法提取 JSON 元素的值。

## 4.结论

在本文中，我们讨论了 Jsoniter 提供的将 JSON 文档解析为 Java 对象的各种方法。

首先，我们看了使用 schema 类解析 JSON 文档的标准方法。

接下来，我们分别使用`Maybe`解码器和`Any`数据类型来处理模糊数据类型和解析 JSON 文档时的动态结构。

最后，我们看了用于将 JSON 手动绑定到 Java 对象的`Iterator` API。

一如既往，本文中使用的示例的源代码 可以在 GitHub 的[上获得。](https://web.archive.org/web/20220625234641/https://github.com/eugenp/tutorials/tree/master/json-modules/json-2)