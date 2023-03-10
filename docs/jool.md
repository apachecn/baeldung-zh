# jOOL 简介

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/jool>

## 1。概述

在本文中，我们将关注 [jOOL](https://web.archive.org/web/20220523150344/https://github.com/jOOQ/jOOL) 库——来自 [jOOQ](https://web.archive.org/web/20220523150344/https://www.jooq.org/) 的另一个产品。

## 2。Maven 依赖关系

让我们从向您的`pom.xml`添加一个 Maven 依赖项开始:

```java
<dependency>
    <groupId>org.jooq</groupId>
    <artifactId>jool</artifactId>
    <version>0.9.12</version>
</dependency> 
```

你可以在这里找到最新版本[。](https://web.archive.org/web/20220523150344/https://search.maven.org/classic/#search%7Cgav%7C1%7Cg%3A%22org.jooq%22%20AND%20a%3A%22jool%22)

## 3。功能接口

在 Java 8 中，函数接口相当有限。它们接受最大数量的两个参数，并且没有太多附加功能。

jOOL 通过提供一组新的功能接口来解决这个问题，这些接口甚至可以接受 16 个参数(从`[Function1](https://web.archive.org/web/20220523150344/http://www.jooq.org/products/jOO%CE%BB/javadoc/0.9.11/index.html?org/jooq/lambda/function/Function1.html)` 到`[Function16](https://web.archive.org/web/20220523150344/http://www.jooq.org/products/jOO%CE%BB/javadoc/0.9.11/index.html?org/jooq/lambda/function/Function16.html))` ),并且增加了额外的简便方法。

例如，要创建一个有三个参数的函数，我们可以使用`Function3:`

```java
Function3<String, String, String, Integer> lengthSum
  = (v1, v2, v3) -> v1.length() + v2.length() + v3.length();
```

在纯 Java 中，您需要自己实现它。除此之外，jOOL 的函数接口有一个方法`applyPartially()`,允许我们轻松地执行部分应用程序:

```java
Function2<Integer, Integer, Integer> addTwoNumbers = (v1, v2) -> v1 + v2;
Function1<Integer, Integer> addToTwo = addTwoNumbers.applyPartially(2);

Integer result = addToTwo.apply(5);

assertEquals(result, (Integer) 7);
```

当我们有一个`Function2` 类型的方法时，我们可以通过使用一个`toBiFunction()`方法很容易地将它转换成一个标准的 Java `BiFunction` :

```java
BiFunction biFunc = addTwoNumbers.toBiFunction();
```

同样的，`Function1`类型中也有一个`toFunction()` 方法。

## 4。元组

元组是函数式编程世界中非常重要的构造。它是一个值的类型化容器，每个值可以有不同的类型。**元组常用作函数自变量**。

在对一系列事件进行转换时，它们也非常有用。在 jOOL 中，我们有可以包装从 1 到 16 个值的元组，由`Tuple1` 到 `Tuple16`类型提供:

```java
tuple(2, 2)
```

对于四个值:

```java
tuple(1,2,3,4); 
```

让我们考虑一个例子，当我们有一个携带 3 个值的元组序列时:

```java
Seq<Tuple3<String, String, Integer>> personDetails = Seq.of(
  tuple("michael", "similar", 49),
  tuple("jodie", "variable", 43));
Tuple2<String, String> tuple = tuple("winter", "summer");

List<Tuple4<String, String, String, String>> result = personDetails
  .map(t -> t.limit2().concat(tuple)).toList();

assertEquals(
  result,
  Arrays.asList(tuple("michael", "similar", "winter", "summer"), tuple("jodie", "variable", "winter", "summer"))
);
```

我们可以对元组使用不同种类的变换。首先，我们调用一个`limit2()` 方法从`Tuple3\.` 中只取两个值，然后，我们调用一个`concat()` 方法来连接两个元组。

结果，我们得到了一个`Tuple4` 类型的值。

## 5。`Seq`

`Seq`构造在`Stream` 上增加了更高级别的方法，同时经常使用它下面的方法。

### 5.1。包含操作

我们可以找到一些方法的变体来检查`Seq.` 中元素的存在，其中一些方法使用来自`Stream` 类的`anyMatch()` 方法:

```java
assertTrue(Seq.of(1, 2, 3, 4).contains(2));

assertTrue(Seq.of(1, 2, 3, 4).containsAll(2, 3));

assertTrue(Seq.of(1, 2, 3, 4).containsAny(2, 5)); 
```

### 5.2。加入操作

当我们有两个流并且想要连接它们时(类似于两个数据集的 SQL 连接操作)，使用标准的`Stream` 类不是一个非常好的方法:

```java
Stream<Integer> left = Stream.of(1, 2, 4);
Stream<Integer> right = Stream.of(1, 2, 3);

List<Integer> rightCollected = right.collect(Collectors.toList());
List<Integer> collect = left
  .filter(rightCollected::contains)
  .collect(Collectors.toList());

assertEquals(collect, Arrays.asList(1, 2));
```

我们需要将`right` 流收集到一个列表中，以防止`java.lang.IllegalStateException: stream has already been operated upon or closed.`接下来，我们需要通过从`filter` 方法访问`rightCollected` 列表来进行副作用操作。这是一种容易出错且不优雅的连接两个数据集的方法。

幸运的是， **`Seq` 有对数据集做内、左、右连接的有用方法。**这些方法隐藏了一个公开优雅 API 的实现。

我们可以使用`innerJoin()`方法进行内部连接:

```java
assertEquals(
  Seq.of(1, 2, 4).innerJoin(Seq.of(1, 2, 3), (a, b) -> a == b).toList(),
  Arrays.asList(tuple(1, 1), tuple(2, 2))
);
```

我们可以相应地进行左右连接:

```java
assertEquals(
  Seq.of(1, 2, 4).leftOuterJoin(Seq.of(1, 2, 3), (a, b) -> a == b).toList(),
  Arrays.asList(tuple(1, 1), tuple(2, 2), tuple(4, null))
);

assertEquals(
  Seq.of(1, 2, 4).rightOuterJoin(Seq.of(1, 2, 3), (a, b) -> a == b).toList(),
  Arrays.asList(tuple(1, 1), tuple(2, 2), tuple(null, 3))
);
```

甚至有一种`crossJoin()` 方法可以对两个数据集进行笛卡尔连接:

```java
assertEquals(
  Seq.of(1, 2).crossJoin(Seq.of("A", "B")).toList(),
  Arrays.asList(tuple(1, "A"), tuple(1, "B"), tuple(2, "A"), tuple(2, "B"))
);
```

### 5.3。操纵着`Seq`

`Seq` 有许多有用的方法来操作元素序列。让我们来看看其中的一些。

我们可以使用`cycle()` 方法从源序列中重复提取元素。这将创建一个无限流，因此我们在将结果收集到一个列表中时需要小心，因此我们需要使用一个`limit()` 方法将无限序列转换为有限序列:

```java
assertEquals(
  Seq.of(1, 2, 3).cycle().limit(9).toList(),
  Arrays.asList(1, 2, 3, 1, 2, 3, 1, 2, 3)
);
```

假设我们想要将一个序列中的所有元素复制到第二个序列中。`duplicate()` 方法正是这样做的:

```java
assertEquals(
  Seq.of(1, 2, 3).duplicate().map((first, second) -> tuple(first.toList(), second.toList())),
  tuple(Arrays.asList(1, 2, 3), Arrays.asList(1, 2, 3))
); 
```

`duplicate()` 方法的返回类型是两个序列的元组。

假设我们有一个整数序列，我们想用一些谓词把这个序列分成两个序列。我们可以使用一种`partition()`方法:

```java
assertEquals(
  Seq.of(1, 2, 3, 4).partition(i -> i > 2)
    .map((first, second) -> tuple(first.toList(), second.toList())),
  tuple(Arrays.asList(3, 4), Arrays.asList(1, 2))
);
```

### 5.4。分组元素

使用`Stream` API 按键对元素进行分组既麻烦又不直观——因为我们需要使用带有`Collectors.groupingBy` 收集器的`collect()` 方法。

`Seq` 将代码隐藏在返回`Map` 的`groupBy()` 方法之后，因此不需要显式使用`collect()`方法:

```java
Map<Integer, List<Integer>> expectedAfterGroupBy = new HashMap<>();
expectedAfterGroupBy.put(1, Arrays.asList(1, 3));
expectedAfterGroupBy.put(0, Arrays.asList(2, 4));

assertEquals(
  Seq.of(1, 2, 3, 4).groupBy(i -> i % 2),
  expectedAfterGroupBy
);
```

### 5.5。跳过元素

假设我们有一个元素序列，当一个谓词不匹配时，我们想跳过元素。当一个谓词被满足时，元素应该以一个结果序列着陆。

我们可以使用一个`skipWhile()` 方法:

```java
assertEquals(
  Seq.of(1, 2, 3, 4, 5).skipWhile(i -> i < 3).toList(),
  Arrays.asList(3, 4, 5)
);
```

我们可以使用`skipUntil()` 方法获得相同的结果:

```java
assertEquals(
  Seq.of(1, 2, 3, 4, 5).skipUntil(i -> i == 3).toList(),
  Arrays.asList(3, 4, 5)
);
```

### 5.6。拉链序列

当我们处理元素序列时，通常需要将它们压缩成一个序列。

可以用来将两个序列压缩成一个序列的 API:

```java
assertEquals(
  Seq.of(1, 2, 3).zip(Seq.of("a", "b", "c")).toList(),
  Arrays.asList(tuple(1, "a"), tuple(2, "b"), tuple(3, "c"))
);
```

结果序列包含两个元素的元组。

当我们压缩两个序列时，但是我们希望以特定的方式压缩它们，我们可以将一个`BiFunction` 传递给一个`zip()` 方法，该方法定义压缩元素的方式:

```java
assertEquals(
  Seq.of(1, 2, 3).zip(Seq.of("a", "b", "c"), (x, y) -> x + ":" + y).toList(),
  Arrays.asList("1:a", "2:b", "3:c")
);
```

有时，通过`zipWithIndex()` API，用序列中的元素索引压缩序列是很有用的:

```java
assertEquals(
  Seq.of("a", "b", "c").zipWithIndex().toList(),
  Arrays.asList(tuple("a", 0L), tuple("b", 1L), tuple("c", 2L))
);
```

## 6。将选中的异常转换为未选中的

假设我们有一个方法，它接受一个字符串并可以抛出一个检查过的异常:

```java
public Integer methodThatThrowsChecked(String arg) throws Exception {
    return arg.length();
}
```

然后我们想要映射一个`Stream` 的元素，将该方法应用于每个元素。没有办法更高层次地处理那个异常，所以我们需要在一个`map()` 方法中处理那个异常:

```java
List<Integer> collect = Stream.of("a", "b", "c").map(elem -> {
    try {
        return methodThatThrowsChecked(elem);
    } catch (Exception e) {
        e.printStackTrace();
        throw new RuntimeException(e);
    }
}).collect(Collectors.toList());

assertEquals(
    collect,
    Arrays.asList(1, 1, 1)
);
```

由于 Java 中函数接口的设计，我们对该异常无能为力，因此在 catch 子句中，我们将检查的异常转换为未检查的异常。

**幸运的是，在 jOOL 中有一个`Unchecked` 类，它的方法可以将检查异常转换成未检查异常:**

```java
List<Integer> collect = Stream.of("a", "b", "c")
  .map(Unchecked.function(elem -> methodThatThrowsChecked(elem)))
  .collect(Collectors.toList());

assertEquals(
  collect,
  Arrays.asList(1, 1, 1)
);
```

我们将对一个`methodThatThrowsChecked()` 的调用包装到一个`Unchecked.function()` 方法中，该方法处理底层异常的转换。

## 7。结论

本文展示了如何使用 jOOL 库，该库向 Java 标准`Stream` API 添加了有用的附加方法。

所有这些示例和代码片段的实现都可以在 [GitHub 项目](https://web.archive.org/web/20220523150344/https://github.com/eugenp/tutorials/tree/master/libraries-5)中找到——这是一个 Maven 项目，因此它应该很容易导入和运行。