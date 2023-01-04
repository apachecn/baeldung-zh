# 杰克逊-马歇尔弦到约翰逊节点

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/jackson-json-to-jsonnode>

## 1。概述

这个快速教程将展示如何**使用 Jackson 2 将一个 JSON 字符串转换成一个`JsonNode`** ( `com.fasterxml.jackson.databind.JsonNode`)。

如果你想更深入地了解和学习其他很酷的事情，你可以用杰克逊 2 号来做——直接去[杰克逊的主要教程](/web/20220629000101/https://www.baeldung.com/jackson "The main Jackson Tutorial")。

## 2。快速解析

很简单，要解析 JSON 字符串，我们只需要一个`ObjectMapper`:

```java
@Test
public void whenParsingJsonStringIntoJsonNode_thenCorrect() 
  throws JsonParseException, IOException {
    String jsonString = "{"k1":"v1","k2":"v2"}";

    ObjectMapper mapper = new ObjectMapper();
    JsonNode actualObj = mapper.readTree(jsonString);

    assertNotNull(actualObj);
}
```

## 3。低级解析

如果出于某种原因，您**需要去比它更低一级的**，下面的例子暴露了负责实际解析字符串的`JsonParser`:

```java
@Test
public void givenUsingLowLevelApi_whenParsingJsonStringIntoJsonNode_thenCorrect() 
  throws JsonParseException, IOException {
    String jsonString = "{"k1":"v1","k2":"v2"}";

    ObjectMapper mapper = new ObjectMapper();
    JsonFactory factory = mapper.getFactory();
    JsonParser parser = factory.createParser(jsonString);
    JsonNode actualObj = mapper.readTree(parser);

    assertNotNull(actualObj);
}
```

## 4。使用`JsonNode`

在 JSON 被解析成 JsonNode 对象之后，我们可以**使用 Jackson JSON 树模型**:

```java
@Test
public void givenTheJsonNode_whenRetrievingDataFromId_thenCorrect() 
  throws JsonParseException, IOException {
    String jsonString = "{"k1":"v1","k2":"v2"}";
    ObjectMapper mapper = new ObjectMapper();
    JsonNode actualObj = mapper.readTree(jsonString);

    // When
    JsonNode jsonNode1 = actualObj.get("k1");
    assertThat(jsonNode1.textValue(), equalTo("v1"));
}
```

## 5。结论

本文展示了如何将 JSON 字符串解析成 Jackson 模型(T2)来支持 JSON 对象的结构化处理。

所有这些示例和代码片段的实现都可以在[我的 github 项目](https://web.archive.org/web/20220629000101/https://github.com/eugenp/tutorials/tree/master/jackson-modules/jackson-conversions#readme "Github Project exemplifying how to parse json strings into JsonNode") 中找到——这是一个基于 Eclipse 的项目，所以它应该很容易导入和运行。