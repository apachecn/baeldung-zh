# Java 8 收集器 toMap

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-collectors-tomap>

## 1.概观

在这个快速教程中，我们将讨论`Collectors`类的`toMap()`方法。我们将用它将`Stream`收集到一个`Map`实例中。

**对于这里涵盖的所有示例，我们将使用一个图书列表作为起点，并将其转换为不同的`Map`实现。**

## 延伸阅读:

## [Java 8 的收集器指南](/web/20220930004318/https://www.baeldung.com/java-8-collectors)

The article discusses Java 8 Collectors, showing examples of built-in collectors, as well as showing how to build custom collector.[Read more](/web/20220930004318/https://www.baeldung.com/java-8-collectors) →

## [将 Java 流收集到一个不可变的集合中](/web/20220930004318/https://www.baeldung.com/java-stream-immutable-collection)

Learn how to collect Java Streams to immutable Collections.[Read more](/web/20220930004318/https://www.baeldung.com/java-stream-immutable-collection) →

## [Java 9 中的新流收集器](/web/20220930004318/https://www.baeldung.com/java9-stream-collectors)

In this article, we explore new Stream collectors that were introduced in JDK 9[Read more](/web/20220930004318/https://www.baeldung.com/java9-stream-collectors) →

## 2.`List`至`Map`

我们将从最简单的情况开始，通过将一个`List`转换成一个`Map`。

下面是我们如何定义我们的`Book`类:

```java
class Book {
    private String name;
    private int releaseYear;
    private String isbn;

    // getters and setters
}
```

我们将创建一个图书列表来验证我们的代码:

```java
List<Book> bookList = new ArrayList<>();
bookList.add(new Book("The Fellowship of the Ring", 1954, "0395489318"));
bookList.add(new Book("The Two Towers", 1954, "0345339711"));
bookList.add(new Book("The Return of the King", 1955, "0618129111"));
```

对于这个场景，我们将使用下面的`toMap()`方法重载:

```java
Collector<T, ?, Map<K,U>> toMap(Function<? super T, ? extends K> keyMapper,
  Function<? super T, ? extends U> valueMapper)
```

**使用`toMap`，我们可以指示如何为地图**获取键和值的策略:

```java
public Map<String, String> listToMap(List<Book> books) {
    return books.stream().collect(Collectors.toMap(Book::getIsbn, Book::getName));
}
```

我们可以很容易地验证它的工作原理:

```java
@Test
public void whenConvertFromListToMap() {
    assertTrue(convertToMap.listToMap(bookList).size() == 3);
}
```

## 3.解决关键冲突

上面的例子运行得很好，但是如果有一个重复的密钥会怎么样呢？

让我们想象一下，我们用每个`Book`的发布年份作为`Map`的关键字:

```java
public Map<Integer, Book> listToMapWithDupKeyError(List<Book> books) {
    return books.stream().collect(
      Collectors.toMap(Book::getReleaseYear, Function.identity()));
}
```

根据我们之前的书单，我们会看到一个`IllegalStateException`:

```java
@Test(expected = IllegalStateException.class)
public void whenMapHasDuplicateKey_without_merge_function_then_runtime_exception() {
    convertToMap.listToMapWithDupKeyError(bookList);
}
```

为了解决这个问题，我们需要使用一个不同的方法和一个额外的参数`mergeFunction`:

```java
Collector<T, ?, M> toMap(Function<? super T, ? extends K> keyMapper,
  Function<? super T, ? extends U> valueMapper,
  BinaryOperator<U> mergeFunction) 
```

让我们引入一个合并函数，它表示在发生冲突的情况下，我们保留现有的条目:

```java
public Map<Integer, Book> listToMapWithDupKey(List<Book> books) {
    return books.stream().collect(Collectors.toMap(Book::getReleaseYear, Function.identity(),
      (existing, replacement) -> existing));
}
```

换句话说，我们得到了先赢行为:

```java
@Test
public void whenMapHasDuplicateKeyThenMergeFunctionHandlesCollision() {
    Map<Integer, Book> booksByYear = convertToMap.listToMapWithDupKey(bookList);
    assertEquals(2, booksByYear.size());
    assertEquals("0395489318", booksByYear.get(1954).getIsbn());
}
```

## 4.其他地图类型

默认情况下，`toMap()`方法将返回一个`HashMap`。

**但是我们可以返回不同的`Map`实现**:

```java
Collector<T, ?, M> toMap(Function<? super T, ? extends K> keyMapper,
  Function<? super T, ? extends U> valueMapper,
  BinaryOperator<U> mergeFunction,
  Supplier<M> mapSupplier)
```

这里的`mapSupplier`是一个函数，它返回一个新的空的`Map`和结果。

### 4.1.`List`至`ConcurrentMap`

让我们举同一个例子，添加一个`mapSupplier`函数来返回一个`ConcurrentHashMap`:

```java
public Map<Integer, Book> listToConcurrentMap(List<Book> books) {
    return books.stream().collect(Collectors.toMap(Book::getReleaseYear, Function.identity(),
      (o1, o2) -> o1, ConcurrentHashMap::new));
}
```

我们将继续测试我们的代码:

```java
@Test
public void whenCreateConcurrentHashMap() {
    assertTrue(convertToMap.listToConcurrentMap(bookList) instanceof ConcurrentHashMap);
}
```

### 4.2.已排序`Map`

最后，让我们看看如何返回一个排序的地图。为此，我们将使用一个 [`TreeMap`](/web/20220930004318/https://www.baeldung.com/java-treemap) 作为`mapSupplier`参数。

因为默认情况下,`TreeMap`是根据其键的自然顺序进行排序的，所以我们不必自己显式地对`books`进行排序:

```java
public TreeMap<String, Book> listToSortedMap(List<Book> books) {
    return books.stream() 
      .collect(
        Collectors.toMap(Book::getName, Function.identity(), (o1, o2) -> o1, TreeMap::new));
}
```

所以在我们的例子中，返回的`TreeMap`将按照书名的字母顺序排序:

```java
@Test
public void whenMapisSorted() {
    assertTrue(convertToMap.listToSortedMap(bookList).firstKey().equals(
      "The Fellowship of the Ring"));
}
```

## 5.结论

在本文中，我们研究了`Collectors`类的`toMap() `方法。它允许我们从一个`Stream`创建一个新的`Map`。

我们还学习了如何解决关键冲突和创建不同的地图实现。

和往常一样，代码可以在 GitHub 的[上获得。](https://web.archive.org/web/20220930004318/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-collections-conversions)