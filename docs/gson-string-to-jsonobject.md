# 用 Gson 将字符串转换成 JsonObject

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/gson-string-to-jsonobject>

## 1。概述

当使用 Gson 库在 Java 中使用 JSON 时，我们有几个选项可以将原始的 JSON 转换成更容易使用的其他类或数据结构。

例如，我们可以[将 JSON 字符串转换成`Map<String, Object>`](/web/20221116220337/https://www.baeldung.com/gson-json-to-map) 或者[创建一个带有映射](/web/20221116220337/https://www.baeldung.com/gson-deserialization-guide)的定制类。然而，有时将我们的 JSON 转换成一个通用对象是很方便的。

在本教程中，我们将学习 [`Gson`](https://web.archive.org/web/20221116220337/https://github.com/google/gson) 如何从`String.`给我们一个`JsonObject`

## 2。Maven 依赖关系

首先，我们需要在我们的`pom.xml`中包含 *gson* 依赖性:

```java
<dependency>
    <groupId>com.google.code.gson</groupId>
    <artifactId>gson</artifactId>
    <version>2.8.5</version>
</dependency>
```

我们可以在 [Maven Central](https://web.archive.org/web/20221116220337/https://search.maven.org/search?q=g:com.google.code.gson%20AND%20a:gson&core=gav) 上找到`gson`的最新版本。

## 3。使用`JsonParser`

我们将研究的将 JSON `String`转换为`JsonObject`的第一种方法是使用`JsonParser`类的两步过程。

第一步，我们需要解析原始的`String`。

Gson 为我们提供了一个名为`JsonParser,`的解析器，它将指定的 JSON `String`解析成一个`JsonElements`的解析树:

```java
public JsonElement parse(String json) throws JsonSyntaxException
```

一旦我们在`JsonElement` 树中解析了`String`，我们将使用` getAsJsonObject()`方法，它将返回期望的结果。

让我们看看如何获得最终的`JsonObject`:

```java
String json = "{ \"name\": \"Baeldung\", \"java\": true }";
JsonObject jsonObject = new JsonParser().parse(json).getAsJsonObject();

Assert.assertTrue(jsonObject.isJsonObject());
Assert.assertTrue(jsonObject.get("name").getAsString().equals("Baeldung"));
Assert.assertTrue(jsonObject.get("java").getAsBoolean() == true);
```

## 4。使用`fromJson`

在我们的第二种方法中，我们将看到如何创建一个`Gson`实例并使用`fromJson`方法。该方法将指定的 JSON `String`反序列化为指定类的对象:

```java
public <T> T fromJson(String json, Class<T> classOfT) throws JsonSyntaxException
```

让我们看看如何使用这个方法来解析我们的 JSON `String`，将`JsonObject`类作为第二个参数传递:

```java
String json = "{ \"name\": \"Baeldung\", \"java\": true }";
JsonObject convertedObject = new Gson().fromJson(json, JsonObject.class);

Assert.assertTrue(convertedObject.isJsonObject());
Assert.assertTrue(convertedObject.get("name").getAsString().equals("Baeldung"));
Assert.assertTrue(convertedObject.get("java").getAsBoolean() == true);
```

## 5。结论

在这篇简短的文章中，我们学习了使用 Gson 库从 Java 中的 JSON 格式的`String`获取`JsonObject`的两种不同方法。我们应该使用更适合我们的中间 JSON 操作的方法。

像往常一样，这些例子的源代码可以在 GitHub 的[上找到。](https://web.archive.org/web/20221116220337/https://github.com/eugenp/tutorials/tree/master/json-modules/gson)