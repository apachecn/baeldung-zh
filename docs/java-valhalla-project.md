# Java Valhalla 项目

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-valhalla-project>

## 1。概述

在这篇文章中，我们将看看[项目](https://web.archive.org/web/20221014191848/https://wiki.openjdk.java.net/display/valhalla/Main)[Valhalla](https://web.archive.org/web/20221014191848/https://wiki.openjdk.java.net/display/valhalla/Main)——它的历史原因、当前的开发状态以及它一旦发布会给日常 Java 开发人员带来什么。

## 2.瓦尔哈拉工程的动机和原因

甲骨文公司的 Java 语言架构师 Brian Goetz 在他的一次演讲中说，Valhalla 项目的主要动机之一是希望 T2 能够让 Java 语言和运行时适应现代硬件。当 Java 语言被构思出来的时候(大约 25 年前写作的时候)，**一次内存获取和一次算术运算的成本大致相同。**

如今，这种情况已经发生了变化，内存获取操作比算术运算贵 200 到 1000 倍。从语言设计的角度来看，这意味着导致指针提取的间接寻址会对整体性能产生不利影响。

由于应用程序中的大多数 Java 数据结构都是对象，所以我们可以认为 Java 是一种指针密集型语言(尽管我们通常不会直接看到或操作它们)。这种基于指针的对象实现用于启用对象标识，对象标识本身被用于语言特性，如多态性、可变性和锁定。默认情况下，每个对象都有这些特性，不管它们是否真的需要。

按照指向指针的标识链和指向间接寻址的指针链，间接寻址具有性能缺陷，一个合乎逻辑的结论是删除那些不需要它们的数据结构。这就是值类型发挥作用的地方。

## 3.值类型

值类型的想法是**表示纯数据集合**。这伴随着放弃常规物体的特征。所以，我们有纯数据，没有身份。当然，这意味着我们也失去了可以使用对象标识实现的特性。因此，**平等比较只能发生在国家的基础上。**因此，我们不能使用表示多态，也不能使用不可变或不可空的对象。

因为我们不再有对象标识，所以与对象相比，我们可以放弃指针，改变值类型的一般内存布局。让我们看一下类`Point`和相应的值类型`Point. `之间的内存布局的比较

常规`Point`类的代码和相应的内存布局应该是:

```java
final class Point {
  final int x;
  final int y;
}
```

[![point class memory](img/80b8f574c3720f0705793f21c8f9f631.png)](/web/20221014191848/https://www.baeldung.com/wp-content/uploads/2019/02/point-class-memory.svg)

另一方面，值类型`Point`的代码和相应的内存布局将是:

```java
value class Point {
  int x;
  int y
}
```

[![point value type memory](img/fb1ca5bd136590ea70eb1bc69ef01d94.png)](/web/20221014191848/https://www.baeldung.com/wp-content/uploads/2019/02/point-value-type-memory.svg)

**这允许 JVM 将值类型扁平化为数组和对象，以及其他值类型。**在下图中，我们展示了在数组中使用`Point`类时间接寻址的负面影响:

[![java point vaue type array](img/df61d8969cafd59c3bf81b9edbbf9d71.png)](/web/20221014191848/https://www.baeldung.com/wp-content/uploads/2019/02/java-point-vaue-type-array.svg)

另一方面，这里我们看到了值类型`Point[]`的对应内存结构:

[![java point vaue type array values](img/6dde252f99577fb73f7a375343864aaf.png)](/web/20221014191848/https://www.baeldung.com/wp-content/uploads/2019/02/java-point-vaue-type-array-values.svg)

它还使 JVM 能够在堆栈上传递值类型，而不是在堆上分配值类型。**最终，这意味着我们将获得运行时行为类似于 Java 原语的数据集合，比如`int`或`float`。**

但与基元不同，值类型可以有方法和字段。我们也可以实现接口并将它们作为泛型类型使用。所以我们可以从两个不同的角度来看值类型:

*   更快的物体
*   用户定义的原语

作为额外的锦上添花，我们可以使用值类型作为泛型类型，而无需装箱。这直接把我们带到了 Valhalla 项目的另一个特性:专门化的泛型。

## 4.专用仿制药

当我们想要泛化语言原语时，我们通常使用装箱类型，比如用`Integer`表示`int`，或者用`Float`表示`float`。这种装箱创建了一个额外的间接层，从而违背了使用原语来增强性能的初衷。

因此，我们在现有的框架和库中看到了许多专用于原语类型的专门化，如`IntStream<T>`或`ToIntFunction<T>`。这样做是为了保持使用原语的性能改进。

所以，专门化的泛型是为了消除对这些“黑客”的需求。相反，Java 语言努力为基本上所有东西启用泛型类型:对象引用、原语、值类型，甚至可能是`void`。

## 5.结论

我们已经看到了 Valhalla 项目将给 Java 语言带来的变化。两个主要目标是增强性能和减少抽象泄漏。

性能增强是通过扁平化对象图和移除间接性来解决的。这导致更高效的内存布局和更少的分配和垃圾收集。

当用作泛型类型时，更好的抽象来自具有更相似行为的原语和对象。

项目 Valhalla 的一个早期原型，将值类型引入到现有的类型系统中，代号为 [LW1](https://web.archive.org/web/20221014191848/https://wiki.openjdk.java.net/display/valhalla/LW1) 。

我们可以在相应的项目页面和 JEPs 中找到更多关于 Valhalla 项目的信息:

*   瓦尔哈拉项目
*   [JEP 169:价值对象](https://web.archive.org/web/20221014191848/https://openjdk.java.net/jeps/169)
*   [JEP 218:原始类型上的泛型](https://web.archive.org/web/20221014191848/https://openjdk.java.net/jeps/218)