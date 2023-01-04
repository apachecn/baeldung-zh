# Java 原语转换

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-primitive-conversions>

## 1。简介

Java 是一种类型化语言，这意味着它利用了类型的概念。有两种不同的类型组:

1.  原始数据类型
2.  抽象数据类型。

在本文中，我们将关注基本类型的转换。

## 2。原语概述

我们必须知道的第一件事是什么样的值可以用于基本类型。有八种基本类型，它们是:

*   **`byte`**–8 位有符号

*   **`short`**–16 位有符号

*   **`char`**–16 位且无符号，以便可以表示 Unicode 字符

*   **`int`**–32 位有符号

*   **`long`**–64 位有符号

*   **`float`**–32 位有符号

*   **`double`**–64 位有符号

*   **`boolean`**–不是数值，可能只有`true`或`false`的值

这并不是要对原语进行广泛的讨论，我们将在转换期间根据需要对它们的细节进行更多的讨论。

## 3。扩大原始转换

当我们需要从比目标类型更简单或更小的原语进行转换时，我们不必为此使用任何特殊的符号:

```
int myInt = 127;
long myLong = myInt;
```

在扩大转换过程中，较小的原始值被放在一个较大的容器上，这意味着该值左侧所有多余的空间都用零填充。这也可以用于从整数组到浮点组:

```
float myFloat = myLong;
double myDouble = myLong;
```

这是可能的，因为移动到更宽的原语不会丢失任何信息。

## 4。缩小原始转换

有时我们需要适应一个比变量声明中使用的类型更大的值。这可能导致信息丢失，因为一些字节将不得不被丢弃。

在这种情况下，我们必须通过使用演员来明确表达我们知道这种情况并且同意这种情况:

```
int myInt = (int) myDouble;
byte myByte = (byte) myInt;
```

## 5。扩大和缩小原语转换

当我们想要从`byte`转换到`char` 时，这种情况发生在**非常特殊的情况下。第一个转换是将`byte`加宽到`int`，然后从`int`变窄到`char`。**

一个例子将阐明这一点:

```
byte myLargeValueByte = (byte) 130;   //0b10000010 -126
```

130 的二进制表示与-126 相同，不同之处在于信号位的解释。现在让我们从`byte`转换到`char`:

```
char myLargeValueChar = (char) myLargeValueByte;
  //0b11111111 10000010 unsigned value
int myLargeValueInt = myLargeValueChar; //0b11111111 10000010 65410
```

`char`表示是一个 Unicode 值，但是转换成`int`向我们展示了一个非常大的值，它的低 8 位与-126 完全相同。

如果我们再次将其转换为`byte`,我们得到:

```
byte myOtherByte = (byte) myLargeValueInt; //0b10000010 -126
```

我们使用的原始值。如果整个代码都以`char`开始，那么值将会不同:

```
char myLargeValueChar2 = 130; //This is an int not a byte! 
  //0b 00000000 10000010 unsigned value

int myLargeValueInt2 = myLargeValueChar2; //0b00000000 10000010  130

byte myOtherByte2 = (byte) myLargeValueInt2; //0b10000010 -126
```

虽然`byte`表示是相同的，即-126，但是`char`表示给了我们两个不同的字符。

## 6。装箱/拆箱转换

在 Java 中，我们为每一个原始类型都提供了一个包装类，这是一种为程序员提供有用的处理方法的聪明方式，而没有将所有东西都作为重量级对象引用的开销。从 Java 1.5 开始，就包含了在原语和对象之间自动转换的能力，这是通过简单的属性实现的:

```
Integer myIntegerReference = myInt;
int myOtherInt = myIntegerReference;
```

## 7。字符串转换

所有的原始类型都可以通过它们的包装类转换成`String`，包装类覆盖了`toString()`方法:

```
String myString = myIntegerReference.toString();
```

如果我们需要回到原始类型，我们需要使用由相应的包装类定义的解析方法:

```
byte  myNewByte   = Byte.parseByte(myString);
short myNewShort  = Short.parseShort(myString);
int   myNewInt    = Integer.parseInt(myString);
long  myNewLong   = Long.parseLong(myString);

float  myNewFloat  = Float.parseFloat(myString);
double myNewDouble = Double.parseDouble(myString); 
```

```
boolean myNewBoolean = Boolean.parseBoolean(myString);
```

这里唯一的例外是`Character`类，因为无论如何`String`是由`char`构成的，这样，考虑到`String`可能是由单个`char`构成的，我们可以使用`String`类的`charAt()`方法:

```
char myNewChar = myString.charAt(0);
```

## 8。数字促销

要执行二元运算，两个操作数的大小必须兼容。

有一套适用的简单规则:

1.  如果其中一个操作数是`double`，则另一个被提升到`double`
2.  否则，如果操作数之一是`float`，则另一个被提升为`float`
3.  否则，如果操作数之一是`long`，则另一个被提升为`long`
4.  否则，两者都被视为`int`

让我们看一个例子:

```
byte op1 = 4;
byte op2 = 5;
byte myResultingByte = (byte) (op1 + op2);
```

两个操作数都被提升到`int`，结果必须再次向下转换到`byte`。

## 9。结论

类型之间的转换是日常编程活动中非常常见的任务。有一套规则来管理静态类型语言操作这些转换的方式。了解这个规则，在试图弄清楚某个代码为什么编译不成功的时候，可能会节省很多时间。

本文中使用的代码可以在 GitHub 上找到[。](https://web.archive.org/web/20220812060356/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-lang-syntax)