# 在 Java 中列出一个范围内的数字

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-listing-numbers-within-a-range>

## 1。概述

在本教程中，我们将探索不同的方法来列出一个范围内的数字序列。

## 2。列出一个范围内的数字

### 2.1。传统`for`循环

我们可以使用传统的`for`循环来生成指定范围内的数字:

```java
public List<Integer> getNumbersInRange(int start, int end) {
    List<Integer> result = new ArrayList<>();
    for (int i = start; i < end; i++) {
        result.add(i);
    }
    return result;
}
```

上面的代码将生成一个包含从`start` (含)到`end` (不含)的数字的列表。

### 2.2。8`IntStream.range`JDK

在 JDK 8 中引入的`IntStream`，可用于生成给定范围内的数字，减少了对`for`循环的需要:

```java
public List<Integer> getNumbersUsingIntStreamRange(int start, int end) {
    return IntStream.range(start, end)
      .boxed()
      .collect(Collectors.toList());
}
```

### 2.3。`IntStream.rangeClosed`

在前一节中，`end`是独占的。为了得到一个范围内的数字，这里有`IntStream.rangeClosed`:

```java
public List<Integer> getNumbersUsingIntStreamRangeClosed(int start, int end) {
    return IntStream.rangeClosed(start, end)
      .boxed()
      .collect(Collectors.toList());
}
```

### 2.4。`IntStream.iterate`

前面几节使用了一个范围来获取一系列数字。当我们知道一个序列中需要多少个数字时，我们可以利用`IntStream.iterate`:

```java
public List<Integer> getNumbersUsingIntStreamIterate(int start, int limit) {
    return IntStream.iterate(start, i -> i + 1)
      .limit(limit)
      .boxed()
      .collect(Collectors.toList());
}
```

这里，`limit`参数限制了要迭代的元素数量。

## 3。结论

在本文中，我们看到了在一个范围内生成数字的不同方法。

代码片段一如既往地可以在 GitHub 上找到[。](https://web.archive.org/web/20220627181314/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-numbers-3)