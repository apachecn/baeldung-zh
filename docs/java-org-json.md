# JSON-Java 简介(org.json)

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-org-json>

## 1。概述

JSON (JavaScript Object Notation)是一种轻量级的数据交换格式，我们最常用它进行客户端-服务器通信。它既易于读写，又与语言无关。一个 JSON 值可以是另一个 JSON `object`、 `array`、 `number`、 `string`、 `boolean`(真/假)或`null`。

在本教程中，我们将看到如何使用一个可用的 JSON 处理库来创建、操作和解析 JSON——[JSON-Java](https://web.archive.org/web/20221012233633/https://stleary.github.io/JSON-java/index.html)库，也称为`org.json`。

## 延伸阅读:

## [迭代 org.json.JSONObject 的实例](/web/20221012233633/https://www.baeldung.com/jsonobject-iteration)

Learn how to iterate and traverse through a JSONObject[Read more](/web/20221012233633/https://www.baeldung.com/jsonobject-iteration) →

## [在 Java 中转义 JSON 字符串](/web/20221012233633/https://www.baeldung.com/java-json-escaping)

Learn ways to escape a JSON String core Java or a library[Read more](/web/20221012233633/https://www.baeldung.com/java-json-escaping) →

## 2。先决条件

我们首先需要在我们的`pom.xml`中添加以下依赖项:

```java
<dependency>
    <groupId>org.json</groupId>
    <artifactId>json</artifactId>
    <version>20180130</version>
</dependency>
```

最新版本可以在 [Maven 中央存储库](https://web.archive.org/web/20221012233633/https://search.maven.org/classic/#search%7Cgav%7C1%7Cg%3A%22org.json%22%20AND%20a%3A%22json%22)中找到。

注意，这个包已经包含在 Android SDK 中，所以我们不应该在使用它的同时包含它。

## 3。Java 中的 JSON【包 org . JSON】

[JSON-Java](https://web.archive.org/web/20221012233633/https://stleary.github.io/JSON-java/index.html) 库也称为`org.json`(不要与 Google 的 [org.json.simple](https://web.archive.org/web/20221012233633/https://code.google.com/archive/p/json-simple/) 混淆)为我们提供了用于在 Java 中解析和操作 JSON 的类。

此外，该库还可以在 JSON、XML、HTTP 头、Cookies、逗号分隔的列表或文本等之间进行转换。

在本教程中，我们将了解以下几个类:

1.  `**JSONObject**`–类似于 Java 的原生`Map`类对象，存储无序的键值对
2.  `**JSONArray**`–类似于 Java 原生向量实现的有序值序列
3.  `**JSONTokener**`–将一段文本分解成一系列`tokens`的工具，这些文本可以被`JSONObject`或`JSONArray`用来解析 JSON 字符串
4.  `**CDL**`–该工具提供了将逗号分隔的文本转换成`JSONArray`的方法，反之亦然
5.  `**Cookie**`–从 JSON `String`转换到 cookies，反之亦然
6.  `**HTTP**`–用于从 JSON `String`转换到 HTTP 头，反之亦然
7.  `**JSONException**`–本库抛出的标准异常

## 4。`JSONObject`

**A `[JSONObject](https://web.archive.org/web/20221012233633/https://stleary.github.io/JSON-java/org/json/JSONObject.html)` 是键和值对的无序集合，类似于 Java 的本机`Map`实现。**

*   **键是唯一的`Strings` ，不能是`null`。**
*   **值可以是从`Boolean`、`Number`、`String`或`JSONArray`到甚至是`JSONObject.NULL`对象的任何值。**
*   一个`JSONObject` 可以用一个`String`表示，用大括号括起来，键和值用冒号分隔，成对的用逗号分隔。
*   它有几个构造函数用来构造一个`JSONObject`。

它还支持以下主要方法:

1.  `get(String key)`–获取与所提供的键相关联的对象，如果没有找到该键，则抛出`JSONException`
2.  `opt(String key)` –获取与所提供的键相关联的对象，否则为`null`
3.  `put(String key, Object value) –` 插入或替换当前`JSONObject.` 中的键值对

`put()` 方法是一个重载方法，它接受一个类型为`String`的键和多个类型的值。

有关`JSONObject`、[支持的方法的完整列表，请访问官方文档。](https://web.archive.org/web/20221012233633/https://stleary.github.io/JSON-java/org/json/JSONObject.html)

现在让我们讨论这个类支持的一些主要操作。

### 4.1。直接从`JSONObject` 创建 JSON

`JSONObject`公开了一个类似 Java 的`Map`接口`.`的 API

我们可以使用`put()` 方法并提供键和值作为参数:

```java
JSONObject jo = new JSONObject();
jo.put("name", "jon doe");
jo.put("age", "22");
jo.put("city", "chicago");
```

现在我们的`JSONObject`看起来像这样:

```java
{"city":"chicago","name":"jon doe","age":"22"}
```

`JSONObject.put()`方法有七种不同的重载签名。虽然键只能是唯一的，但非空值可以是任何值。

### 4.2。从映射创建 JSON

我们可以构造一个自定义的`Map`，然后将其作为参数传递给`JSONObject`的构造函数，而不是直接将键和值放入`JSONObject`。

该示例将产生与上面相同的结果:

```java
Map<String, String> map = new HashMap<>();
map.put("name", "jon doe");
map.put("age", "22");
map.put("city", "chicago");
JSONObject jo = new JSONObject(map);
```

### 4.3。从 JSON `String` 创建`JSONObject`

为了将 JSON `String`解析为`JSONObject`，我们可以将`String`传递给构造函数。

该示例将产生与上面相同的结果:

```java
JSONObject jo = new JSONObject(
  "{\"city\":\"chicago\",\"name\":\"jon doe\",\"age\":\"22\"}"
);
```

传递的`String`参数必须是有效的 JSON 否则，这个构造函数可能会抛出一个`JSONException`。

### 4.4。将 Java 对象序列化为 JSON

一个`JSONObject'`的构造函数将一个 POJO 作为它的参数。在下面的例子中，包使用了来自`DemoBean`类的 getters，并为其创建了一个合适的`JSONObject`。

为了从 Java 对象中获取一个`JSONObject`，我们必须使用一个有效的 [Java Bean](https://web.archive.org/web/20221012233633/https://en.wikipedia.org/wiki/JavaBeans) 的类:

```java
DemoBean demo = new DemoBean();
demo.setId(1);
demo.setName("lorem ipsum");
demo.setActive(true);

JSONObject jo = new JSONObject(demo);
```

这里是`JSONObject jo`:

```java
{"name":"lorem ipsum","active":true,"id":1}
```

虽然我们有办法将 Java 对象序列化为 JSON 字符串，但是没有办法使用这个库将它转换回来。如果我们想要那种灵活性，我们可以切换到其他库，比如 [Jackson](/web/20221012233633/https://www.baeldung.com/jackson) 。

## 5。`JSONArray`

**A [`JSONArray`](https://web.archive.org/web/20221012233633/https://stleary.github.io/JSON-java/org/json/JSONArray.html) 是值的有序集合，类似于 Java 的本机 `Vector`实现。**

*   值可以是从`Number`、`String`、`Boolean`、`JSONArray`或`JSONObject`到甚至`JSONObject.NULL`对象的任何值。
*   它由方括号中的`String`表示，由逗号分隔的值集合组成。
*   像`JSONObject`一样，它有一个构造器，接受一个源`String`并解析它以构造一个`JSONArray`。

这些是`JSONArray`类的主要方法:

1.  `get(int index)` –返回指定索引处的值(介于 0 和总长度–1 之间)，否则抛出`JSONException`
2.  `opt(int index)`–返回与索引相关的值(介于 0 和总长度–1 之间)。如果在那个索引处没有值，那么返回一个`null` 。
3.  `put(Object value)`–将一个对象值追加到这个`JSONArray.`中。这个方法是重载的，支持多种数据类型。

关于 JSONArray 支持的方法的完整列表，[请访问官方文档](https://web.archive.org/web/20221012233633/https://stleary.github.io/JSON-java/org/json/JSONArray.html)。

### 5.1。创造`JSONArray`

一旦我们初始化了 JSONArray 对象，我们就可以使用`put()`和`get()` 方法简单地添加和检索元素:

```java
JSONArray ja = new JSONArray();
ja.put(Boolean.TRUE);
ja.put("lorem ipsum");

JSONObject jo = new JSONObject();
jo.put("name", "jon doe");
jo.put("age", "22");
jo.put("city", "chicago");

ja.put(jo);
```

以下是我们的`JSONArray` 的内容(为了清楚起见，代码被格式化):

```java
[
    true,
    "lorem ipsum",
    {
        "city": "chicago",
        "name": "jon doe",
        "age": "22"
    }
]
```

### 5.2。直接从 JSON 字符串创建`JSONArray`

和`JSONObject`一样，`JSONArray`也有一个直接从 JSON `String`创建 Java 对象的构造函数:

```java
JSONArray ja = new JSONArray("[true, \"lorem ipsum\", 215]");
```

如果源`String`不是有效的 JSON `String`，这个构造函数可能会抛出一个`JSONException`。

### 5.3。直接从集合或数组中创建`JSONArray`

`JSONArray` 的构造函数也支持集合和数组对象作为参数。

我们简单地将它们作为参数传递给构造函数，它将返回一个`JSONArray`对象:

```java
List<String> list = new ArrayList<>();
list.add("California");
list.add("Texas");
list.add("Hawaii");
list.add("Alaska");

JSONArray ja = new JSONArray(list);
```

现在我们的`JSONArray`包括以下内容:

```java
["California","Texas","Hawaii","Alaska"]
```

## 6。 `**JSONTokener**`

一个 [`JSONTokener`](https://web.archive.org/web/20221012233633/https://stleary.github.io/JSON-java/org/json/JSONTokener.html) 将一个源`String`作为其构造函数的输入，并从中提取字符和标记。这个包的类(如`JSONObject`、`JSONArray`)在内部使用它来解析 JSON `Strings`。

我们可能不会在很多情况下直接使用这个类，因为我们可以使用其他更简单的方法(比如`string.toCharArray()`)来实现相同的功能:

```java
JSONTokener jt = new JSONTokener("lorem");

while(jt.more()) {
    Log.info(jt.next());
}
```

现在我们可以像迭代器一样访问一个`JSONTokener`，使用`more()`方法检查是否有剩余元素，使用`next()`访问下一个元素。

以下是从上一个示例中收到的令牌:

```java
l
o
r
e
m
```

## 7。`CDL`

我们提供了一个**(逗号分隔列表)**类来将逗号分隔文本转换成`JSONArray` ，反之亦然。

### 7.1。直接从逗号分隔的文本生成`JSONArray`

为了直接从逗号分隔的文本中产生一个`JSONArray`，我们可以使用静态方法`rowToJSONArray()`，它接受一个`JSONTokener`:

```java
JSONArray ja = CDL.rowToJSONArray(new JSONTokener("England, USA, Canada"));
```

现在我们的`JSONArray`由以下内容组成:

```java
["England","USA","Canada"]
```

### 7.2。从 JSONArray 生成逗号分隔的文本

让我们看看如何颠倒上一步，从`JSONArray`中取回逗号分隔的文本:

```java
JSONArray ja = new JSONArray("[\"England\",\"USA\",\"Canada\"]");
String cdt = CDL.rowToString(ja);
```

`String` `cdt`现在包含以下内容:

```java
England,USA,Canada
```

### 7.3。使用逗号分隔的文本生成`JSONObject`的`JSONArray`

为了生成一个由`JSONObject`组成的`JSONArray`，我们将使用一个文本`String`，其中包含由逗号分隔的标题和数据。

我们使用回车符`(\r)`或换行符`(\n).`来分隔不同的行

第一行被解释为标题列表，所有后续行被视为数据:

```java
String string = "name, city, age \n" +
  "john, chicago, 22 \n" +
  "gary, florida, 35 \n" +
  "sal, vegas, 18";

JSONArray result = CDL.toJSONArray(string);
```

对象`JSONArray result` 现在由以下内容组成(为了清楚起见，输出被格式化):

```java
[
    {
        "name": "john",
        "city": "chicago",
        "age": "22"
    },
    {
        "name": "gary",
        "city": "florida",
        "age": "35"
    },
    {
        "name": "sal",
        "city": "vegas",
        "age": "18"
    }
]
```

请注意，数据和报头都是在同一个`String`中提供的。**我们有一个替代的方法，通过提供一个`JSONArray`来获取标题，一个逗号分隔的`String`作为数据，我们可以实现相同的功能。**

同样，我们使用回车符`(\r)`或换行符`(\n)`来分隔不同的行:

```java
JSONArray ja = new JSONArray();
ja.put("name");
ja.put("city");
ja.put("age");

String string = "john, chicago, 22 \n"
  + "gary, florida, 35 \n"
  + "sal, vegas, 18";

JSONArray result = CDL.toJSONArray(ja, string);
```

这里我们将像以前一样获取对象`result` 的内容。

## 8。Cookie

[`Cookie`](https://web.archive.org/web/20221012233633/https://stleary.github.io/JSON-java/org/json/Cookie.html) 类处理 web 浏览器 cookie，并具有将浏览器 cookie 转换为`JSONObject`的方法，反之亦然。

下面是`Cookie`类的主要方法:

1.  `toJsonObject(String sourceCookie) –`将一个 cookie 字符串转换成一个`JSONObject` 
2.  `toString(JSONObject jo)`–与前面的方法相反，将一个`JSONObject` 转换成一个 cookie `String`

### 8.1。将 Cookie `String`转换为`JSONObject`

为了将 cookie `String`转换成`JSONObject`，我们将使用静态方法`Cookie.toJSONObject()`:

```java
String cookie = "username=John Doe; expires=Thu, 18 Dec 2013 12:00:00 UTC; path=/";
JSONObject cookieJO = Cookie.toJSONObject(cookie);
```

### 8.2。将一个`JSONObject`转换成 Cookie `String`

现在我们将把一个`JSONObject` 转换成 cookie `String`。这与上一步相反:

```java
String cookie = Cookie.toString(cookieJO);
```

## 9。`HTTP`

`[HTTP](https://web.archive.org/web/20221012233633/https://stleary.github.io/JSON-java/org/json/HTTP.html)`类包含静态方法，用于将 HTTP 头转换为`JSONObject`，反之亦然。

这个类也有两个主要方法:

1.  `toJsonObject(String sourceHttpHeader) –`将一个`HttpHeader String`转换为`JSONObject`
2.  `toString(JSONObject jo)`–将提供的`JSONObject`转换为`String`

### 9.1。将`JSONObject`转换为 HTTP 报头

`HTTP.toString()`方法用于将一个`JSONObject`转换成 HTTP 头`String`:

```java
JSONObject jo = new JSONObject();
jo.put("Method", "POST");
jo.put("Request-URI", "http://www.example.com/");
jo.put("HTTP-Version", "HTTP/1.1");
String httpStr = HTTP.toString(jo);
```

以下是我们的`String httpStr` 将包含的内容:

```java
POST "http://www.example.com/" HTTP/1.1
```

注意，在转换 HTTP 请求头时，`JSONObject`必须包含`“Method”`、`“Request-URI”`和`“HTTP-Version”`键。对于响应头，对象必须包含`“HTTP-Version”`、`“Status-Code”`和`“Reason-Phrase”`参数。

### 9.2。将 HTTP 报头`String`转换回`JSONObject`

这里我们将把上一步中得到的 HTTP 字符串转换回我们在那一步中创建的那个`JSONObject`:

```java
JSONObject obj = HTTP.toJSONObject("POST \"http://www.example.com/\" HTTP/1.1");
```

## 10。`JSONException`

`[JSONException](https://web.archive.org/web/20221012233633/https://stleary.github.io/JSON-java/org/json/JSONException.html)` 是这个包在遇到任何错误时抛出的标准异常。

这个包中的所有类都使用它。异常后通常会有一条消息，说明到底哪里出错了。

## 11。结论

在本文中，我们看了一个使用 Java 的 JSON—`org.json`——我们重点关注了这里可用的一些核心功能。

本文中使用的完整代码片段可以在 GitHub 上找到[。](https://web.archive.org/web/20221012233633/https://github.com/eugenp/tutorials/tree/master/json-modules/json)