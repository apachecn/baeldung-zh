# 在 Java 中检查一个数是否是质数

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-prime-numbers>

## 1。简介

首先，让我们复习一些基本理论。

简单地说，如果一个数只能被 1 和它本身整除，那么它就是质数。非质数叫做合数。第一个既不是质数也不是合数。

在本文中，我们将看看在 Java 中检查一个数的素性的不同方法。

## 2。自定义实现

用这种方法，我们可以检查一个介于 2 和(数的平方根)之间的数是否能准确地除数。

如果数字是质数，以下逻辑将返回`true`:

```java
public boolean isPrime(int number) {
    return number > 1 
      && IntStream.rangeClosed(2, (int) Math.sqrt(number))
      .noneMatch(n -> (number % n == 0));
}
```

## 3。使用`BigInteger`

类通常用于存储大整数，即大于 64 位的整数。它提供了一些有用的 API 来处理`int`和`long`值。

其中一个 API 是`isProbablePrime`。如果这个数肯定是一个合数，这个 API 返回`false`,如果它有可能是质数，这个 API 返回`true`。这在处理大整数时很有用，因为完全验证这些整数可能是相当密集的计算。

**快速补充说明**—`isProbablePrime`API 使用所谓的“米勒-拉宾和卢卡斯-莱默”素性测试来检查数字是否可能是质数。在数小于 100 位的情况下，仅使用“米勒-拉宾”检验，否则，两种检验都用于检查数的素性。

```java
public boolean isPrime(int number) {
    BigInteger bigInt = BigInteger.valueOf(number);
    return bigInt.isProbablePrime(100);
}
```

## 4。使用 Apache Commons Math

Apache Commons Math API 提供了一个名为`org.apache.commons.math3.primes.Primes,`的方法，我们将用它来检查一个数的素性。

首先，我们需要通过在我们的`pom.xml`中添加以下依赖项来导入 Apache Commons 数学库:

```java
<dependency>
    <groupId>org.apache.commons</groupId>
    <artifactId>commons-math3</artifactId>
    <version>3.6.1</version>
</dependency>
```

commons-math3 的最新版本可以在[这里](https://web.archive.org/web/20221206160915/https://search.maven.org/classic/#search%7Cga%7C1%7Ca%3A%22commons-math3%22)找到。

我们可以通过调用方法来进行检查:

```java
Primes.isPrime(number);
```

## 5。结论

在这篇快速的文章中，我们看到了三种检验数的素性的方法。

这个代码可以在 Github 上的包`com.baeldung.algorithms.primechecker` [中找到。](https://web.archive.org/web/20221206160915/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-numbers-2)