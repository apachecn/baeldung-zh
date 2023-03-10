# 使用数学。sin 与度

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-math-sin-degrees>

## 1。简介

在这个简短的教程中，我们将看看如何使用 Java 的`Math.sin()`函数计算正弦值，以及如何在角度和弧度之间转换角度值。

## 2。弧度与度数

默认情况下， **Java `Math`库期望其三角函数的值以弧度**为单位。

提醒一下， **`radians `只是表示角度**的另一种方式，换算为:

```java
double inRadians = inDegrees * PI / 180;
inDegrees = inRadians * 180 / PI;
```

Java 通过`toRadians `和`toDegrees`让这变得简单:

```java
double inRadians = Math.toRadians(inDegrees);
double inDegrees = Math.toDegrees(inRadians);
```

每当我们使用 Java 的三角函数时，**我们应该首先考虑我们的输入单位**是什么。

## 3。使用`Math.sin`

我们可以通过查看`Math.s` `in`方法来了解这一原理，这是 Java 提供的众多方法之一:

```java
public static double sin(double a)
```

它相当于数学上的正弦函数，**它期望它的输入以弧度为单位**。假设我们有一个已知的角度，单位是度:

```java
double inDegrees = 30;
```

我们首先需要将其转换为弧度:

```java
double inRadians = Math.toRadians(inDegrees);
```

然后我们可以计算正弦值:

```java
double sine = Math.sin(inRadians);
```

但是，**如果我们知道它已经是弧度，那么我们不需要做转换**:

```java
@Test
public void givenAnAngleInDegrees_whenUsingToRadians_thenResultIsInRadians() {
    double angleInDegrees = 30;
    double sinForDegrees = Math.sin(Math.toRadians(angleInDegrees)); // 0.5

    double thirtyDegreesInRadians = 1/6 * Math.PI;
    double sinForRadians = Math.sin(thirtyDegreesInRadians); // 0.5

    assertTrue(sinForDegrees == sinForRadians);
}
```

由于`thirtyDegreesInRadians `已经是弧度，我们不需要首先转换它来获得相同的结果。

## 4。结论

在这篇简短的文章中，我们回顾了弧度和度数，然后看了一个如何使用`Math.sin.`的例子

和往常一样，在 GitHub 上查看这个例子[的源代码。](https://web.archive.org/web/20221129012602/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-numbers)