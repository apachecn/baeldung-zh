# Java 8 和无限流

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-inifinite-streams>

## 1。概述

在本文中，我们将会看到一个`[java.util.Stream](https://web.archive.org/web/20221205130616/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/stream/Stream.html)` API，我们将会看到如何使用这个构造来操作无限的数据/元素流。

处理无限元素序列的可能性是基于这样一个事实，即流是为懒惰而构建的。

这种惰性是通过将可以在流上执行的两种类型的操作分开来实现的:`intermediate`和`terminal`操作。

## 2。中间和终端操作

**所有`Stream`操作分为`intermediate`和`terminal`操作**，并组合成流水线。

流管道由一个源(比如一个`Collection`，一个数组，一个生成器函数，一个 I/O 通道，或者无限序列生成器)组成；随后是零个或多个中间操作和一个终止操作。

### 2.1。`Intermediate`作战

在调用某个`terminal` 操作之前，不会执行`Intermediate`操作。

它们组成了一个`Stream`执行的流水线。可以通过以下方法将`intermediate`操作添加到`Stream`管道中:

*   `filter()`
*   `map()`
*   `flatMap()`
*   `distinct()`
*   `sorted()`
*   `peek()`
*   `limit()`
*   `skip()`

所有的`Intermediate`操作都是懒惰的，所以直到真正需要一个处理结果时才会执行。

基本上，`intermediate`操作返回一个新的流。执行中间操作实际上并不执行任何操作，而是创建一个新的流，该流在被遍历时包含与给定谓词匹配的初始流的元素。

**这样，`Stream`的遍历直到流水线的`terminal`操作执行后才开始。**

这是一个非常重要的属性，对于无限流尤其重要——因为它允许我们创建仅在调用`Terminal`操作时才实际调用的流。

### 2.2.`Terminal`操作

操作可能会遍历流以产生结果或副作用。

在终端操作被执行之后，流管道被认为被消耗，并且不能再被使用。几乎在所有情况下，终端操作都是急切的，在返回之前完成它们对数据源的遍历和对管道的处理。

对于无限流，终端操作的急切性是很重要的，因为在处理的时候**我们需要仔细考虑我们的`Stream`是否被**恰当地限制，例如一个`limit()` 转换。`Terminal`操作有:

*   `forEach()`
*   `forEachOrdered()`
*   `toArray()`
*   `reduce()`
*   `collect()`
*   `min()`
*   `max()`
*   `count()`
*   `anyMatch()`
*   `allMatch()`
*   `noneMatch()`
*   `findFirst()`
*   `findAny()`

这些操作中的每一个都将触发所有中间操作的执行。

## 3。无限流

既然我们理解了这两个概念——`Intermediate`和`Terminal`操作——我们就能够编写一个利用流惰性的无限流。

假设我们想从零开始创建一个无限的元素流，这个元素流的增量是 2。那么我们需要在调用终端操作之前限制这个序列。

**在执行一个终端操作`collect()` 方法**之前使用一个`limit()` 方法是至关重要的，否则我们的程序将无限期运行:

```java
// given
Stream<Integer> infiniteStream = Stream.iterate(0, i -> i + 2);

// when
List<Integer> collect = infiniteStream
  .limit(10)
  .collect(Collectors.toList());

// then
assertEquals(collect, Arrays.asList(0, 2, 4, 6, 8, 10, 12, 14, 16, 18));
```

我们使用`iterate()` 方法创建了一个无限流。然后我们调用了一个`limit()` 转换和一个`collect()` 终端操作。那么在我们得到的`List,` 中，由于`Stream.`的懒惰，我们将有一个无限序列的前 10 个元素

## 4。自定义元素类型的无限流

假设我们想要创建一个无限的随机流`UUIDs`。

使用`Stream` API 实现这一点的第一步是创建这些随机值的`Supplier`:

```java
Supplier<UUID> randomUUIDSupplier = UUID::randomUUID;
```

当我们定义供应商时，我们可以使用`generate()` 方法创建无限流:

```java
Stream<UUID> infiniteStreamOfRandomUUID = Stream.generate(randomUUIDSupplier);
```

然后我们可以从这条溪流中提取一些元素。如果我们希望程序在有限的时间内完成，我们需要记住使用一个`limit()` 方法:

```java
List<UUID> randomInts = infiniteStreamOfRandomUUID
  .skip(10)
  .limit(10)
  .collect(Collectors.toList());
```

我们使用一个`skip()` 转换来丢弃前 10 个结果，并获取接下来的 10 个元素。通过将`Supplier` 接口的函数传递给`Stream`上的`generate()` 方法，我们可以创建任意自定义类型元素的无限流。

## 6。`Do-While`–河道

假设我们有一个简单的 do..在代码中循环时:

```java
int i = 0;
while (i < 10) {
    System.out.println(i);
    i++;
}
```

我们正在打印十次`i` 计数器。我们可以预期，使用`Stream` API 可以很容易地编写这样的构造，理想情况下，我们将在流上有一个`doWhile()` 方法。

不幸的是，在流上没有这样的方法，当我们想要实现类似于标准`do-while`循环的功能时，我们需要使用一个`limit()`方法:

```java
Stream<Integer> integers = Stream
  .iterate(0, i -> i + 1);
integers
  .limit(10)
  .forEach(System.out::println);
```

我们用更少的代码实现了与命令式 while 循环相同的功能，但是对`limit()` 函数的调用不如对`Stream`对象使用`doWhile()`方法那样具有描述性。

## 5。结论

这篇文章解释了我们如何使用`Stream API` 来创建无限的流。当与诸如`limit() –` 这样的转换一起使用时，这些可以使一些场景更容易理解和实现。

支持所有这些例子的代码可以在 [GitHub 项目](https://web.archive.org/web/20221205130616/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-streams)中找到——这是一个 Maven 项目，所以它应该很容易导入和运行。