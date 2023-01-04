# 如何在地图中创建新条目

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-map-new-entry>

## 1.概观

在本教程中，我们将讨论如何使用 Java 的内置类、第三方库和我们的自定义实现来创建一个代表 [`Map`](/web/20220816153818/https://www.baeldung.com/java-map-entry) 中键值关联的`Entry`对象。

## 2.使用 Java 内置类

Java 提供了`Map`。`Entry`用两个简单的实现接口创建一个`Entry`。让我们来看看它们。

### 2.1.使用`AbstractMap`。`SimpleEntry`

`SimpleEntry`类是`AbstractMap` 类中的静态嵌套类。它提供了两种不同的构造函数来初始化实例:

```java
AbstractMap.SimpleEntry<String, String> firstEntry = new AbstractMap.SimpleEntry<>("key1", "value1");
AbstractMap.SimpleEntry<String, String> secondEntry = new AbstractMap.SimpleEntry<>("key2", "value2");
AbstractMap.SimpleEntry<String, String> thirdEntry = new AbstractMap.SimpleEntry<>(firstEntry);
thirdEntry.setValue("a different value");

assertThat(Stream.of(firstEntry, secondEntry, thirdEntry))
  .extracting("key", "value")
  .containsExactly(
    tuple("key1", "value1"),
    tuple("key2", "value2"),
    tuple("key1", "a different value"));
```

正如我们在这里看到的，其中一个构造函数接受键和值，而另一个接受一个`Entry`实例来初始化一个新的`Entry`实例。

### 2.2.使用`AbstractMap`。`SimpleImmutableEntry`

就像使用`SimpleEntry`一样，我们可以使用`SimpleImmutableEntry`来创建条目:

```java
AbstractMap.SimpleImmutableEntry<String, String> firstEntry = new AbstractMap.SimpleImmutableEntry<>("key1", "value1");
AbstractMap.SimpleImmutableEntry<String, String> secondEntry = new AbstractMap.SimpleImmutableEntry<>("key2", "value2");
AbstractMap.SimpleImmutableEntry<String, String> thirdEntry = new AbstractMap.SimpleImmutableEntry<>(firstEntry);

assertThat(Stream.of(firstEntry, secondEntry, thirdEntry))
  .extracting("key", "value")
  .containsExactly(
    tuple("key1", "value1"),
    tuple("key2", "value2"),
    tuple("key1", "value1"));
```

与`SimpleEntry`，**，`SimpleImmutableEntry`不同，它不允许我们在初始化`Entry`实例后改变值。**如果我们试图改变这个值，它会抛出`java.lang.UnsupportedOperationException.`

### 2.3.使用`Map`。`entry`

从版本 9 开始，Java 在`Map` 接口中有一个静态方法`entry()` 来创建一个`Entry`:

```java
Map.Entry<String, String> entry = Map.entry("key", "value");

assertThat(entry.getKey()).isEqualTo("key");
assertThat(entry.getValue()).isEqualTo("value");
```

我们需要记住，以这种方式创建的条目**也是不可变的**，如果我们试图在初始化后改变值，就会导致`java.lang.UnsupportedOperationException`。

## 3.第三方库

除了 Java 本身，还有一些流行的库提供了创建条目的好方法。

### 3.1.使用 Apache `commons-collections4`库

让我们首先包括我们的 [Maven](https://web.archive.org/web/20220816153818/https://mvnrepository.com/artifact/org.apache.commons/commons-collections4/4.4) 依赖项:

```java
<dependency>
    <groupId>org.apache.commons</groupId>
    <artifactId>commons-collections4</artifactId>
</dependency>
```

我们应该提到，除了`Entry`接口，这个库还提供了一个名为`KeyValue:`的接口

```java
Map.Entry<String, String> firstEntry = new DefaultMapEntry<>("key1", "value1");
KeyValue<String, String> secondEntry = new DefaultMapEntry<>("key2", "value2");

KeyValue<String, String> thirdEntry = new DefaultMapEntry<>(firstEntry);
KeyValue<String, String> fourthEntry = new DefaultMapEntry<>(secondEntry);

firstEntry.setValue("a different value");

assertThat(firstEntry)
  .extracting("key", "value")
  .containsExactly("key1", "a different value");

assertThat(Stream.of(secondEntry, thirdEntry, fourthEntry))
  .extracting("key", "value")
  .containsExactly(
    tuple("key2", "value2"),
    tuple("key1", "value1"),
    tuple("key2", "value2"));
```

`DefaultMapEntry`类提供了三种不同的构造函数。第一个接受键-值对，第二个和第三个分别接受参数类型`Entry `和`KeyValue,` 。

`UnmodifiableMapEntry`类也有相同的行为方式:

```java
Map.Entry<String, String> firstEntry = new UnmodifiableMapEntry<>("key1", "value1");
KeyValue<String, String> secondEntry = new UnmodifiableMapEntry<>("key2", "value2");

KeyValue<String, String> thirdEntry = new UnmodifiableMapEntry<>(firstEntry);
KeyValue<String, String> fourthEntry = new UnmodifiableMapEntry<>(secondEntry);

assertThat(firstEntry)
  .extracting("key", "value")
  .containsExactly("key1", "value1");

assertThat(Stream.of(secondEntry, thirdEntry, fourthEntry))
  .extracting("key", "value")
  .containsExactly(
    tuple("key2", "value2"),
    tuple("key1", "value1"),
    tuple("key2", "value2"));
```

但是，从它的名字我们可以理解， **`UnmodifiableMapEntry` 也不允许我们在初始化** `**.**`后改变值

### 3.2.使用谷歌番石榴图书馆

让我们首先包括我们的 [Maven](https://web.archive.org/web/20220816153818/https://mvnrepository.com/artifact/com.google.guava/guava/31.0.1-jre) 依赖关系:

```java
<dependency>
    <groupId>com.google.guava</groupId>
    <artifactId>guava</artifactId>
 </dependency>
```

现在，让我们看看如何使用`immutableEntry()`方法:

```java
Map.Entry<String, String> firstEntry = Maps.immutableEntry("key1", "value1");
Map.Entry<String, String> secondEntry = Maps.immutableEntry("key2", "value2");

assertThat(Stream.of(firstEntry, secondEntry))
  .extracting("key", "value")
  .containsExactly(
    tuple("key1", "value1"),
    tuple("key2", "value2"));
```

**因为它创建了一个不可变的条目，如果我们试图改变这个值，它抛出`java.lang.UnsupportedOperationException.`**

## 4.自定义实现

到目前为止，我们已经看到了一些创建一个`Entry`实例来表示键值关联的选项。**这些类被设计成必须符合`Map`接口实现的内部逻辑，例如 [`HashMap`](/web/20220816153818/https://www.baeldung.com/java-hashmap) 。**

这意味着只要我们遵守相同的，我们就可以创建我们自己的`Entry`接口的实现。首先，让我们添加一个简单的实现:

```java
public class SimpleCustomKeyValue<K, V> implements Map.Entry<K, V> {

    private final K key;
    private V value;

    public SimpleCustomKeyValue(K key, V value) {
        this.key = key;
        this.value = value;
    }
    // standard getters and setters
    // standard equals and hashcode
    // standard toString
}
```

最后，让我们来看几个用法示例:

```java
Map.Entry<String, String> firstEntry = new SimpleCustomKeyValue<>("key1", "value1");

Map.Entry<String, String> secondEntry = new SimpleCustomKeyValue<>("key2", "value2");
secondEntry.setValue("different value");

Map<String, String> map = Map.ofEntries(firstEntry, secondEntry);

assertThat(map)
  .isEqualTo(ImmutableMap.<String, String>builder()
    .put("key1", "value1")
    .put("key2", "different value")
    .build());
```

## 5.结论

在本文中，我们学习了如何使用 Java 提供的现有选项和一些流行的第三方库提供的替代选项来创建一个`Entry`实例。此外，我们还创建了一个自定义实现，并展示了一些使用示例。

和往常一样，这些例子的代码可以在 GitHub 的 [中找到。](https://web.archive.org/web/20220816153818/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-collections-maps-4)