# 用 Jackson 忽略空字段

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/jackson-ignore-null-fields>

## 1。概述

这个快速教程将介绍如何设置 **Jackson 在序列化**一个 java 类时忽略空字段。

如果我们想更深入地挖掘，并了解杰克逊 2 的其他酷事情，我们可以前往杰克逊的主要教程。

## 延伸阅读:

## [Jackson–更改字段名称](/web/20221127070640/https://www.baeldung.com/jackson-name-of-property)

Jackson - Change the name of a field to adhere to a specific JSON format.[Read more](/web/20221127070640/https://www.baeldung.com/jackson-name-of-property) →

## [Jackson–决定序列化/反序列化哪些字段](/web/20221127070640/https://www.baeldung.com/jackson-field-serializable-deserializable-or-not)

How to control which fields get serialized/deserialized by Jackson and which fields get ignored.[Read more](/web/20221127070640/https://www.baeldung.com/jackson-field-serializable-deserializable-or-not) →

## 2。忽略类上的空字段

**Jackson 允许我们在班级级别控制这种行为:**

```java
@JsonInclude(Include.NON_NULL)
public class MyDto { ... }
```

**或者在字段级别具有更大的粒度:**

```java
public class MyDto {

    @JsonInclude(Include.NON_NULL)
    private String stringValue;

    private int intValue;

    // standard getters and setters
}
```

现在我们应该能够测试出`null`值确实不是最终 JSON 输出的一部分:

```java
@Test
public void givenNullsIgnoredOnClass_whenWritingObjectWithNullField_thenIgnored()
  throws JsonProcessingException {
    ObjectMapper mapper = new ObjectMapper();
    MyDto dtoObject = new MyDto();

    String dtoAsString = mapper.writeValueAsString(dtoObject);

    assertThat(dtoAsString, containsString("intValue"));
    assertThat(dtoAsString, not(containsString("stringValue")));
}
```

## 3。全局忽略空字段

Jackson 还允许我们在`ObjectMapper` 上**全局配置此行为:**

```java
mapper.setSerializationInclusion(Include.NON_NULL);
```

现在，通过该映射器序列化的任何类中的任何`null`字段都将被忽略:

```java
@Test
public void givenNullsIgnoredGlobally_whenWritingObjectWithNullField_thenIgnored() 
  throws JsonProcessingException {
    ObjectMapper mapper = new ObjectMapper();
    mapper.setSerializationInclusion(Include.NON_NULL);
    MyDto dtoObject = new MyDto();

    String dtoAsString = mapper.writeValueAsString(dtoObject);

    assertThat(dtoAsString, containsString("intValue"));
    assertThat(dtoAsString, containsString("booleanValue"));
    assertThat(dtoAsString, not(containsString("stringValue")));
}
```

## 4。结论

忽略`null`字段是一种常见的 Jackson 配置，因为我们经常需要更好地控制 JSON 输出。本文演示了如何为类实现这一点。然而，还有更高级的用例，比如在序列化 Map 时[忽略空值。](/web/20221127070640/https://www.baeldung.com/jackson-map-null-values-or-null-key)

所有这些例子和代码片段的实现都可以在 Github 项目中找到。