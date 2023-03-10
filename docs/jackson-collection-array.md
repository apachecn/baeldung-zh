# Jackson–解组到集合/阵列

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/jackson-collection-array>

## 1。概述

本教程将展示如何用 Jackson 2 将 JSON 数组反序列化为 Java 数组或集合。

如果你想更深入地了解和学习其他很酷的事情，你可以用杰克逊 2 号来做——直接去[杰克逊的主要教程](/web/20221129001443/https://www.baeldung.com/jackson "The Jackson 2 Tutorial")。

## 2。解组到数组

Jackson 可以很容易地反序列化为 Java 数组:

```java
@Test
public void givenJsonArray_whenDeserializingAsArray_thenCorrect() 
  throws JsonParseException, JsonMappingException, IOException {

    ObjectMapper mapper = new ObjectMapper();
    List<MyDto> listOfDtos = Lists.newArrayList(
      new MyDto("a", 1, true), new MyDto("bc", 3, false));
    String jsonArray = mapper.writeValueAsString(listOfDtos);

    // [{"stringValue":"a","intValue":1,"booleanValue":true},
    // {"stringValue":"bc","intValue":3,"booleanValue":false}]

    MyDto[] asArray = mapper.readValue(jsonArray, MyDto[].class);
    assertThat(asArray[0], instanceOf(MyDto.class));
}
```

## 3。解组到集合

将同一个 JSON 数组读入一个 Java 集合有点困难——默认情况下， **Jackson 将不能获得完整的泛型类型信息**,而是创建一个`LinkedHashMap`实例的集合:

```java
@Test
public void givenJsonArray_whenDeserializingAsListWithNoTypeInfo_thenNotCorrect() 
  throws JsonParseException, JsonMappingException, IOException {

    ObjectMapper mapper = new ObjectMapper();

    List<MyDto> listOfDtos = Lists.newArrayList(
      new MyDto("a", 1, true), new MyDto("bc", 3, false));
    String jsonArray = mapper.writeValueAsString(listOfDtos);

    List<MyDto> asList = mapper.readValue(jsonArray, List.class);
    assertThat((Object) asList.get(0), instanceOf(LinkedHashMap.class));
}
```

有两种方法**帮助 Jackson 理解正确的类型信息**——我们可以使用图书馆提供的`TypeReference`来达到这个目的:

```java
@Test
public void givenJsonArray_whenDeserializingAsListWithTypeReferenceHelp_thenCorrect() 
  throws JsonParseException, JsonMappingException, IOException {

    ObjectMapper mapper = new ObjectMapper();

    List<MyDto> listOfDtos = Lists.newArrayList(
      new MyDto("a", 1, true), new MyDto("bc", 3, false));
    String jsonArray = mapper.writeValueAsString(listOfDtos);

    List<MyDto> asList = mapper.readValue(
      jsonArray, new TypeReference<List<MyDto>>() { });
    assertThat(asList.get(0), instanceOf(MyDto.class));
}
```

或者我们可以使用接受一个`JavaType`的重载`readValue`方法:

```java
@Test
public void givenJsonArray_whenDeserializingAsListWithJavaTypeHelp_thenCorrect() 
  throws JsonParseException, JsonMappingException, IOException {
    ObjectMapper mapper = new ObjectMapper();

    List<MyDto> listOfDtos = Lists.newArrayList(
      new MyDto("a", 1, true), new MyDto("bc", 3, false));
    String jsonArray = mapper.writeValueAsString(listOfDtos);

    CollectionType javaType = mapper.getTypeFactory()
      .constructCollectionType(List.class, MyDto.class);
    List<MyDto> asList = mapper.readValue(jsonArray, javaType);

    assertThat(asList.get(0), instanceOf(MyDto.class));
}
```

最后要注意的是，`MyDto`类需要无参数的默认构造函数——如果没有的话， **Jackson 将不能实例化它**:

```java
com.fasterxml.jackson.databind.JsonMappingException: 
No suitable constructor found for type [simple type, class org.baeldung.jackson.ignore.MyDto]: 
can not instantiate from JSON object (need to add/enable type information?)
```

## 4。结论

将 JSON 数组映射到 java 集合是 Jackson 常用的任务之一，这些解决方案**对于获得正确的、类型安全的映射**至关重要。

所有这些例子和代码片段**的实现可以在我们的 [GitHub 项目](https://web.archive.org/web/20221129001443/https://github.com/eugenp/tutorials/tree/master/jackson-modules/jackson-conversions#readme "Github Project exemplifying how map the Json Array")** 中找到——这是一个基于 Maven 的项目，所以它应该很容易导入和运行。