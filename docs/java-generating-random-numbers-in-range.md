# 在 Java 中生成一定范围内的随机数

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-generating-random-numbers-in-range>

## 1。概述

在本教程中，我们将探索在一个范围内产生随机数的不同方法。

## 延伸阅读:

## [在 Java 中生成随机数](/web/20221027120734/https://www.baeldung.com/java-generating-random-numbers)

Learn different ways of generating random numbers in Java.[Read more](/web/20221027120734/https://www.baeldung.com/java-generating-random-numbers) →

## [Java–随机长整型、浮点型、整型和双精度型](/web/20221027120734/https://www.baeldung.com/java-generate-random-long-float-integer-double)

Learn how to generate random numbers in Java - both unbounded as well as within a given interval.[Read more](/web/20221027120734/https://www.baeldung.com/java-generate-random-long-float-integer-double) →

## [Java–生成随机字符串](/web/20221027120734/https://www.baeldung.com/java-random-string)

Generate Bounded and Unbounded Random Strings using plain Java and the Apache Commons Lang library.[Read more](/web/20221027120734/https://www.baeldung.com/java-random-string) →

## 2。生成一个范围内的随机数

### 2.1。`Math.random`

**`Math.random`给出大于等于 0.0 小于 1.0 的随机`double`值。**

让我们使用`Math.random` 方法在给定范围`[min, max)`内生成一个随机数:

```java
public int getRandomNumber(int min, int max) {
    return (int) ((Math.random() * (max - min)) + min);
}
```

为什么会这样？让我们看看当`Math.random`返回 0.0 时会发生什么，这是可能的最低输出:

```java
0.0 * (max - min) + min => min
```

所以，我们能得到的最低数字是`min`。

由于 1.0 是`Math.random`的独占上限，所以我们得到的是:

```java
1.0 * (max - min) + min => max - min + min => max
```

所以我们方法返回的独占上限是`max`。

在下一节中，我们将看到相同的模式在`Random#nextInt`中重复。

### 2.2。`java.util.Random.nextInt`

我们也可以使用`java.util.Random`的一个实例来做同样的事情。

让我们利用`java.util.Random.nextInt` 方法得到一个随机数:

```java
public int getRandomNumberUsingNextInt(int min, int max) {
    Random random = new Random();
    return random.nextInt(max - min) + min;
}
```

`min`参数(原点)是包含性的，而上限`max`是排他性的。

### 2.3。`java.util.Random.ints`

**`java.util.Random.ints` 方法返回一个随机整数的`IntStream` 。**

因此，我们可以利用`java.util.Random.ints`方法并返回一个随机数:

```java
public int getRandomNumberUsingInts(int min, int max) {
    Random random = new Random();
    return random.ints(min, max)
      .findFirst()
      .getAsInt();
}
```

这里同样，指定的原点`min`是包含性的，而`max`是排他性的。

## 3。结论

在本文中，我们看到了在一个范围内生成随机数的替代方法。

代码片段一如既往地可以在 GitHub 上找到[。](https://web.archive.org/web/20221027120734/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-numbers-3)