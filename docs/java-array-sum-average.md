# 在 Java 数组中求和与平均值

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-array-sum-average>

## 1.介绍

在这个快速教程中，我们将介绍如何使用 Java 标准循环和`Stream` API 计算数组中的 sum & average。

## 2.求数组元素的和

### 2.1。使用`For`循环求和

为了找到数组中所有元素的总和，**我们可以简单地迭代数组，并将每个元素添加到一个*总和*累加变量中。**

这非常简单，从 0 的`sum`开始，并随着我们的进行添加数组中的每一项:

```java
public static int findSumWithoutUsingStream(int[] array) {
    int sum = 0;
    for (int value : array) {
        sum += value;
    }
    return sum;
}
```

### 2.2。用 Java 流 API 求和

**我们可以使用流 API 来实现相同的结果:**

```java
public static int findSumUsingStream(int[] array) {
    return Arrays.stream(array).sum();
}
```

重要的是要知道`sum()` 方法只支持[原始类型流](/web/20220628084227/https://www.baeldung.com/java-8-primitive-streams)。

如果我们想在装箱的`Integer`值上使用流，我们必须首先使用`mapToInt` 方法将流转换成`IntStream` 。

之后，我们可以将`sum()` 方法应用于我们新转换的`IntStream`:

```java
public static int findSumUsingStream(Integer[] array) {
    return Arrays.stream(array)
      .mapToInt(Integer::intValue)
      .sum();
}
```

你可以在这里阅读更多关于流 API [的内容。](/web/20220628084227/https://www.baeldung.com/java-8-streams)

## 3.在 Java 数组中查找平均值

### 3.1。没有流 API 的平均值

一旦我们知道如何计算数组元素的总和，求平均值就相当容易了——正如`Average = Sum of Elements / Number of Elements`:

```java
public static double findAverageWithoutUsingStream(int[] array) {
    int sum = findSumWithoutUsingStream(array);
    return (double) sum / array.length;
}
```

`Notes`:

1.  将一个`int`除以另一个`int`返回一个`int`结果。**为了得到准确的平均值，我们首先将`sum`转换为`double`。**
2.  Java `Array`有一个`length`字段，存储数组中元素的数量。

### 3.2。使用 Java 流 API 进行平均

```java
public static double findAverageUsingStream(int[] array) {
    return Arrays.stream(array).average().orElse(Double.NaN);
}
```

`IntStream.average()`返回一个`OptionalDouble`，它可能不包含值，需要特殊处理。

阅读[这篇](/web/20220628084227/https://www.baeldung.com/java-optional)文章中关于`Optionals`的更多内容，以及 [Java 8 文档](https://web.archive.org/web/20220628084227/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/OptionalDouble.html#orElse(double))中关于`OptionalDouble` 类的更多内容。

## 4.结论

在本文中，我们探讨了如何找到`int`数组元素的总和/平均值。

和往常一样，代码可以在 Github 的[上获得。](https://web.archive.org/web/20220628084227/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-arrays-operations-advanced)