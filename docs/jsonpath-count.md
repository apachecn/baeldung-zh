# 用 JsonPath 计数

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/jsonpath-count>

## 1.概观

在这个快速教程中，**我们将探索如何使用 JsonPath 对 JSON 文档中的对象和数组进行计数。**

JsonPath 提供了一种标准机制来遍历 JSON 文档的特定部分。我们可以说 JsonPath 对于 JSON 就像 XPath 对于 XML 一样。

## 2.必需的依赖关系

我们使用下面的 [JsonPath](https://web.archive.org/web/20220715114006/https://github.com/json-path/JsonPath) Maven 依赖项，当然，它可以在 [Maven Central](https://web.archive.org/web/20220715114006/https://search.maven.org/classic/#search%7Cga%7C1%7Cg%3A%22com.jayway.jsonpath%22) 上获得:

```java
<dependency>
    <groupId>com.jayway.jsonpath</groupId>
    <artifactId>json-path</artifactId>
    <version>2.4.0</version>
</dependency>
```

## 3.样本 JSON

下面的 JSON 将用于说明这些示例:

```java
{
    "items":{
        "book":[
            {
                "author":"Arthur Conan Doyle",
                "title":"Sherlock Holmes",
                "price":8.99
            },
            {
                "author":"J. R. R. Tolkien",
                "title":"The Lord of the Rings",
                "isbn":"0-395-19395-8",
                "price":22.99
            }
        ],
        "bicycle":{
            "color":"red",
            "price":19.95
        }
    },
    "url":"mystore.com",
    "owner":"baeldung"
}
```

## 4.计算 JSON 对象数

根元素由美元符号“`$”`表示。在下面的 JUnit 测试中，我们用 JSON `String`和我们想要计数的 JSON 路径“$”调用`JsonPath.read()`:

```java
public void shouldMatchCountOfObjects() {
    Map<String, String> objectMap = JsonPath.read(json, "$");
    assertEquals(3, objectMap.keySet().size());
}
```

通过计算结果`Map,`的大小，我们知道 JSON 结构中给定路径上有多少元素。

## 5.计算 JSON 数组大小

在下面的 JUnit 测试中，我们查询 JSON 来查找包含所有在`items`元素下的`books`的数组:

```java
public void shouldMatchCountOfArrays() {
    JSONArray jsonArray = JsonPath.read(json, "$.items.book[*]");
    assertEquals(2, jsonArray.size());
}
```

## 6.结论

在本文中，我们介绍了一些关于如何在 JSON 结构中计数的基本例子。

您可以在[官方 JsonPath 文档](https://web.archive.org/web/20220715114006/https://github.com/json-path/JsonPath#path-examples)中探索更多的路径示例。

和往常一样，代码示例可以在 GitHub 库中找到。