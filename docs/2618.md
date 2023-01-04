# 在 Java 中将字符串转换为浮点型，然后再转换回来

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-string-to-float>

## 1.介绍

在 Java 中，将数据从`Float`转换到`String`，反之亦然，这是一个普通的操作。然而，有许多方法可以做到这一点，这可能会导致混乱和不确定的选择。

在本文中，我们将展示和比较所有可用的选项。

## 2.`Float`至`String`

首先，让我们看看将`Float`值转换为`String`的最常见方法。

### 2.1.`String`串联

我们可以使用的最直接的解决方案是将浮点值与空的`String`连接起来。

让我们看一个例子:

```
float givenFloat = 1.25f;

String result = givenFloat + "";

assertEquals("1.25", result);
```

类似地，我们可以向空的`String`添加一个`Float`对象，得到相同的结果。当我们使用一个`Float`对象时，它的`toString()`方法被自动调用:

```
Float givenFloat = 1.25f;

String result = givenFloat + "";

assertEquals("1.25", result); 
```

**如果`Float`对象为空，拼接结果将是一个“空”** `**String**:`

```
Float givenFloat = null;

String result = givenFloat + "";

assertEquals("null", result);
```

### 2.2.`Float.toString()`

我们可以使用的另一个选项是用于`String`转换的`Float`类的静态`toString()`方法。**我们可以将一个`float`原始值或者一个`Float`对象传递给`toString()`方法:**

```
Float givenFloat = 1.25f;

String result = Float.toString(givenFloat);

assertEquals("1.25", result);
```

如果我们将 null 作为参数传递给方法，我们将在运行时得到一个`NullPointerException`:

```
Float givenFloat = null;

assertThrows(NullPointerException.class, () -> Float.toString(givenFloat));
```

### 2.3.`String.valueOf()`

同样，我们可以使用`String`的静态`valueOf`方法:

```
Float givenFloat = 1.25f;

String result = String.valueOf(givenFloat);

assertEquals("1.25", result);
```

**与`Float.toString()`不同，如果我们将 null 作为参数传递，`String.valueOf()`不会抛出异常，而是返回“null”`String`**:

```
Float givenFloat = null;

String result = String.valueOf(givenFloat);

assertEquals("null", result);
```

### 2.4.`String.format()`

[`String`的`format()`静态方法](/web/20221023005054/https://www.baeldung.com/string/format)为我们提供了额外的格式化选项。我们必须知道，如果不限制小数位数，即使没有小数部分，结果也会包含尾随零，如下例所示:

```
Float givenFloat = 1.25f;

String result = String.format("%f", givenFloat);

assertEquals("1.250000", result);
```

当我们格式化指定小数位数的浮点数时，**`format()`方法也会将结果**四舍五入:

```
Float givenFloat = 1.256f;

String result = String.format("%.2f", givenFloat);

assertEquals("1.26", result);
```

如果我们传递一个空值`Float`，那么转换后的结果将是一个“空值”`String`:

```
Float givenFloat = null;

String result = String.format("%f", givenFloat);

assertEquals("null", result);
```

### 2.5.`DecimalFormat`

最后，**[`DecimalFormat`](/web/20221023005054/https://www.baeldung.com/java-decimalformat)类有一个`format()`方法，允许将浮点值转换成定制格式的`Strings`** 。这样做的好处是，我们可以精确地定义在我们得到的`String`中需要多少位小数。

让我们看看如何在一个例子中使用它:

```
Float givenFloat = 1.25f;

String result = new DecimalFormat("#.0000").format(givenFloat);

assertEquals("1.2500", result);
```

如果我们应用格式后，没有小数部分， **`DecimalFormat`将返回整数**:

```
Float givenFloat = 1.0025f;

String result = new DecimalFormat("#.##").format(givenFloat);

assertEquals("1", result);
```

如果我们将 null 作为参数传递，那么我们将得到一个`IllegalArgumentException`:

```
Float givenFloat = null;

assertThrows(IllegalArgumentException.class, () -> new DecimalFormat("#.000").format(givenFloat));
```

## 3.`String`至`Float`

接下来，让我们看看将*字符串的*值转换为`Float`的最常见方法。

### 3.1.`Float.parseFloat()`

最常见的一种方式就是使用`Float`的静态方法:`parseFloat()`。**它将返回一个原始的`float`值，由`String`参数**表示。此外，前导空格和尾随空格会被忽略:

```
String givenString = "1.25";

float result = Float.parseFloat(givenString);

assertEquals(1.25f, result);
```

如果`String`参数为空，我们得到一个`NullPointerException`:

```
String givenString = null;

assertThrows(NullPointerException.class, () -> Float.parseFloat(givenString));
```

如果`String`参数不包含可解析的`float`，我们得到一个`NumberFormatException:`

```
String givenString = "1.23x";

assertThrows(NumberFormatException.class, () -> Float.parseFloat(givenString));
```

### 3.2.`Float.valueOf()`

同样，我们可以使用`Float`的静态`valueOf()`方法。**不同的是`valueOf()`返回一个`Float`对象**。具体来说，它调用了`parseFloat()`方法并将其打包成一个`Float`对象:

```
String givenString = "1.25";

Float result = Float.valueOf(givenString);

assertEquals(1.25f, result);
```

同样，如果我们传递一个不可解析的`String`，我们将得到一个`NumberFormatException`:

```
String givenString = "1.25x";

assertThrows(NumberFormatException.class, () -> Float.valueOf(givenString));
```

### 3.3.`DecimalFormat`

我们也可以使用`DecimalFormat`将`String`转换成`Float`。**主要优势之一是指定自定义小数点分隔符**。

```
String givenString = "1,250";
DecimalFormatSymbols symbols = new DecimalFormatSymbols();
symbols.setDecimalSeparator(',');
DecimalFormat decimalFormat = new DecimalFormat("#.000");
decimalFormat.setDecimalFormatSymbols(symbols);

Float result = decimalFormat.parse(givenString).floatValue();

assertEquals(1.25f, result);
```

### 3.4.Float 的构造函数

最后，我们可以直接使用`Float`的构造函数进行转换。**内部将使用`Float`的静态`parseFloat()`方法并创建`Float`对象:**

```
String givenString = "1.25";

Float result = new Float(givenString);

assertEquals(1.25f, result);
```

从 Java 9 开始，**这个构造函数已经被[否决了。](https://web.archive.org/web/20221023005054/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/lang/Float.html#%3Cinit%3E(java.lang.String))** 相反，我们应该考虑使用其他静态工厂方法，如`parseFloat()`或`valueOf()`。

## 4.结论

在本文中，我们探索了多种将`String`实例转换为*浮动*或`Float`实例的方法。

对于简单的转换，`String`串联和`Float.toString()`是转换为`String`的更好的选择。如果我们需要更复杂的格式，那么`DecimalFormat`是最好的工具。为了将字符串转换成浮点值，如果我们需要一个`float`原语，我们可以使用`Float.parseFloat()`，或者如果我们更喜欢一个`Float`对象，我们可以使用`Float.valueOf()`。同样，对于自定义格式，`DecimalFormat`是最好的选择。

和往常一样，这些例子的代码可以在 GitHub 的[上找到。](https://web.archive.org/web/20221023005054/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-string-conversions-2)