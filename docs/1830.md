# 检查 BigDecimal 值是否为零

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-bigdecimal-zero>

## 1.概观

当我们想用 Java 做十进制数计算时，可以考虑使用`[BigDecimal](/web/20221208143919/https://www.baeldung.com/java-bigdecimal-biginteger)`类。

在这个简短的教程中，我们将探索如何检查一个`BigDecimal`对象的值是否为零。

## 2.问题简介

这个问题其实很简单。假设我们有一个非空的`BigDecimal`对象。我们想知道它的值是否等于零。

眼尖的可能已经意识到，要求“`whether` `its value is equal to zero`”已经隐含了解决方案:使用 [`equals()`](/web/20221208143919/https://www.baeldung.com/java-equals-method-operator-difference) 的方法。此外，`BigDecimal`类提供了一个方便的`ZERO`常量对象来指示零值。

的确，这个问题听起来很简单。我们可以简单地检查`BigDecimal.ZERO.equals(givenBdNumber)`来决定`givenBdNumber`对象的值是否为零。然而，**如果我们不知道`BigDeicmal`的比较错综复杂，我们可能会陷入一个常见的陷阱**。

接下来，让我们仔细看看它，并提出解决它的正确方法。

## 3.`BigDecimal`比较的常见陷阱:使用`equals`方法

首先，让我们创建一个值为零的`BigDecimal`对象:

```
BigDecimal BD1 = new BigDecimal("0");
```

现在，让我们使用`equals`方法检查`BD1`的值是否为零。为了简单起见，让我们用一种单元测试方法来做这件事:

```
assertThat(BigDecimal.ZERO.equals(BD1)).isTrue();
```

如果我们进行测试，它会通过的。到目前为止，一切顺利。我们可能认为这是解决方案。接下来，让我们创建另一个`BigDecimal`对象:

```
BigDecimal BD2 = new BigDecimal("0.0000");
```

显然，`BD2`对象的值是零，尽管我们已经用一个四进制的字符串构造了它。众所周知，0.0000 和 0 在价值上是一样的。

现在，让我们再次用`equals`方法测试 BD2:

```
assertThat(BigDecimal.ZERO.equals(BD2)).isTrue();
```

这一次，如果我们运行方法，**令人惊讶的是，测试将失败**。

这是因为`BigDecimal`对象具有值和比例属性。此外，**`equals`方法认为只有当两个`BigDecimal`对象的值和比例**相等时，它们才相等。也就是说，`BigDecimal` 42 与`equals`相比，不等于 42.0。

另一方面，`BigDecimal.ZERO`常量的值和刻度也是零。所以，当我们检查"`0 equals 0.0000`"时，`equals`方法返回`false`。

因此，我们需要找到一种方法，只比较两个`BigDecimal`对象的值，而忽略它们的比例。

接下来，让我们看看解决这个问题的几种方法。

## 4.使用`compareTo`方法

`BigDecimal`类实现了`[Comparable](/web/20221208143919/https://www.baeldung.com/java-comparator-comparable)`接口。因此，我们可以使用`compareTo`方法来比较两个`BigDecimal`对象的值。

此外，`compareTo`方法的 Javadoc 明确声明:

> 值相等但比例不同的两个`BigDecimal`对象(如 2.0 和 2.00)被该方法视为相等。

因此，**我们可以检查`BigDecimal.ZERO.compareTo(givenBdNumber) == 0`** 来决定 givenBdNumber 的值是否为零。

接下来，让我们测试这个方法是否能正确地判断出`BigDecimal`对象`BD1`和`BD2`是否都为零:

```
assertThat(BigDecimal.ZERO.compareTo(BD1)).isSameAs(0);
assertThat(BigDecimal.ZERO.compareTo(BD2)).isSameAs(0);
```

当我们运行测试时，它通过了。因此，我们已经使用`compareTo`方法解决了这个问题。

## 5.使用`signum`方法

**`BigDeicmal`类提供了`[signum](https://web.archive.org/web/20221208143919/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/math/BigDecimal.html#signum())`方法来判断给定的`BigDecimal`对象的值是负(-1)、零(0)还是正(1)** 。`signum`方法将忽略 scale 属性。

所以我们可以通过查`(givenBdNumber.signum() == 0)`来解决问题。

同样，让我们编写一个测试来验证这种方法是否适用于这两个示例:

```
assertThat(BD1.signum()).isSameAs(0);
assertThat(BD2.signum()).isSameAs(0);
```

如果我们试一试，上面的测试就通过了。

## 6.结论

在这篇短文中，我们提出了两种检查`BigDecimal`对象的值是否为零的正确方法:`compareTo`方法或`signum`方法。

像往常一样，这篇文章的完整代码可以在 GitHub 上找到[。](https://web.archive.org/web/20221208143919/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-numbers-4)