# Jackson–JsonMappingException(未找到类的序列化程序)

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/jackson-jsonmappingexception>

## 1。概述

在这个快速教程中，我们将分析没有 getters 的实体的编组，以及 Jackson `JsonMappingException`异常的解决方案。

如果你想更深入地了解和学习其他很酷的事情，你可以用杰克逊 2 号来做——直接去[杰克逊的主要教程](/web/20220629004613/https://www.baeldung.com/jackson "The main Jackson Tutorial")。

## 延伸阅读:

## [杰克逊对象映射器简介](/web/20220629004613/https://www.baeldung.com/jackson-object-mapper-tutorial)

The article discusses Jackson's central ObjectMapper class, basic serialization and deserialization as well as configuring the two processes.[Read more](/web/20220629004613/https://www.baeldung.com/jackson-object-mapper-tutorial) →

## [与杰克逊一起使用可选](/web/20220629004613/https://www.baeldung.com/jackson-optional)

A quick overview of how we can use the Optional with Jackson.[Read more](/web/20220629004613/https://www.baeldung.com/jackson-optional) →

## [Spring JSON-P 与杰克逊](/web/20220629004613/https://www.baeldung.com/spring-jackson-jsonp)

The article is focused on showing how to use the new JSON-P support in Spring 4.1.[Read more](/web/20220629004613/https://www.baeldung.com/spring-jackson-jsonp) →

## 2。问题

默认情况下，Jackson 2 将只处理公共字段，或者具有公共 getter 方法的字段——**序列化所有字段都是私有的或包私有的实体将会失败**:

```
public class MyDtoNoAccessors {
    String stringValue;
    int intValue;
    boolean booleanValue;

    public MyDtoNoAccessors() {
        super();
    }

    // no getters
}
```

```
@Test(expected = JsonMappingException.class)
public void givenObjectHasNoAccessors_whenSerializing_thenException() 
  throws JsonParseException, IOException {
    String dtoAsString = new ObjectMapper().writeValueAsString(new MyDtoNoAccessors());

    assertThat(dtoAsString, notNullValue());
}
```

**满异常**是:

```
com.fasterxml.jackson.databind.JsonMappingException: 
No serializer found for class dtos.MyDtoNoAccessors 
and no properties discovered to create BeanSerializer 
(to avoid exception, disable SerializationFeature.FAIL_ON_EMPTY_BEANS) )
```

## 3。解决方案

显而易见的解决方案是为字段添加 getters 如果实体在我们的控制之下。如果情况不是这样，并且**修改实体的源代码是不可能的**——那么 Jackson 为我们提供了一些替代方案。

### 3.1。全局自动检测具有任何可见性的字段

这个问题的第一个解决方案是全局配置`ObjectMapper`来检测所有字段，不管它们的可见性如何:

```
objectMapper.setVisibility(PropertyAccessor.FIELD, Visibility.ANY);
```

这将**允许在没有获取器的情况下检测私有和包私有字段**，并且序列化将正确工作:

```
@Test
public void givenObjectHasNoAccessors_whenSerializingWithAllFieldsDetected_thenNoException() 
  throws JsonParseException, IOException {
    ObjectMapper objectMapper = new ObjectMapper();
    objectMapper.setVisibility(PropertyAccessor.FIELD, Visibility.ANY);
    String dtoAsString = objectMapper.writeValueAsString(new MyDtoNoAccessors());

    assertThat(dtoAsString, containsString("intValue"));
    assertThat(dtoAsString, containsString("stringValue"));
    assertThat(dtoAsString, containsString("booleanValue"));
}
```

### 3.2。检测到类别级别的所有字段

Jackson 2 提供的另一个选项是——而不是全局配置——**通过`@JsonAutoDetect`注释在类级别**控制字段可见性:

```
@JsonAutoDetect(fieldVisibility = Visibility.ANY)
public class MyDtoNoAccessors { ... }
```

有了这个注释，序列化现在应该可以正确地处理这个特定的类了:

```
@Test
public void givenObjectHasNoAccessorsButHasVisibleFields_whenSerializing_thenNoException() 
  throws JsonParseException, IOException {
    ObjectMapper objectMapper = new ObjectMapper();
    String dtoAsString = objectMapper.writeValueAsString(new MyDtoNoAccessors());

    assertThat(dtoAsString, containsString("intValue"));
    assertThat(dtoAsString, containsString("stringValue"));
    assertThat(dtoAsString, containsString("booleanValue"));
}
```

## 4。结论

本文展示了如何通过在`ObjectMapper`或单个类上全局配置一个定制的可见性来绕过 Jackson 中的默认字段可见性。Jackson 通过提供选项来精确控制映射器如何看到具有特定可见性的 getters、setters 或 fields，从而允许进一步的定制。

所有这些示例和代码片段的实现都可以在[我的 GitHub 项目](https://web.archive.org/web/20220629004613/https://github.com/eugenp/tutorials/tree/master/jackson-modules/jackson-exceptions#readme "Github Project exemplifying how to set custom field visibilities in Jackson 2") 中找到——这是一个基于 Eclipse 的项目，所以应该很容易导入和运行。