# 在 Java 中使用 Math.pow

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-math-pow>

## 1。概述

一个数的幂指的是这个数在乘法中要用多少次。这在 Java 中很容易计算。

## 2。`Math.pow`例子

在看这个例子之前，让我们看一下这个方法的签名:

```java
public double pow(double a, double b)
```

该方法将`a`提升到`b`的幂，并将结果作为`double`返回。换句话说，`a`是自身乘以`b`倍。

现在让我们看一个简单的例子:

```java
int intResult = (int) Math.pow(2, 3);
```

输出将是 8。请注意，如果我们想要一个`Integer`的结果，那么**上面例子中的`int`造型是必需的**。

现在让我们传递一个`double`作为参数，看看结果:

```java
double dblResult = Math.pow(4.2, 3);
```

输出将是 74.0880000000001。

这里我们没有将结果强制转换成一个`int`，因为我们感兴趣的是一个`double`值。由于我们有一个`double`值，我们可以很容易地配置并使用一个`DecimalFormat`将该值四舍五入到两位小数，结果是 74.09:

```java
DecimalFormat df = new DecimalFormat(".00");
double dblResult = Math.pow(4.2, 3);
```

## 3。结论

在这篇简短的文章中，我们看到了如何使用 Java 的 [Math.pow()](https://web.archive.org/web/20221207133541/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/lang/Math.html#pow(double,double)) 方法来计算任意给定基数的幂。

像往常一样，完整的源代码可以在 GitHub 上获得[。](https://web.archive.org/web/20221207133541/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-numbers-2)