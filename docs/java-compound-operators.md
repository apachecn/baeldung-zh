# Java 复合运算符

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-compound-operators>

## 1。概述

在本教程中，我们将看看 Java 复合运算符、它们的类型以及 Java 如何对它们求值。

我们还将解释隐式转换是如何工作的。

## 2。复合赋值运算符

赋值运算符是一种二元运算符，它将右边的结果赋给左边的变量。最简单的是`“=”`赋值操作符:

```java
int x = 5;
```

该语句声明一个新变量`x`，给`x`赋值`5`的值，并返回`5`。

复合赋值运算符是应用算术或位运算并将运算值赋给左侧变量的一种较短方式。

例如，以下两个乘法语句是等价的，这意味着`a`和`b`将具有相同的值:

```java
int a = 3, b = 3, c = -2;
a = a * c; // Simple assignment operator
b *= c; // Compound assignment operator
```

需要注意的是，复合赋值操作符左边的变量必须已经声明。换句话说，**复合运算符不能用来声明一个新变量。**

像“=”赋值运算符一样，复合运算符返回表达式的赋值结果:

```java
long x = 1;
long y = (x+=2);
```

`x`和`y`都将保存值`3`。

赋值`(x+=2)`做两件事:首先，它给变量`x`的值加 2，这个值就变成了`3;`其次，它返回赋值的值，这个值也是`3`。

## 3。复合赋值运算符的类型

Java 支持 11 种复合赋值运算符。我们可以将这些分为算术和按位运算符。

让我们看看算术运算符及其执行的运算:

*   增量:`+=`
*   递减:`-=`
*   乘法:`*=`
*   分部:`/=`
*   模数:`%=`

然后，我们还有按位运算符:

*   并且，二进制:`&=`
*   异或，二进制:`^=`
*   包含或，二进制:`|=`
*   左移，二进制:`<<=`
*   右移，二进制:`>>=`
*   右移补零:`>>>=`

让我们来看看这些操作的几个例子:

```java
// Simple assignment
int x = 5; // x is 5

// Incrementation
x += 5; // x is 10

// Decrementation
x -= 2; // x is 8

// Multiplication
x *= 2; // x is 16

// Modulus
x %= 3; // x is 1

// Binary AND
x &= 4; // x is 0

// Binary exclusive OR
x ^= 4; // x is 4

// Binary inclusive OR
x |= 8; // x is 12
```

正如我们在这里看到的，使用这些操作符的语法是一致的。

## 4。复合赋值运算的求值

Java 有两种方法来计算复合运算。

首先，**当左边的操作数不是数组时，**那么 Java 会，按顺序:

1.  验证操作数是已声明的变量
2.  保存左侧操作数的值
3.  计算右边的操作数
4.  按照复合运算符的指示执行二元运算
5.  将二元运算的结果转换为左侧变量的类型(隐式转换)
6.  将转换后的结果赋给左边的变量

接下来，**当左边的操作数是一个数组，**接下来的步骤有点不同:

1.  验证左边的数组表达式，如果不正确，抛出一个`NullPointerException`或`ArrayIndexOutOfBoundsException`
2.  将数组元素保存在索引中
3.  计算右边的操作数
4.  检查选定的数组组件是基元类型还是引用类型，然后继续执行与第一个列表相同的步骤，就好像左边的操作数是变量一样。

如果评估的任何一步失败，Java 不会继续执行下面的步骤。

**让我们举一些与对一个数组元素进行这些运算的求值相关的例子:**

```java
int[] numbers = null;

// Trying Incrementation
numbers[2] += 5;
```

正如我们所料，这将抛出一个`NullPointerException`。

但是，如果我们给数组赋值一个初始值:

```java
int[] numbers = {0, 1};

// Trying Incrementation
numbers[2] += 5;
```

我们会去掉`NullPointerException,`，但我们仍然会得到一个`ArrayIndexOutOfBoundsException`，因为使用的索引不正确。

如果我们解决了这个问题，操作将会成功完成:

```java
int[] numbers = {0, 1};

// Incrementation
numbers[1] += 5; // x is now 6
```

最后，`x`变量在赋值结束时将是`6`。

## 5。隐式转换

复合运算符有用的原因之一是，它们不仅提供了一种更短的运算方式，而且还可以隐式转换变量。

形式上，复合赋值表达式的形式为:

`E1 op= E2`

相当于:

`E1 – (T)(E1 op E2)`

其中`T`是`E1`的类型。

让我们考虑下面的例子:

```java
long number = 10;
int i = number;
i = i * number; // Does not compile
```

让我们回顾一下为什么最后一行不会编译。

当它们在一个操作中在一起时，Java 自动将较小的数据类型提升为较大的数据类型，但是当试图从较大的类型转换为较小的类型时**会抛出一个错误。**

因此，首先，`i`将被提升到`long`，然后乘法将给出结果`10L.`，长结果将被赋值给`i`，这是一个`int`，这将抛出一个错误。

这可以通过显式强制转换来解决:

```java
i = (int) i * number;
```

**在这种情况下，Java 复合赋值操作符是完美的，因为它们进行隐式转换:**

```java
i *= number;
```

这个语句工作得很好，将乘法结果转换为`int`，并将值赋给左边的变量`i`。

## 6。结论

在本文中，我们研究了 Java 中的复合运算符，给出了一些例子和不同类型的复合运算符。我们解释了 Java 如何评估这些操作。

最后，我们还回顾了隐式转换，这是这些速记操作符有用的原因之一。

和往常一样，本文中提到的所有代码片段都可以在我们的 [GitHub 资源库](https://web.archive.org/web/20220625164519/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-lang-operators)中找到。