# Java 文字

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-literals>

## 1.概观

在使用 Java 编程语言时，我们将使用大量的文字。

在本教程中，我们将看看所有类型的文字以及如何使用它们。

## 2.什么是 Java 文字？

Java 文字是我们在代码中指定为常量的任何值。可以是任何类型——`integer`、 `float`、 `double`、 `long`、 `String`、 `char`、 `or boolean.`

在下面的例子中，数字`1` 和字符串`literal_string`是文字。

```java
int x = 1;
String s = "literal_string";
```

当处理文字时，很好地理解 [Java 原语](/web/20221024103657/https://www.baeldung.com/java-primitives)同样重要。

## 3.文字的类型

让我们通过一些例子来看看不同类型的文字。

### 3.1.整数文字

对于整数(`int, long, short, byte`)，我们可以用不同的方式来指定。

首先，我们可以使用**十进制文字(基数为 10)** 。这些是最常用的，因为它们是我们日常使用的数字。

```java
int x = 1;
```

其次，我们可以用**八进制形式(基数 8)** 指定整数文字量。在这种形式下，他们不得不以**开始，以**开始`**0**.`

```java
int x = 05;
```

第三，整数文字可以用在**十六进制形式(基数 16)** 中。他们必须以`0x `或 T1 开始

```java
int x = 0x12ef;
```

最后，我们有**二进制形式(基数 2)** 。这种形式是在 Java 1.7 中引入的，这些文字必须以`0b `或 `**0B**.`开始

```java
int x = 0b1101;
```

在上面的例子中，我们可以将`int`更改为任何整数类型，只要文字值不超过类型限制。

默认情况下，这些文字被认为是`int` 。对于 `long`、 `short`或`byte`，编译器检查值是否符合类型的限制，如果是，它将它们视为该类型的文字。

我们可以通过对长文本使用`l`或`L`来覆盖默认`int`文本。只有当文字值超过`int `限制时，我们才需要使用它。

### 3.2.浮点文字

默认情况下，浮点文字被视为`double`。我们也可以通过使用`d`或`D`来指定文字是一个`double`。这不是强制性的，但这是一个很好的做法。

我们可以通过使用`f`或`F`T5 来**指定我们想要一个`float`。这对于类型为`float`的文字是必需的。**

浮点文字只能以十进制形式指定(基数为 10):

```java
double d = 123.456;
float f = 123.456;
float f2 = 123.456d;
float f3 = 123.456f;
```

由于类型不匹配，第二个和第三个示例无法编译。

### 3.3.字符文字

对于`char`数据类型，文字有几种方式。

单引号字符很常见，对于特殊字符尤其有用。

```java
char c = 'a';
char c2 = '\n';
```

我们可以**在单引号**之间只使用一个字符或一个特殊字符。

用于字符的整数值被转换为该字符的 Unicode 值:

```java
char c = 65;
```

我们可以像指定整数一样指定它。

最后，我们可以使用 Unicode 表示法:

```java
char c= '\u0041';
```

最后两个例子中的表达式计算结果为字符`A.`

### 3.4.字符串文字

双引号之间的任何文本都是一个`String`文字:

```java
String s = "string_literal";
```

文字只能在一行中。为了有一个多行的`String`，我们可以使用一个将在编译时执行的表达式:

```java
String multiLineText = "When we want some text that is on more than one line,\n"
+ "then we can use expressions to add text to a new line.\n";
```

## 4.短文本和字节文本

上面，我们看到了如何声明每种类型的文字。对于所有类型，除了`byte and short`，我们不需要创建变量。我们可以简单地使用文字值。

例如，当向方法传递参数时，我们可以传递文字:

```java
static void literals(int i, long l, double d, float f, char c, String s) {
    // do something
}
//Call literals method
literals(1, 123L, 1.0D, 1.0F, 'a', "a");
```

有符号来指定文字的类型，所以它们不用作默认类型。在上面的例子中，`F`是唯一的强制符号。

令人惊讶的是，`byte`和`short`类型出现了问题:

```java
static void shortAndByteLiterals(short s, byte b) {
    // do something
}
//Call the method
shortAndByteLiterals(1, 0); // won't compile
```

尽管有这个限制，还是有 2 个解决方案。

第一个解决方案是使用我们之前声明的一些变量:

```java
short s = 1;
byte b = 1;
shortAndByteLiterals(s, b);
```

另一个选择是转换文字值:

```java
shortAndByteLiterals((short) 1, (byte) 0);
```

为了让编译器使用适当的类型，这是必要的。

## 5.结论

在本文中，我们研究了在 Java 中指定和使用文字的不同方式。****