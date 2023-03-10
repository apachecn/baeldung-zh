# 在 Java 中初始化数组

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-initialize-array>

## 1。概述

在这个快速教程中，我们将研究初始化数组的不同方法，以及它们之间的细微差别。

## 延伸阅读:

## Java 中的数组:参考指南

A simple and complete reference guide to understanding and using Arrays in Java.[Read more](/web/20221012104208/https://www.baeldung.com/java-arrays-guide) →

## [Java 中的数组操作](/web/20221012104208/https://www.baeldung.com/java-common-array-operations)

Learn how we can handle common array operations in Java.[Read more](/web/20221012104208/https://www.baeldung.com/java-common-array-operations) →

## [一行 Java 列表初始化](/web/20221012104208/https://www.baeldung.com/java-init-list-one-line)

In this quick tutorial, we'll investigate how can we initialize a List using one-liners.[Read more](/web/20221012104208/https://www.baeldung.com/java-init-list-one-line) →

## 2。一次一个元素

让我们从一个简单的基于循环的方法开始:

```java
for (int i = 0; i < array.length; i++) {
    array[i] = i + 2;
}
```

我们还将看到如何一次一个元素地初始化多维数组:

```java
for (int i = 0; i < 2; i++) {
    for (int j = 0; j < 5; j++) {
        array[i][j] = j + 1;
    }
}
```

## 3。申报时

现在让我们在声明时初始化一个数组:

```java
String array[] = new String[] { 
  "Toyota", "Mercedes", "BMW", "Volkswagen", "Skoda" };
```

在实例化数组时，我们不必指定它的类型:

```java
int array[] = { 1, 2, 3, 4, 5 };
```

注意，使用这种方法不可能在声明之后初始化数组；试图这样做将导致编译错误。

## 4。使用`Arrays.fill()`

`java.util.Arrays`类有几个名为`fill(),`的方法，它们接受不同类型的参数并用相同的值填充整个数组:

```java
long array[] = new long[5];
Arrays.fill(array, 30);
```

该方法还有几个替代方法，将数组的范围设置为特定值:

```java
int array[] = new int[5];
Arrays.fill(array, 0, 3, -50);
```

请注意，该方法接受数组、第一个元素的索引、元素的数量和值。

## 5。使用`Arrays.copyOf()`

方法`Arrays.copyOf()`通过复制另一个数组来创建一个新数组。该方法有许多重载，这些重载接受不同类型的参数。

让我们看一个简单的例子:

```java
int array[] = { 1, 2, 3, 4, 5 };
int[] copy = Arrays.copyOf(array, 5);
```

这里有一些注意事项:

*   该方法接受源数组和要创建的副本的长度。
*   如果长度大于要复制的数组的长度，那么多余的元素将使用它们的默认值进行初始化。
*   如果源数组没有初始化，那么抛出一个`NullPointerException`。
*   最后，如果源数组长度是负的，那么抛出一个`NegativeArraySizeException`。

## 6。使用`Arrays.setAll()`

方法`Arrays.setAll()`使用生成器函数设置数组的所有元素:

```java
int[] array = new int[20];
Arrays.setAll(array, p -> p > 9 ? 0 : p);

// [0, 1, 2, 3, 4, 5, 6, 7, 8, 9, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0]
```

如果生成器函数为空，那么抛出一个`NullPointerException`。

## 7。使用`ArrayUtils.clone()`

最后，让我们利用 Apache Commons Lang 3 的`ArrayUtils.clone()` API，它通过创建另一个数组的直接副本来初始化一个数组:

```java
char[] array = new char[] {'a', 'b', 'c'};
char[] copy = ArrayUtils.clone(array);
```

请注意，此方法对于所有基元类型都是重载的。

## 8。结论

在这篇简短的文章中，我们探索了在 Java 中初始化数组的不同方法。

和往常一样，GitHub 上有完整版本的代码[。](https://web.archive.org/web/20221012104208/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-arrays-operations-basic)