# 用 Java 计算对数

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-logarithms>

## 1.介绍

在这个简短的教程中，我们将学习如何用 Java 计算对数。我们将讨论普通的和自然的对数，以及带有自定义基数的对数。

## 2.对数

对数是一个数学公式，表示我们必须对一个固定数(底数)进行幂运算以产生一个给定的数。

以最简单的形式，它回答了这个问题:一个数乘以多少次才能得到另一个数？

我们可以通过以下等式定义对数:

![{\displaystyle \log _{b}(x)=y\quad }](img/12aa51d5b6be84e76efddaba0c4cda0c.png)exactly if![{\displaystyle \quad b^{y}=x.}](img/564b46b5a7e133b24312d1a671b9cb1c.png)

## 3.计算常用对数

以 10 为底的对数叫做普通对数。

要在 Java 中计算常用对数，我们可以简单地使用`Math.log10()`方法:

```java
@Test
public void givenLog10_shouldReturnValidResults() {
    assertEquals(Math.log10(100), 2);
    assertEquals(Math.log10(1000), 3);
}
```

## 4.计算自然对数

以 e 为底的对数叫做自然对数。

为了在 Java 中计算自然对数，我们使用了`Math.log()`方法:

```java
@Test
public void givenLog10_shouldReturnValidResults() {
    assertEquals(Math.log(Math.E), 1);
    assertEquals(Math.log(10), 2.30258);
}
```

## 5.使用自定义基数计算对数

为了在 Java 中使用自定义底数计算对数，我们使用以下恒等式:

![{\displaystyle \log _{b}x={\frac {\log _{10}x}{\log _{10}b}}={\frac {\log _{e}x}{\log _{e}b}}.\,}](img/9669a66d7b1d7e92355f0df8f257fdc1.png)

```java
@Test
public void givenCustomLog_shouldReturnValidResults() {
    assertEquals(customLog(2, 256), 8);
    assertEquals(customLog(10, 100), 2);
}

private static double customLog(double base, double logNumber) {
    return Math.log(logNumber) / Math.log(base);
}
```

## 6.结论

在本教程中，我们学习了如何用 Java 计算对数。

和往常一样，源代码可以在 GitHub 上的[处获得。](https://web.archive.org/web/20221018155259/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-lang-math)