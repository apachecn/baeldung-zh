# 将 Double 转换为 String，删除小数位

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-double-to-string>

## 1。简介

在本教程中，我们将看看**将`double`值转换为`String`值的不同方法，去掉它的小数位。**

当我们想要截断小数部分，或者想要四舍五入时，我们会看看如何做。

## 2。使用铸件截断

如果我们的`double`值在`int`范围内，我们可以**将其转换为`int`** `. `该转换会截断小数部分，这意味着它不做任何舍入就将其截断。

**这种方法比我们将要研究的其他方法快 10 倍左右。**

一旦它是一个`int,` **，那么我们就可以将它传递给`String`类**上的`valueOf`方法 **:**

```java
String truncated = String.valueOf((int) doubleValue);
```

当我们保证[double 值在`int`](/web/20220909060320/https://www.baeldung.com/java-primitives) 的范围内时，我们可以放心地使用这种方法。但是**如果我们的价值超过那个，选角就不会像我们想要的那样进行**。

## 3。舍入使用*string . format()*

现在，剩下的方法不像铸造那样受限制，但是它们有自己的细微差别。

例如，另一种方法是使用`String`类的`format`方法。该方法的第一个参数指定我们正在格式化小数点后有零位数的浮点值:

```java
String rounded = String.format("%.0f", doubleValue);
```

**`format` 方法使用`HALF_UP `舍入**，如果小数部分后的值为 0 . 5 或更大，则舍入。否则，它返回小数点前的数字。

虽然简单，`**String.format**` **是完成这个**最慢的方法。

## 4。使用`NumberFormat.format()`

`NumberFormat` 类也提供了一个类似于`String`类的`format`方法，但是 **`NumberFormat `更快，有了它，我们可以指定舍入模式来实现截断或舍入。**

`setMaximumFractionDigits()`方法告诉格式化程序在输出中要包含小数点后多少位小数:

```java
NumberFormat nf = NumberFormat.getNumberInstance();
nf.setMaximumFractionDigits(0);
String rounded = nf.format(doubleValue);
```

**奇怪的是，`NumberFormat`默认不使用`HALF_UP` 。**相反，它默认使用`HALF_EVEN`舍入，这意味着它会像正常情况一样舍入，除了在. 5，**处，在这种情况下，它会选择最接近的偶数**。

**虽然`HALF_EVEN`有助于统计分析，**让我们使用`HALF_UP`来保持一致:

```java
nf.setRoundingMode(RoundingMode.HALF_UP);
String rounded = nf.format(doubleValue);
```

并且，**我们可以改变这一点，通过设置格式化程序使用`FLOOR` 舍入模式来实现截断:**

```java
nf.setRoundingMode(RoundingMode.FLOOR);
String truncated = nf.format(doubleValue)
```

而现在，它会截断而不是倒圆。

## 5。使用`DecimalFormat.format()`

与`NumberFormat`类似，`DecimalFormat`类可以用来格式化`double`值。然而，不是用方法调用来设置输出格式，**我们可以通过给构造函数提供一个特定的模式来告诉格式化程序我们想要什么样的输出:**

```java
DecimalFormat df = new DecimalFormat("#,###");
df.setRoundingMode(RoundingMode.HALF_UP);
String rounded = df.format(doubleValue);
```

“# # # #”模式表示我们希望格式化程序只返回输入的整数部分。它还表示我们希望用逗号将数字分成三组。

同样的舍入默认值也适用于此处，因此如果我们想要输出截断值，我们可以将舍入模式设置为`FLOOR`:

```java
df.setRoundingMode(RoundingMode.FLOOR);
String truncated = df.format(doubleValue)
```

## 6。使用`BigDecimal.toString()`

我们要看的最后一种方法是`BigDecimal`，我们将包括它，因为对于更大的`double` s ，它的表现优于`NumberFormat `和`DecimalFormat`。

我们可以使用`BigDecimal`的`setScale `方法来判断我们是想要舍入还是截断:

```java
double largeDouble = 345_345_345_345.56;
BigDecimal big = new BigDecimal(largeDouble);
big = big.setScale(0, RoundingMode.HALF_UP);
```

**记住`BigDecimal`是不可变的，所以像字符串一样，我们需要重置值。**

然后，我们称`BigDecimal`的`toString`:

```java
String rounded = big.toString();
```

## 7。结论

在本教程中，**我们看了在移除小数位的同时将`double`转换成`String`的不同方法。**我们提供了输出舍入值或截断值的方法。

像往常一样，可以在 GitHub 上的[处获得样本和基准。](https://web.archive.org/web/20220909060320/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-numbers)