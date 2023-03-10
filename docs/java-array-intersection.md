# 两个整数数组的交集

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-array-intersection>

## 1。概述

在这个快速教程中，我们将看看**如何计算两个整数数组** `‘a'`和 `‘b'`的交集。

我们还将关注如何处理重复条目。

对于实现，我们将使用`Streams.`

## 2。数组的成员谓词

根据定义，两个集合的交集是一个集合，其中所有值都来自一个集合，也是第二个集合的一部分。

因此，我们需要一个`Function`或者更确切地说是一个`Predicate`来决定第二个数组中的成员。由于`List`提供了这样一个现成的方法，我们将把它转换成一个`List`:

```java
Predicate isContainedInB = Arrays.asList(b)::contains; 
```

## 3。建设十字路口

为了构建结果数组，我们将依次考虑第一个集合的元素，并验证它们是否也包含在第二个数组中。然后我们将在此基础上创建一个新的数组。

API 为我们提供了所需的方法。**首先，我们将创建一个`Stream`，然后使用成员资格- `Predicate`进行过滤，最后我们将创建一个新数组:**

```java
public static Integer[] intersectionSimple(Integer[] a, Integer[] b){
    return Stream.of(a)
      .filter(Arrays.asList(b)::contains)
      .toArray(Integer[]::new);
}
```

## 4。重复条目

由于 Java 中的数组没有`Set`实现，我们面临输入和结果中重复条目的问题。请注意，结果中的出现次数取决于第一个参数中的出现次数。

但是对于集合，元素不能出现多次。**我们可以通过使用`distinct()`方法进行归档:**

```java
public static Integer[] intersectionSet(Integer[] a, Integer[] b){
    return Stream.of(a)
      .filter(Arrays.asList(b)::contain)
      .distinct()
      .toArray(Integer[]::new);
}
```

因此交集的长度不再取决于参数顺序。

然而，数组与其自身的交集可能不再是数组，因为我们删除了双重条目。

## 5。多重集交集

一个更普遍的概念是多重集，它允许多个相等的条目。对他们来说，交集是由最小数量的输入事件定义的。因此，我们的 membership- `Predicate`必须记录我们向结果中添加元素的频率。

为此可以使用`remove()`方法，该方法返回成员并使用元素。因此，在消耗完`‘b'`中所有相等的元素后，不会再有相等的元素添加到结果中:

```java
public static Integer[] intersectionSet(Integer[] a, Integer[] b){
    return Stream.of(a)
      .filter(new LinkedList<>(Arrays.asList(b))::remove)
      .toArray(Integer[]::new);
} 
```

由于`Arrays ` API 只返回一个不可变的`List,`,我们必须生成一个专用的可变 API。

## 6.结论

在本文中，我们看到了如何使用`contains`和`remove `方法在 Java 中实现两个数组的交集。

所有的实现、代码片段和测试都可以在我们的 [GitHub 库](https://web.archive.org/web/20221208143830/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-arrays-operations-advanced)中找到——这是一个基于 Maven 的项目，所以它应该很容易导入和运行。