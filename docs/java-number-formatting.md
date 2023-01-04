# Java 中的数字格式

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-number-formatting>

## 1.概观

在本教程中，我们将研究 Java 中数字格式化的不同方法以及如何实现它们。

## 2.用`String#format`进行基本数字格式化

[`String#format`](https://web.archive.org/web/20220923221946/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/lang/String.html#format(java.util.Locale,java.lang.String,java.lang.Object...)) 方法对于格式化数字非常有用。该方法有两个参数。第一个参数描述了我们想要查看的小数位数的模式，第二个参数是给定值:

```java
double value = 4.2352989244d;
assertThat(String.format("%.2f", value)).isEqualTo("4.24");
assertThat(String.format("%.3f", value)).isEqualTo("4.235");
```

## 3.通过舍入设置小数格式

在 Java 中，我们有两种代表十进制数的基本类型**–`float `和`decimal`:**

```java
double myDouble = 7.8723d;
float myFloat = 7.8723f;
```

根据所执行的操作，小数位数可能会有所不同。在大多数情况下，[我们只对前几个小数位](/web/20220923221946/https://www.baeldung.com/java-round-decimal-number)感兴趣。让我们来看一些通过舍入来格式化小数的方法。

### 3.1.使用`BigDecimal`进行数字格式化

[`BigDecimal`](https://web.archive.org/web/20220923221946/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/math/BigDecimal.html) 类提供了四舍五入到指定小数位数的方法。让我们创建一个 helper 方法，该方法将返回一个 double 值，舍入到所需的位数:

```java
public static double withBigDecimal(double value, int places) {
    BigDecimal bigDecimal = new BigDecimal(value);
    bigDecimal = bigDecimal.setScale(places, RoundingMode.HALF_UP);
    return bigDecimal.doubleValue();
}
```

我们用原始的十进制值开始一个新的`BigDecimal`实例。然后，**通过设置刻度，我们提供我们想要的小数位数以及我们想要如何舍入我们的数字**。使用这种方法，我们可以轻松地格式化一个`double`值:

```java
double D = 4.2352989244d;
assertThat(withBigDecimal(D, 2)).isEqualTo(4.24);
assertThat(withBigDecimal(D, 3)).isEqualTo(4.235);
```

### 3.2.使用`**Math#round**`

我们还可以利用 [`Math`类](https://web.archive.org/web/20220923221946/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/lang/Math.html)中的静态方法将`double`值四舍五入到指定的小数位。在这种情况下，我们可以通过**乘以`10^n`再除以**来调整小数位数。让我们检查一下我们的 helper 方法:

```java
public static double withMathRound(double value, int places) {
    double scale = Math.pow(10, places);
    return Math.round(value * scale) / scale;
}
```

```java
assertThat(withMathRound(D, 2)).isEqualTo(4.24);
assertThat(withMathRound(D, 3)).isEqualTo(4.235);
```

然而，这种方法仅在特殊情况下被推荐，因为有时输出在打印前可能会与预期的舍入不同。

这是因为 **`Math#round`正在截断值**。让我们看看这是如何发生的:

```java
System.out.println(withMathRound(1000.0d, 17));
// Gives: 92.23372036854776 !!
System.out.println(withMathRound(260.775d, 2));
// Gives: 260.77 instead of expected 260.78 
```

因此，所列出的方法仅用于学习目的。

## 4.格式化不同类型的数字

在某些特殊情况下，我们可能希望将数字格式化为特定的类型，如货币、大整数或百分比。

### 4.1.用逗号格式化大整数

每当我们的应用程序中有一个大整数时，我们可能希望用逗号来显示它，通过使用具有预定义模式的 [`DecimalFormat`](https://web.archive.org/web/20220923221946/https://docs.oracle.com/en/java/javase/14/docs/api/java.base/java/text/DecimalFormat.html) :

```java
public static String withLargeIntegers(double value) {
    DecimalFormat df = new DecimalFormat("###,###,###");
    return df.format(value);
}

int value = 123456789;
assertThat(withLargeIntegers(value)).isEqualTo("123,456,789");
```

### 4.2.填充数字

在某些情况下，我们可能希望用零填充一个数字以达到指定的长度。这里，我们可以使用`String#format`方法，如前所述:

```java
public static String byPaddingZeros(int value, int paddingLength) {
    return String.format("%0" + paddingLength + "d", value);
}

int value = 1;
assertThat(byPaddingOutZeros(value, 3)).isEqualTo("001");
```

### 4.3.格式化小数点后有两个零的数字

为了能够打印小数点后有两个零的任何给定数字，我们将使用一个带有预定义模式的 time `DecimalFormat`类:

```java
public static double withTwoDecimalPlaces(double value) {
    DecimalFormat df = new DecimalFormat("#.00");
    return new Double(df.format(value));
}

int value = 12; 
assertThat(withTwoDecimalPlaces(value)).isEqualTo(12.00);
```

在本例中，**我们创建了一个新的格式，在小数点**后指定两个零。

### 4.4.格式和百分比

有时我们可能需要显示百分比。

在这种情况下，我们可以使用 [`NumberFormat#` `getPercentInstance`](https://web.archive.org/web/20220923221946/https://docs.oracle.com/en/java/javase/14/docs/api/java.base/java/text/NumberFormat.html#getPercentInstance(java.util.Locale)) 的方法。这个方法允许我们提供一个`Locale`,以您指定的国家正确的格式打印值:

```java
public static String forPercentages(double value, Locale locale) {
    NumberFormat nf = NumberFormat.getPercentInstance(locale);
    return nf.format(value);
}

double value = 25f / 100f;
assertThat(forPercentages(value, new Locale("en", "US"))).isEqualTo("25%");
```

### 4.5.货币数字格式

在我们的应用程序中，[存储货币](/web/20220923221946/https://www.baeldung.com/java-money-and-currency)的一种常见方式是使用`BigDecimal`。如果我们想要[向用户显示它们](/web/20220923221946/https://www.baeldung.com/java-money-into-words)呢？在这种情况下，我们可以使用`NumberFormat`类:

```java
public static String currencyWithChosenLocalisation(double value, Locale locale) {
    NumberFormat nf = NumberFormat.getCurrencyInstance(locale);
    return nf.format(value);
}
```

我们获取给定的`Locale`的 currency 实例，然后简单地用该值调用`format`方法。结果是显示为指定国家货币的数字:

```java
double value = 23_500;
assertThat(currencyWithChosenLocalisation(value, new Locale("en", "US"))).isEqualTo("$23,500.00");
assertThat(currencyWithChosenLocalisation(value, new Locale("zh", "CN"))).isEqualTo("￥23,500.00");
assertThat(currencyWithChosenLocalisation(value, new Locale("pl", "PL"))).isEqualTo("23 500 zł");
```

## 5.高级格式化用例

`[DecimalFormat](https://web.archive.org/web/20220923221946/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/text/DecimalFormat.html) `是 Java 中格式化十进制数最流行的方法之一。与前面的例子相似，我们将编写一个帮助器方法:

```java
public static double withDecimalFormatLocal(double value) {
    DecimalFormat df = (DecimalFormat) NumberFormat.getNumberInstance(Locale.getDefault());
    return new Double(df.format(value));
}
```

我们的格式类型将获得给定本地化的默认设置。

不同的国家使用不同的数字系统来处理十进制格式。例如，分组字符(在美国是逗号，但在其他语言环境中是空格或点号)、分组大小(在美国和大多数语言环境中是三，但在印度不同)或十进制字符(在美国是点号，但在其他语言环境中是逗号)。

```java
double D = 4.2352989244d;
assertThat(withDecimalFormatLocal(D)).isEqualTo(4.235);
```

我们还可以扩展这个功能来提供一些特定的模式:

```java
public static double withDecimalFormatPattern(double value, int places) {
    DecimalFormat df2 = new DecimalFormat("#,###,###,##0.00");
    DecimalFormat df3 = new DecimalFormat("#,###,###,##0.000");
    if (places == 2)
        return new Double(df2.format(value));
    else if (places == 3)
        return new Double(df3.format(value));
    else
        throw new IllegalArgumentException();
}

assertThat(withDecimalFormatPattern(D, 2)).isEqualTo(4.24); 
assertThat(withDecimalFormatPattern(D, 3)).isEqualTo(4.235);
```

在这里，我们允许用户根据空格数选择模式来配置`DecimalFormat`。

## 6.结论

在本文中，我们简要介绍了 Java 中不同的数字格式化方法。正如我们所看到的，没有一个最好的方法可以做到这一点。可以采用许多方法，因为每种方法都有自己的特点。

和往常一样，这些例子的代码可以在 GitHub 的[上找到。](https://web.archive.org/web/20220923221946/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-numbers-3)**