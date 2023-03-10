# Map.computeIfAbsent()方法

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-map-computeifabsent>

## 1.概观

在本教程中，我们将简单看一下 Java 8 中引入的`Map`接口的新默认方法`computeIfAbsent`。

具体来说，我们将查看它的签名、用法以及它如何处理不同的情况。

## 2.`Map.computeIfAbsent`方法

先来看一下`computeIfAbsent`的签名:

```java
default V computeIfAbsent(K key, Function<? super K, ? extends V> mappingFunction)
```

`computeIfAbsent`方法有两个参数。第一个参数是`key` ,第二个参数是`mappingFunction.` ,重要的是要知道映射函数只有在映射不存在时才会被调用。

### 2.1.与非空值相关的键

首先，它检查`key`是否出现在地图中。如果 `key`存在，并且一个非空值与该键相关，那么它返回该值:

```java
Map<String, Integer> stringLength = new HashMap<>();
stringLength.put("John", 5);
assertEquals((long)stringLength.computeIfAbsent("John", s -> s.length()), 5);
```

正如我们看到的，`key “John”` 有一个非空映射，它返回值 5。如果使用我们的映射函数，我们希望该函数返回长度 4。

### 2.2.使用映射函数计算值

此外，如果地图中不存在`key`或者空值与`key,`相关，那么它会尝试使用给定的`mappingFunction`来计算值。此外，除非计算值为空，否则它会将计算值输入到地图中。

让我们来看看`computeIfAbsent`方法中 `mappingFunction`的用法:

```java
Map<String, Integer> stringLength = new HashMap<>();
assertEquals((long)stringLength.computeIfAbsent("John", s -> s.length()), 4);
assertEquals((long)stringLength.get("John"), 4);
```

由于`key “John”`不存在，它通过将`key`作为参数传递给`mappingFunction`来计算值。

### 2.3.映射函数返回`null`

此外，如果`mappingFunction`返回`null`，则地图不记录任何映射:

```java
Map<String, Integer> stringLength = new HashMap<>();
assertEquals(stringLength.computeIfAbsent("John", s -> null), null);
assertNull(stringLength.get("John"));
```

### 2.4.映射函数抛出异常

最后，如果`mappingFunction`抛出了一个未检查的异常，那么该异常将被再次抛出，并且 map 没有记录任何映射:

```java
@Test(expected = RuntimeException.class)
public void whenMappingFunctionThrowsException_thenExceptionIsRethrown() {
    Map<String, Integer> stringLength = new HashMap<>();
    stringLength.computeIfAbsent("John", s -> { throw new RuntimeException(); });
}
```

我们看到,`mappingFunction`抛出了一个`RuntimeException`,它传播回了`computeIfAbsent`方法。

## 3.结论

在这篇简短的文章中，我们研究了`computeIfAbsent`方法、它的签名和用法。最后，我们看到了它如何处理不同的情况。

和往常一样，所有这些代码示例都可以在 GitHub 上的[处获得。](https://web.archive.org/web/20220627082518/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-collections-maps-3)