# 杰克逊-更改字段名称

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/jackson-name-of-property>

## 1。概述

这个快速教程演示了如何在序列化过程中**更改一个字段的名称以映射到另一个 JSON 属性**。

如果你想更深入地了解和学习其他你可以用杰克逊 2 做的很酷的事情——直接去[杰克逊的主要教程](/web/20220625072359/https://www.baeldung.com/jackson "The main Jackson Tutorial")。

## 2。更改序列化字段的名称

使用简单实体:

```java
public class MyDto {
    private String stringValue;

    public MyDto() {
        super();
    }

    public String getStringValue() {
        return stringValue;
    }

    public void setStringValue(String stringValue) {
        this.stringValue = stringValue;
    }
}
```

将其序列化将产生以下 JSON:

```java
{"stringValue":"some value"}
```

**为了定制输出，我们需要简单地注释 getter:** ，而不是`stringValue`

```java
@JsonProperty("strVal")
public String getStringValue() {
    return stringValue;
}
```

现在，在序列化时，我们将得到期望的输出:

```java
{"strVal":"some value"}
```

简单的单元测试应该可以验证输出是否正确:

```java
@Test
public void givenNameOfFieldIsChanged_whenSerializing_thenCorrect() 
  throws JsonParseException, IOException {
    ObjectMapper mapper = new ObjectMapper();
    MyDtoFieldNameChanged dtoObject = new MyDtoFieldNameChanged();
    dtoObject.setStringValue("a");

    String dtoAsString = mapper.writeValueAsString(dtoObject);

    assertThat(dtoAsString, not(containsString("stringValue")));
    assertThat(dtoAsString, containsString("strVal"));
}
```

## 3。结论

编组一个实体以遵循特定的 JSON 格式是一项常见的任务——本文展示了如何通过使用`@JsonProperty`注释来完成。

所有这些例子和代码片段的实现都可以在[我的 github 项目](https://web.archive.org/web/20220625072359/https://github.com/eugenp/tutorials/tree/master/jackson-simple "Github Project exemplifying how to change the json key of a filed")中找到。