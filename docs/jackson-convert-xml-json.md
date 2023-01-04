# 使用 Jackson 将 XML 转换成 JSON

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/jackson-convert-xml-json>

## 1。概述

在本教程中，我们将看到如何使用 Jackson 将 XML 消息转换成 JSON。

对于不熟悉杰克逊的读者来说，可以考虑先熟悉一下基础知识。

## 2。杰克逊简介

我们可以考虑用 Jackson 以三种不同的方式解析 JSON:

*   第一个也是最常见的是使用`[ObjectMapper](https://web.archive.org/web/20221129004411/https://fasterxml.github.io/jackson-databind/javadoc/2.9/com/fasterxml/jackson/databind/ObjectMapper.html)`的数据绑定
*   第二个是映射到带有`[TreeTraversingParser](https://web.archive.org/web/20221129004411/https://fasterxml.github.io/jackson-databind/javadoc/2.9/com/fasterxml/jackson/databind/node/TreeTraversingParser.html)`和`[JsonNode](https://web.archive.org/web/20221129004411/https://fasterxml.github.io/jackson-databind/javadoc/2.9/com/fasterxml/jackson/databind/JsonNode.html)`的树形数据结构
*   第三种是使用`[JsonParser](https://web.archive.org/web/20221129004411/https://fasterxml.github.io/jackson-core/javadoc/2.9/com/fasterxml/jackson/core/JsonParser.html)`和`[JsonGenerator](https://web.archive.org/web/20221129004411/https://fasterxml.github.io/jackson-core/javadoc/2.9/com/fasterxml/jackson/core/JsonGenerator.html?is-external=true)`通过令牌流式传输树数据结构

现在，Jackson 也支持前两种 XML 数据。因此，让我们看看 Jackson 如何帮助我们完成从一种格式到另一种格式的转换。

## 3。依赖性

首先，我们需要将`[jackson-databind](https://web.archive.org/web/20221129004411/https://search.maven.org/classic/#search%7Cga%7C1%7Cg%3A%22com.fasterxml.jackson.core%22%20AND%20a%3A%22jackson-databind%22)`依赖项添加到我们的 `pom.xml`中:

```
<dependency>
    <groupId>com.fasterxml.jackson.core</groupId>
    <artifactId>jackson-databind</artifactId>
    <version>2.13.3</version>
</dependency>
```

这个库将允许我们使用数据绑定 API。

第二个是添加了 Jackson 的 XML 支持的`[jackson-dataformat-xml](https://web.archive.org/web/20221129004411/https://search.maven.org/classic/#search|gav|1|g%3A%22com.fasterxml.jackson.dataformat%22%20AND%20a%3A%22jackson-dataformat-xml%22)`:

```
<dependency>
    <groupId>com.fasterxml.jackson.dataformat</groupId>
    <artifactId>jackson-dataformat-xml</artifactId>
    <version>2.13.3</version>
</dependency>
```

## 4。数据绑定

**数据绑定，简单来说就是我们要把序列化的数据直接映射到一个 Java 对象的时候。**

为了探究这一点，**让我们用`Flower`和`Color `属性**来定义我们的 XML:

```
<Flower>
    <name>Poppy</name>
    <color>RED</color>
    <petals>9</petals>
</Flower> 
```

这类似于这个 Java 符号:

```
public class Flower {
    private String name;
    private Color color;
    private Integer petals;
    // getters and setters
}

public enum Color { PINK, BLUE, YELLOW, RED; }
```

**我们的第一步是将 XML 解析成一个`Flower`实例**。为此，让我们创建一个`XmlMapper`的实例，Jackson 的 XML 对`ObjectMapper`的等效物，并使用它的`readValue `方法:

```
XmlMapper xmlMapper = new XmlMapper();
Flower poppy = xmlMapper.readValue(xml, Flower.class);
```

**一旦我们有了`Flower`实例，我们将希望使用熟悉的`ObjectMapper` :** 把它写成 JSON

```
ObjectMapper mapper = new ObjectMapper();
String json = mapper.writeValueAsString(poppy);
```

结果，我们得到了我们的 JSON 等价物:

```
{
    "name":"Poppy",
    "color":"RED",
    "petals":9
}
```

## 5。树遍历

有时，直接查看树结构可以提供更多的灵活性，比如我们不想维护一个中间类，或者我们只想转换结构的一部分。

尽管如此，正如我们将看到的，它也伴随着一些权衡。

第一步类似于我们使用数据绑定时的第一步。不过，这一次我们将使用`readTree`方法:

```
XmlMapper xmlMapper = new XmlMapper();
JsonNode node = xmlMapper.readTree(xml.getBytes());
```

这样做之后，我们将有一个包含 3 个孩子的`JsonNode`，正如我们所料:`name, color,` 和 `petals`。

然后，我们可以再次使用`ObjectMapper`，只是发送我们的`JsonNode `:

```
ObjectMapper jsonMapper = new ObjectMapper();
String json = jsonMapper.writeValueAsString(node);
```

**现在，结果与我们上一个例子略有不同:**

```
{
    "name":"Poppy",
    "color":"RED",
    "petals":"9"
}
```

**仔细检查，我们可以看到 petals 属性被序列化为一个字符串，而不是一个数字！** **这是因为在没有明确定义的情况下，`readTree`不会推断出数据类型。**

### 5.1。局限性

而且，Jackson 的 XML 树遍历支持有一些限制:

*   杰克逊无法区分对象和数组。由于 XML 缺乏本地结构来区分对象和对象列表，Jackson 将简单地把重复的元素整理成一个值。
*   而且，因为 Jackson 想要将每个 XML 元素映射到一个 JSON 节点，所以它不支持混合内容。

出于这些原因，[杰克逊官方文档不推荐使用树模型来解析 XML](https://web.archive.org/web/20221129004411/https://github.com/FasterXML/jackson-dataformat-xml#known-limitations) 。

## 6。内存限制

**现在，这两种方法都有一个明显的缺点，那就是为了执行转换，整个 XML 都需要立刻存在内存中。**在 Jackson 支持将树结构作为令牌进行流式传输之前，我们将被这个约束所困，或者我们需要看看如何用类似`[XMLStreamReader](https://web.archive.org/web/20221129004411/https://docs.oracle.com/en/java/javase/11/docs/api/java.xml/javax/xml/stream/XMLStreamReader.html).`的东西来滚动我们自己的树结构

## 7 .**。结论**

在本教程中，我们简要了解了 Jackson 读取 XML 数据并将其写入 JSON 的不同方式。此外，我们快速查看了每种受支持方法的局限性。

像往常一样，教程附带的完整源代码可以在 GitHub 上的[处获得。](https://web.archive.org/web/20221129004411/https://github.com/eugenp/tutorials/tree/master/jackson-modules/jackson-conversions-2)