# 将迭代器转换为列表

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-convert-iterator-to-list>

## 1.概观

在这个简短的教程中，我们将学习如何在 Java 中将一个`[Iterator](/web/20221126225628/https://www.baeldung.com/java-iterator)` 转换成一个`[List](/web/20221126225628/https://www.baeldung.com/java-init-list-one-line)` 。我们将讨论一些使用 while 循环、Java 8 和一些公共库的例子。

我们将在所有示例中使用带`Integer` s 的`Iterator`:

```java
Iterator<Integer> iterator = Arrays.asList(1, 2, 3).iterator(); 
```

## 2.使用 While 循环

让我们从 Java 8 之前传统使用的方法开始。我们将用**`while`循环**将`Iterator`转换为`List`:

```java
List<Integer> actualList = new ArrayList<Integer>();
while (iterator.hasNext()) {
    actualList.add(iterator.next());
}

assertThat(actualList, containsInAnyOrder(1, 2, 3)); 
```

## 3.使用 Java 8 `Iterator.forEachRemaining`

在 Java 8 和更高版本中，我们可以使用`Iterator`的`forEachRemaining()`方法来构建我们的`List`。我们将通过`List`接口的`add()`方法作为[方法引用](/web/20221126225628/https://www.baeldung.com/java-method-references):

```java
List<Integer> actualList = new ArrayList<Integer>();
iterator.forEachRemaining(actualList::add);

assertThat(actualList, containsInAnyOrder(1, 2, 3)); 
```

## 4.使用 Java 8 流 API

接下来，我们将使用 [Java 8 Streams API](/web/20221126225628/https://www.baeldung.com/java-8-streams) 将`Iterator`转换为`List`。为了使用`Stream` API，我们需要**首先将`Iterator`转换为`[Iterable](/web/20221126225628/https://www.baeldung.com/java-iterable-to-stream)`** 。我们可以使用 Java 8 Lambda 表达式做到这一点:

```java
Iterable<Integer> iterable = () -> iterator; 
```

现在，**可以使用`StreamSupport` class' `stream()`和`collect()`的方法来构建`List`** :

```java
List<Integer> actualList = StreamSupport
  .stream(iterable.spliterator(), false)
  .collect(Collectors.toList());

assertThat(actualList, containsInAnyOrder(1, 2, 3));
```

## 5.用番石榴

Google 的 [**番石榴库**](/web/20221126225628/https://www.baeldung.com/guava-lists) **提供了创建可变和不可变`List`s`,`**的选项，因此我们将看到这两种方法。

让我们首先使用`ImmutableList.copyOf()`方法创建一个不可变的`List`:

```java
List<Integer> actualList = ImmutableList.copyOf(iterator);

assertThat(actualList, containsInAnyOrder(1, 2, 3));
```

现在，让我们使用`Lists.newArrayList()`方法创建一个可变的`List`:

```java
List<Integer> actualList = Lists.newArrayList(iterator);

assertThat(actualList, containsInAnyOrder(1, 2, 3));
```

## 6.使用 Apache Commons

**[Apache Commons Collections](/web/20221126225628/https://www.baeldung.com/apache-commons-collection-utils)库提供了处理`List. `** 的选项，我们将使用`IteratorUtils`进行转换:

```java
List<Integer> actualList = IteratorUtils.toList(iterator);

assertThat(actualList, containsInAnyOrder(1, 2, 3));
```

## 7.结论

在本文中，我们介绍了几种将`Iterator`转换成`List`的方法。虽然还有其他一些方法可以实现这一点，但我们已经介绍了几种常用的方法。

所有这些例子和代码片段的实现都可以在 GitHub 上找到[。](https://web.archive.org/web/20221126225628/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-collections-conversions)