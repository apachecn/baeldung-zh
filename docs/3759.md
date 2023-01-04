# Jackson–使用地图和空值

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/jackson-map-null-values-or-null-key>

## 1。概述

在这篇简短的文章中，我们将看看使用 **Jackson 的一个更高级的用例——使用包含空值或空键**的`Maps`。

## 2。忽略地图中的空值

Jackson 有一个简单但有用的方法来全局控制当映射被序列化时空值会发生什么:

```
ObjectMapper mapper = new ObjectMapper();
mapper.setSerializationInclusion(Include.NON_NULL);
```

现在，通过此映射器序列化的映射对象中的任何空值都将被忽略:

```
@Test
public void givenIgnoringNullValuesInMap_whenWritingMapObjectWithNullValue_thenIgnored() 
  throws JsonProcessingException {
    ObjectMapper mapper = new ObjectMapper();
    mapper.setSerializationInclusion(Include.NON_NULL);

    MyDto dtoObject1 = new MyDto();

    Map<String, MyDto> dtoMap = new HashMap<String, MyDto>();
    dtoMap.put("dtoObject1", dtoObject1);
    dtoMap.put("dtoObject2", null);

    String dtoMapAsString = mapper.writeValueAsString(dtoMap);

    assertThat(dtoMapAsString, containsString("dtoObject1"));
    assertThat(dtoMapAsString, not(containsString("dtoObject2")));
}
```

## 3。用空键序列化映射

默认情况下， **Jackson 不允许序列化带有空键**的地图。如果您尝试写出这样的地图，您将会得到以下异常:

```
c.f.j.c.JsonGenerationException: 
  Null key for a Map not allowed in JSON (use a converting NullKeySerializer?)
    at c.f.j.d.s.i.FailingSerializer.serialize(FailingSerializer.java:36)
```

然而，这个库足够灵活，您可以定义一个定制的空键序列化器并覆盖默认行为:

```
class MyDtoNullKeySerializer extends StdSerializer<Object> {
    public MyDtoNullKeySerializer() {
        this(null);
    }

    public MyDtoNullKeySerializer(Class<Object> t) {
        super(t);
    }

    @Override
    public void serialize(Object nullKey, JsonGenerator jsonGenerator, SerializerProvider unused) 
      throws IOException, JsonProcessingException {
        jsonGenerator.writeFieldName("");
    }
}
```

现在，带有空键的映射就可以正常工作了——空键将被写成一个空字符串:

```
@Test
public void givenAllowingMapObjectWithNullKey_whenWriting_thenCorrect() 
throws JsonProcessingException {
    ObjectMapper mapper = new ObjectMapper();
    mapper.getSerializerProvider().setNullKeySerializer(new MyDtoNullKeySerializer());

    MyDto dtoObject = new MyDto();
    dtoObject.setStringValue("dtoObjectString");

    Map<String, MyDto> dtoMap = new HashMap<String, MyDto>();
    dtoMap.put(null, dtoObject);

    String dtoMapAsString = mapper.writeValueAsString(dtoMap);

    assertThat(dtoMapAsString, containsString("\"\""));
    assertThat(dtoMapAsString, containsString("dtoObjectString"));
}
```

## 4。忽略空字段

除了地图之外，Jackson 还提供了很多配置和灵活性来忽略/处理一般的`null`字段。你可以看看这个教程，看看它到底是如何工作的。

## 5。结论

序列化 Map 对象非常常见，我们需要一个能够很好地处理序列化过程细微差别的库。Jackson 提供了一些方便的定制选项来帮助您很好地形成这个序列化过程的输出。

它还为[提供了许多更一般意义上的使用集合](/web/20220127172631/https://www.baeldung.com/jackson-collection-array)的可靠方法。

所有这些示例和代码片段**的实现可以在 GitHub** 上的[中找到——这是一个基于 Maven 的项目，所以它应该很容易导入和运行。](https://web.archive.org/web/20220127172631/https://github.com/eugenp/tutorials/tree/master/jackson-modules/jackson-conversions#readme "Github Project covering all Jackson examples")