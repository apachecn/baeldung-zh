# 在 Java 中计算第 n 个根

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-nth-root>

## 1.概观

试图在 Java 中使用`pow()` 寻找 n 次根在某些情况下是不准确的。原因是双精度数可能会在途中丢失精度。因此，我们可能需要润色结果来处理这些情况。

## 2.问题是

假设我们想计算 N 次方根，如下所示:

```java
base = 125, exponent = 3
```

换句话说，哪个数字的 3 次方是 125？

假设一个数 x 的 n 次方根等于这个数 x 的`1/n` 的幂。因此，我们将等式转换为:

```java
N-th root = Math.pow(125, 1/3)
```

结果是 4.9999999999999999。而 4.9999999999999999 的 3 次方不是 125。那么我们如何解决这个问题呢？

## 3.正确计算 N 次方根

上述问题的解决方案主要是一种数学变通方法，而且非常简单。众所周知**一个数 x 的 n 次方根等于数 x 的`1/n`** 次方。

有几种方法可以利用上面的等式。首先，我们可以使用一个`BigDecimal`并实现我们版本的[牛顿-拉夫森](https://web.archive.org/web/20221128051908/https://en.wikipedia.org/wiki/Newton%27s_method)方法。其次，我们可以将结果四舍五入到最接近的数字，最后，我们可以定义结果可以接受的误差范围。我们将关注最后两种方法。

### 3.1.轮次

我们现在将使用舍入来解决我们的问题。让我们重复使用前面的例子，看看如何获得正确的结果:

```java
public void whenBaseIs125AndNIs3_thenNthIs5() {
    double nth = Math.round(Math.pow(125, 1.0 / 3.0));
    assertEquals(5, nth, 0);
}
```

### 3.2.误差幅度

这种方法与上面的非常相似。我们只需要定义一个可接受的误差范围，假设 0.00001:

```java
public void whenBaseIs625AndNIs4_thenNthIs5() {
    double nth = Math.pow(625, 1.0 / 4.0);
    assertEquals(5, nth, 0.00001);
}
```

测试证明我们的方法正确地计算了 n 次方根。

## 4.结论

作为开发人员，我们必须了解数据类型及其行为。上面描述的数学方法工作得非常好，精确度相当高。您可以选择一个更适合您的用例。上述解决方案的代码可以在 GitHub 上找到[。](https://web.archive.org/web/20221128051908/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-numbers)