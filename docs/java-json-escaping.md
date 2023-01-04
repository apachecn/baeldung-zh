# 在 Java 中转义 JSON 字符串

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-json-escaping>

## 1.概观

在这个简短的教程中，我们将展示一些在 Java 中转义 JSON 字符串的方法。

我们将快速浏览一下最流行的 JSON 处理库，以及它们如何让转义成为一项简单的任务。

## 2.什么会出错？

让我们考虑一个简单但常见的用例，向 web 服务发送用户指定的消息。天真地，我们可能会尝试:

```java
String payload = "{\"message\":\"" + message + "\"}";
sendMessage(payload);
```

但是，真的，这会带来很多问题。

最简单的是，如果消息包含引用:

```java
{ "message" : "My "message" breaks json" }
```

更糟糕的是**用户可能故意破坏请求**的语义。如果他发送:

```java
Hello", "role" : "admin
```

那么消息就变成了:

```java
{ "message" : "Hello", "role" : "admin" }
```

最简单的方法是用适当的转义序列替换引号:

```java
String payload = "{\"message\":\"" + message.replace("\"", "\\\"") + "\"}";
```

然而，这种方法相当脆弱:

*   需要对每一个连接的值都这样做，我们需要时刻记住我们已经转义了哪些字符串
*   此外，随着消息结构随时间的变化，**这可能会成为一个令人头疼的维护问题**
*   而且**很难阅读，这使得它更容易出错**

简单地说，我们需要采用一种更通用的方法。遗憾的是，**原生 JSON 处理特性还处于 [JEP 阶段](https://web.archive.org/web/20220707143817/https://openjdk.java.net/jeps/198)T3，所以我们不得不将目光转向各种开源 JSON 库。**

还好有[几个](https://web.archive.org/web/20220707143817/https://json.org/) JSON 处理库。让我们快速浏览一下最受欢迎的三个。

## 3.JSON-java 库

在我们的评论中，最简单和最小的库是 [JSON-java](/web/20220707143817/https://www.baeldung.com/java-org-json) ，也称为`org.json`。

为了构造一个 JSON 对象，**我们简单地创建了一个`JSONObject `的实例，并且基本上把它当作一个`Map`** :

```java
JSONObject jsonObject = new JSONObject();
jsonObject.put("message", "Hello \"World\"");
String payload = jsonObject.toString();
```

这将使用“World”周围的引号并对其进行转义:

```java
{
   "message" : "Hello \"World\""
}
```

## 4.杰克逊图书馆

用于 JSON 处理的最流行和最通用的 Java 库之一是 [Jackson](/web/20220707143817/https://www.baeldung.com/jackson) 。

乍一看，**杰克森的行为与`org.json` :** 相似

```java
Map<String, Object> params = new HashMap<>();
params.put("message", "Hello \"World\"");
String payload = new ObjectMapper().writeValueAsString(params);
```

然而，Jackson 也可以支持 Java 对象的序列化。

因此，让我们通过将我们的消息包装在一个自定义类中来增强我们的示例:

```java
class Payload {
    Payload(String message) {
        this.message = message;
    }

    String message;

    // getters and setters
} 
```

然后，我们需要一个`ObjectMapper`的实例，我们可以将对象的实例传递给它:

```java
String payload = new ObjectMapper().writeValueAsString(new Payload("Hello \"World\"")); 
```

在这两种情况下，我们得到了和以前一样的结果:

```java
{
   "message" : "Hello \"World\""
}
```

如果我们有一个已经转义的属性，并且需要序列化它而不需要任何进一步的转义，我们可能希望在那个字段上使用 Jackson 的`@JsonRawValue`注释。

## 5.Gson 图书馆

Gson 是谷歌的一个库，T2 经常和 Jackson 并肩作战。

**当然，我们可以像对`org.json `一样再做一次:**

```java
JsonObject json = new JsonObject();
json.addProperty("message", "Hello \"World\"");
String payload = new Gson().toJson(gsonObject);
```

或者我们可以使用自定义对象，比如 Jackson:

```java
String payload = new Gson().toJson(new Payload("Hello \"World\""));
```

我们会再次得到相同的结果。

## 6.结论

在这篇短文中，我们看到了如何使用不同的开源库在 Java 中转义 JSON 字符串。

与本文相关的所有代码都可以在 Github 上找到[。](https://web.archive.org/web/20220707143817/https://github.com/eugenp/tutorials/tree/master/json-modules/json)