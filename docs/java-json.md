# Java 中的 JSON

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-json>

## 1。概述

在 Java 中处理 JSON 数据可能很容易，但是——像 Java 中的大多数东西一样——有很多选项和库可供我们选择。

本指南将使选择变得更容易，并让你对生态系统有一个坚实的理解。我们将讨论 Java 中最常见的 JSON 处理库:

*   [杰克逊](#jackson)
*   [Gson](#gson)
*   json-io
*   [基因](#genson)

我们对每个库都遵循一个简单的结构——首先是一些有用的资源(包括 Baeldung 上的和外部的)。然后我们将回顾一个基本的代码示例，看看这个库实际上是怎样工作的。

## 2。人气和基本统计

首先，让我们以一些统计数据作为每个图书馆受欢迎程度的代表:

### 2.1。杰克逊

*   Maven 用法:[数据绑定](https://web.archive.org/web/20221221194720/https://mvnrepository.com/artifact/com.fasterxml.jackson.core/jackson-databind) ( **2362** )，[核心](https://web.archive.org/web/20221221194720/https://mvnrepository.com/artifact/com.fasterxml.jackson.core/jackson-core) ( **1377** )
*   [Github](https://web.archive.org/web/20221221194720/https://github.com/FasterXML/jackson) 明星: **1457**
*   Github 分叉: **585**

### 2.2。Gson

*   [美芬](https://web.archive.org/web/20221221194720/https://mvnrepository.com/artifact/com.google.code.gson/gson)用法: **1588**
*   [Github](https://web.archive.org/web/20221221194720/https://github.com/google/gson) 明星: **2079**
*   Github 分叉: **471**

### 2.3。json-io

*   [Maven](https://web.archive.org/web/20221221194720/https://mvnrepository.com/artifact/com.cedarsoftware/json-io) 用法: **11**
*   [Github](https://web.archive.org/web/20221221194720/https://github.com/jdereg/json-io) 明星: **129**
*   Github Forks: **40**

### 2.4。Genson

*   [Maven](https://web.archive.org/web/20221221194720/https://mvnrepository.com/artifact/com.owlike/genson) 用法: **8**
*   [Github](https://web.archive.org/web/20221221194720/https://github.com/owlike/genson) 明星: **67**
*   Github 分叉: **15**

## 3。杰克逊

接下来，我们来看看这其中最受欢迎的——杰克逊。Jackson 是一个用于处理 JSON 数据的多用途 Java 库。

### 3.1。有用的资源

以下是该图书馆的一些官方资源:

*   [杰克逊官方维基](https://web.archive.org/web/20221221194720/https://en.wikipedia.org/wiki/Jackson_(API))
*   [杰克逊在 Github 上](https://web.archive.org/web/20221221194720/https://github.com/FasterXML/jackson)

**上登出:**

*   [杰克逊教程](/web/20221221194720/https://www.baeldung.com/jackson)
*   [杰克逊日期](/web/20221221194720/https://www.baeldung.com/jackson-serialize-dates)
*   [杰克逊 JSON 观点](/web/20221221194720/https://www.baeldung.com/jackson-json-view-annotation)
*   [杰克逊注解指南](/web/20221221194720/https://www.baeldung.com/jackson-annotations)
*   [杰克逊异常——问题和解决方案](/web/20221221194720/https://www.baeldung.com/jackson-exception)
*   [Jackson–决定序列化/反序列化哪些字段](/web/20221221194720/https://www.baeldung.com/jackson-field-serializable-deserializable-or-not)
*   [杰克逊-双向关系](/web/20221221194720/https://www.baeldung.com/jackson-bidirectional-relationships-and-infinite-recursion)
*   [Jackson–定制串行器](/web/20221221194720/https://www.baeldung.com/jackson-custom-serialization)
*   [Jackson–定制解串器](/web/20221221194720/https://www.baeldung.com/jackson-deserialization)

**其他有趣的报道:**

*   [杰克逊 Java 中 JSON 处理 API 示例教程](https://web.archive.org/web/20221221194720/http://www.journaldev.com/2324/jackson-json-processing-api-in-java-example-tutorial)
*   [杰克逊–对象映射器](https://web.archive.org/web/20221221194720/https://jenkov.com/tutorials/java-json/jackson-objectmapper.html)
*   [Jackson 2–将 Java 对象转换为 JSON](https://web.archive.org/web/20221221194720/https://mkyong.com/java/jackson-2-convert-java-object-to-from-json/)

### 3.2。Maven 依赖关系

要使用这个库——下面是要添加到您的`pom.xml`中的 Maven 依赖项:

```java
<dependency>
    <groupId>com.fasterxml.jackson.core</groupId>
    <artifactId>jackson-databind</artifactId>
    <version>${jackson.version}</version>
</dependency>
```

注意[现在杰克森的最新版本](https://web.archive.org/web/20221221194720/https://search.maven.org/classic/#search|gav|1|g%3A%22com.fasterxml.jackson.core%22%20AND%20a%3A%22jackson-databind%22)是 **2.13** 。

### 3.3。杰克逊的简单例子

现在，让我们看看如何在一个简单的例子中使用 Jackson:

```java
@Test
public void whenSerializeAndDeserializeUsingJackson_thenCorrect() 
  throws IOException{
    Foo foo = new Foo(1,"first");
    ObjectMapper mapper = new ObjectMapper();

    String jsonStr = mapper.writeValueAsString(foo);
    Foo result = mapper.readValue(jsonStr, Foo.class);
    assertEquals(foo.getId(),result.getId());
}
```

请注意:

*   `ObjectMapper.writeValueAsString()`用于将对象序列化为 JSON 字符串。
*   `ObjectMapper.readValue()`用于将 JSON 字符串反序列化为 Java 对象。
*   JSON 输出示例:

```java
{
    "id":1,
    "name":"first"
}
```

## 4。Gson

Gson 是我们要研究的下一个 Java JSON 库。

### 4.1。有用的资源

以下是该图书馆的一些官方资源:

*   [Github 上的 Gson](https://web.archive.org/web/20221221194720/https://github.com/google/gson)
*   [Gson 用户指南](https://web.archive.org/web/20221221194720/https://sites.google.com/site/gson/gson-user-guide)

**上登出:**

*   [Gson 连载食谱](/web/20221221194720/https://www.baeldung.com/gson-serialization-guide)
*   [Gson 反序列化食谱](/web/20221221194720/https://www.baeldung.com/gson-deserialization-guide)

**其他有趣的报道:**

*   [Gson 排斥策略](https://web.archive.org/web/20221221194720/https://www.studytrails.com/2016/09/12/java-google-json-exclusion-strategy/)
*   [Gson 定制串行器/解串器](https://web.archive.org/web/20221221194720/https://www.studytrails.com/2016/09/12/java-google-json-custom-serializer-deserializer/)
*   [Java Gson + JSON 教程示例](https://web.archive.org/web/20221221194720/http://www.concretepage.com/google-api/java-gson-json-tutorial-examples)

### 4.2。Maven 依赖关系

```java
<dependency>
    <groupId>com.google.code.gson</groupId>
    <artifactId>gson</artifactId>
    <version>${gson.version}</version>
</dependency>
```

注意[目前 Gson 的最新版本](https://web.archive.org/web/20221221194720/https://search.maven.org/classic/#search|gav|1|g%3A%22com.google.code.gson%22%20AND%20a%3A%22gson%22)是 **2.8.8** 。

### 4.3。Gson 的简单例子

下面是一个简单例子，阐明了如何使用 Gson 来序列化/反序列化 JSON:

```java
@Test
public void whenSerializeAndDeserializeUsingGson_thenCorrect(){
    Gson gson = new Gson();
    Foo foo = new Foo(1,"first");

    String jsonStr = gson.toJson(foo);
    Foo result = gson.fromJson(jsonStr, Foo.class);
    assertEquals(foo.getId(),result.getId());
}
```

请注意:

*   `Gson.toJson()`用于将对象序列化到 JSON
*   `Gson.fromJson()`用于将 JSON 反序列化为 Java 对象

## 5。Json-io

Json-io 是一个简单的 Java 库，用于序列化/反序列化 Json。

### 5.1。有用的资源

以下是该图书馆的一些官方资源:

*   [谷歌代码上的 JSON-io](https://web.archive.org/web/20221221194720/https://code.google.com/archive/p/json-io/)
*   [Github 上的 JSON-io](https://web.archive.org/web/20221221194720/https://github.com/jdereg/json-io)

### 5.2。Maven 依赖关系

```java
<dependency>
    <groupId>com.cedarsoftware</groupId>
    <artifactId>json-io</artifactId>
    <version>${json-io.version}</version>
</dependency>
```

注意[目前 json-io 的最新版本](https://web.archive.org/web/20221221194720/https://search.maven.org/classic/#search|gav|1|g%3A%22com.cedarsoftware%22%20AND%20a%3A%22json-io%22)是 **4.13.0** 。

### 5.3。json-io 的简单例子

现在，让我们看一个使用 json-io 的简单例子:

```java
@Test
public void whenSerializeAndDeserializeUsingJsonio_thenCorrect(){
    Foo foo = new Foo(1,"first");

    String jsonStr = JsonWriter.objectToJson(foo);
    Foo result = (Foo) JsonReader.jsonToJava(jsonStr);
    assertEquals(foo.getId(),result.getId());
}
```

请注意:

*   `JsonWriter.objectToJson()`用于将对象序列化到 JSON。
*   `JsonReader.jsonToJava()`用于将 Json 反序列化为 Java 对象。
*   JSON 输出示例:

```java
{
    "@type":"org.baeldung.Foo",
    "id":1,
    "name":"first"
}
```

## 6。Genson

Genson 是一个 Java 和 Scala 到 JSON 的转换库，提供完整的数据绑定和流。

### 6.1。有用的资源

以下是该图书馆的一些官方资源:

*   [Genson 官网](https://web.archive.org/web/20221221194720/https://owlike.github.io/genson/)
*   [Github 上的 Genson](https://web.archive.org/web/20221221194720/https://github.com/owlike/genson)
*   [Genson 用户指南](https://web.archive.org/web/20221221194720/https://owlike.github.io/genson/Documentation/UserGuide/)
*   [字节数组的 Genson JSON 格式](https://web.archive.org/web/20221221194720/https://iten.rs/blog/it/genson-json-format-for-byte-arrays/)

### 6.2。Maven 依赖关系

```java
<dependency>
    <groupId>com.owlike</groupId>
    <artifactId>genson</artifactId>
    <version>${genson.version}</version>
</dependency>
```

注意[现在 Genson 的最新版本](https://web.archive.org/web/20221221194720/https://search.maven.org/classic/#search|gav|1|g%3A%22com.owlike%22%20AND%20a%3A%22genson%22)是 **1.6。**

### 6.3。带有 Genson 的简单示例

下面是一个使用库的简单示例:

```java
@Test
public void whenSerializeAndDeserializeUsingGenson_thenCorrect(){
    Genson genson = new Genson();
    Foo foo = new Foo(1,"first");

    String jsonStr = genson.serialize(foo);
    Foo result = genson.deserialize(jsonStr, Foo.class);
    assertEquals(foo.getId(),result.getId());
}
```

请注意:

*   `Genson.serialize()`用于将对象序列化到 JSON
*   `Genson.desrialize()`用于将 JSON 反序列化为 Java 对象

## 7。结论

在这篇快速概述文章中，我们了解了 Java 中最常见的 JSON 处理库。