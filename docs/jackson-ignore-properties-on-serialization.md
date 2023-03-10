# Jackson 忽略编组属性

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/jackson-ignore-properties-on-serialization>

## 1。概述

本教程将展示如何在使用 Jackson 2.x 将一个对象序列化为 JSON 时**忽略某些字段。**

当 Jackson 缺省值不够时，这非常有用，我们需要精确地控制序列化到 JSON 的内容——有几种方法可以忽略属性。

为了更深入地挖掘和了解我们可以和杰克逊一起做的其他很酷的事情，请前往杰克逊的主要教程。

## 延伸阅读:

## [杰克逊对象映射器简介](/web/20220629004455/https://www.baeldung.com/jackson-object-mapper-tutorial)

The article discusses Jackson's central ObjectMapper class, basic serialization and deserialization as well as configuring the two processes.[Read more](/web/20220629004455/https://www.baeldung.com/jackson-object-mapper-tutorial) →

## [杰克逊流媒体应用编程接口](/web/20220629004455/https://www.baeldung.com/jackson-streaming-api)

A quick overview of Jackson's Streaming API for processing JSON, including the examples[Read more](/web/20220629004455/https://www.baeldung.com/jackson-streaming-api) →

## [杰克逊@JsonFormat 指南](/web/20220629004455/https://www.baeldung.com/jackson-jsonformat)

A quick and practical guide to the @JsonFormat annotation in Jackson.[Read more](/web/20220629004455/https://www.baeldung.com/jackson-jsonformat) →

## 2。忽略类级别的字段

我们可以忽略类级别的特定字段，使用@ `JsonIgnoreProperties`注释并通过名称指定字段:

```java
@JsonIgnoreProperties(value = { "intValue" })
public class MyDto {

    private String stringValue;
    private int intValue;
    private boolean booleanValue;

    public MyDto() {
        super();
    }

    // standard setters and getters are not shown
}
```

我们现在可以测试，在对象被写入 JSON 之后，该字段确实不是输出的一部分:

```java
@Test
public void givenFieldIsIgnoredByName_whenDtoIsSerialized_thenCorrect()
  throws JsonParseException, IOException {

    ObjectMapper mapper = new ObjectMapper();
    MyDto dtoObject = new MyDto();

    String dtoAsString = mapper.writeValueAsString(dtoObject);

    assertThat(dtoAsString, not(containsString("intValue")));
}
```

## 3。在字段级别忽略字段

我们也可以通过字段上的@ `JsonIgnore`注释直接忽略字段:

```java
public class MyDto {

    private String stringValue;
    @JsonIgnore
    private int intValue;
    private boolean booleanValue;

    public MyDto() {
        super();
    }

    // standard setters and getters are not shown
}
```

我们现在可以测试`intValue`字段确实不是序列化 JSON 输出的一部分:

```java
@Test
public void givenFieldIsIgnoredDirectly_whenDtoIsSerialized_thenCorrect() 
  throws JsonParseException, IOException {

    ObjectMapper mapper = new ObjectMapper();
    MyDto dtoObject = new MyDto();

    String dtoAsString = mapper.writeValueAsString(dtoObject);

    assertThat(dtoAsString, not(containsString("intValue")));
}
```

## 4。按类型忽略所有字段

最后，我们可以使用@ `JsonIgnoreType`注释**忽略指定类型的所有字段。**如果我们控制了类型，那么我们可以直接对类进行注释:

```java
@JsonIgnoreType
public class SomeType { ... }
```

然而，大多数情况下，我们无法控制类本身。在这种情况下，**我们可以好好利用杰克逊 mixins。**

首先，我们为我们想要忽略的类型定义一个 MixIn，并用`@JsonIgnoreType`代替:

```java
@JsonIgnoreType
public class MyMixInForIgnoreType {}
```

然后我们注册 mixin 来替换(并忽略)编组期间的所有`String[]`类型:

```java
mapper.addMixInAnnotations(String[].class, MyMixInForIgnoreType.class);
```

此时，所有字符串数组都将被忽略，而不是被封送到 JSON:

```java
@Test
public final void givenFieldTypeIsIgnored_whenDtoIsSerialized_thenCorrect()
  throws JsonParseException, IOException {

    ObjectMapper mapper = new ObjectMapper();
    mapper.addMixIn(String[].class, MyMixInForIgnoreType.class);
    MyDtoWithSpecialField dtoObject = new MyDtoWithSpecialField();
    dtoObject.setBooleanValue(true);

    String dtoAsString = mapper.writeValueAsString(dtoObject);

    assertThat(dtoAsString, containsString("intValue"));
    assertThat(dtoAsString, containsString("booleanValue"));
    assertThat(dtoAsString, not(containsString("stringValue")));
}
```

这是我们的 DTO:

```java
public class MyDtoWithSpecialField {
    private String[] stringValue;
    private int intValue;
    private boolean booleanValue;
}
```

注意:从 2.5 版本开始，似乎就不能用这个方法忽略原语数据类型了，但是可以用于自定义数据类型和数组。

## 5。使用过滤器忽略字段

最后，**我们还可以使用过滤器来忽略 Jackson 中的特定字段**。

首先，我们需要在 Java 对象上定义过滤器:

```java
@JsonFilter("myFilter")
public class MyDtoWithFilter { ... }
```

然后我们定义一个简单的过滤器，它将忽略`intValue`字段:

```java
SimpleBeanPropertyFilter theFilter = SimpleBeanPropertyFilter
  .serializeAllExcept("intValue");
FilterProvider filters = new SimpleFilterProvider()
  .addFilter("myFilter", theFilter);
```

现在我们可以序列化对象，并确保 JSON 输出中不存在`intValue`字段:

```java
@Test
public final void givenTypeHasFilterThatIgnoresFieldByName_whenDtoIsSerialized_thenCorrect() 
  throws JsonParseException, IOException {

    ObjectMapper mapper = new ObjectMapper();
    SimpleBeanPropertyFilter theFilter = SimpleBeanPropertyFilter
      .serializeAllExcept("intValue");
    FilterProvider filters = new SimpleFilterProvider()
      .addFilter("myFilter", theFilter);

    MyDtoWithFilter dtoObject = new MyDtoWithFilter();
    String dtoAsString = mapper.writer(filters).writeValueAsString(dtoObject);

    assertThat(dtoAsString, not(containsString("intValue")));
    assertThat(dtoAsString, containsString("booleanValue"));
    assertThat(dtoAsString, containsString("stringValue"));
    System.out.println(dtoAsString);
}
```

## 6。结论

本文演示了如何在序列化中忽略字段。我们首先通过名称来实现，然后直接实现，最后，我们忽略了整个 java 类型和 MixIns，并使用过滤器来更好地控制输出。

所有这些例子和代码片段的实现都可以在 GitHub 项目中找到。