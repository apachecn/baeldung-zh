# 如何检查映射中是否存在键

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-map-key-exists>

## 1。概述

在这个简短的教程中，我们将看看如何检查一个键是否存在于`Map`中。

具体来说，我们将关注`containsKey `和`get.`

## 2。`containsKey`

如果我们看一下[的 JavaDoc 对于`Map#containsKey`的](https://web.archive.org/web/20220628145741/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/Map.html#containsKey(java.lang.Object)):

> 如果此映射包含指定键的映射，则返回`true`

我们可以看到，这种方法是做我们想做的事情的一个很好的选择。

让我们创建一个非常简单的地图，并用`containsKey`验证它的内容:

```java
@Test
public void whenKeyIsPresent_thenContainsKeyReturnsTrue() {
    Map<String, String> map = Collections.singletonMap("key", "value");

    assertTrue(map.containsKey("key"));
    assertFalse(map.containsKey("missing"));
}
```

**简单来说，`containsKey `告诉我们地图是否包含那个键。**

## 3。`get`

现在，`get `有时也能工作，但是它有一些麻烦，这取决于`Map`实现是否支持空值。

再来看看`Map`的 JavaDoc，这次是针对 [`Map#put`](https://web.archive.org/web/20220628145741/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/Map.html#put(K,V)) ，我们看到它只会抛出一个`NullPointerException`:

> 如果指定的键或值为空**并且该映射不允许空键或值**

因为`Map `的一些实现可以有空值(比如`HashMap`)，所以即使键存在，`get`也有可能返回`null`。

**所以，如果我们的目标是查看一个键是否有值，那么`get`就可以了:**

```java
@Test
public void whenKeyHasNullValue_thenGetStillWorks() {
    Map<String, String> map = Collections.singletonMap("nothing", null);

    assertTrue(map.containsKey("nothing"));
    assertNull(map.get("nothing"));
}
```

**但是，如果我们只是试图检查密钥是否存在，那么我们应该坚持使用`containsKey`。**

## 4。结论

在本文中，我们看了一下`containsKey`。我们还仔细研究了为什么使用`get`来验证一个密钥的存在是有风险的。

像往常一样，在 Github 上查看代码示例[。](https://web.archive.org/web/20220628145741/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-collections-maps)