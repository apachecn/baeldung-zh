# 爪哇的南

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-not-a-number>

## 1.概观

简单来说，`NaN`是一个数值数据类型值，代表“不是一个数字”。

在这个快速教程中，我们将解释 Java 中的`NaN` 值以及可以产生或涉及这个值的各种操作。

## 2.什么是`NaN`？

**`NaN`通常表示无效操作的结果。**例如，试图将零除以零就是这样一种运算。

**我们也用 `NaN`表示不可代表的值。**-1 的平方根就是这样一种情况，因为我们只能用复数来描述值(`i`)。

[IEEE 浮点运算标准(IEEE 754)](https://web.archive.org/web/20220523135107/https://en.wikipedia.org/wiki/IEEE_754) 定义了`NaN`值。**在 Java 中，浮点类型`float`和`double`实现了这个标准。**

Java 将`float`和`double`类型的 `NaN`常量定义为 [`Float`。楠](https://web.archive.org/web/20220523135107/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/lang/Float.html#NaN)和[`Double.NaN`T8:](https://web.archive.org/web/20220523135107/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/lang/Double.html#NaN)

”`A constant holding a Not-a-Number (NaN) value of type double. It is equivalent to the value returned by Double.longBitsToDouble(0x7ff8000000000000L).”`

并且:

`“A constant holding a Not-a-Number (NaN) value of type float. It is equivalent to the value returned by Float.intBitsToFloat(0x7fc00000).”`

在 Java 中，其他数值数据类型没有这种类型的常量。

## 3.与`NaN`的比较

在用 Java 编写方法时，我们应该检查输入是否有效，是否在预期的范围内。`NaN`值在大多数情况下不是有效的输入。因此，我们应该验证输入值不是一个`NaN`值，并适当地处理这些输入值。

`NaN`不能与任何浮点型值进行比较。这意味着对于所有涉及`NaN`的比较操作，我们将得到`false`(除了"！= "为此我们得到`true`。

当且仅当`x`是`NaN:`时，我们得到`x != x”`的`true`

```
System.out.println("NaN == 1 = " + (NAN == 1));
System.out.println("NaN > 1 = " + (NAN > 1));
System.out.println("NaN < 1 = " + (NAN < 1));
System.out.println("NaN != 1 = " + (NAN != 1));
System.out.println("NaN == NaN = " + (NAN == NAN));
System.out.println("NaN > NaN = " + (NAN > NAN));
System.out.println("NaN < NaN = " + (NAN < NAN));
System.out.println("NaN != NaN = " + (NAN != NAN)); 
```

让我们看看运行上面代码的结果:

```
NaN == 1 = false
NaN > 1 = false
NaN < 1 = false
NaN != 1 = true
NaN == NaN = false
NaN > NaN = false
NaN < NaN = false
NaN != NaN = true 
```

因此，**我们无法通过使用“==”或”与`NaN`进行比较来检查`NaN`！= ".**其实我们应该很少用“==”或者”！= "带有`float`或`double` 类型的操作员。

相反，我们可以使用表达式"`x !` = x" `.` 这个表达式只对`NAN.`返回 true

我们还可以使用方法`Float.isNaN`和`Double.isNaN` 来检查这些值`.` 这是首选方法，因为它更具可读性和可理解性:

```
double x = 1;
System.out.println(x + " is NaN = " + (x != x));
System.out.println(x + " is NaN = " + (Double.isNaN(x)));

x = Double.NaN;
System.out.println(x + " is NaN = " + (x != x));
System.out.println(x + " is NaN = " + (Double.isNaN(x))); 
```

运行这段代码时，我们将得到以下结果:

```
1.0 is NaN = false
1.0 is NaN = false
NaN is NaN = true
NaN is NaN = true
```

## 4.生产操作`NaN`

在进行涉及`float`和`double`类型的操作时，我们需要知道`NaN`值。

**一些浮点方法和操作产生`NaN`值，而不是抛出一个`Exception.`** 我们可能需要显式地处理这样的结果。

导致非数字值的常见情况是**数学上未定义的数字运算**:

```
double ZERO = 0;
System.out.println("ZERO / ZERO = " + (ZERO / ZERO));
System.out.println("INFINITY - INFINITY = " + 
  (Double.POSITIVE_INFINITY - Double.POSITIVE_INFINITY));
System.out.println("INFINITY * ZERO = " + (Double.POSITIVE_INFINITY * ZERO)); 
```

这些示例会产生以下输出:

```
ZERO / ZERO = NaN
INFINITY - INFINITY = NaN
INFINITY * ZERO = NaN 
```

**没有实数结果的数值运算也产生`NaN:`**

```
System.out.println("SQUARE ROOT OF -1 = " + Math.sqrt(-1));
System.out.println("LOG OF -1 = " +  Math.log(-1)); 
```

这些声明将导致:

```
SQUARE ROOT OF -1 = NaN
LOG OF -1 = NaN 
```

所有以`NaN`作为操作数的数字运算都会产生结果`NaN`:

```
System.out.println("2 + NaN = " +  (2 + Double.NaN));
System.out.println("2 - NaN = " +  (2 - Double.NaN));
System.out.println("2 * NaN = " +  (2 * Double.NaN));
System.out.println("2 / NaN = " +  (2 / Double.NaN)); 
```

而以上的结果是:

```
2 + NaN = NaN
2 - NaN = NaN
2 * NaN = NaN
2 / NaN = NaN 
```

最后，我们不能将`null`赋给`double`或`float`类型的变量。相反，我们可以显式地将`NaN`赋给这样的变量来表示缺失或未知的值:

```
double maxValue = Double.NaN;
```

## 5.结论

在本文中，我们讨论了`NaN`和与之相关的各种操作。我们还讨论了在 Java 中显式进行浮点计算时处理`NaN`的需求。

完整的源代码可以在 GitHub 上找到[。](https://web.archive.org/web/20220523135107/https://github.com/eugenp/tutorials/tree/master/java-numbers-2)