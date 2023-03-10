# 如何在 Java 8 流中使用 if/else 逻辑

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-8-streams-if-else-logic>

## 1。概述

在本教程中，我们将演示如何用 Java 8 `Streams`实现 if/else 逻辑。作为教程的一部分，我们将创建一个简单的算法来识别奇数和偶数。

我们可以看看[这篇文章](/web/20221208022743/https://www.baeldung.com/java-8-streams-introduction)来了解 Java 8 `Stream`的基础知识。

## 2。`forEach()` 内的常规`if/else`逻辑

首先，让我们创建一个`Integer List` ，然后在`Integer`流`forEach()`方法中使用传统的 if/else 逻辑:

```java
List<Integer> ints = Arrays.asList(1, 2, 3, 4, 5, 6, 7, 8, 9, 10);

ints.stream()
    .forEach(i -> {
        if (i.intValue() % 2 == 0) {
            Assert.assertTrue(i.intValue() % 2 == 0);
        } else {
            Assert.assertTrue(i.intValue() % 2 != 0);
        }
    });
```

**我们的`forEach`方法包含 if-else 逻辑，它使用 Java 模数运算符验证`Integer`是奇数还是偶数**。

## 3。`if/else`逻辑同`filter()`

其次，让我们看一个使用`Stream filter()`方法的更优雅的实现:

```java
Stream<Integer> evenIntegers = ints.stream()
    .filter(i -> i.intValue() % 2 == 0);
Stream<Integer> oddIntegers = ints.stream()
    .filter(i -> i.intValue() % 2 != 0);

evenIntegers.forEach(i -> Assert.assertTrue(i.intValue() % 2 == 0));
oddIntegers.forEach(i -> Assert.assertTrue(i.intValue() % 2 != 0));
```

上面我们使用`Stream filter()`方法**实现了 if/else 逻辑，将`Integer List`分成两个`Stream`，一个用于偶数，另一个用于奇数。**

## 4。结论

在这篇简短的文章中，我们探索了如何创建 Java 8 `Stream`以及如何使用`forEach()`方法实现 if/else 逻辑。

此外，我们学习了如何使用`Stream filter`方法以更优雅的方式获得类似的结果。

最后，本教程中使用的完整源代码可以从 Github 上的[处获得。](https://web.archive.org/web/20221208022743/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-streams-3)