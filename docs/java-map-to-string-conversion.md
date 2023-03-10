# Java 中的映射到字符串转换

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-map-to-string-conversion>

## 1.概观

在本教程中，我们将重点关注从`Map`到`String`的转换，反之亦然。

首先，我们将看到如何使用核心 Java 方法来实现这些，然后，我们将使用一些第三方库。

## 2.基本`Map`示例

在所有示例中，我们将使用相同的`Map`实现:

```java
Map<Integer, String> wordsByKey = new HashMap<>();
wordsByKey.put(1, "one");
wordsByKey.put(2, "two");
wordsByKey.put(3, "three");
wordsByKey.put(4, "four");
```

## 3.通过迭代将一个`Map`转换成一个`String`

让我们遍历我们的`Map`中的所有键，并为它们中的每一个，将键值组合附加到我们生成的`[StringBuilder](/web/20221101145551/https://www.baeldung.com/java-string-builder-string-buffer) `对象中。

出于格式化的目的，我们可以用花括号将结果括起来:

```java
public String convertWithIteration(Map<Integer, ?> map) {
    StringBuilder mapAsString = new StringBuilder("{");
    for (Integer key : map.keySet()) {
        mapAsString.append(key + "=" + map.get(key) + ", ");
    }
    mapAsString.delete(mapAsString.length()-2, mapAsString.length()).append("}");
    return mapAsString.toString();
}
```

为了检查我们是否正确地转换了我们的`Map`,让我们运行下面的测试:

```java
@Test
public void givenMap_WhenUsingIteration_ThenResultingStringIsCorrect() {
    String mapAsString = MapToString.convertWithIteration(wordsByKey);
    Assert.assertEquals("{1=one, 2=two, 3=three, 4=four}", mapAsString);
}
```

## 4.使用 Java 流将一个`Map`转换成一个`String`

为了使用流执行转换，我们首先需要用可用的`Map`键创建一个流。

其次，我们将每个键映射到人类可读的`String`。

最后，我们连接这些值，为了方便起见，我们使用`Collectors.joining()`方法添加了一些格式规则:

```java
public String convertWithStream(Map<Integer, ?> map) {
    String mapAsString = map.keySet().stream()
      .map(key -> key + "=" + map.get(key))
      .collect(Collectors.joining(", ", "{", "}"));
    return mapAsString;
}
```

## 5.使用番石榴将`Map`转化为`String`

让我们将 [Guava](https://web.archive.org/web/20221101145551/https://search.maven.org/search?q=g:com.google.guava%20AND%20a:guava) 添加到我们的项目中，看看我们如何在一行代码中实现转换:

```java
<dependency>
    <groupId>com.google.guava</groupId>
    <artifactId>guava</artifactId>
    <version>31.0.1-jre</version>
</dependency>
```

为了使用 Guava 的`Joiner`类执行转换，我们需要在不同的`Map`条目之间定义一个分隔符，在键和值之间定义一个分隔符:

```java
public String convertWithGuava(Map<Integer, ?> map) {
    return Joiner.on(",").withKeyValueSeparator("=").join(map);
}
```

## 6.使用 Apache Commons 将一个`Map`转换成一个`String`

要使用 [Apache Commons](https://web.archive.org/web/20221101145551/https://search.maven.org/search?q=a:commons-collections4%20AND%20g:org.apache.commons) ，让我们先添加以下依赖项:

```java
<dependency>
    <groupId>org.apache.commons</groupId>
    <artifactId>commons-collections4</artifactId>
    <version>4.2</version>
</dependency>
```

连接非常简单——我们只需要调用`StringUtils.join`方法:

```java
public String convertWithApache(Map map) {
    return StringUtils.join(map);
}
```

特别值得一提的是 Apache Commons 中的`debugPrint`方法。这对于调试非常有用。

当我们打电话时:

```java
MapUtils.debugPrint(System.out, "Map as String", wordsByKey);
```

调试文本将被写入控制台:

```java
Map as String = 
{
    1 = one java.lang.String
    2 = two java.lang.String
    3 = three java.lang.String
    4 = four java.lang.String
} java.util.HashMap
```

## 7.使用流将一个`String`转换成一个`Map`

要执行从`String`到`Map`的转换，让我们定义在哪里拆分以及如何提取键和值:

```java
public Map<String, String> convertWithStream(String mapAsString) {
    Map<String, String> map = Arrays.stream(mapAsString.split(","))
      .map(entry -> entry.split("="))
      .collect(Collectors.toMap(entry -> entry[0], entry -> entry[1]));
    return map;
}
```

## 8.使用番石榴将`String`转化为`Map`

上面的一个更紧凑的版本是依靠番石榴在一行程序中为我们做分割和转换:

```java
public Map<String, String> convertWithGuava(String mapAsString) {
    return Splitter.on(',').withKeyValueSeparator('=').split(mapAsString);
}
```

## 9.结论

在本文中，我们看到了如何使用核心 Java 方法和第三方库将一个`Map`转换成一个`String`，反之亦然。

所有这些例子的实现都可以在 GitHub 上找到[。](https://web.archive.org/web/20221101145551/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-collections-maps-2)