# Java 和 Vavr 之间的互操作性

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-vavr>

## 1。概述

由于 Vavr 主要在 Java 生态系统中工作，所以总是需要将 Vavr 的数据结构转换成 Java 可理解的数据结构。

例如，考虑一个返回`io.vavr.collection.List`的函数，我们需要将结果传递给另一个接受`java.util.List.`的函数，这就是 Java-Vavr 互操作性派上用场的地方。

在本教程中，我们将看看如何**将几个 Vavr 数据结构转换成我们的标准 Java 集合，反之亦然**。

## 2。Vavr 到 Java 的转换

Vavr 中的`Value`接口是大多数 Vavr 工具的基础接口。因此，所有 Vavr 的集合都继承了`Value.`的属性

这很有用，因为**`Value`接口附带了许多`toJavaXXX()`方法**，允许我们将 Vavr 数据结构转换成 Java 等价物。

我们来看看如何从 Vavr 的`List`或`Stream`中获取一个 Java `List`:

```java
List<String> vavrStringList = List.of("JAVA", "Javascript", "Scala");
java.util.List<String> javaStringList = vavrStringList.toJavaList();
```

```java
Stream<String> vavrStream = Stream.of("JAVA", "Javascript", "Scala");
java.util.List<String> javaStringList = vavrStream.toJavaList();
```

第一个示例将 Vavr 列表转换为 Java 列表，下一个示例将流转换为 Java 列表。这两个例子都依赖于`toJavaList()`方法。

同样，**我们可以从 Vavr 对象**中获取其他 Java 集合。

让我们看另一个将 Vavr `Map`转换成 Java `Map:`的例子

```java
Map<String, String> vavrMap = HashMap.of("1", "a", "2", "b", "3", "c");
java.util.Map<String, String> javaMap = vavrMap.toJavaMap();
```

除了标准的 Java 集合， **Vavr 还提供了将值转换成 Java 流的 API 和** `**Optionals**.`

让我们看一个使用`toJavaOptional()`方法获得`Optional `的例子:

```java
List<String> vavrList = List.of("Java");
Optional<String> optional = vavrList.toJavaOptional();
assertEquals("Java", optional.get());
```

作为这种类型的 Vavr 方法的概述，我们有:

*   `toJavaArray()`
*   `toJavaCollection()`
*   `toJavaList()`
*   `toJavaMap()`
*   `toJavaSet()`
*   `toJavaOptional()`
*   `toJavaParallelStream()`
*   `toJavaStream()`

有用 API 的完整列表可以在[这里](https://web.archive.org/web/20220627173524/https://www.javadoc.io/doc/io.vavr/vavr/0.9.2)找到。

## 3。Java 到 Vavr 的转换

Vavr 中的所有集合实现都有一个基本类型`Traversable. `，因此，**每个集合类型都有一个静态方法`ofAll()`，它接受一个`Iterable`并将其转换为相应的 Vavr 集合**。

让我们看看如何将`java.util.List `转换成 Vavr `List`:

```java
java.util.List<String> javaList = Arrays.asList("Java", "Haskell", "Scala");
List<String> vavrList = List.ofAll(javaList); 
```

类似地，我们可以使用`ofAll()`方法将 Java 流转换成 Vavr 集合:

```java
java.util.stream.Stream<String> javaStream 
  = Arrays.asList("Java", "Haskell", "Scala").stream();
Stream<String> vavrStream = Stream.ofAll(javaStream);
```

## 4。Java 集合视图

Vavr 库还提供了 **Java 集合视图，这些视图将调用委托给底层 Vavr 集合**。

Vavr 到 Java 的转换方法通过遍历所有元素来构建 Java 集合，从而创建一个新的实例。这意味着转换的性能是线性的，而创建集合视图的性能是恒定的。

截至撰写本文时，Vavr 中仅支持`List`视图。

对于`List`，有两种方法可以获得我们的视图。第一个是返回不可变的 T2 的 T1，下一个是 T3

这里有一个展示不可变 Java `List`的例子:

```java
@Test(expected = UnsupportedOperationException.class)
public void givenParams_whenVavrListConverted_thenException() {
    java.util.List<String> javaList 
      = List.of("Java", "Haskell", "Scala").asJava();

    javaList.add("Python");
    assertEquals(4, javaList.size());
}
```

由于`List`是不可变的，对它执行任何修改都会抛出一个`UnsupportedOperationException`。

我们也可以通过调用`List.`上的`asJavaMutable() `方法来获得一个可变的`List`

我们是这样做的:

```java
@Test
public void givenParams_whenVavrListConvertedToMutable_thenRetunMutableList() {
    java.util.List<String> javaList = List.of("Java", "Haskell", "Scala")
      .asJavaMutable();
    javaList.add("Python");

    assertEquals(4, javaList.size());
}
```

## 5。Vavr 对象之间的转换

类似于 Java 到 Vavr 的转换，反之亦然，我们可以将 Vavr 中的一个`Value`类型转换成其他的`Value`类型。此转换功能有助于在需要时在 Vavr 对象之间进行转换。

例如，我们有一个`List`条目，我们希望在保留顺序的同时过滤掉重复的条目。在这种情况下，我们需要一个`LinkedHashSet`。这里有一个演示使用案例的示例:

```java
List<String> vavrList = List.of("Java", "Haskell", "Scala", "Java");
Set<String> linkedSet = vavrList.toLinkedSet();
assertEquals(3, linkedSet.size());
assertTrue(linkedSet instanceof LinkedHashSet);
```

在 Value 接口中有许多其他方法可以帮助我们根据用例将集合转换成不同的类型。

API 的完整列表可以在[这里](https://web.archive.org/web/20220627173524/https://static.javadoc.io/io.javaslang/javaslang/2.1.0-alpha/javaslang/Value.html)找到。

## 6。结论

在本文中，我们学习了 Vavr 和 Java 集合类型之间的转换。要查看框架根据用例为转换提供的更多 API，请参考 [JavaDoc](https://web.archive.org/web/20220627173524/https://static.javadoc.io/io.vavr/vavr/0.9.0/io/vavr/collection/package-frame.html) 和[用户指南](https://web.archive.org/web/20220627173524/http://www.vavr.io/vavr-docs/)。

本文中所有示例的完整源代码可以在 Github 的[中找到。](https://web.archive.org/web/20220627173524/https://github.com/eugenp/tutorials/tree/master/vavr-2)