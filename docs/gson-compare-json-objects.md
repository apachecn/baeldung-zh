# 用 Gson 比较两个 JSON 对象

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/gson-compare-json-objects>

## 1.概观

JSON 是数据的字符串表示。我们可能希望在我们的算法或测试中比较这些数据。尽管可以比较包含 JSON 的字符串，**字符串比较对表示的差异**敏感，而不是内容。

为了克服这个问题并从语义上比较 JSON 数据，我们需要将数据加载到内存中的一个结构中，这个结构不受空白或对象键的顺序等因素的影响。

在这个简短的教程中，我们将使用 [Gson](https://web.archive.org/web/20221128052325/https://github.com/google/gson) 来解决这个问题，这是一个 JSON 序列化\反序列化库，可以在 JSON 对象之间进行深度比较。

## 2.不同字符串中语义相同的 JSON

让我们仔细看看我们试图解决的问题。

假设我们有两个字符串，代表相同的 JSON 数据，但是其中一个在末尾有一些额外的空格:

```java
String string1 = "{\"fullName\": \"Emily Jenkins\", \"age\": 27    }";
String string2 = "{\"fullName\": \"Emily Jenkins\", \"age\": 27}";
```

虽然 JSON 对象的内容是相同的，但是将上面的内容作为字符串进行比较会发现不同之处:

```java
assertNotEquals(string1, string2);
```

如果对象中键的顺序不同，也会发生同样的情况，即使 JSON 通常对此不敏感:

```java
String string1 = "{\"fullName\": \"Emily Jenkins\", \"age\": 27}";
String string2 = "{\"age\": 27, \"fullName\": \"Emily Jenkins\"}";
assertNotEquals(string1, string2);
```

这就是为什么我们会受益于使用 JSON 处理库来比较 JSON 数据。

## 3.Maven 依赖性

要使用 Gson，让我们首先添加 [Gson Maven 依赖关系](https://web.archive.org/web/20221128052325/https://search.maven.org/artifact/com.google.code.gson/gson):

```java
<dependency>
    <groupId>com.google.code.gson</groupId>
    <artifactId>gson</artifactId>
    <version>2.8.6</version>
</dependency>
```

## 4.将 JSON 解析成 Gson 对象

在我们深入比较对象之前，让我们看看 Gson 是如何在 Java 中表示 JSON 数据的。

在 Java 中使用 JSON 时，我们首先需要将 JSON 字符串转换成 Java 对象。Gson 提供了 [`JsonParser`](https://web.archive.org/web/20221128052325/https://www.javadoc.io/doc/com.google.code.gson/gson/2.6.2/com/google/gson/JsonParser.html) ，它将源 JSON 解析成一个`[JsonElement](https://web.archive.org/web/20221128052325/https://www.javadoc.io/doc/com.google.code.gson/gson/2.8.5/com/google/gson/JsonElement.html) `树:

```java
JsonParser parser = new JsonParser();
String objectString = "{\"customer\": {\"fullName\": \"Emily Jenkins\", \"age\": 27 }}";
String arrayString = "[10, 20, 30]";

JsonElement json1 = parser.parse(objectString);
JsonElement json2 = parser.parse(arrayString);
```

`JsonElement `是一个抽象类，代表 JSON 的一个元素。`parse` 方法返回一个`JsonElement`的实现；不是`JsonObject, JsonArray, JsonPrimitive`就是`JsonNull:`

```java
assertTrue(json1.isJsonObject());
assertTrue(json2.isJsonArray());
```

这些子类中的每一个。)**覆盖了`Object.equals` 方法，提供了有效的深度 JSON 比较。**

## 5.Gson 对比用例

### 5.1.比较两个简单的 JSON 对象

假设我们有两个字符串，代表简单的 JSON 对象，其中键的顺序是不同的:

第一个对象的`fullName`早于`age`:

```java
{
    "customer": {
        "id": 44521,
        "fullName": "Emily Jenkins",
        "age": 27
    }
}
```

第二个颠倒了顺序:

```java
{
    "customer": {
        "id": 44521,
        "age": 27,
        "fullName": "Emily Jenkins"
    }
}
```

我们可以简单地分析和比较它们:

```java
assertEquals(parser.parse(string1), parser.parse(string2));
```

在这种情况下，`JsonParser`返回一个`JsonObject`，它的 **`equals` 实现不是** **顺序敏感的**。

### 5.2.比较两个 JSON 数组

在 JSON 数组的情况下，`JsonParser` 将返回一个`JsonArray.`

如果我们有一个有序的数组:

```java
[10, 20, 30]
```

```java
assertTrue(parser.parse(string1).isJsonArray());
```

我们可以以不同的顺序将它与另一个进行比较:

```java
[20, 10, 30]
```

与`JsonObject`，**，`JsonArray`的`equals` 方法不同，是顺序敏感的**，所以这些数组不相等，这在语义上是正确的:

```java
assertNotEquals(parser.parse(string1), parser.parse(string2));
```

### 5.3.比较两个嵌套的 JSON 对象

正如我们前面看到的，`JsonParser` 可以解析 JSON 的树状结构。每个`JsonObject`和`JsonArray`包含其他`JsonElement`对象，这些对象本身可以是类型`JsonObject` 或`JsonArray`。

当我们使用`equals`时，它递归地比较所有成员，这意味着**嵌套对象也是可比较的:**

如果这是`string1`:

```java
{
  "customer": {
    "id": "44521",
    "fullName": "Emily Jenkins",
    "age": 27,
    "consumption_info": {
      "fav_product": "Coke",
      "last_buy": "2012-04-23"
    }
  }
}
```

而这个 JSON 是`string2`:

```java
{
  "customer": {
    "fullName": "Emily Jenkins",
    "id": "44521",
    "age": 27,
    "consumption_info": {
      "last_buy": "2012-04-23",
      "fav_product": "Coke"
   }
  }
}
```

那么我们还是可以用`equals` 的方法来比较它们:

```java
assertEquals(parser.parse(string1), parser.parse(string2));
```

## 6.结论

在这篇短文中，我们已经看到了将 JSON 作为`String`进行比较的挑战。我们已经看到 Gson 如何允许我们将这些字符串解析成支持比较的对象结构。

和往常一样，上面例子的源代码可以在 [GitHub](https://web.archive.org/web/20221128052325/https://github.com/eugenp/tutorials/tree/master/json-modules/gson) 上找到。