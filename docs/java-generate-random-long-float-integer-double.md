# Java–随机长整型、浮点型、整数型和双精度型

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-generate-random-long-float-integer-double>

这个快速教程将演示如何使用普通 Java 和 Apache Commons 数学库生成一个 long first。

本文是 Baeldung 网站上的“Java 回归基础”系列文章的一部分。

## 1。生成一个无界 Long

让我们从生成一个长:

```java
@Test
public void givenUsingPlainJava_whenGeneratingRandomLongUnbounded_thenCorrect() {
    long generatedLong = new Random().nextLong();
}
```

## 2。生成一个范围内的 Long】

### 2.1。带普通 Java 的随机 Long】

接下来，让我们看看如何创建一个随机有界长整型值，即在给定范围或区间内的长整型值:

```java
@Test
public void givenUsingPlainJava_whenGeneratingRandomLongBounded_thenCorrect() {
    long leftLimit = 1L;
    long rightLimit = 10L;
    long generatedLong = leftLimit + (long) (Math.random() * (rightLimit - leftLimit));
}
```

### 2.2。带 Apache Commons Math 的 Random Long】

让我们来看看用一个更简洁的 API 和 Commons Math 来生成 random Long:

```java
@Test
public void givenUsingApacheCommons_whenGeneratingRandomLongBounded_thenCorrect() {
    long leftLimit = 10L;
    long rightLimit = 100L;
    long generatedLong = new RandomDataGenerator().nextLong(leftLimit, rightLimit);
}
```

## 3。生成一个无界整数

让我们继续生成一个没有边界的随机整数:

```java
@Test
public void givenUsingPlainJava_whenGeneratingRandomIntegerUnbounded_thenCorrect() {
    int generatedInteger = new Random().nextInt();
}
```

如你所见，它非常接近于产生一个长的。

## 4。生成一个范围内的整数

### 4.1。带普通 Java 的随机整数

下一个–给定范围内的随机整数:

```java
@Test
public void givenUsingPlainJava_whenGeneratingRandomIntegerBounded_thenCorrect() {
    int leftLimit = 1;
    int rightLimit = 10;
    int generatedInteger = leftLimit + (int) (new Random().nextFloat() * (rightLimit - leftLimit));
}
```

### 4.2。具有通用数学的随机整数

普通数学也是如此:

```java
@Test
public void givenUsingApache_whenGeneratingRandomIntegerBounded_thenCorrect() {
    int leftLimit = 1;
    int rightLimit = 10;
    int generatedInteger = new RandomDataGenerator().nextInt(leftLimit, rightLimit);
}
```

## 5。生成一个无界浮点数

现在，让我们回顾一下生成随机浮点数——首先是无界的:

```java
@Test
public void givenUsingPlainJava_whenGeneratingRandomFloatUnbouned_thenCorrect() {
    float generatedFloat = new Random().nextFloat();
}
```

## 6。在范围内生成浮点

### 6.1。用普通 Java 实现随机浮点运算

和一个有界随机浮点数:

```java
@Test
public void givenUsingPlainJava_whenGeneratingRandomFloatBouned_thenCorrect() {
    float leftLimit = 1F;
    float rightLimit = 10F;
    float generatedFloat = leftLimit + new Random().nextFloat() * (rightLimit - leftLimit);
}
```

### 6.2。使用 Commons Math 的随机浮点运算

现在——一个有限的随机浮动，带有公共数学:

```java
@Test
public void givenUsingApache_whenGeneratingRandomFloatBounded_thenCorrect() {
    float leftLimit = 1F;
    float rightLimit = 10F;
    float randomFloat = new RandomDataGenerator().getRandomGenerator().nextFloat();
    float generatedFloat = leftLimit + randomFloat * (rightLimit - leftLimit);
}
```

## 7。生成一个无界的 Double

### 7.1。带普通 Java 的随机无界 Double

最后，我们将生成随机双精度值，首先使用 Java Math API:

```java
@Test
public void givenUsingPlainJava_whenGeneratingRandomDoubleUnbounded_thenCorrect() {
    double generatedDouble = Math.random();
}
```

### 7.2。随机无界双精度公地数学

以及 Apache Commons 数学库的随机双精度值:

```java
@Test
public void givenUsingApache_whenGeneratingRandomDoubleUnbounded_thenCorrect() {
    double generatedDouble = new RandomDataGenerator().getRandomGenerator().nextDouble();
}
```

## 8。生成一个范围内的 Double 值

### 8.1。带普通 Java 的随机有界 Double

在这个例子中，让我们看看在一个时间间隔内用 Java 生成的随机双精度数:

```java
@Test
public void givenUsingPlainJava_whenGeneratingRandomDoubleBounded_thenCorrect() {
    double leftLimit = 1D;
    double rightLimit = 10D;
    double generatedDouble = leftLimit + new Random().nextDouble() * (rightLimit - leftLimit);
}
```

### 8.2。随机有界双带公地数学

最后——使用 Apache Commons 数学库，在一个区间内随机双精度:

```java
@Test
public void givenUsingApache_whenGeneratingRandomDoubleBounded_thenCorrect() {
    double leftLimit = 1D;
    double rightLimit = 100D;
    double generatedDouble = new RandomDataGenerator().nextUniform(leftLimit, rightLimit);
}
```

现在你已经知道了——如何为 Java 中最常见的数字原语生成无界值和有界值的简明例子。

## 9.结论

本教程演示了如何使用不同的技术和库来生成绑定或未绑定的随机数。

和往常一样，所有这些例子和片段的实现都可以在 GitHub 项目中找到。这是一个基于 Maven 的项目，因此应该很容易导入和运行。