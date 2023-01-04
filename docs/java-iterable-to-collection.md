# 在 Java 中将 Iterable 转换为集合

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-iterable-to-collection>

## 1。概述

在本教程中，**我们将探索在 Java** 中将`Iterable`转换成`Collection`的不同方法。

我们将从简单的 Java 解决方案开始，然后看看 Guava 和 Apache Commons 库也提供的选项。

## 2。`Iterable`和`Iterator`

首先，我们将定义我们的`Iterable`:

```java
Iterable<String> iterable = Arrays.asList("john", "tom", "jane");
```

我们还将定义一个简单的`Iterator`——来强调将`Iterable`转换为`Collection`和将`Iterator`转换为`Collection`之间的区别:

```java
Iterator<String> iterator = iterable.iterator();
```

## 3。使用普通 Java

### 3.1。`Iterable`到`Collection`到

我们可以**使用 Java 8 `forEach()`方法将所有元素添加到`List`** :

```java
@Test
public void whenConvertIterableToListUsingJava8_thenSuccess() {
    List<String> result = new ArrayList<String>();
    iterable.forEach(result::add);

    assertThat(result, contains("john", "tom", "jane"));
}
```

或者**使用`Spliterator` 类将我们的`Iterable`转换成`Stream`，然后转换成`Collection`** :

```java
List<String> result = 
  StreamSupport.stream(iterable.spliterator(), false)
    .collect(Collectors.toList());
```

### 3.2。`Iterator`到`Collection`到

另一方面，我们将把`forEachRemaining()`和`Iterator`一起使用，而不是使用`forEach()`:

```java
@Test
public void whenConvertIteratorToListUsingJava8_thenSuccess() {
    List<String> result = new ArrayList<String>();
    iterator.forEachRemaining(result::add);

    assertThat(result, contains("john", "tom", "jane"));
}
```

我们也可以从我们的`Iterator`创建一个`Spliterator`，然后用它将`Iterator`转换成`Stream`:

```java
List<String> result = 
  StreamSupport.stream(Spliterators.spliteratorUnknownSize(iterator, Spliterator.ORDERED), false)
    .collect(Collectors.toList());
```

### 3.3.使用 For 循环

让我们来看看一个解决方案，它使用一个非常简单的 for 循环将我们的`Iterable`转换为`List`:

```java
@Test
public void whenConvertIterableToListUsingJava_thenSuccess() {
    List<String> result = new ArrayList<String>();
    for (String str : iterable) {
        result.add(str);
    }

    assertThat(result, contains("john", "tom", "jane"));
}
```

另一方面，我们将使用`hasNext()`和`next()`以及`Iterator`:

```java
@Test
public void whenConvertIteratorToListUsingJava_thenSuccess() {
    List<String> result = new ArrayList<String>();
    while (iterator.hasNext()) {
        result.add(iterator.next());
    }

    assertThat(result, contains("john", "tom", "jane"));
}
```

## 4。使用番石榴

也有一些可用的库提供了方便的方法来帮助我们实现这一点。

让我们看看如何使用[番石榴](/web/20221126180429/https://www.baeldung.com/guava-collections)从`Iterable`转变为`List`:

**我们可以使用`Lists.newArrayList()` :** 从`Iterable`或`Iterator`创建一个新的`List`

```java
List<String> result = Lists.newArrayList(iterable);
```

或者我们可以使用`ImmutableList.copyOf()`:

```java
List<String> result = ImmutableList.copyOf(iterable);
```

## 5。使用 Apache Commons

最后，**我们将使用 [Apache Commons](/web/20221126180429/https://www.baeldung.com/java-commons-lang-3) `IterableUtils`从`Iterable` :** 创建一个`List`

```java
List<String> result = IterableUtils.toList(iterable);
```

类似地，我们将使用`IteratorUtils`从我们的`Iterator`创建一个`List`:

```java
List<String> result = IteratorUtils.toList(iterator);
```

## 6。结论

在这篇短文中，我们学习了如何使用 Java 将`Iterable`和`Iterator`转换成`Collection`。我们探索了使用普通 Java 和两个外部库的不同方式:Guava 和 Apache Commons。

和往常一样，完整的源代码可以在 [GitHub](https://web.archive.org/web/20221126180429/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-collections-conversions) 上获得。