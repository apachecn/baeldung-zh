# 十进制格式实用指南

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-decimalformat>

## 1。概述

在本文中，我们将探索`[DecimalFormat](https://web.archive.org/web/20220915122120/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/text/DecimalFormat.html)`类及其实际用法。

这是`NumberFormat`的一个子类，允许使用预定义的模式格式化十进制数字的`String`表示。

它也可以反过来使用，将字符串解析成数字。

## 2。它是如何工作的？

为了格式化一个数字，我们必须定义一个模式，这是一个可能与文本混合的特殊字符序列。

有 11 种特殊模式字符，但最重要的是:

*   0–如果提供，则打印一个数字，否则打印 0
*   #–如果提供，则打印一个数字，否则不打印
*   。–指示放置小数点分隔符的位置
*   ，–指示放置分组分隔符的位置

当模式应用于一个数字时，它的格式规则被执行，结果根据 JVM 的`Locale`的`DecimalFormatSymbol`被打印，除非指定了一个特定的`Locale`。

下面的例子的输出来自于一个运行在 English `Locale`上的 JVM。

## 3。基本格式

现在让我们看看当用下面的模式格式化相同的数字时会产生哪些输出。

### 3.1。简单小数

```java
double d = 1234567.89;    
assertThat(
  new DecimalFormat("#.##").format(d)).isEqualTo("1234567.89");
assertThat(
  new DecimalFormat("0.00").format(d)).isEqualTo("1234567.89"); 
```

正如我们所看到的，整数部分永远不会被丢弃，无论模式是否小于数字。

```java
assertThat(new DecimalFormat("#########.###").format(d))
  .isEqualTo("1234567.89");
assertThat(new DecimalFormat("000000000.000").format(d))
  .isEqualTo("001234567.890"); 
```

相反，如果模式大于数字，则添加零，而丢弃散列，无论是整数部分还是小数部分。

### 3.2。舍入

如果模式的小数部分不能包含输入数字的完整精度，它会被四舍五入。

这里，0.89 部分被舍入到 0.90，然后 0 被删除:

```java
assertThat(new DecimalFormat("#.#").format(d))
  .isEqualTo("1234567.9"); 
```

在这里，0.89 部分被舍入到 1.00，然后 0.00 被丢弃，1 被求和为 7:

```java
assertThat(new DecimalFormat("#").format(d))
  .isEqualTo("1234568"); 
```

默认的舍入模式是`[HALF_EVEN](https://web.archive.org/web/20220915122120/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/math/RoundingMode.html#HALF_EVEN),`，但是可以通过`[setRoundingMode](https://web.archive.org/web/20220915122120/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/text/DecimalFormat.html#setRoundingMode(java.math.RoundingMode))`方法定制。

### 3.3。分组

分组分隔符用于指定自动重复的子模式:

```java
assertThat(new DecimalFormat("#,###.#").format(d))
  .isEqualTo("1,234,567.9");
assertThat(new DecimalFormat("#,###").format(d))
  .isEqualTo("1,234,568"); 
```

### 3.4。多种分组模式

一些国家的编号系统中有不同数量的分组模式。

印度的编号系统使用#、#、###格式。##，其中只有第一个分组分隔符保存三个数字，而所有其他分隔符保存两个数字。

这是不可能使用`DecimalFormat`类实现的，它只保留从左到右遇到的最新模式，并将其应用于整数，忽略以前的分组模式。

尝试使用模式###，##，##，##，##，会导致重组到######，##，并以重新分配到####，# # # #结束。

为了实现多分组模式匹配，需要编写我们自己的`String`操纵代码，或者尝试 [Icu4J 的`DecimalFormat`](https://web.archive.org/web/20220915122120/https://ssl.icu-project.org/apiref/icu4j/com/ibm/icu/text/DecimalFormat.html) ，这就允许了。

### 3.5。混合字符串文字

可以在模式中混合使用`String`文字:

```java
assertThat(new DecimalFormat("The # number")
  .format(d))
  .isEqualTo("The 1234568 number"); 
```

也可以通过转义使用特殊字符作为`String`文字:

```java
assertThat(new DecimalFormat("The '#' # number")
  .format(d))
  .isEqualTo("The # 1234568 number"); 
```

## 4。本地化格式

许多[国家](https://web.archive.org/web/20220915122120/https://en.wikipedia.org/wiki/Decimal_separator#Hindu%E2%80%93Arabic_numeral_system)不使用英文符号，使用逗号作为小数点分隔符，使用点号作为分组分隔符。

运行# # # #。例如，带有意大利语`Locale`的 JVM 上的# pattern 将输出 1.234.567，89。

虽然在某些情况下，这可能是一个有用的 i18n 特性，但在其他情况下，我们可能希望实施一种特定的、独立于 JVM 的格式。

我们可以这样做:

```java
assertThat(new DecimalFormat("#,###.##", 
  new DecimalFormatSymbols(Locale.ENGLISH)).format(d))
  .isEqualTo("1,234,567.89");
assertThat(new DecimalFormat("#,###.##", 
  new DecimalFormatSymbols(Locale.ITALIAN)).format(d))
  .isEqualTo("1.234.567,89"); 
```

如果我们感兴趣的`Locale`不在`DecimalFormatSymbols`构造函数覆盖的范围内，我们可以用 [`getInstance`](https://web.archive.org/web/20220915122120/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/text/NumberFormat.html#getInstance()) 方法指定它:

```java
Locale customLocale = new Locale("it", "IT");
assertThat(new DecimalFormat(
  "#,###.##", 
   DecimalFormatSymbols.getInstance(customLocale)).format(d))
  .isEqualTo("1.234.567,89");
```

## 5。科学符号

[科学符号](https://web.archive.org/web/20220915122120/https://en.wikipedia.org/wiki/Scientific_notation)代表尾数和十的指数的乘积。数字 1234567.89 也可以表示为 12.3456789 * 10^5(点移动 5 位)。

### 5.1。`E`-批注

可以使用代表十的指数的`E`模式字符用科学记数法来表示一个数:

```java
assertThat(new DecimalFormat("00.#######E0").format(d))
  .isEqualTo("12.3456789E5");
assertThat(new DecimalFormat("000.000000E0").format(d))
  .isEqualTo("123.456789E4"); 
```

我们应该记住指数后面的字符数是相关的，所以如果我们需要表达 10^12，我们需要`E00`而不是`E0`。

### 5.2。工程符号

通常使用一种特殊形式的科学记数法，称为工程记数法，这种记数法会调整结果，以便将其表示为三的倍数，例如，当使用基洛(10^3)、兆(10^6)、吉(10^9)等测量单位时。

我们可以通过调整整数数字的最大数量(用#表示的字符，在小数点分隔符的左边)来加强这种记数法，使它大于最小数量(用 0 表示的那个)并且大于 1。

这迫使指数是最大数的倍数，因此对于这个用例，我们希望最大数是 3:

```java
assertThat(new DecimalFormat("##0.######E0")
  .format(d)).isEqualTo("1.23456789E6");		
assertThat(new DecimalFormat("###.000000E0")
  .format(d)).isEqualTo("1.23456789E6"); 
```

## 6。解析

让我们看看如何用 parse 方法将 `String`解析成`Number`:

```java
assertThat(new DecimalFormat("", new DecimalFormatSymbols(Locale.ENGLISH))
  .parse("1234567.89"))
  .isEqualTo(1234567.89);
assertThat(new DecimalFormat("", new DecimalFormatSymbols(Locale.ITALIAN))
  .parse("1.234.567,89"))
  .isEqualTo(1234567.89);
```

由于返回值不是由小数点的存在推断出来的，我们可以使用返回的`Number`对象的`.doubleValue()`、`.longValue()`等方法在输出中强制执行特定的原语。

我们还可以获得如下的`BigDecimal`:

```java
NumberFormat nf = new DecimalFormat(
  "", 
  new DecimalFormatSymbols(Locale.ENGLISH));
((DecimalFormat) nf).setParseBigDecimal(true);

assertThat(nf.parse("1234567.89"))
  .isEqualTo(BigDecimal.valueOf(1234567.89)); 
```

## 7。线程安全

**`DecimalFormat`不是线程安全的**，因此我们在线程间共享同一个实例时要特别注意。

## 8。结论

我们已经看到了`DecimalFormat`类的主要用法，以及它的优点和缺点`.`

和往常一样，完整的源代码可以在 Github 上找到[。](https://web.archive.org/web/20220915122120/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-numbers)