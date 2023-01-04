# Java 中遍历列表的方法

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-iterate-list>

## 1.介绍

遍历列表中的元素是程序中最常见的任务之一。

在本教程中，我们将回顾在 Java 中实现这一点的不同方法。我们将专注于按顺序遍历列表，尽管逆着遍历[也很简单。](/web/20221208143845/https://www.baeldung.com/java-list-iterate-backwards)

## 延伸阅读:

## [在 Java 中迭代一个集合](/web/20221208143845/https://www.baeldung.com/java-iterate-set)

Learn look at how to iterate over the elements of a `Set` in Java.[Read more](/web/20221208143845/https://www.baeldung.com/java-iterate-set) →

## [在 Java 中迭代地图](/web/20221208143845/https://www.baeldung.com/java-iterate-map)

Learn different ways of iterating through the entries of a Map in Java.[Read more](/web/20221208143845/https://www.baeldung.com/java-iterate-map) →

## [如何迭代带有索引的流](/web/20221208143845/https://www.baeldung.com/java-stream-indices)

Learn several ways of iterating over Java 8 Streams using indices[Read more](/web/20221208143845/https://www.baeldung.com/java-stream-indices) →

## 2.`for`循环

首先，让我们回顾一些 [`for`循环](/web/20221208143845/https://www.baeldung.com/java-loops)选项。

我们将首先为我们的示例定义一个国家列表:

```
List<String> countries = Arrays.asList("Germany", "Panama", "Australia");
```

### 2.1.基本`for`循环

迭代最常见的流控制语句是基本的`for`循环。

`for`循环定义了用分号分隔的三种类型的语句。第一条语句是初始化语句。第二个定义了终止条件。最后一个语句是 update 子句。

这里我们简单地使用一个整数变量作为索引:

```
for (int i = 0; i < countries.size(); i++) {
    System.out.println(countries.get(i));
}
```

在初始化时，我们必须声明一个整数变量来指定起始点。该变量通常充当列表索引。

终止条件是一个计算后返回布尔值的表达式。一旦该表达式的值为 *false，*循环结束。

update 子句用于修改索引变量的当前状态，增加或减少它，直到终止点。

### 2.2.增强型`for`回路

增强的`for`循环是一个简单的结构，允许我们访问列表中的每个元素。它类似于基本的 for 循环，但可读性更强，也更紧凑。因此，它是遍历列表最常用的形式之一。

注意，增强的`for`循环比基本的`for`循环简单:

```
for (String country : countries) {
    System.out.println(country); 
}
```

## 3 .迭代器

一个 [`Iterator`](/web/20221208143845/https://www.baeldung.com/java-iterator) 是一种设计模式，它为我们提供了一个标准接口来遍历数据结构，而不必担心内部表示。

这种遍历数据结构的方式提供了许多优势，其中之一是我们可以强调我们的代码不依赖于实现。

因此，该结构可以是二叉树或双向链表，因为`Iterator`将我们从执行遍历的方式中抽象出来。这样，我们可以轻松地替换代码中的数据结构，而不会出现令人不快的问题。

### 3.1.`Iterator`

在 Java 中，`Iterator`模式反映在`java.util.Iterator`类中。它在 Java `Collections`中被广泛使用。在一个`Iterator`中有两个关键方法，`hasNext()`和`next()`方法。

这里我们将演示两者的用法:

```
Iterator<String> countriesIterator = countries.iterator();

while(countriesIterator.hasNext()) {
    System.out.println(countriesIterator.next()); 
}
```

方法**检查列表**中是否还有剩余的元素。

方法**返回迭代**中的下一个元素。

### 3.2.`ListIterator`

一个`ListIterator`允许我们以向前或向后的顺序遍历一个元素列表。

用`ListIterator`向前滚动列表遵循类似于`Iterator`所使用的机制。这样，我们可以用`next()`方法向前移动迭代器，我们可以用`hasNext()`方法找到列表的末尾。

正如我们所见，`ListIterator`看起来与我们之前使用的`Iterator`非常相似:

```
ListIterator<String> listIterator = countries.listIterator();

while(listIterator.hasNext()) {
    System.out.println(listIterator.next());
}
```

## 4.`forEach()`

### 4.1.`Iterable.forEach()`

**从 Java 8 开始，我们可以使用[`forEach()`方法](/web/20221208143845/https://www.baeldung.com/foreach-java)来迭代链表**的元素。该方法在 [`Iterable`](https://web.archive.org/web/20221208143845/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/lang/Iterable.html) 接口中定义，可以接受 Lambda 表达式作为参数。

语法非常简单:

```
countries.forEach(System.out::println);
```

在`forEach`函数之前，Java 中的所有迭代器都是活动的，这意味着它们包含一个 for 或 while 循环，该循环遍历数据集合，直到满足某个条件。

随着在`Iterable`接口中引入`forEach`作为函数，所有实现`Iterable`的类都增加了`forEach`函数。

### 4.2. `Stream.forEach()`

我们还可以将一组值转换成一个流，并访问诸如`forEach()`、`map(),`和`filter().`之类的操作

这里我们将演示流的一个典型用法:

```
countries.stream().forEach((c) -> System.out.println(c));
```

## 5.结论

在本文中，我们演示了使用 Java API 遍历列表元素的不同方法。这些选项包括`for`循环、增强的`for `循环、`Iterator`、`ListIterator,`和`forEach()`方法(包含在 Java 8 中)。

然后我们学习了如何用`Streams`来使用`forEach()`方法。

最后，本文中使用的所有代码都可以在我们的 [Github repo](https://web.archive.org/web/20221208143845/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-collections-list-2) 中找到。