# Java 8 forEach 指南

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/foreach-java>

## 1。概述

Java 8 中引入的`forEach`循环为程序员提供了一种新的、简洁而有趣的方法来迭代集合。

在本教程中，我们将看到如何对集合使用`forEach`，它需要什么样的参数，以及这个循环与增强的`for-loop`有何不同。

如果你需要温习一些 Java 8 概念，我们的文章集可以帮到你。

## 延伸阅读:

## 【Collection.stream()的区别。forEach()和 Collection.forEach()

A quick and practical overview of the difference between Collection.stream().forEach() and Collection.forEach().[Read more](/web/20220925220222/https://www.baeldung.com/java-collection-stream-foreach) →

## [如何从 Java 流中断 forEach](/web/20220925220222/https://www.baeldung.com/java-break-stream-foreach)

Java Streams are often a good replacement for loops. Where loops provide the break keyword, we have do something a little different to stop a Stream.[Read more](/web/20220925220222/https://www.baeldung.com/java-break-stream-foreach) →

## [Java 8 Stream API 教程](/web/20220925220222/https://www.baeldung.com/java-8-streams)

The article is an example-heavy introduction of the possibilities and operations offered by the Java 8 Stream API.[Read more](/web/20220925220222/https://www.baeldung.com/java-8-streams) →

## 2。`forEach` 基础知识

在 Java 中，`Collection`接口将`Iterable`作为它的超级接口。从 Java 8 开始，这个接口有了一个新的 API:

```java
void forEach(Consumer<? super T> action)
```

简单地说，`forEach`的 [Javadoc](https://web.archive.org/web/20220925220222/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/lang/Iterable.html#forEach(java.util.function.Consumer)) 声明它“对`Iterable`的每个元素执行给定的动作，直到所有元素都被处理完或者动作抛出异常为止”

因此，使用`forEach`，我们可以迭代一个集合并对每个元素执行给定的操作，就像任何其他的`Iterator`一样。

例如，考虑迭代和打印`Strings`的`Collection`的`for-loop`版本:

```java
for (String name : names) {
    System.out.println(name);
}
```

我们可以用`forEach`来写这个:

```java
names.forEach(name -> {
    System.out.println(name);
});
```

## 3。使用`forEach`方法

我们使用`forEach` 迭代一个集合，并对每个元素执行特定的操作。**要执行的动作包含在实现`Consumer`接口的类中，并作为参数传递给`forEach `。**

`Consumer`接口是[一个函数接口](/web/20220925220222/https://www.baeldung.com/java-8-functional-interfaces)(一个具有单一抽象方法的接口)。它接受输入，不返回任何结果。

定义如下:

```java
@FunctionalInterface
public interface Consumer {
    void accept(T t);
}
```

因此，任何实现，例如，一个简单地打印一个`String`:

```java
Consumer<String> printConsumer = new Consumer<String>() {
    public void accept(String name) {
        System.out.println(name);
    };
};
```

可以作为参数传递给`forEach` :

```java
names.forEach(printConsumer);
```

但这并不是通过消费者创建动作并使用`forEach` API 的唯一方式。

让我们看看使用`forEach`方法的三种最流行的方式。

### 3.1。匿名`Consumer` 实施

我们可以使用匿名类实例化`Consumer`接口的实现，然后将其作为参数应用于`forEach`方法:

```java
Consumer<String> printConsumer= new Consumer<String>() {
    public void accept(String name) {
        System.out.println(name);
    }
};
names.forEach(printConsumer);
```

这个很好用。但是如果我们分析这个例子，我们会发现有用的部分实际上是在`accept()`方法内部的代码。

尽管 Lambda 表达式现在是标准的，也是一种更简单的方法，但了解如何实现`Consumer`接口仍然是值得的。

### 3.2。λ表达式

Java 8 函数接口的主要好处是我们可以使用 Lambda 表达式来实例化它们，避免使用庞大的匿名类实现。

由于 `Consumer`接口是一个函数接口，我们可以用 Lambda 来表示:

```java
(argument) -> { //body }
```

因此，我们的`printConsumer`被简化为:

```java
name -> System.out.println(name)
```

我们可以把它传给`forEach`:

```java
names.forEach(name -> System.out.println(name));
```

自从在 Java 8 中引入 Lambda 表达式以来，这可能是使用`forEach`方法最常见的方式。

Lambdas 确实有一个非常真实的学习曲线，所以如果你刚刚开始，这篇文章将介绍一些使用新语言特性的良好实践。

### 3.3。方法参考

我们可以使用方法引用语法来代替普通的 Lambda 语法，其中已经存在一个方法来对类执行操作:

```java
names.forEach(System.out::println);
```

## 4.使用`forEach`

### 4.1.迭代一个集合

**任何`Collection` — `list`、 `set`、 `queue` 等类型的可重复项。—使用`forEach.`** 的语法相同

因此，如我们所见，我们可以这样迭代列表元素:

```java
List<String> names = Arrays.asList("Larry", "Steve", "James");

names.forEach(System.out::println);
```

还有一套是类似的:

```java
Set<String> uniqueNames = new HashSet<>(Arrays.asList("Larry", "Steve", "James"));

uniqueNames.forEach(System.out::println);
```

最后，我们来看一个`Queue`，它也是一个`Collection`:

```java
Queue<String> namesQueue = new ArrayDeque<>(Arrays.asList("Larry", "Steve", "James"));

namesQueue.forEach(System.out::println);
```

### 4.2.使用地图的`forEach`迭代地图

地图不是`Iterable`，但它们确实**提供了自己的`forEach `变体，接受一个** `**BiConsumer**` **。**

Java 8 引入了一个`BiConsumer`来代替 Iterable 的`forEach`中的`Consumer`，这样就可以同时对`Map`的键和值执行一个动作。

让我们用这些条目创建一个`Map`:

```java
Map<Integer, String> namesMap = new HashMap<>();
namesMap.put(1, "Larry");
namesMap.put(2, "Steve");
namesMap.put(3, "James");
```

接下来，让我们使用 Map 的`forEach`迭代`namesMap`:

```java
namesMap.forEach((key, value) -> System.out.println(key + " " + value));
```

正如我们在这里看到的，我们使用了一个`BiConsumer`来迭代`Map`的条目:

```java
(key, value) -> System.out.println(key + " " + value)
```

### 4.3.通过迭代 `entrySet`来迭代一个`Map`

我们还可以使用 Iterable 的`forEach`来迭代`Map `的`EntrySet `。

由于**的条目被存储在一个叫做`EntrySet,`的`Set`中，我们可以使用一个`forEach`T5 来迭代:**

```java
namesMap.entrySet().forEach(entry -> System.out.println(
  entry.getKey() + " " + entry.getValue()));
```

## 5。Foreach vs For-Loop

从简单的角度来看，这两个循环提供了相同的功能:遍历集合中的元素。

**它们之间的主要区别在于它们是不同的迭代器。增强的`for-loop`是一个外部迭代器，而新的`forEach`方法是内部的。**

### 5.1。内部迭代器— `forEach`

这种类型的迭代器在后台管理迭代，让程序员编写代码来处理集合中的元素。

相反，迭代器管理迭代，并确保逐个处理元素。

让我们看一个内部迭代器的例子:

```java
names.forEach(name -> System.out.println(name));
```

在上面的`forEach` 方法中，我们可以看到提供的参数是一个 lambda 表达式。这意味着该方法只需要知道**要做什么**，所有的迭代工作将由内部处理。

### 5.2。外部迭代器— `for-loop`

外部迭代器混合了**什么**和**如何**完成循环。

`Enumerations`、`Iterators` 和增强的`for-loop`都是外部迭代器(还记得方法`iterator()`、`next()`或者`hasNext()`？).在所有这些迭代器中，我们的工作是指定如何执行迭代。

考虑这个熟悉的循环:

```java
for (String name : names) {
    System.out.println(name);
}
```

尽管我们在遍历列表时没有显式地调用`hasNext()`或`next()` 方法，但是使这种迭代工作的底层代码使用了这些方法。这意味着这些操作的复杂性对程序员来说是隐藏的，但它仍然存在。

与集合自己进行迭代的内部迭代器相反，这里我们需要外部代码从集合中取出每个元素。

## 6。 **结论**

在本文中，我们展示了`forEach`循环比普通的`for-loop`循环更方便。

我们还看到了`forEach`方法是如何工作的，以及为了对集合中的每个元素执行操作，哪种实现可以接收参数。

最后，本文中使用的所有代码片段都可以在我们的 [GitHub](https://web.archive.org/web/20220925220222/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-8) 资源库中找到。