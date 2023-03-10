# 如何在 For Each 循环中访问迭代计数器

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-foreach-counter>

## 1.概观

在 Java 中迭代数据时，我们可能希望访问当前项及其在数据源中的位置。

这在经典的`for`循环中很容易实现，在这里位置通常是循环计算的焦点，但是当我们使用像 for each 循环或流这样的结构时，这需要更多的工作。

在这个简短的教程中，我们将看看每个操作包含一个计数器的几种方式。

## 2.实现计数器

让我们从一个简单的例子开始。我们将获取一个有序的电影列表，并输出它们的排名。

```java
List<String> IMDB_TOP_MOVIES = Arrays.asList("The Shawshank Redemption",
  "The Godfather", "The Godfather II", "The Dark Knight");
```

### 2.1.`for`循环

一个 [`for`循环](/web/20220703122515/https://www.baeldung.com/java-for-loop)使用一个计数器来引用当前项，所以这是一个操作列表中数据及其索引的简单方法:

```java
List rankings = new ArrayList<>();
for (int i = 0; i < movies.size(); i++) {
    String ranking = (i + 1) + ": " + movies.get(i);
    rankings.add(ranking);
}
```

由于这个`List`可能是一个`ArrayList`，`get`操作是高效的，上面的代码是我们问题的简单解决方案。

```java
assertThat(getRankingsWithForLoop(IMDB_TOP_MOVIES))
  .containsExactly("1: The Shawshank Redemption",
      "2: The Godfather", "3: The Godfather II", "4: The Dark Knight");
```

然而，并不是所有的 Java 数据源都可以用这种方式迭代。有时`get`是一个时间密集型操作，或者我们只能使用`Stream`或`Iterable.`来处理数据源的下一个元素

### 2.2.`for`每个循环

我们将继续使用我们的电影列表，但是让我们假设我们只能使用 Java 的 for each 构造来迭代它:

```java
for (String movie : IMDB_TOP_MOVIES) {
   // use movie value
}
```

这里我们需要使用一个单独的变量来跟踪当前的索引。我们可以构造循环外部，并在内部递增:

```java
int i = 0;
for (String movie : movies) {
    String ranking = (i + 1) + ": " + movie;
    rankings.add(ranking);

    i++;
}
```

我们应该注意到**在循环中使用了**之后，我们必须增加计数器的值。

## 3.各一个功能性`for`

每次我们需要时都编写计数器扩展可能会导致代码重复，并且可能会有关于何时更新计数器变量的意外错误的风险。因此，我们可以使用 Java 的[函数接口](/web/20220703122515/https://www.baeldung.com/java-8-functional-interfaces)来概括上述内容。

首先，我们应该将循环内部的行为视为集合中的项和索引的消费者。这可以使用`BiConsumer`来建模，它定义了一个带两个参数的`accept`函数

```java
@FunctionalInterface
public interface BiConsumer<T, U> {
   void accept(T t, U u);
}
```

由于循环内部使用了两个值，我们可以编写一个通用的循环操作。它可以获取源数据的【the for each 循环将在其上运行),以及对每个项目及其索引执行操作的`BiConsumer`。我们可以用类型参数`T`使其通用化:

```java
static <T> void forEachWithCounter(Iterable<T> source, BiConsumer<Integer, T> consumer) {
    int i = 0;
    for (T item : source) {
        consumer.accept(i, item);
        i++;
    }
}
```

我们可以通过将`BiConsumer`的实现作为 lambda 来使用这个电影排名示例:

```java
List rankings = new ArrayList<>();
forEachWithCounter(movies, (i, movie) -> {
    String ranking = (i + 1) + ": " + movies.get(i);
    rankings.add(ranking);
});
```

## 4.用`Stream`给`forEach`增加一个计数器

[Java `Stream`](/web/20220703122515/https://www.baeldung.com/java-8-streams-introduction) API 允许我们表达我们的数据如何通过过滤器和转换。它还提供了一个`forEach`功能。让我们试着把它转换成一个包含计数器的操作。

`Stream forEach`函数使用一个`Consumer `来处理下一个项目。然而，我们可以创建`Consumer`来跟踪计数器，并将商品传递给`BiConsumer`:

```java
public static <T> Consumer<T> withCounter(BiConsumer<Integer, T> consumer) {
    AtomicInteger counter = new AtomicInteger(0);
    return item -> consumer.accept(counter.getAndIncrement(), item);
}
```

这个函数返回一个新的 lambda。lambda 使用`AtomicInteger`对象在迭代过程中跟踪计数器。每次有新项目时，都会调用`getAndIncrement`函数。

这个函数创建的 lambda 委托给传入的`BiConsumer`,这样算法就可以处理条目及其索引。

让我们来看看我们的电影排名示例与名为`movies`的`Stream`的对比:

```java
List rankings = new ArrayList<>();
movies.forEach(withCounter((i, movie) -> {
    String ranking = (i + 1) + ": " + movie;
    rankings.add(ranking);
}));
```

在`forEach`中有一个对`withCounter`函数的调用，用来创建一个对象，该对象既跟踪计数，又充当`forEach`操作传递其值的`Consumer`。

## 5.结论

在这篇短文中，我们看了三种给 Java `for`每个操作附加计数器的方法。

我们看到了如何在一个循环的每个实现中跟踪当前项的索引。然后，我们研究了如何推广这种模式，以及如何将它添加到流操作中。

一如既往，本文的示例代码可以在 GitHub 上的[处获得。](https://web.archive.org/web/20220703122515/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-lang-3)