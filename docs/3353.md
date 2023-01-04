# java.util.Formatter 指南

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-string-formatter>

## 1。概述

在本文中，我们将使用 [`java.util.Formatter`](https://web.archive.org/web/20220628064236/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/Formatter.html) 类讨论 Java 中的`String`格式，该类为布局对齐提供支持。

## 2。如何使用`Formatter`

还记得 C 的`printf?` 在 Java 里格式化一个`String` 感觉很像。

`Formatter`的`format()`方法通过`String`类的静态方法公开。这个方法接受一个模板`String`和一个参数列表来填充模板:

```
String greetings = String.format(
  "Hello Folks, welcome to %s !", 
  "Baeldung");
```

由此产生的`String`是:

```
"Hello Folks, welcome to Baeldung !"
```

模板是一个`String`，它包含一些静态文本和一个或多个格式说明符，这些说明符指示哪个参数应该放在特定的位置。

在本例中，**只有一个格式说明符`%s`，它被相应的参数替换。**

## 3。格式说明符

### 3.1。通用语法

类型`General, Character,`和`Numeric`的格式说明符语法为:

```
%[argument_index$][flags][width][.precision]conversion
```

说明符`argument_index, flag, width`和`precision`是可选的。

*   `argument_index`部分是整数`i`——表示这里应该使用参数列表中的`ith`参数
*   `flags`是用于修改输出格式的一组字符
*   `width`为正整数，表示写入输出的最小字符数
*   `precision`是一个整数，通常用来限制字符数，其具体行为取决于转换
*   是强制部分。它是一个字符，表示参数应该如何格式化。给定参数的有效转换集取决于参数的数据类型

在上面的例子中，如果我们想要明确地指定一个参数的编号，我们可以使用`1$`和`2$`参数索引来编写它。

这两个分别是第一个和第二个参数:

```
String greetings = String.format(
  "Hello %2$s, welcome to %1$s !", 
  "Baeldung", 
  "Folks");
```

### 3.2。对于`Date/Time`代表

```
%[argument_index$][flags][width]conversion
```

同样`argument_index, flags`和`width`是可选的。

我们举个例子来理解这个:

```
@Test
public void whenFormatSpecifierForCalendar_thenGotExpected() {
    Calendar c = new GregorianCalendar(2017, 11, 10);
    String s = String.format(
      "The date is: %tm %1$te,%1$tY", c);

    assertEquals("The date is: 12 10,2017", s);
}
```

这里，对于每个格式说明符，将使用第一个参数，因此称为`1$`。这里，如果我们跳过第二个和第三个格式说明符的`argument_index` ，它试图找到 3 个参数，但是我们需要对所有 3 个格式说明符使用相同的参数。

所以，如果我们不为第一个指定`argument _index` 是可以的，但是我们需要为另外两个指定。

这里的`flag`由两个字组成。其中第一个角色总是一个`‘t'`或 `‘T'`。第二个字符取决于要显示`Calendar`的哪一部分。

在我们的例子中，第一个格式说明符`tm`表示两个数字格式的月份，`te`表示一个月中的第几天，`tY`表示四个数字格式的年份。

### 3.3。没有参数的格式说明符

```
%[flags][width]conversion
```

可选的`flags`和`width`与以上章节中定义的相同。

必需的`conversion`是一个字符或`String`表示要插入输出的内容。目前，只有 `‘%'`和换行符 `‘n'`可以用这个打印

```
@Test
public void whenNoArguments_thenExpected() {
    String s = String.format("John scored 90%% in Fall semester");

    assertEquals("John scored 90% in Fall semester", s);
} 
```

在`format()`里面，如果我们想要打印`‘%'`——我们需要使用`‘%%'`来转义它。

## 4。转换

现在让我们深入研究格式说明符语法的每个细节，从`conversion`开始。请注意，您可以在 [`Formatter` javadocs](https://web.archive.org/web/20220628064236/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/Formatter.html) 中找到所有细节。

正如我们在上面的例子中注意到的，`conversion`部分在所有格式说明符中都是必需的，它可以分为几类。

让我们通过举例来看看每一个。

### 4.1。将军

用于任何参数类型。一般转换为:

1.  `‘b’`或 `‘B'`——对于`Boolean`值
2.  `‘h’`或`‘H'`——代表`HashCode`
3.  `‘s'`或`‘S'`—`String`时，如果`null`打印“空”，否则`arg.toString()`

我们现在将尝试显示`boolean`和`String`值，使用相应的转换:

```
@Test
public void givenString_whenGeneralConversion_thenConvertedString() {
    String s = String.format("The correct answer is %s", false);
    assertEquals("The correct answer is false", s);

    s = String.format("The correct answer is %b", null);
    assertEquals("The correct answer is false", s);

    s = String.format("The correct answer is %B", true);
    assertEquals("The correct answer is TRUE", s);
}
```

### 4.2。`**Character**`

用于表示 Unicode 字符的基本类型:`char, Character, byte, Byte, short,` 和 `Short`。当`Character.isValidCodePoint(int)`为类型`int`和`Integer`返回`true`时，该转换也可用于它们。

**根据我们想要的案例可以写成`‘c’`或者`’C’`。**

让我们试着打印一些字符:

```
@Test
public void givenString_whenCharConversion_thenConvertedString() {
    String s = String.format("The correct answer is %c", 'a');
    assertEquals("The correct answer is a", s);

    s = String.format("The correct answer is %c", null);
    assertEquals("The correct answer is null", s);

    s = String.format("The correct answer is %C", 'b');
    assertEquals("The correct answer is B", s);

    s = String.format("The valid unicode character: %c", 0x0400);
    assertTrue(Character.isValidCodePoint(0x0400));
    assertEquals("The valid unicode character: Ѐ", s);
}
```

让我们再举一个无效代码点的例子:

```
@Test(expected = IllegalFormatCodePointException.class)
public void whenIllegalCodePointForConversion_thenError() {
    String s = String.format("The valid unicode character: %c", 0x11FFFF);

    assertFalse(Character.isValidCodePoint(0x11FFFF));
    assertEquals("The valid unicode character: Ā", s);
}
```

### 4.3。数字-**积分**

这些用于 Java 整型:`byte, Byte, short, Short, int`和 `Integer, long, Long,`和`BigInteger`。此类别中有三种转换:

1.  `‘d'`–为十进制数
2.  `‘o'`–用于八进制数
3.  `‘X'`或`‘x'`–十六进制数

让我们试着把这些打印出来:

```
@Test
public void whenNumericIntegralConversion_thenConvertedString() {
    String s = String.format("The number 25 in decimal = %d", 25);
    assertEquals("The number 25 in decimal = 25", s);

    s = String.format("The number 25 in octal = %o", 25);
    assertEquals("The number 25 in octal = 31", s);

    s = String.format("The number 25 in hexadecimal = %x", 25);
    assertEquals("The number 25 in hexadecimal = 19", s);
}
```

### 4.4。数字–浮点

用于 Java 浮点类型:`float, Float, double, Double,`和`BigDecimal`

1.  `‘e'`或`‘E'`**–**用计算机化的科学记数法格式化为十进制数
2.  `‘f'`**–**格式化为十进制数
3.  `‘g'`或`‘G'`**–**根据四舍五入后的精度值，转换成计算机化的科学记数法或十进制格式

让我们试着打印浮点数:

```
@Test
public void whenNumericFloatingConversion_thenConvertedString() {
    String s = String.format(
      "The computerized scientific format of 10000.00 "
      + "= %e", 10000.00);

    assertEquals(
      "The computerized scientific format of 10000.00 = 1.000000e+04", s);

    String s2 = String.format("The decimal format of 10.019 = %f", 10.019);
    assertEquals("The decimal format of 10.019 = 10.019000", s2);
}
```

### 4.5。其他转换

*   `Date/Time`–对于能够编码日期或时间的 Java 类型:`long, Long, Calendar,` `Date`和`TemporalAccessor.` 为此，我们需要使用前缀 `‘t'`或`‘T'`，就像我们之前看到的
*   `Percent`–打印文字`‘%' (‘\u0025')`
*   `Line Separator`–打印特定于平台的行分隔符

让我们看一个简单的例子:

```
@Test
public void whenLineSeparatorConversion_thenConvertedString() {
    String s = String.format("First Line %nSecond Line");

    assertEquals("First Line \n" + "Second Line", s);
}
```

## 5。标志

通常，标志用于格式化输出。而在日期和时间的情况下，它们用于指定显示日期的哪一部分，正如我们在第 4 节的例子中看到的。

有许多标志可用，其列表可在文档中找到。

让我们看一个旗帜示例来了解它的用法。`‘-‘`用于将输出格式化为左对齐:

```
@Test
public void whenSpecifyFlag_thenGotFormattedString() {
    String s = String.format("Without left justified flag: %5d", 25);
    assertEquals("Without left justified flag:    25", s);

    s = String.format("With left justified flag: %-5d", 25);
    assertEquals("With left justified flag: 25   ", s);
}
```

## 6。精度

对于一般转换， **precision 就是要写入输出**的最大字符数。而 f 或浮点转换的精度是小数点后的位数。

第一个语句是浮点数的精度示例，第二个语句是一般转换的示例:

```
@Test
public void whenSpecifyPrecision_thenGotExpected() {
    String s = String.format(
      "Output of 25.09878 with Precision 2: %.2f", 25.09878);

    assertEquals("Output of 25.09878 with Precision 2: 25.10", s);

    String s2 = String.format(
      "Output of general conversion type with Precision 2: %.2b", true);

    assertEquals("Output of general conversion type with Precision 2: tr", s2);
}
```

## 7。自变量索引

如前所述， **`argument_index`是一个整数，表示自变量在自变量列表**中的位置。`1$` 表示第一个参数，`2$` 表示第二个参数，依此类推。

此外，还有另一种通过位置引用参数的方法，通过使用 `‘<‘ (‘\u003c')`标志，这意味着来自先前格式说明符的参数将被重用。例如，这两条语句会产生相同的输出:

```
@Test
public void whenSpecifyArgumentIndex_thenGotExpected() {
    Calendar c = Calendar.getInstance();
    String s = String.format("The date is: %tm %1$te,%1$tY", c);
    assertEquals("The date is: 12 10,2017", s);

    s = String.format("The date is: %tm %<te,%<tY", c);
    assertEquals("The date is: 12 10,2017", s);
}
```

## 8。`Formatter`其他使用方式

到目前为止，我们已经看到了`Formatter` 类的`format()` 方法的使用。我们还可以创建一个`Formatter`实例，并使用它来调用`format()` 方法。

**我们可以通过传入一个`Appendable`、`OutputStream`、`File`或文件名**来创建一个实例。基于此，格式化后的`String`分别存储在`Appendable`、`OutputStream`、`File` 中。

让我们看一个和`Appendable.` 一起使用的例子，我们可以用同样的方式和其他人一起使用。

### 8.1。使用`Formatter`与`Appendable`和

让我们创建一个 S `tringBuilder` 实例`sb`，并使用它创建一个`Formatter` 。然后我们将调用`format()` 来格式化一个`String`:

```
@Test
public void whenCreateFormatter_thenFormatterWithAppendable() {
    StringBuilder sb = new StringBuilder();
    Formatter formatter = new Formatter(sb);
    formatter.format("I am writting to a %s Instance.", sb.getClass());

    assertEquals(
      "I am writting to a class java.lang.StringBuilder Instance.", 
      sb.toString());
}
```

## 9。结论

在本文中，我们看到了由`java.util.Formatter`类提供的格式化工具。我们看到了可用于格式化`String`的各种语法，以及可用于不同数据类型的转换类型。

像往常一样，我们看到的例子的代码可以在 Github 上找到[。](https://web.archive.org/web/20220628064236/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-string-apis)