# 用 Java 将一个列表复制到另一个列表

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-copy-list-to-another>

## 1.概观

在这个快速教程中，我们将探索将一个`List`复制到另一个`List,` 的不同方法，以及在这个过程中产生的一个常见错误。

关于`Collections`的使用介绍，请参考[这里的](/web/20221209095055/https://www.baeldung.com/java-collections)这篇文章。

## 2.构造器

复制`List`的一个简单方法是使用将集合作为参数的构造函数:

```java
List<Plant> copy = new ArrayList<>(list);
```

因为我们在这里复制引用，而不是克隆对象，所以对一个元素的任何修改都会影响两个列表。

因此，最好使用构造函数来复制不可变对象:

```java
List<Integer> copy = new ArrayList<>(list);
```

`Integer`是不可变的类；它的值是在创建实例时设置的，并且永远不能更改。

因此，一个引用可以被多个列表和线程共享，任何人都无法改变它的值。

## 3.`List` `ConcurrentAccessException`

**使用列表的一个常见问题是`ConcurrentAccessException`。**这通常意味着我们在试图复制列表时修改了它，很可能是在另一个线程中。

要解决这个问题，我们必须:

*   使用为并发访问设计的集合
*   适当地锁定集合以循环访问它
*   找到一种方法来避免需要复制原始集合

考虑到我们的最后一种方法，它不是线程安全的。如果我们想用第一个选项解决我们的问题，我们可能想使用`CopyOnWriteArrayList`，其中所有的变异操作都是通过制作底层数组的新副本来实现的。

欲了解更多信息，请参考本文中的[。](/web/20221209095055/https://www.baeldung.com/java-copy-on-write-arraylist)

如果我们想锁定`Collection`，可以使用锁原语来序列化读/写访问，比如`ReentrantReadWriteLock`。

## 4.`AddAll`

复制元素的另一种方法是使用 `addAll`方法:

```java
List<Integer> copy = new ArrayList<>();
copy.addAll(list);
```

无论何时使用这个方法，记住这一点是很重要的:和构造函数一样，两个列表的内容将引用相同的对象。

## 5.`Collections.copy`

`Collections`类只包含操作集合或返回集合的静态方法。

其中之一是`copy`，它需要一个源列表和一个至少和源一样长的目的列表。

它将维护目标列表中每个复制元素的索引，例如原始元素:

```java
List<Integer> source = Arrays.asList(1,2,3);
List<Integer> dest = Arrays.asList(4,5,6);
Collections.copy(dest, source);
```

在上面的例子中， `dest`列表中所有先前的元素都被覆盖了，因为两个列表具有相同的大小。

如果目标列表比源列表大:

```java
List<Integer> source = Arrays.asList(1, 2, 3);
List<Integer> dest = Arrays.asList(5, 6, 7, 8, 9, 10);
Collections.copy(dest, source);
```

这里，只有前三项被覆盖，而列表中的其余元素被保留。

## 6.使用 Java 8

这个版本的 Java 通过添加新工具扩展了我们的可能性。我们将在下面的例子中探索的是`Stream`:

```java
List<String> copy = list.stream()
  .collect(Collectors.toList());
```

这个选项的主要优点是能够使用跳过和过滤器。在下一个例子中，我们将跳过第一个元素:

```java
List<String> copy = list.stream()
  .skip(1)
  .collect(Collectors.toList());
```

也可以通过`String,`的长度或者通过比较我们对象的属性来过滤:

```java
List<String> copy = list.stream()
  .filter(s -> s.length() > 10)
  .collect(Collectors.toList());
```

```java
List<Flower> flowers = list.stream()
  .filter(f -> f.getPetals() > 6)
  .collect(Collectors.toList());
```

很可能我们希望以一种零安全的方式工作:

```java
List<Flower> flowers = Optional.ofNullable(list)
  .map(List::stream)
  .orElseGet(Stream::empty)
  .collect(Collectors.toList());
```

我们可能也想以这种方式跳过一个元素:

```java
List<Flower> flowers = Optional.ofNullable(list)
  .map(List::stream).orElseGet(Stream::empty)
  .skip(1)
  .collect(Collectors.toList());
```

## 7.使用 Java 10

最后，最后一个 Java 版本允许我们创建一个不可变的`List`，包含给定的`Collection:`的元素

```java
List<T> copy = List.copyOf(list);
```

The only conditions are that the given Collection mustn't be null, or contain any null elements.

## 8.结论

In this article, we learned various ways to copy a `List` to another `List` with different Java versions. We also examined a common error produced in the process.As always, code samples can be found over on GitHub, [here](https://web.archive.org/web/20221209095055/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-collections-list-3) and [here](https://web.archive.org/web/20221209095055/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-10).