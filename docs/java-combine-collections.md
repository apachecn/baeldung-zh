# 在 Java 中组合不同类型的集合

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-combine-collections>

## 1。简介

在这个快速教程中，我们将探索在 Java 中组合集合的不同方法。

我们将探索使用 Java 和外部框架(如 Guava、Apache 等)的各种方法。关于系列的介绍，请看这个系列的[这里](/web/20221109224619/https://www.baeldung.com/java-collections)。

## 2。使用集合的外部库

除了本地方法，我们还将使用外部库。请在`pom.xml`中添加以下依赖项:

```java
<dependency>
    <groupId>org.apache.commons</groupId>
    <artifactId>commons-collections4</artifactId>
    <version>4.2</version>
</dependency>
<dependency>
    <groupId>org.apache.commons</groupId>
    <artifactId>commons-exec</artifactId>
    <version>1.3</version>
</dependency>
<dependency>
    <groupId>com.google.guava</groupId>
    <artifactId>guava</artifactId>
    <version>31.0.1-jre</version>
</dependency>
```

最新版本可以在 Maven Central 上找到 [Commons](https://web.archive.org/web/20221109224619/https://search.maven.org/classic/#search%7Cga%7C1%7Ca%3A%22commons-collections4%22) 、 [Commons-exec](https://web.archive.org/web/20221109224619/https://search.maven.org/classic/#search%7Cga%7C1%7Ca%3A%22commons-exec%22) 和 [Guava](https://web.archive.org/web/20221109224619/https://search.maven.org/classic/#search%7Cgav%7C1%7Cg%3A%22com.google.guava%22%20AND%20a%3A%22guava%22) 。

## 3。在 Java 中组合数组

### 3.1。原生 Java 解决方案

**Java 自带一个内置的`[void arraycopy()](https://web.archive.org/web/20221109224619/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/lang/System.html#arraycopy(java.lang.Object,int,java.lang.Object,int,int))`方法，它将给定的源数组复制到目标数组。**

我们可以按以下方式使用它:

```java
Object[] combined = new Object[first.length + second.length];
System.arraycopy(first, 0, combined, 0, first.length);
System.arraycopy(second, 0, combined, first.length, second.length);
```

在这个方法中，除了数组对象，我们还指定了需要复制的位置，并且我们还传递了长度参数。

这是一个本地 Java 解决方案，所以它不需要任何外部库。

### 3.2。使用 Java 8 `Stream` API

流提供了一种有效的方法来迭代几种不同类型的集合。要开始使用 streams，请阅读 [Java 8 Stream API 教程](/web/20221109224619/https://www.baeldung.com/java-8-streams)。

要使用`Stream`组合数组，我们可以使用以下代码:

```java
Object[] combined = Stream.concat(Arrays.stream(first), Arrays.stream(second)).toArray();
```

**`[Stream.concat()](https://web.archive.org/web/20221109224619/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/stream/Stream.html#concat(java.util.stream.Stream,java.util.stream.Stream))`创建一个串接的流，其中第一个流的元素后面跟随着第二个流的元素，然后使用`toArray() `方法将其转换为一个数组。**

不同类型的集合创建流的过程是相同的。但是，我们可以用不同的方式收集它，从中检索不同的数据结构。

我们将在 4.2 节中再次讨论这个方法。和 5.2。看看我们如何在`Lists`和`Sets`上使用相同的方法。

### 3.3。使用来自 Apache Commons 的`ArrayUtils`

Apache commons 库为我们提供了来自 [`ArrayUtils`](https://web.archive.org/web/20221109224619/https://commons.apache.org/proper/commons-lang/javadocs/api-release/org/apache/commons/lang3/ArrayUtils.html) 包的 [`addAll()`](https://web.archive.org/web/20221109224619/https://commons.apache.org/proper/commons-lang/javadocs/api-release/org/apache/commons/lang3/ArrayUtils.html#addAll-T:A-T...-) 方法。我们可以提供目标和源数组作为参数，这个方法将返回一个组合数组:

```java
Object[] combined = ArrayUtils.addAll(first, second);
```

在使用 Apache Commons Lang 3 文章的[数组处理中也详细讨论了这个方法。](/web/20221109224619/https://www.baeldung.com/array-processing-commons-lang)

### 3.4。使用番石榴

出于同样的目的，番石榴为我们提供了`[concat()](https://web.archive.org/web/20221109224619/https://google.github.io/guava/releases/19.0/api/docs/com/google/common/collect/ObjectArrays.html#concat(T[],%20T[],%20java.lang.Class))`方法:

```java
Object [] combined = ObjectArrays.concat(first, second, Object.class);
```

它可以用于不同的数据类型，并且它接受两个源数组和类文本来返回组合数组。

## 4。Java 中的组合`List`

### 4.1。使用`Collection`原生`addAll()`方法

[`Collection`](https://web.archive.org/web/20221109224619/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/Collection.html) 接口本身为我们提供了 [`addAll()`](https://web.archive.org/web/20221109224619/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/Collection.html#addAll(java.util.Collection)) 方法，将指定集合中的所有元素添加到调用者对象中。这也在[这篇 Baeldung 文章中详细讨论:](/web/20221109224619/https://www.baeldung.com/java-combine-multiple-collections)

```java
List<Object> combined = new ArrayList<>();
combined.addAll(first);
combined.addAll(second);
```

由于该方法是在集合框架的最父接口，即`Collection`接口中提供的，因此可以应用于所有的`List`和`Set`

### 4.2。使用 Java 8

我们可以按照以下方式使用`Stream`和`Collectors`来组合`Lists`:

```java
List<Object> combined = Stream.concat(first.stream(), second.stream()).collect(Collectors.toList());
```

这与我们在 3.2 节中对`Arrays` 所做的一样，但是我们没有将它转换成数组，而是使用收集器将它转换成列表。要详细了解`Collectors`，请访问[Java 8 收集器指南](/web/20221109224619/https://www.baeldung.com/java-8-collectors)。

我们也可以这样使用`flatMaps` :

```java
List<Object> combined = Stream.of(first, second).flatMap(Collection::stream).collect(Collectors.toList());
```

**首先，我们使用 [`Stream.of()`](https://web.archive.org/web/20221109224619/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/stream/Stream.html#of(T...)) 返回两个列表的顺序流——`first`和`second`。然后我们将它传递给`flatMap`，它将在应用映射函数后返回映射流的内容。**这个方法也在[在 Java 中合并流](/web/20221109224619/https://www.baeldung.com/java-merge-streams)一文中讨论过。

要了解更多关于`flatMap`的信息，请阅读[这篇关于](/web/20221109224619/https://www.baeldung.com/java-difference-map-and-flatmap)的文章。

### 4.3。使用来自 Apache Commons 的`ListUtils`

`CollectionUtils.union `执行两个集合的并集并返回一个包含所有元素的集合:

```java
List<Object> combined = ListUtils.union(first, second);
```

这个方法在[Apache Commons Collections 指南`CollectionUtils`](/web/20221109224619/https://www.baeldung.com/apache-commons-collection-utils) 中也有讨论。欲了解更多信息，请参阅第 4.9 节。这篇文章的。

### 4.4。使用番石榴

**要使用番石榴合并一个`List`，我们将使用由`concat()`方法组成的`Iterable`。**连接所有集合后，我们可以快速获得组合的`List`对象，如下例所示:

```java
Iterable<Object> combinedIterables = Iterables
  .unmodifiableIterable(Iterables.concat(first, second));
List<Object> combined = Lists.newArrayList(combinedIterables);
```

## 5。Java 中的组合`Set`

### 5.1。普通 Java 解决方案

正如我们在 4.1 节已经讨论过的。，[集合](https://web.archive.org/web/20221109224619/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/Collection.html)接口自带 [`addAll()`](https://web.archive.org/web/20221109224619/https://commons.apache.org/proper/commons-lang/javadocs/api-release/org/apache/commons/lang3/ArrayUtils.html#addAll-T:A-T...-) 方法，也可用于复制`Lists`和`Sets`:

```java
Set<Object> combined = new HashSet<>();
combined.addAll(first);
combined.addAll(second);
```

### 5.2。使用 Java 8 流

我们用于`List `对象的相同函数可以应用于此:

```java
Set<Object> combined = Stream
  .concat(first.stream(), second.stream())
  .collect(Collectors.toSet());
```

**与 list 相比，这里唯一值得注意的区别是，我们没有使用 [`Collectors.toList()`](https://web.archive.org/web/20221109224619/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/stream/Collectors.html#toList()) ，而是使用 [`Collectors.toSet()`](https://web.archive.org/web/20221109224619/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/stream/Collectors.html#toSet()) 将提供的两个流中的所有元素累积到一个新的`Set`中。**

与`Lists`类似，当在`Sets`上使用`flatMaps `时，它看起来像:

```java
Set<Object> combined = Stream.of(first, second)
  .flatMap(Collection::stream)
  .collect(Collectors.toSet());
```

### 5.3。使用 Apache Commons

类似于`ListUtils`，我们也可以使用`SetUtils`来完成`Set`元素的联合:

```java
Set<Object> combined = SetUtils.union(first, second);
```

### 5.4。使用番石榴

番石榴库为我们提供了简单明了的`Sets.union()`方法来组合 Java 中的`Sets`:

```java
Set<Object> combined = Sets.union(first, second);
```

## 6。Java 中的组合`Map`

### 6.1。普通 Java 解决方案

我们可以利用`Map`接口，该接口本身为我们提供了`putAll()`方法，该方法将所有映射从`Map`对象的参数复制到调用者`Map`对象:

```java
Map<Object, Object> combined = new HashMap<>();
combined.putAll(first);
combined.putAll(second);
```

### 6.2。使用 Java 8

**从 Java 8 开始，`Map`类由接受一个键、值和一个双函数的`merge()` 方法组成。**我们可以使用 Java 8 [forEach](/web/20221109224619/https://www.baeldung.com/foreach-java) 语句来实现合并功能:

```java
second.forEach((key, value) -> first.merge(key, value, String::concat));
```

第三个参数，即重映射函数，在两个源映射中都存在相同的键-值对时很有用。该函数指定应该如何处理这些类型的值。

我们也可以这样使用`flatMap`:

```java
Map<String, String> combined = Stream.of(first, second)
  .map(Map::entrySet)
  .flatMap(Collection::stream)
  .collect(Collectors.toMap(Map.Entry::getKey, Map.Entry::getValue, String::concat));
```

### 6.3。使用 Apache Commons Exec

[Apache Commons Exec](https://web.archive.org/web/20221109224619/https://commons.apache.org/proper/commons-exec/) 为我们提供了一个直截了当的 [`merge(Map<K,V> first, Map<K,V> second)`](https://web.archive.org/web/20221109224619/https://commons.apache.org/proper/commons-exec/apidocs/org/apache/commons/exec/util/MapUtils.html#merge(java.util.Map,%20java.util.Map)) 方法:

```java
Map<String, String> combined = MapUtils.merge(first, second);
```

### 6.4。使用谷歌番石榴

我们可以使用 Google 的番石榴库提供的`[ImmutableMap](https://web.archive.org/web/20221109224619/https://google.github.io/guava/releases/21.0/api/docs/com/google/common/collect/ImmutableMap.html) `。它的`[putAll()](https://web.archive.org/web/20221109224619/https://google.github.io/guava/releases/21.0/api/docs/com/google/common/collect/ImmutableMap.Builder.html#putAll-java.util.Map-)`方法将所有给定映射的键和值关联到构建的映射中:

```java
Map<String, String> combined = ImmutableMap.<String, String>builder()
  .putAll(first)
  .putAll(second)
  .build();
```

## 7。结论

在本文中，我们通过不同的方法来组合不同类型的`Collections`。我们合并了`arrays`、`Lists`、`Sets`和`Maps`。

和往常一样，完整的代码片段及其适当的单元测试可以在 GitHub 上找到。