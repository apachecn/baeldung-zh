# 将 Java 流收集到一个不可变的集合中

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-stream-immutable-collection>

## 1。简介

我们经常希望将一个 Java `Stream`转换成一个集合。这通常会导致一个可变的集合，但是我们可以定制它。

在这个简短的教程中，我们将仔细看看**如何将 Java 流收集到一个不可变的集合**——首先使用普通 Java，然后使用 Guava 库。

## 2。 **使用标准 Java**

### 2.1.使用 Java 的`toUnmodifiableList`

从 Java 10 开始，我们可以使用 Java 的`Collectors`类中的`toUnmodifiableList`方法:

```
List<String> givenList = Arrays.asList("a", "b", "c");
List<String> result = givenList.stream()
  .collect(toUnmodifiableList());
```

通过使用这种方法，我们得到了一个不支持来自 Java 的`ImmutableCollections`的`null`值的`List`实现:

```
class java.util.ImmutableCollections$ListN
```

### 2.2。使用 Java 的`collectingAndThen`

Java 的`Collectors`类中的`collectingAndThen`方法接受一个`Collector`和一个`finisher` `Function`。该`finisher` 应用于从`Collector:`返回的结果

```
List<String> givenList = Arrays.asList("a", "b", "c");
List<String> result = givenList.stream()
  .collect(collectingAndThen(toList(), ImmutableList::copyOf));

System.out.println(result.getClass());
```

使用这种方法，**因为我们不能直接使用`toCollection Collector`，我们需要将元素收集到一个临时列表中。**然后，我们从中构造一个不可变列表。

### 2.3。使用`Stream.toList()`方法

Java 16 在[流 API](/web/20220926195021/https://www.baeldung.com/java-8-streams) 上引入了一个名为`toList().` 的新方法。这个简便的方法返回一个包含流元素的**不可修改的`List` :**

```
@Test
public void whenUsingStreamToList_thenReturnImmutableList() {
    List<String> immutableList = Stream.of("a", "b", "c", "d").toList();

    Assertions.assertThrows(UnsupportedOperationException.class, () -> {
        immutableList.add("e");
    });
}
```

正如我们在单元测试中看到的，`Stream.toList()` 返回一个不可变的列表`.` ,因此，试图向列表中添加一个新元素只会导致`[UnsupportedOperationException](/web/20220926195021/https://www.baeldung.com/java-list-unsupported-operation-exception).`

请记住，新的`Stream.toList()` 方法与现有的 [`Collectors.toList()`](/web/20220926195021/https://www.baeldung.com/java-8-collectors#1-collectorstolist) 略有不同，因为它返回一个不可修改的列表。

## 3。`Collector`建筑风俗

我们也可以选择[实现自定义`Collector`](/web/20220926195021/https://www.baeldung.com/java-8-collectors#Custom) 。

### 3.1.一个基本不可变的`Collector`

为了实现这一点，我们可以使用静态的`Collector.of`方法:

```
public static <T> Collector<T, List<T>, List<T>> toImmutableList() {
    return Collector.of(ArrayList::new, List::add,
      (left, right) -> {
        left.addAll(right);
        return left;
      }, Collections::unmodifiableList);
}
```

我们可以像使用任何内置的`Collector`一样使用这个函数:

```
List<String> givenList = Arrays.asList("a", "b", "c", "d");
List<String> result = givenList.stream()
  .collect(MyImmutableListCollector.toImmutableList());
```

最后，让我们检查输出类型:

```
class java.util.Collections$UnmodifiableRandomAccessList
```

### 3.2。使`MyImmutableListCollector`通用

我们的实现有一个限制——它总是返回一个由`ArrayList`支持的不可变实例。但是，稍加改进，我们可以让这个收集器返回用户指定的类型:

```
public static <T, A extends List<T>> Collector<T, A, List<T>> toImmutableList(
  Supplier<A> supplier) {

    return Collector.of(
      supplier,
      List::add, (left, right) -> {
        left.addAll(right);
        return left;
      }, Collections::unmodifiableList);
}
```

所以现在，我们不是在方法实现中确定`Supplier`，而是向用户请求`Supplier`:

```
List<String> givenList = Arrays.asList("a", "b", "c", "d");
List<String> result = givenList.stream()
  .collect(MyImmutableListCollector.toImmutableList(LinkedList::new));
```

此外，我们使用的是`LinkedList`而不是`ArrayList`。

```
class java.util.Collections$UnmodifiableList
```

这一次，我们得到了`UnmodifiableList`而不是`UnmodifiableRandomAccessList`。

## 4。用芭乐的`Collector` s

在本节中，我们将使用 [Google Guava](https://web.archive.org/web/20220926195021/https://search.maven.org/classic/#search%7Cgav%7C1%7Cg%3A%22com.google.guava%22%20AND%20a%3A%22guava%22) 库来驱动我们的一些示例:

```
<dependency>
    <groupId>com.google.guava</groupId>
    <artifactId>guava</artifactId>
    <version>31.1-jre</version>
</dependency>
```

从 Guava 21 开始，每个不可变的类都附带了一个像 Java 的标准`Collector` s `:`一样容易使用的`Collector`

```
List<Integer> list = IntStream.range(0, 9)
  .boxed()
  .collect(ImmutableList.toImmutableList());
```

产生的实例是`RegularImmutableList`:

```
class com.google.common.collect.RegularImmutableList
```

## 5。结论

在这篇短文中，我们看到了将一个`Stream`收集到一个不可变的`Collection`中的各种方法。

和往常一样，本文的完整源代码在 GitHub 上。它们被 Java 版本分成[第 3-4 节](https://web.archive.org/web/20220926195021/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-streams-2)、[第 2.2 节](https://web.archive.org/web/20220926195021/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-10/)和[第 2.3 节](https://web.archive.org/web/20220926195021/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-16/)的例子。