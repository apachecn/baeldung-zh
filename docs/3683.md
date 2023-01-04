# 杰克逊流媒体 API

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/jackson-streaming-api>

## 1 **。概述**

在本文中，我们将关注杰克逊流式 API。它支持读和写，通过使用它，我们可以编写高性能和快速的 JSON 解析器。

另一方面，使用起来有点困难 JSON 数据的每个细节都需要在代码中显式处理。

## 2。Maven 依赖关系

首先，我们需要给 [`jackson-core`](https://web.archive.org/web/20220703150611/https://search.maven.org/classic/#search%7Cgav%7C1%7Cg%3A%22com.fasterxml.jackson.core%22%20AND%20a%3A%22jackson-databind%22) 添加一个 Maven 依赖项:

```
<dependency>
    <groupId>com.fasterxml.jackson.core</groupId>
    <artifactId>jackson-core</artifactId>
    <version>2.11.1</version>
</dependency>
```

## 3。写入 JSON

我们可以通过使用一个`[JsonGenerator](https://web.archive.org/web/20220703150611/https://fasterxml.github.io/jackson-core/javadoc/2.6/com/fasterxml/jackson/core/JsonGenerator.html)` 类将 JSON 内容直接写到`OutputStream` 中。首先，我们需要创建该对象的实例:

```
ByteArrayOutputStream stream = new ByteArrayOutputStream();
JsonFactory jfactory = new JsonFactory();
JsonGenerator jGenerator = jfactory
  .createGenerator(stream, JsonEncoding.UTF8);
```

接下来，假设我们想要编写一个具有以下结构的 JSON:

```
{  
   "name":"Tom",
   "age":25,
   "address":[  
      "Poland",
      "5th avenue"
   ]
}
```

我们可以使用`JsonGenerator` 的实例将特定字段直接写入`OutputStream:`

```
jGenerator.writeStartObject();
jGenerator.writeStringField("name", "Tom");
jGenerator.writeNumberField("age", 25);
jGenerator.writeFieldName("address");
jGenerator.writeStartArray();
jGenerator.writeString("Poland");
jGenerator.writeString("5th avenue");
jGenerator.writeEndArray();
jGenerator.writeEndObject();
jGenerator.close();
```

为了检查是否创建了正确的 JSON，我们可以创建一个包含 JSON 对象的`String` 对象:

```
String json = new String(stream.toByteArray(), "UTF-8");
assertEquals(
  json, 
  "{\"name\":\"Tom\",\"age\":25,\"address\":[\"Poland\",\"5th avenue\"]}");
```

## 4。解析 JSON

当我们得到一个 JSON `String` 作为输入，并希望从中提取特定的字段时，可以使用一个`[JsonParser](https://web.archive.org/web/20220703150611/https://fasterxml.github.io/jackson-core/javadoc/2.6/com/fasterxml/jackson/core/JsonParser.html)` 类:

```
String json
  = "{\"name\":\"Tom\",\"age\":25,\"address\":[\"Poland\",\"5th avenue\"]}";
JsonFactory jfactory = new JsonFactory();
JsonParser jParser = jfactory.createParser(json);

String parsedName = null;
Integer parsedAge = null;
List<String> addresses = new LinkedList<>();
```

我们想从输入 JSON 中获得`parsedName, parsedAge, and addresses` 字段。为了实现这一点，我们需要处理底层解析逻辑并自己实现它:

```
while (jParser.nextToken() != JsonToken.END_OBJECT) {
    String fieldname = jParser.getCurrentName();
    if ("name".equals(fieldname)) {
        jParser.nextToken();
        parsedName = jParser.getText();
    }

    if ("age".equals(fieldname)) {
        jParser.nextToken();
        parsedAge = jParser.getIntValue();
    }

    if ("address".equals(fieldname)) {
        jParser.nextToken();
        while (jParser.nextToken() != JsonToken.END_ARRAY) {
            addresses.add(jParser.getText());
        }
    }
}
jParser.close();
```

根据字段名，我们提取它并分配给一个适当的字段。解析文档后，所有字段都应该有正确的数据:

```
assertEquals(parsedName, "Tom");
assertEquals(parsedAge, (Integer) 25);
assertEquals(addresses, Arrays.asList("Poland", "5th avenue"));
```

## 5。提取 JSON 部件

有时，当我们解析一个 JSON 文档时，我们只对一个特定的领域感兴趣。

理想情况下，在这些情况下，我们只想解析文档的开头，一旦找到所需的字段，我们就可以中止处理。

假设我们只对输入 JSON 的`age`字段感兴趣。在这种情况下，我们可以实现解析逻辑，一旦找到需要的字段就停止解析:

```
while (jParser.nextToken() != JsonToken.END_OBJECT) {
    String fieldname = jParser.getCurrentName();

    if ("age".equals(fieldname)) {
        jParser.nextToken();
        parsedAge = jParser.getIntValue();
        return;
    }

}
jParser.close();
```

处理后，唯一的`parsedAge` 字段将有一个值:

```
assertNull(parsedName);
assertEquals(parsedAge, (Integer) 25);
assertTrue(addresses.isEmpty());
```

由于这一点，JSON 文档的解析将会快得多，因为我们不需要阅读整个文档，只需要阅读其中的一小部分。

## 6。结论

在这篇简短的文章中，我们将探讨如何利用 Jackson 的流处理 API。

所有这些例子和代码片段的实现都可以在 GitHub 上找到[——这是一个 Maven 项目，所以它应该很容易导入和运行。](https://web.archive.org/web/20220703150611/https://github.com/eugenp/tutorials/tree/master/jackson-modules/jackson-conversions-2)