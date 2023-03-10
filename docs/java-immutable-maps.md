# Java 中的不可变映射实现

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-immutable-maps>

## 1.概观

有时最好不允许对`java.util.Map `进行修改，比如跨线程共享只读数据。为此，我们可以使用不可修改的映射或不可变的映射。

在这个快速教程中，我们将看到它们之间的区别。然后，我们将介绍创建不可变地图的各种方法。

## 2.不可修改与不可改变

**不可修改的地图只是可修改地图的包装，不允许直接修改:**

```java
Map<String, String> mutableMap = new HashMap<>();
mutableMap.put("USA", "North America");

Map<String, String> unmodifiableMap = Collections.unmodifiableMap(mutableMap);
assertThrows(UnsupportedOperationException.class,
  () -> unmodifiableMap.put("Canada", "North America"));
```

但是底层的可变映射仍然可以被改变，并且修改也反映在不可修改的映射中:

```java
mutableMap.remove("USA");
assertFalse(unmodifiableMap.containsKey("USA"));

mutableMap.put("Mexico", "North America");
assertTrue(unmodifiableMap.containsKey("Mexico"));
```

另一方面，不可变地图包含自己的私有数据，不允许对其进行修改。因此，一旦创建了不可变映射的实例，数据就不能以任何方式改变。

## 3.番石榴不变的地图

[番石榴](https://web.archive.org/web/20220816153817/https://github.com/google/guava)提供了每个`java.util`不变的版本。`Map`使用`ImmutableMap`。每当我们试图修改它时，它就会抛出一个`UnsupportedOperationException`。

因为它包含自己的私有数据，所以当原始地图改变时，这些数据不会改变。

我们现在将讨论创建`ImmutableMap.`实例的各种方法

### 3.1.使用`copyOf()`方法

首先，让我们使用 `ImmutableMap.copyOf()`方法返回原始地图中所有条目的副本:

```java
ImmutableMap<String, String> immutableMap = ImmutableMap.copyOf(mutableMap);
assertTrue(immutableMap.containsKey("USA"));
```

它不能被直接或间接修改:

```java
assertThrows(UnsupportedOperationException.class,
  () -> immutableMap.put("Canada", "North America"));

mutableMap.remove("USA");
assertTrue(immutableMap.containsKey("USA"));

mutableMap.put("Mexico", "North America");
assertFalse(immutableMap.containsKey("Mexico"));
```

### 3.2.使用`builder()`方法

我们还可以使用`ImmutableMap.builder()`方法创建原始地图中所有条目的副本。

此外，我们可以使用这种方法添加原始地图中不存在的附加条目:

```java
ImmutableMap<String, String> immutableMap = ImmutableMap.<String, String>builder()
  .putAll(mutableMap)
  .put("Costa Rica", "North America")
  .build();
assertTrue(immutableMap.containsKey("USA"));
assertTrue(immutableMap.containsKey("Costa Rica"));
```

与前面的例子一样，我们不能直接或间接地修改它:

```java
assertThrows(UnsupportedOperationException.class,
  () -> immutableMap.put("Canada", "North America"));

mutableMap.remove("USA");
assertTrue(immutableMap.containsKey("USA"));

mutableMap.put("Mexico", "North America");
assertFalse(immutableMap.containsKey("Mexico"));
```

### 3.3.使用`of()`方法

**最后，我们可以使用`ImmutableMap.of()`方法创建一个不可变的映射，其中包含一组动态提供的条目。它最多支持五个键/值对:**

```java
ImmutableMap<String, String> immutableMap
  = ImmutableMap.of("USA", "North America", "Costa Rica", "North America");
assertTrue(immutableMap.containsKey("USA"));
assertTrue(immutableMap.containsKey("Costa Rica"));
```

我们也不能修改它:

```java
assertThrows(UnsupportedOperationException.class,
  () -> immutableMap.put("Canada", "North America"));
```

## 4.结论

在这篇简短的文章中，我们讨论了不可修改的映射和不可变的映射之间的区别。

我们还看了制作番石榴的不同方法

和往常一样，GitHub 上的[提供了完整的代码示例。](https://web.archive.org/web/20220816153817/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-collections-maps)