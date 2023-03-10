# Java 中 Map 和 MultivaluedMap 的区别

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-map-vs-multivaluedmap>

## 1.概观

在本教程中，我们将学习 Java 中`Map`和`MultivaluedMap`的区别。但在此之前，我们先来看一些例子。

## 2.*图*示例

`HashMap`实现`Map`接口，也允许`null`值和`null`键:

```java
@Test
public void givenHashMap_whenEquals_thenTrue() {
    Map<String, Integer> map = new HashMap<>();

    // Putting key-value pairs into our map.
    map.put("first", 1);
    map.put(null, 2);
    map.put("third", null);

    // The assert statements. The last arguments is what's printed if the assertion fails.
    assertNotNull(map, "The HashMap is null!");
    assertEquals(1, map.get("first"), "The key isn't mapped to the right value!");
    assertEquals(2, map.get(null), "HashMap didn't accept null as key!");
    assertEquals(null, map.get("third"), "HashMap didn't accept null value!");
}
```

上述单元测试成功通过。正如我们所看到的，每个键都映射到一个值。

## 3.如何将`MultivaluedMap`添加到我们的项目中？

在使用`MultivaluedMap`接口及其实现类之前，我们需要将它的库 [Jakarta RESTful WS API](https://web.archive.org/web/20230103152512/https://search.maven.org/artifact/jakarta.ws.rs/jakarta.ws.rs-api) 添加到我们的 Maven 项目中。为此，我们需要在项目的`pom.xml`文件中声明一个依赖项:

```java
<dependency>
    <groupId>jakarta.ws.rs</groupId>
    <artifactId>jakarta.ws.rs-api</artifactId>
    <version>3.1.0</version>
</dependency>
```

我们已经成功地将该库添加到我们的项目中。

## 4.*多值地图*的示例

`MultivaluedHashMap`实现`MultivaluedMap`接口，它允许`null`值和`null `键:

```java
@Test
public void givenMultivaluedHashMap_whenEquals_thenTrue() {
    MultivaluedMap<String, Integer> mulmap = new MultivaluedHashMap<>();

    // Mapping keys to values.
    mulmap.addAll("first", 1, 2, 3);
    mulmap.add(null, null);

    // The assert statements. The last argument is what's printed if the assertion fails.
    assertNotNull(mulmap, "The MultivaluedHashMap is null!");
    assertEquals(1, mulmap.getFirst("first"), "The key isn't mapped to the right values!");
    assertEquals(null, mulmap.getFirst(null), "MultivaluedHashMap didn't accept null!");
}
```

上述单元测试成功通过。这里，每个键都映射到一个值列表，该列表可以包含零个或多个元素。

## 5.有什么区别？

在 Java 生态系统中，`[Map](https://web.archive.org/web/20230103152512/https://docs.oracle.com/en/java/javase/12/docs/api/java.base/java/util/Map.html)`和 [`MultivaluedMap`](https://web.archive.org/web/20230103152512/https://docs.oracle.com/javaee/7/api/javax/ws/rs/core/MultivaluedMap.html) 都是接口。**不同之处在于，在`Map`中，每个键都被映射到一个对象。然而，在一个`MultivaluedMap`中，我们可以有零个或多个对象与同一个键相关联。**

此外，`MultivaluedMap`是`Map,`的一个子接口，所以它有自己的所有方法和一些方法。一些实现`Map`的类有`[HashMap](https://web.archive.org/web/20230103152512/https://docs.oracle.com/en/java/javase/12/docs/api/java.base/java/util/HashMap.html)`、[、`LinkedHashMap`、](https://web.archive.org/web/20230103152512/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/LinkedHashMap.html)`[ConcurrentHashMap](https://web.archive.org/web/20230103152512/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/concurrent/ConcurrentHashMap.html)`、`[WeakHashMap](https://web.archive.org/web/20230103152512/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/WeakHashMap.html)`、`[EnumMap](https://web.archive.org/web/20230103152512/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/EnumMap.html)`和`[TreeMap](https://web.archive.org/web/20230103152512/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/TreeMap.html)`。而且， [`MultivaluedHashMap`](https://web.archive.org/web/20230103152512/https://docs.oracle.com/javaee/7/api/javax/ws/rs/core/MultivaluedHashMap.html) 是同时实现了`Map`和`MultivaluedMap`的类。

例如，`addFirst(K key, V value)`是`MultivaluedMap`的方法之一，它将一个值添加到所提供键的当前值列表的第一个位置:

```java
MultivaluedMap<String, String> mulmap = new MultivaluedHashMap<>();
mulmap.addFirst("firstKey", "firstValue");
```

另一方面，`getFirst(K key)`获得所提供的键的第一个值:

```java
String value = mulmap.getFirst("firstKey");
```

最后，`addAll(K key, V… newValues)`将多个值添加到所提供的键的当前值列表中:

```java
mulmap.addAll("firstKey", "secondValue", "thirdValue");
```

## 6.摘要

在这篇文章中，我们看到了`Map`和`MultivaluedMap`的区别。

和往常一样，完整的源代码可以在 GitHub 上找到[。](https://web.archive.org/web/20230103152512/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-collections-maps-4)