# 在 Java 中检查一个数字是正数还是负数

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-check-number-positive-negative>

## 1.概观

在 Java 中，当我们处理像`Integer`、`Long`、`Float`和`Double`这样的类型时，我们经常想要检查数字是正还是负。这是一个基本且常见的数字运算。

在这个快速教程中，我们将讨论如何检查一个给定的数字是正数还是负数。

## 2.问题简介

检查一个数字是正数还是负数是一个非常简单的问题。然而，在我们开始研究实现之前，让我们先了解一下积极和消极的定义。

给定一个[实数](https://web.archive.org/web/20221012124242/https://en.wikipedia.org/wiki/Real_number) `n`，如果`n`大于零，则为正数。否则，如果`n`小于零，则为负。所以，我们还有一个特例:零。 **[零既不是正也不是负](https://web.archive.org/web/20221012124242/https://en.wikipedia.org/wiki/0)** 。

所以，我们可以创建一个 [`enum`](/web/20221012124242/https://www.baeldung.com/a-guide-to-java-enums) 来涵盖这三种可能性:

```java
enum Result {
    POSITIVE, NEGATIVE, ZERO
} 
```

在本教程中，我们将介绍两种不同的方法来检查一个数字是正数、负数还是零。为了简单起见，我们将使用单元测试断言来验证结果。

接下来，让我们看看他们的行动。

## 3.使用'`<`'和'`>`'运算符

根据定义，一个数是正数还是负数取决于与零的比较结果。所以，**我们可以用 Java 的“大于(>)”和“小于(<)”[运算符](/web/20221012124242/https://www.baeldung.com/java-operators)来解决问题**。

接下来，让我们以`Integer`类型为例创建一个方法来进行检查:

```java
static Result byOperator(Integer integer) {
    if (integer > 0) {
        return POSITIVE;
    } else if (integer < 0) {
        return NEGATIVE;
    }
    return ZERO;
} 
```

上面的代码解释得很清楚。根据与零的比较结果，我们确定结果是正、负还是零。

让我们创建一个测试来验证我们的方法:

```java
assertEquals(POSITIVE, PositiveOrNegative.byOperator(42));
assertEquals(ZERO, PositiveOrNegative.byOperator(0));
assertEquals(NEGATIVE, PositiveOrNegative.byOperator(-700));
```

不出所料，如果我们执行它，测试就会通过。

当然，如果我们可以将`Integer`参数更改为`Long`、`Float,`或`Double`，同样的逻辑也适用。

## 4.使用`signum()`方法

我们已经看到了如何使用< and the >操作符检查一个数字是正还是负。或者，我们可以使用`signum()`方法获得给定数字的符号。

**对于`Integer`和`Long`的数字，我们可以调用 [`Integer.signum()`](https://web.archive.org/web/20221012124242/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/lang/Integer.html#signum(int)) 和 [`Long.signum()`](https://web.archive.org/web/20221012124242/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/lang/Long.html#signum(long)) 的方法。**

当`n`为负、零或正时，`signum(n)`方法返回`-1`、`0`和`1`。

让我们以一个`Integer`为例来创建一个检查方法:

```java
static Result bySignum(Integer integer) {
    int result = Integer.signum(integer);
    if (result == 1) {
        return Result.POSITIVE;
    } else if (result == -1) {
        return Result.NEGATIVE;
    }
    return Result.ZERO;
}
```

下面的测试验证了我们的方法的预期效果:

```java
assertEquals(POSITIVE, PositiveOrNegative.bySignum(42));
assertEquals(ZERO, PositiveOrNegative.bySignum(0));
assertEquals(NEGATIVE, PositiveOrNegative.bySignum(-700));
```

与`Integer`和`Long`不同，`Float`和`Double`类不提供`signum()`方法。然而，**[`Math.signum()`](/web/20221012124242/https://www.baeldung.com/java-lang-math#signum)方法接受`Float`和`Double`数字作为参数**，例如:

```java
static Result bySignum(Float floatNumber) {
    float result = Math.signum(floatNumber);

    if (result.compareTo(1.0f) == 0) {
        return Result.POSITIVE;
    } else if (result.compareTo(-1.0f) == 0) {
        return Result.NEGATIVE;
    }
    return Result.ZERO;
}
```

最后，让我们创建一个测试来验证该方法是否可以检查浮点数是正数还是负数:

```java
assertEquals(POSITIVE, PositiveOrNegative.bySignum(4.2f));
assertEquals(ZERO, PositiveOrNegative.bySignum(0f));
assertEquals(NEGATIVE, PositiveOrNegative.bySignum(-7.7f));
```

如果我们试一试，测试就会通过。

## 5.结论

在本文中，我们学习了两种方法来确定一个给定的数字是正数、负数还是零。

像往常一样，这里展示的所有代码片段都可以在 GitHub 上获得。