# Java 中的原始整数值列表

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-list-primitive-int>

## 1。概述

在本教程中，**我们将学习如何构建一个包含原始整数值**的列表。

我们将探索使用核心 Java 和外部库的解决方案。

## 2。自动装箱

在 Java 中，[泛型类型](/web/20220815130058/https://www.baeldung.com/java-generics)参数必须是引用类型。**这意味着我们不能做类似`List<int>`的事情。**

相反，我们可以使用`List<Integer>`并利用自动装箱。[自动装箱](/web/20220815130058/https://www.baeldung.com/java-wrapper-classes)帮助我们使用`List<Integer>`接口，就好像它包含原始的`int`值一样。引擎盖下依然是`Objects`的集合而不是原语。

核心 Java 解决方案只是一个调整，以便能够将原语与通用的[集合](/web/20220815130058/https://www.baeldung.com/java-collections)一起使用。此外，**它伴随着装箱和拆箱转换的成本。**

然而，在 Java 和其他第三方库中还有其他选项可供我们使用。下面我们来看看如何使用它们。

## 3。使用流 API

通常，我们实际上并不需要创建一个列表，我们只需要对它进行操作。

在这些情况下，使用 Java 8 的[流 API](/web/20220815130058/https://www.baeldung.com/java-8-streams-introduction) 可能会有效，而不是完全创建一个列表。**`IntSream`类提供了一系列支持顺序聚合操作的原始`int`元素。**

让我们快速看一个例子:

```java
IntStream stream = IntStream.of(5, 10, 0, 2, -8);
```

`IntStream.of()` `static`方法返回一个连续的`IntStream`。

类似地，我们可以从现有的`ints`数组中创建一个`IntStream`:

```java
int[] primitives = {5, 10, 0, 2, -8};
IntStream stream = IntStream.of(primitives);
```

此外，我们可以应用标准的流 API 操作来迭代、过滤和聚合`ints`。例如，我们可以计算正`int`值的平均值:

```java
OptionalDouble average = stream.filter(i -> i > 0).average();
```

最重要的是，**在处理流时没有使用自动装箱。**

但是，如果我们确实需要一个具体的列表，我们会想看看下面的第三方库中的一个。

## 4。使用 Trove

Trove 是一个高性能的库，它为 Java 提供了原始集合。

为了用 Maven 设置 Trove，我们需要在我们的`pom.xml`中包含[和`trov4j `依赖关系](https://web.archive.org/web/20220815130058/https://search.maven.org/search?q=net.sf.trove4j%20trove4j):

```java
<dependency>
    <groupId>net.sf.trove4j</groupId>
    <artifactId>trove4j</artifactId>
    <version>3.0.2</version>
</dependency>
```

使用 Trove，**我们可以创建列表、地图和集合。**

例如，有一个接口`TIntList`及其`TIntArrayList`实现来处理一列`int`值:

```java
TIntList tList = new TIntArrayList();
```

即使`TIntList`不能直接实现`List`，但它的方法很有可比性。我们讨论的其他解决方案也遵循类似的模式。

**使用`TIntArrayList`最大的好处就是性能和内存消耗增益**。不需要额外的装箱/拆箱，因为它将数据存储在一个`int[]`数组中。

## 5。使用 Fastutil

另一个处理原语的高性能库是 [Fastutil](https://web.archive.org/web/20220815130058/http://fastutil.di.unimi.it/) 。让我们添加 [`fastutil`依赖](https://web.archive.org/web/20220815130058/https://search.maven.org/search?q=it.unimi.dsi%20fastutil):

```java
<dependency>
    <groupId>it.unimi.dsi</groupId>
    <artifactId>fastutil</artifactId>
    <version>8.1.0</version>
</dependency>
```

现在，我们可以使用它了:

```java
IntArrayList list = new IntArrayList();
```

**默认构造函数`IntArrayList()`在内部创建一个原语数组，默认容量为 16** 。同样，我们可以从现有数组初始化它:

```java
int[] primitives = new int[] {5, 10, 0, 2, -8};
IntArrayList list = new IntArrayList(primitives);
```

## 6。使用 Colt

**[Colt](https://web.archive.org/web/20220815130058/https://dst.lbl.gov/ACSSoftware/colt/api/index.html) 是一个开源的，用于科学技术计算的高性能库**。`cern.colt`包包含可调整大小的列表，这些列表包含原始数据类型，如``int`.`

首先，让我们添加 [`colt`依赖关系](https://web.archive.org/web/20220815130058/https://search.maven.org/search?q=colt%20colt):

```java
<dependency>
    <groupId>colt</groupId>
    <artifactId>colt</artifactId>
    <version>1.2.0</version>
</dependency>
```

提供这个库的原语列表是`cern.colt.list.IntArrayList:`

```java
cern.colt.list.IntArrayList coltList = new cern.colt.list.IntArrayList();
```

默认初始容量为 10。

## 7。使用番石榴

**[Guava](/web/20220815130058/https://www.baeldung.com/whats-new-in-guava-18) 提供了一些原始数组和集合 API**之间的接口方式。`com.google.common.primitives`包有所有的类来适应原始类型。

例如，`ImmutableIntArray`类让我们创建一个不可变的`int`元素列表。

让我们假设，我们有下面的`int`值数组:

```java
int[] primitives = new int[] {5, 10, 0, 2};
```

我们可以简单地用数组创建一个列表:

```java
ImmutableIntArray list = ImmutableIntArray.builder().addAll(primitives).build();
```

此外，它提供了一个列表 API，其中包含了我们所期望的所有标准方法。

## 8。结论

在这篇简短的文章中，**我们展示了用原始整数**创建列表的多种方法。**在我们的例子中，我们使用了 Trove、Fastutil、Colt 和 Guava 库**。

和往常一样，这篇文章的完整代码可以在 GitHub 的[上找到。](https://web.archive.org/web/20220815130058/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-collections-list-3)