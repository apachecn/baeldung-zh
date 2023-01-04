# Jackson 用未知属性解组 JSON

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/jackson-deserialize-json-unknown-properties>

## 1。概述

在本教程中，我们将看看 Jackson 2.x 的解组过程，特别是**如何处理带有未知属性**的 JSON 内容。

为了更深入地挖掘和了解我们可以和杰克逊一起做的其他很酷的事情，我们可以查看杰克逊的主要教程。

## 延伸阅读:

## [杰克逊忽略编组上的属性](/web/20220628100222/https://www.baeldung.com/jackson-ignore-properties-on-serialization)

Control your JSON Output - Ignore certain fields directly, by name or by type (with mixins) for Jackson bliss.[Read more](/web/20220628100222/https://www.baeldung.com/jackson-ignore-properties-on-serialization) →

## [Jackson–更改字段名称](/web/20220628100222/https://www.baeldung.com/jackson-name-of-property)

Jackson - Change the name of a field to adhere to a specific JSON format.[Read more](/web/20220628100222/https://www.baeldung.com/jackson-name-of-property) →

## 2。解组带有附加/未知字段的 JSON】

JSON 输入有各种形状和大小，大多数时候，我们需要将它映射到预先定义的 Java 对象，这些对象有一定数量的字段。目标是简单地**忽略任何不能映射到现有 Java 字段**的 JSON 属性。

例如，假设我们需要将 JSON 解组到以下 Java 实体:

```java
public class MyDto {

    private String stringValue;
    private int intValue;
    private boolean booleanValue;

    // standard constructor, getters and setters 
}
```

### 2.1。`UnrecognizedPropertyException`在未知的领域

试图将一个属性未知的 JSON 解组到这个简单的 Java 实体会导致一个`com.fasterxml.jackson.databind.exc.UnrecognizedPropertyException`:

```java
@Test(expected = UnrecognizedPropertyException.class)
public void givenJsonHasUnknownValues_whenDeserializing_thenException()
  throws JsonParseException, JsonMappingException, IOException {
    String jsonAsString = 
        "{"stringValue":"a"," +
        ""intValue":1," +
        ""booleanValue":true," +
        ""stringValue2":"something"}";
    ObjectMapper mapper = new ObjectMapper();

    MyDto readValue = mapper.readValue(jsonAsString, MyDto.class);

    assertNotNull(readValue);
}
```

这将失败，并出现以下异常:

```java
com.fasterxml.jackson.databind.exc.UnrecognizedPropertyException: 
Unrecognized field "stringValue2" (class org.baeldung.jackson.ignore.MyDto), 
not marked as ignorable (3 known properties: "stringValue", "booleanValue", "intValue"])
```

### 2.2。使用`ObjectMapper` 处理未知字段

**我们现在可以配置完整的`ObjectMapper`来忽略 JSON:** 中的未知属性

```java
new ObjectMapper()
  .configure(DeserializationFeature.FAIL_ON_UNKNOWN_PROPERTIES, false)
```

然后，我们应该能够将这种 JSON 读入预定义的 Java 实体:

```java
@Test
public void givenJsonHasUnknownValuesButJacksonIsIgnoringUnknowns_whenDeserializing_thenCorrect()
  throws JsonParseException, JsonMappingException, IOException {

    String jsonAsString = 
        "{"stringValue":"a"," +
        ""intValue":1," +
        ""booleanValue":true," +
        ""stringValue2":"something"}";
    ObjectMapper mapper = new ObjectMapper()
      .configure(DeserializationFeature.FAIL_ON_UNKNOWN_PROPERTIES, false);

    MyDto readValue = mapper.readValue(jsonAsString, MyDto.class);

    assertNotNull(readValue);
    assertThat(readValue.getStringValue(), equalTo("a"));
    assertThat(readValue.isBooleanValue(), equalTo(true));
    assertThat(readValue.getIntValue(), equalTo(1));
}
```

### 2.3。在类级别处理未知字段

**我们也可以将单个类标记为接受未知字段**，而不是整个 Jackson `ObjectMapper`:

```java
@JsonIgnoreProperties(ignoreUnknown = true)
public class MyDtoIgnoreUnknown { ... }
```

现在我们应该能够像以前一样测试相同的行为。未知字段被忽略，只映射已知字段:

```java
@Test
public void givenJsonHasUnknownValuesButIgnoredOnClass_whenDeserializing_thenCorrect() 
  throws JsonParseException, JsonMappingException, IOException {

    String jsonAsString =
        "{"stringValue":"a"," +
        ""intValue":1," +
        ""booleanValue":true," +
        ""stringValue2":"something"}";
    ObjectMapper mapper = new ObjectMapper();

    MyDtoIgnoreUnknown readValue = mapper
      .readValue(jsonAsString, MyDtoIgnoreUnknown.class);

    assertNotNull(readValue);
    assertThat(readValue.getStringValue(), equalTo("a"));
    assertThat(readValue.isBooleanValue(), equalTo(true));
    assertThat(readValue.getIntValue(), equalTo(1));
}
```

## 3。解组不完整的 JSON

与其他未知字段类似，解组一个不完整的 JSON，一个不包含 Java 类中所有字段的 JSON，对 Jackson 来说不是问题:

```java
@Test
public void givenNotAllFieldsHaveValuesInJson_whenDeserializingAJsonToAClass_thenCorrect() 
  throws JsonParseException, JsonMappingException, IOException {
    String jsonAsString = "{"stringValue":"a","booleanValue":true}";
    ObjectMapper mapper = new ObjectMapper();

    MyDto readValue = mapper.readValue(jsonAsString, MyDto.class);

    assertNotNull(readValue);
    assertThat(readValue.getStringValue(), equalTo("a"));
    assertThat(readValue.isBooleanValue(), equalTo(true));
}
```

## 4。结论

在本文中，我们讨论了使用 Jackson 反序列化带有额外未知属性的 JSON。

这是使用 Jackson 时最常见的配置之一，因为我们经常需要**将外部 REST APIs 的 JSON 结果映射到 API 实体的内部 Java 表示**。

所有这些例子和代码片段的实现都可以在我的 GitHub 项目中找到。