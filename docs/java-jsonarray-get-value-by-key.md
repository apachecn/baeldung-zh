# 通过 JSONArray 中的键获取值

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-jsonarray-get-value-by-key>

## 1.概观

JSON 是一种轻量级和独立于语言的数据交换格式，用于大多数客户机-服务器通信。

`JSONObject`和`JSONArray`是大多数 JSON 处理库中常见的两个类。**`JSONObject`存储无序的键值对**，很像 Java `Map`实现。另一方面，`JSONArray`是有序的值序列，很像 Java 中的`List`或`Vector`。

在本教程中，我们将使用`[JSON-Java](https://web.archive.org/web/20220822140111/https://stleary.github.io/JSON-java/index.html)` ( `org.json`)库，并学习如何处理一个`JSONArray` 来提取给定键的值。如果需要，我们有这个图书馆的简介。

## 2.Maven 依赖性

我们将首先在 POM 中添加以下依赖项:

```java
<dependency>
    <groupId>org.json</groupId>
    <artifactId>json</artifactId>
    <version>20180813</version>
</dependency>
```

我们总能在 [Maven Central](https://web.archive.org/web/20220822140111/https://search.maven.org/classic/#search%7Cgav%7C1%7Cg%3A%22org.json%22%20AND%20a%3A%22json%22) 上找到`JSON-Java`的最新版本。

## 3.上下文构建

JSON 消息通常由 JSON 对象和数组组成，它们可以相互嵌套。一个`JSONArray`对象被括在方括号 `[ ]`中，而一个`JSONObject`被括在花括号`{}`中。例如，让我们考虑这个 JSON 消息:

```java
[
    {
        "name": "John",
        "city": "chicago",
        "age": "22"
    },
    {
        "name": "Gary",
        "city": "florida",
        "age": "35"
    },
    {
        "name": "Selena",
        "city": "vegas",
        "age": "18"
    }
]
```

显然，这是一个 JSON 对象数组。这个数组中的每个 JSON 对象代表我们的客户记录，属性或键是 `a name, age,` 和 `city`。

## 4.处理`JSONArray`

给定上面的 JSON，如果我们希望找出我们所有客户的名字，该怎么办？换句话说，给定一个键，在我们的例子中是`“name”` ，我们如何在给定的 JSON 数组中找到映射到该键的所有值呢？

我们知道，`JSONArray`是 JSON 对象的列表。所以，让我们找出给定键的所有值:

```java
public List<String> getValuesForGivenKey(String jsonArrayStr, String key) {
    JSONArray jsonArray = new JSONArray(jsonArrayStr);
    return IntStream.range(0, jsonArray.length())
      .mapToObj(index -> ((JSONObject)jsonArray.get(index)).optString(key))
      .collect(Collectors.toList());
}
```

在前面的示例中:

*   首先，我们遍历 JSON 数组中的整个对象列表
*   然后对于每个`JSONObject`，我们得到映射到给定键的值

同样，如果不存在这样的键，方法`optString()`返回一个空字符串。

在调用`getValuesForGivenKey(jsonArrayStr, “name”)` 时，其中 `jsonArrayStr` 是我们的示例 JSON，我们将得到所有名字的`List`作为输出:

```java
[John, Gary, Selena]
```

## 5.结论

在这篇简短的文章中，我们学习了如何解析一个`JSONArray`来获取给定键的所有映射值。在这里，我们使用了`JSON-Java (org.json)`库。

JSON.simple 是在 Java 中使用 JSON 的另一个类似且强大的选择。请随意探索。

像往常一样，Github 上有完整的源代码。