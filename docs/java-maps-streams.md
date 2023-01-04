# 使用流处理地图

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-maps-streams>

## 1.介绍

在本教程中，我们将讨论一些如何使用[Java`Stream`s](/web/20220930004311/https://www.baeldung.com/java-8-streams-introduction)与`[Map](/web/20220930004311/https://www.baeldung.com/java-hashmap)`s 协同工作的例子。值得注意的是，其中一些练习可以使用双向`Map`数据结构来解决，但是我们在这里对函数方法感兴趣。

首先，我们将解释使用`Maps`和`Stream` s 的基本思想。然后，我们将介绍几个与`Maps`相关的不同问题以及使用`Stream` s 的具体解决方案

## 延伸阅读:

## [用 Java 8 合并两张地图](/web/20220930004311/https://www.baeldung.com/java-merge-maps)

Learn different techniques for merging maps in Java 8[Read more](/web/20220930004311/https://www.baeldung.com/java-merge-maps) →

## [Java 8 Collectors toMap](/web/20220930004311/https://www.baeldung.com/java-collectors-tomap)

Learn how to use the toMap() method of the Collectors class.[Read more](/web/20220930004311/https://www.baeldung.com/java-collectors-tomap) →

## [Java 8 Stream API 教程](/web/20220930004311/https://www.baeldung.com/java-8-streams)

The article is an example-heavy introduction of the possibilities and operations offered by the Java 8 Stream API.[Read more](/web/20220930004311/https://www.baeldung.com/java-8-streams) →

## 2.基本想法

要注意的主要事情是*流*是可以从`Collection`中容易地获得的元素序列。

`Maps`具有不同的结构，具有从键到值的映射，没有顺序。然而，这并不意味着我们不能将一个`Map`结构转换成不同的序列，然后允许我们以自然的方式使用 Stream API。

让我们看看从一个`Map`中获得不同的`Collection`的方法，然后我们可以将它们转换成一个`Stream`:

```java
Map<String, Integer> someMap = new HashMap<>();
```

我们可以获得一组键值对:

```java
Set<Map.Entry<String, Integer>> entries = someMap.entrySet();
```

我们还可以获得与`Map`相关联的键集:

```java
Set<String> keySet = someMap.keySet();
```

或者我们可以直接使用这些值:

```java
Collection<Integer> values = someMap.values();
```

这些都为我们提供了一个通过从这些集合中获取流来处理这些集合的入口点:

```java
Stream<Map.Entry<String, Integer>> entriesStream = entries.stream();
Stream<Integer> valuesStream = values.stream();
Stream<String> keysStream = keySet.stream();
```

## 3.使用`Stream` s 获取`Map`的密钥

### 3.1.输入数据

让我们假设我们有一个`Map`:

```java
Map<String, String> books = new HashMap<>();
books.put(
"978-0201633610", "Design patterns : elements of reusable object-oriented software");
books.put(
  "978-1617291999", "Java 8 in Action: Lambdas, Streams, and functional-style programming");
books.put("978-0134685991", "Effective Java");
```

我们有兴趣找到名为“有效的 Java”的图书的 ISBN。

### 3.2.检索匹配项

由于书名不可能存在于我们的`Map`中，我们希望能够指出它没有相关的 ISBN。我们可以用一个 **[`Optional`](/web/20220930004311/https://www.baeldung.com/java-optional)** 来表示:

在这个例子中，我们假设我们对与书名相匹配的任何一本书的关键字感兴趣:

```java
Optional<String> optionalIsbn = books.entrySet().stream()
  .filter(e -> "Effective Java".equals(e.getValue()))
  .map(Map.Entry::getKey)
  .findFirst();

assertEquals("978-0134685991", optionalIsbn.get());
```

我们来分析一下代码。首先，**我们从`Map`** 中获得`entrySet`，正如我们之前看到的。

我们只考虑标题为“有效 Java”的条目，所以第一个中间操作将是一个[过滤器](/web/20220930004311/https://www.baeldung.com/java-stream-filter-lambda)。

**我们对整个`Map`条目不感兴趣，而是对每个条目的关键字感兴趣。**所以下一个链接的中间操作就是这样做的:这是一个`map`操作，它将生成一个新的流作为输出，该流将只包含匹配我们正在寻找的标题的条目的键。

**因为我们只想要一个结果，我们可以应用`findFirst()`** 终端操作，它将在`Stream`中提供初始值作为`Optional`对象。

让我们看一个标题不存在的情况:

```java
Optional<String> optionalIsbn = books.entrySet().stream()
  .filter(e -> "Non Existent Title".equals(e.getValue()))
  .map(Map.Entry::getKey).findFirst();

assertEquals(false, optionalIsbn.isPresent());
```

### 3.3.检索多个结果

现在让我们改变这个问题，看看如何处理返回多个结果而不是一个结果。

为了返回多个结果，让我们将下面的书添加到我们的`Map`:

```java
books.put("978-0321356680", "Effective Java: Second Edition"); 
```

所以现在如果我们寻找以“有效的 Java”开头的`all`书，我们将得到不止一个结果:

```java
List<String> isbnCodes = books.entrySet().stream()
  .filter(e -> e.getValue().startsWith("Effective Java"))
  .map(Map.Entry::getKey)
  .collect(Collectors.toList());

assertTrue(isbnCodes.contains("978-0321356680"));
assertTrue(isbnCodes.contains("978-0134685991"));
```

在这种情况下，我们所做的是替换过滤条件，以验证`Map`中的值是否以“有效 Java”开始，而不是比较`String`是否相等。

这一次**我们`collect`出了结果**，而不是只挑第一名，并把比赛分成了`List`。

## 4.使用`Stream` s 获取`Map`的值

现在让我们来关注地图的另一个问题。**我们将尝试基于`ISBNs.`** 获取`titles `，而不是基于`titles`获取`ISBNs`

还是用原来的`Map`吧。我们希望找到 ISBN 以“978-0”开头的图书。

```java
List<String> titles = books.entrySet().stream()
  .filter(e -> e.getKey().startsWith("978-0"))
  .map(Map.Entry::getValue)
  .collect(Collectors.toList());

assertEquals(2, titles.size());
assertTrue(titles.contains(
  "Design patterns : elements of reusable object-oriented software"));
assertTrue(titles.contains("Effective Java"));
```

这个解决方案类似于我们前面一组问题的解决方案；我们对条目集进行流式处理，然后进行过滤、映射和收集。

同样像以前一样，如果我们想只返回第一个匹配，那么在`map`方法之后，我们可以调用`findFirst()`方法，而不是在`List`中收集所有结果。

## 5.结论

在本文中，我们已经演示了如何以函数方式处理一个`Map``.`

特别是，我们已经看到，一旦我们切换到使用关联的集合到`Map` s，使用`Stream` s 的处理变得更加容易和直观。

当然，本文中的所有例子都可以在 [GitHub 项目](https://web.archive.org/web/20220930004311/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-streams-2)中找到。