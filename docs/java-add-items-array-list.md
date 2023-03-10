# 向 Java 数组列表中添加多个项目

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-add-items-array-list>

## 1.`ArrayList`概述

在这个快速教程中，我们将展示如何向一个已经初始化的`ArrayList`添加多个项目。

关于`ArrayList`的使用介绍，请参考[这里的](/web/20221118233956/https://www.baeldung.com/java-arraylist)这篇文章。

## 2.`AddAll`

首先，我们将介绍一种简单的方法来将多个项目添加到一个`ArrayList`中。

首先，我们将使用`addAll()`，它将一个集合作为它的参数:

```java
List<Integer> anotherList = Arrays.asList(5, 12, 9, 3, 15, 88);
list.addAll(anotherList);
```

**重要的是要记住，添加到第一个列表中的元素将引用与`anotherList`中的元素相同的对象。**

因此，对其中一个要素的任何修改都会影响到两个列表。

## 3.`Collections.addAll`

`Collections`类只包含操作集合或返回集合的静态方法。

其中一个是`addAll`，它需要一个目的地列表，要添加的项目可以单独指定，也可以作为一个数组指定。

下面是如何将它用于单个元素的示例:

```java
List<Integer> list = new ArrayList<>();
Collections.addAll(list, 1, 2, 3, 4, 5);
```

另一个示例了两个阵列的操作:

```java
List<Integer> list = new ArrayList<>();
Integer[] otherList = new Integer[] {1, 2, 3, 4, 5};
Collections.addAll(list, otherList);
```

**与上一节所述的方式类似，这里两个列表的内容将指向相同的对象。**

## 4.使用 Java 8

这个版本的 Java 通过添加新工具为我们打开了可能性。我们将在接下来的例子中探索的是`Stream`:

```java
List<Integer> source = ...;
List<Integer> target = ...;

source.stream()
  .forEachOrdered(target::add);
```

这种方式的主要优点是有机会使用跳过和过滤器。在下一个例子中，我们将跳过第一个元素:

```java
source.stream()
  .skip(1)
  .forEachOrdered(target::add);
```

根据我们的需要来过滤元素是可能的。例如，整数值:

```java
source.stream()
  .filter(i -> i > 10)
  .forEachOrdered(target::add);
```

最后，在一些场景中，我们希望以零安全的方式工作。对于那些，我们可以使用`Optional`:

```java
Optional.ofNullable(source).ifPresent(target::addAll)
```

在上面的例子中，我们通过方法`addAll`添加了从`source`到`target`的元素。

## 5.结论

在本文中，我们探索了向已经初始化的`ArrayList`添加多个项目的不同方法。

和往常一样，代码样本可以在 GitHub 上找到[。](https://web.archive.org/web/20221118233956/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-collections-array-list)