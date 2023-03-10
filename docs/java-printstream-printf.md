# 用 Java 中的 printf()格式化输出

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-printstream-printf>

## 1。概述

在本教程中，我们将用`printf()` **方法演示不同的格式化例子。**

该方法是`java.io.PrintStream`类的一部分，提供类似于 c 语言中的`printf()`函数的字符串格式。

## 延伸阅读:

## [Java . util . formatter 指南](/web/20220908014517/https://www.baeldung.com/java-string-formatter)

Introduction to formatting Strings in Java using the java.util.Formatter.[Read more](/web/20220908014517/https://www.baeldung.com/java-string-formatter) →

## [datetime formatter 指南](/web/20220908014517/https://www.baeldung.com/java-datetimeformatter)

Learn how to use the Java 8 DateTimeFormatter class to format and parse dates and times[Read more](/web/20220908014517/https://www.baeldung.com/java-datetimeformatter) →

## [在 Java 中用零或空格填充字符串](/web/20220908014517/https://www.baeldung.com/java-pad-string)

Learn how to pad a String in Java with a specific character.[Read more](/web/20220908014517/https://www.baeldung.com/java-pad-string) →

## 2。语法

我们可以使用这些`PrintStream`方法之一来格式化输出:

```java
System.out.printf(format, arguments);
System.out.printf(locale, format, arguments);
```

我们使用`format`参数指定格式化规则。规则以`%`字符开始。

在深入研究各种格式规则的细节之前，我们先看一个简单的例子:

```java
System.out.printf("Hello %s!%n", "World");
```

这会产生以下输出:

```java
Hello World!
```

如上所示，格式字符串包含纯文本和两个格式规则。第一个规则用于格式化字符串参数。第二条规则在字符串末尾添加一个换行符。

### 2.1.格式规则

让我们更仔细地看看格式字符串。它由文字和格式说明符组成。**格式说明符包括标志、宽度、精度和转换字符**，顺序如下:

```java
%[flags][width][.precision]conversion-character
```

括号中的说明符是可选的。

在内部，`printf()`使用 [java.util.Formatter](/web/20220908014517/https://www.baeldung.com/java-string-formatter) 类解析格式字符串并生成输出。其他格式字符串选项可以在[格式化程序 Javadoc](https://web.archive.org/web/20220908014517/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/Formatter.html#syntax) 中找到。

### 2.2。转换字符

**`conversion-character`是必需的，它决定了参数的格式。**

转换字符仅对某些数据类型有效。以下是一些常见的:

*   `s` 格式化字符串。
*   `d`格式化十进制整数。
*   `f`格式化浮点数。
*   `t` 格式化日期/时间值。

我们将在教程的后面探索这些和其他一些。

### 2.3。可选修饰符

**`[flags]`定义了修改输出**的标准方法，最常用于格式化整数和浮点数。

**`The [width]`指定输出自变量的字段宽度。**它表示写入输出的最小字符数。

**T0 指定输出浮点值时精度**的位数。此外，我们可以用它来定义从`String`中提取的子串的长度。

## 3。行分隔符

**为了将字符串分成单独的行，我们使用了% `n`说明符**:

```java
System.out.printf("baeldung%nline%nterminator");
```

上面的代码片段将产生以下输出:

```java
baeldung
line
terminator
```

**`%n` 分隔符` printf()`会自动插入主机系统的本机行分隔符。**

## 4。布尔格式

**为了格式化布尔值，我们使用`%b`格式。**其工作方式如下:如果输入值为`true`，则输出为`true`。否则输出`false`。

所以，如果我们做了以下事情:

```java
System.out.printf("%b%n", null);
System.out.printf("%B%n", false);
System.out.printf("%B%n", 5.3);
System.out.printf("%b%n", "random text");
```

然后我们会看到:

```java
false
FALSE
TRUE
true 
```

注意，我们可以使用`%B`进行大写格式。

## 5。字符串格式化

**为了格式化一个简单的字符串，我们将使用`%s`组合。**另外，我们可以让字符串大写:

```java
printf("'%s' %n", "baeldung");
printf("'%S' %n", "baeldung");
```

这是输出结果:

```java
'baeldung' 
'BAELDUNG'
```

同样，为了指定最小长度，我们可以指定一个`width`:

```java
printf("'%15s' %n", "baeldung");
```

这给了我们:

```java
'       baeldung'
```

如果我们需要左对齐我们的字符串，我们可以使用`– flag`:

```java
printf("'%-10s' %n", "baeldung");
```

这是输出:

```java
'baeldung  '
```

甚至，我们可以通过指定一个`precision`来限制输出中的字符数:

```java
System.out.printf("%2.2s", "Hi there!");
```

`%x.ys`语法中的第一个`x`数字是填充。`y`是字符数。

对于我们这里的例子，输出是`Hi`。

## 6。字符格式

`%c`的结果是一个 Unicode 字符:

```java
System.out.printf("%c%n", 's');
System.out.printf("%C%n", 's');
```

大写字母`C`将大写结果:

```java
s
S
```

但是如果我们给它一个无效的参数，那么`Formatter`就会抛出`IllegalFormatConversionException`。

## 7。 **数字格式**

### 7.1。整数格式化

`printf()`方法接受语言中所有可用的整数— `byte`、 `short`、 `int`、 `long`和`BigInteger `，如果我们使用`%d`:

```java
System.out.printf("simple integer: %d%n", 10000L);
```

在`d`角色的帮助下，我们会有这样的结果:

```java
simple integer: 10000
```

如果我们需要**用千位分隔符格式化我们的数字，我们可以使用`,` `flag` `.`** ，我们也可以为不同的地区格式化我们的结果:

```java
System.out.printf(Locale.US, "%,d %n", 10000);
System.out.printf(Locale.ITALY, "%,d %n", 10000);
```

正如我们所见，美国的格式与意大利不同:

```java
10,000 
10.000
```

### 7.2。浮点和双精度格式

要格式化一个浮点数，我们需要`f`格式:

```java
System.out.printf("%f%n", 5.1473);
```

它将输出:

```java
5.147300
```

当然，首先想到的是控制`precision`:

```java
System.out.printf("'%5.2f'%n", 5.1473);
```

这里我们定义我们的数的`width`为`5`，小数部分的长度为`2:`

```java
' 5.15'
```

这里，我们从数字的开头开始填充一个空格，以支持预定义的宽度。

为了用科学符号表示我们的输出**，我们只需使用`e conversion-character`** :

```java
System.out.printf("'%5.2e'%n", 5.1473);
```

这是我们的结果:

```java
'5.15e+00'
```

## 8。日期和时间格式

对于日期和时间格式，**转换字符串是两个字符的序列:`t`或`T`字符和转换后缀。**

让我们通过例子来探索最常见的时间和日期格式后缀字符。

当然，对于更高级的格式化，我们可以使用从 Java 8 开始就有的 [`DateTimeFormatter`](/web/20220908014517/https://www.baeldung.com/java-datetimeformatter) 。

### 8.1。时间格式

首先，让我们看看一些对时间格式有用的后缀字符列表:

*   **`H`、 `M`、 `S`字符负责从输入`Date`中提取小时、分钟、秒**。
*   `L`、 `N`相应地以毫秒和纳秒表示时间。
*   **`p`添加上午/下午格式。**
*   **`z`打印时区偏移量。**

现在，假设我们想要打印出`Date`的时间部分:

```java
Date date = new Date();
System.out.printf("%tT%n", date);
```

上面的代码和`%tT`组合产生以下输出:

```java
13:51:15
```

如果我们需要更详细的格式，我们可以调用不同的时间段:

```java
System.out.printf("hours %tH: minutes %tM: seconds %tS%n", date, date, date);
```

在使用了`H`、`M`和`S`之后，我们得到这个结果:

```java
hours 13: minutes 51: seconds 15
```

然而，多次列出`date `是一种痛苦。

或者，**为了去掉多个参数，我们可以使用输入参数的索引引用，在我们的例子中是`1$`**:

```java
System.out.printf("%1$tH:%1$tM:%1$tS %1$tp %1$tL %1$tN %1$tz %n", date);
```

这里我们希望输出当前时间，上午/下午，以毫秒和纳秒为单位的时间，以及时区偏移量:

```java
13:51:15 pm 061 061000000 +0400
```

### 8.2。日期格式

像时间格式一样，我们有特殊的格式字符用于日期格式:

*   **`A`打印出一周的全天。**
*   `d`格式化一个月中的两位数日期。
*   **`B`为整月名称。**
*   `m`格式化一个两位数的月份。
*   **`Y`以四位数输出年份。**
*   `y`输出年份的最后两位数字。

假设我们想显示星期几，然后是月份:

```java
System.out.printf("%1$tA, %1$tB %1$tY %n", date);
```

然后，使用`A`、`B`和`Y`，我们将得到以下输出:

```java
Thursday, November 2018
```

为了让我们的结果都是数字格式，我们可以用`d`、 `m`、 `y`替换`A`、 `B`、 `Y`字母:

```java
System.out.printf("%1$td.%1$tm.%1$ty %n", date);
```

这将导致:

```java
22.11.18
```

## 9。结论

在本文中，我们讨论了如何使用`PrintStream#printf`方法格式化输出。我们研究了用于控制常见数据类型输出的不同格式模式。

最后，和往常一样，讨论中使用的代码可以在 GitHub 上找到[。](https://web.archive.org/web/20220908014517/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-console)