# 用 Java 从 URL 读取 JSON

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-read-json-from-url>

## 1.介绍

在这个快速教程中，我们将创建能够从任何 URL 读取 JSON 数据的方法。我们将从核心 Java 类开始。然后，我们将使用一些库来简化我们的代码。

## 2.使用核心 Java 类

**用 Java 从 URL 读取数据的最简单的方法之一是使用`URL`类。**要使用它，我们打开一个到`URL`的输入流，创建一个输入流阅读器，然后读取所有字符。我们将这些字符添加到一个`StringBuilder` 中，然后将其作为一个`String`返回:

```java
public static String stream(URL url) {
    try (InputStream input = url.openStream()) {
        InputStreamReader isr = new InputStreamReader(input);
        BufferedReader reader = new BufferedReader(isr);
        StringBuilder json = new StringBuilder();
        int c;
        while ((c = reader.read()) != -1) {
            json.append((char) c);
        }
        return json.toString();
    }
}
```

因此，代码中包含了许多样板文件。此外，如果我们想将我们的 [JSON](/web/20221011150328/https://www.baeldung.com/java-json) 转换成地图或者 [POJO](/web/20221011150328/https://www.baeldung.com/java-pojo-class) ，这将需要更多的代码。**即使使用新的 Java 11 [HttpClient](https://web.archive.org/web/20221011150328/https://baeldung.com/java-9-http-client) ，对于一个简单的 GET 请求来说也有很多代码。**此外，它对将响应从字符串转换成 POJO 没有帮助。因此，让我们探索更简单的方法来做到这一点。

## 3.使用 commons-io 和 org.json

一个非常受欢迎的图书馆是 [Apache Commons IO](/web/20221011150328/https://www.baeldung.com/apache-commons-io) 。**我们将使用`IOUtils`来读取一个 URL 并返回一个`String`。然后，为了将它转换成一个`JSONObject`，我们将使用 [JSON-Java (org.json)](/web/20221011150328/https://www.baeldung.com/java-org-json) 库。**这是来自[json.org](https://web.archive.org/web/20221011150328/https://json.org/)的 Java 参考实现。让我们用一种新的方法将它们结合起来:

```java
public static JSONObject getJson(URL url) {
    String json = IOUtils.toString(url, Charset.forName("UTF-8"));
    return new JSONObject(json);
}
```

有了`JSONObject`，我们可以为任何属性调用`get()`并获得一个`Object`。特定类型有类似命名的方法。例如:

```java
jsonObject.getString("stringProperty");
```

## 4.少跟杰克逊码了`ObjectMapper`

有许多解决方案可以将 JSON 转换成 POJO，反之亦然。但是，[杰克森](/web/20221011150328/https://www.baeldung.com/jackson)被广泛用于像[泽西](/web/20221011150328/https://www.baeldung.com/jersey-rest-api-with-spring)和其他 [JAX-RS](/web/20221011150328/https://www.baeldung.com/jax-rs-spec-and-implementations) 实现的项目中。让我们将我们需要的依赖项添加到我们的`pom.xml`:

```java
<dependency>
    <groupId>com.fasterxml.jackson.core</groupId>
    <artifactId>jackson-databind</artifactId>
    <version>2.13.3</version>
</dependency>
```

有了这个，我们不仅可以毫不费力地从 URL 读取 JSON，同时还可以将其转换成 POJO。

### 4.1.反序列化为泛型对象

杰克逊的大部分动作来自于 [`ObjectMapper`](/web/20221011150328/https://www.baeldung.com/jackson-object-mapper-tutorial) 。对于`ObjectMapper`来说，最常见的场景是给它一个`String`输入，然后获取一个对象。幸运的是， **`ObjectMapper`还可以直接从互联网 URL** 读取输入:

```java
public static JsonNode get(URL url) {
    ObjectMapper mapper = new ObjectMapper();
    return mapper.readTree(url);
}
```

有了`readTree()`，我们得到一个`JsonNode`，它是一个[树状的](/web/20221011150328/https://www.baeldung.com/java-binary-tree)结构。我们用它的`get()`方法读取属性:

```java
json.get("propertyName");
```

因此，如果我们不想的话，我们不需要将我们的响应映射到特定的类。

### 4.2.反序列化为自定义类

但是，对于更复杂的对象，创建一个表示我们期望的 JSON 结构的类是有帮助的。**我们可以使用[泛型](/web/20221011150328/https://www.baeldung.com/java-generics)来创建我们方法的一个版本，该版本能够将响应映射到任何我们想要的类，使用`readValue()`** :

```java
public static <T> T get(URL url, Class<T> type) {
    ObjectMapper mapper = new ObjectMapper();
    return mapper.readValue(url, type);
}
```

然后，只要我们的对象的属性和结构匹配，我们就会得到一个新的实例，其中填充了来自 JSON 响应的值。

## 5.结论

在本文中，我们学习了如何向 URL 发出请求并获得一个 JSON 字符串。然后，我们使用一些库来简化我们的代码。最后，我们读取了一个 JSON 响应，同时用几行代码将它映射到一个 POJO。

和往常一样，源代码可以从 GitHub 上的[处获得。](https://web.archive.org/web/20221011150328/https://github.com/eugenp/tutorials/tree/master/jackson-modules/jackson-conversions-2)