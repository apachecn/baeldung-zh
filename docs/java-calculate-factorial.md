# 用 Java 计算阶乘

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-calculate-factorial>

## 1.概观

给定一个非负整数`n`，阶乘是所有小于等于`n`的正整数的乘积。

在这个快速教程中，我们将探索用 Java 计算给定数字的阶乘的不同方法。

## 2.20 以内数字的阶乘

### 2.1.使用`for`循环的阶乘

让我们看一个使用`for`循环的基本阶乘算法:

```java
public long factorialUsingForLoop(int n) {
    long fact = 1;
    for (int i = 2; i <= n; i++) {
        fact = fact * i;
    }
    return fact;
}
```

上面的解决方案对 20 个 T2 以内的数字都有效。但是，如果我们尝试大于 20 的东西，那么它会失败，因为**结果会太大而不适合`long`** ，导致溢出。

让我们再看几个，注意每一个都只对少数人有效。

### 2.2.使用 Java 8 流的阶乘

我们也可以使用 [Java 8 `Stream` API](/web/20221126223232/https://www.baeldung.com/java-8-streams-introduction) 非常容易地计算阶乘:

```java
public long factorialUsingStreams(int n) {
    return LongStream.rangeClosed(1, n)
        .reduce(1, (long x, long y) -> x * y);
}
```

在这个程序中，我们首先使用`LongStream`来迭代 1 和`n`之间的数字。然后我们使用了`reduce()`，它为归约步骤使用了一个恒等式和累加器函数。

### 2.3.使用递归的阶乘

让我们看看阶乘程序的另一个例子，这次使用递归:

```java
public long factorialUsingRecursion(int n) {
    if (n <= 2) {
        return n;
    }
    return n * factorialUsingRecursion(n - 1);
}
```

### 2.4.使用 Apache Commons 数学计算阶乘

[Apache Commons Math](/web/20221126223232/https://www.baeldung.com/apache-commons-math) 有一个带静态`factorial`方法的`CombinatoricsUtils`类，我们可以用它来计算阶乘。

为了包含 Apache Commons Math，我们将把[的`commons-math3`依赖关系](https://web.archive.org/web/20221126223232/https://search.maven.org/search?q=a:commons-math3)添加到我们的`pom`中:

```java
<dependency>
    <groupId>org.apache.commons</groupId>
    <artifactId>commons-math3</artifactId>
    <version>3.6.1</version>
</dependency>
```

让我们看一个使用`CombinatoricsUtils `类的例子:

```java
public long factorialUsingApacheCommons(int n) {
    return CombinatoricsUtils.factorial(n);
}
```

注意，它的返回类型是`long`，就像我们自己开发的解决方案一样。

这意味着如果计算出的值超过了`Long.MAX_VALUE`，就会抛出一个`MathArithmeticException`。

为了变得更大，我们需要一个不同的返回类型。

## 3.大于 20 的数字的阶乘

### 3.1.使用`BigInteger`的阶乘

如前所述，`long`数据类型只能用于`n <= 20`的阶乘。

对于更大的`n`、**值，我们可以使用`java.math`包中的`BigInteger`类**，它可以保存**值直到`2^Integer.MAX_VALUE`、**:

```java
public BigInteger factorialHavingLargeResult(int n) {
    BigInteger result = BigInteger.ONE;
    for (int i = 2; i <= n; i++)
        result = result.multiply(BigInteger.valueOf(i));
    return result;
}
```

### 3.2.番石榴析因

Google 的 [Guava](/web/20221126223232/https://www.baeldung.com/category/guava/) 库也提供了一个计算较大数目的阶乘的实用方法。

为了包含这个库，我们可以将它的[`guava `依赖项](https://web.archive.org/web/20221126223232/https://search.maven.org/search?q=guava)添加到我们的`pom`中:

```java
<dependency>
    <groupId>com.google.guava</groupId>
    <artifactId>guava</artifactId>
    <version>31.0.1-jre</version>
</dependency>
```

现在，我们可以使用来自`BigIntegerMath`类的静态`factorial`方法来计算给定数字的阶乘:

```java
public BigInteger factorialUsingGuava(int n) {
    return BigIntegerMath.factorial(n);
}
```

## 4.结论

在本文中，我们看到了使用核心 Java 和一些外部库计算阶乘的几种方法。

我们第一次看到使用`long`数据类型计算数字的阶乘直到`20`的解决方案。然后，我们看到了对大于 20 的数字使用`BigInteger `的几种方法。

本文中的代码可以从 Github 上的[处获得。](https://web.archive.org/web/20221126223232/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-lang-math-2)