# Java 按位运算符

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-bitwise-operators>

## 1.概观

[运算符](https://web.archive.org/web/20220820082115/https://docs.oracle.com/javase/tutorial/java/nutsandbolts/opsummary.html)在 Java 语言中用于对数据和变量进行操作。

在本教程中，我们将探索按位运算符以及它们在 Java 中的工作方式。

## 2.按位运算符

**按位运算符对输入值的[二进制数字](https://web.archive.org/web/20220820082115/https://medium.com/coderscorner/number-systems-decimal-binary-octal-and-hexadecimal-5e567e55ab28)或位进行运算。**我们可以将这些应用于整数类型—`long, int, short, char,`和`byte.`

在探索不同的位运算符之前，让我们先了解它们是如何工作的。

**按位运算符处理十进制数的二进制等效值，并按照给定的运算符对它们逐位执行运算:**

*   首先，操作数被转换成二进制表示
*   接下来，将运算符应用于每个二进制数，并计算结果
*   最后，结果被转换回十进制表示

我们用一个例子来理解；让我们来看两个`integers:`

```java
int value1 = 6;
int value2 = 5;
```

接下来，让我们对这些数字应用按位 OR 运算符:

```java
int result = 6 | 5;
```

要执行此操作，首先，将计算这些数字的二进制表示:

```java
Binary number of value1 = 0110
Binary number of value2 = 0101
```

那么该操作将被应用于每一位。结果返回一个新的二进制数:

```java
0110
0101
-----
0111
```

最后，结果`0111 `将被转换回等于`7`的十进制:

```java
result : 7
```

按位运算符进一步分为按位逻辑运算符和按位移位运算符。现在让我们来看看每一种类型。

## 3.按位逻辑运算符

按位逻辑运算符是 AND(&)、OR(|)、XOR(^)和 NOT(~)。

### 3.1.按位`OR` (|)

OR 运算符比较两个整数的每个二进制数字，如果其中一个为 1，则返回 1。

这类似于用于布尔值的||逻辑运算符。当比较两个布尔值时，如果其中一个为`true.`，结果为`true`。类似地，当其中一个为 1 时，输出为 1。

在上一节中，我们看到了该运算符的一个示例:

```java
@Test
public void givenTwoIntegers_whenOrOperator_thenNewDecimalNumber() {
    int value1 = 6;
    int value2 = 5;
    int result = value1 | value2;
    assertEquals(7, result);
}
```

让我们看看这个操作的二进制表示:

```java
0110
0101
-----
0111
```

在这里，我们可以看到，使用 OR，0 和 0 将导致 0，而任何至少包含 1 的组合将导致 1。

### 3.2.按位`AND` ( &)

**AND 运算符比较两个整数的每个二进制数字，如果都是 1，则返回 1，否则返回 0。**

这类似于带有`boolean`值的&&运算符。当两个`booleans`的值为`true`时，一个&操作的结果为`true.`

让我们使用与上面相同的例子，除了现在使用&操作符代替|操作符:

```java
@Test
public void givenTwoIntegers_whenAndOperator_thenNewDecimalNumber() {
    int value1 = 6;
    int value2 = 5;
    int result = value1 & value2;
    assertEquals(4, result);
}
```

让我们看看这个操作的二进制表示:

```java
0110
0101
-----
0100
```

`0100`是`4`的小数，因此，结果是:

```java
result : 4
```

### 3.3.按位异或(^)

XOR 运算符比较两个整数的每个二进制数字，如果两个被比较的位不同，则返回 1。这意味着如果两个整数的位都是 1 或 0，结果将是 0；否则，结果将是 1:

```java
@Test
public void givenTwoIntegers_whenXorOperator_thenNewDecimalNumber() {
    int value1 = 6;
    int value2 = 5;
    int result = value1 ^ value2;
    assertEquals(3, result);
}
```

二进制表示:

```java
0110
0101
-----
0011
```

*0011* 是十进制的 3，因此，结果是:

```java
result : 3
```

### 3.4.按位补码(~)

**按位非或补码运算符简单地表示输入值的每一位的反运算。它只需要一个整数，就相当于！接线员。**

这个运算符改变整数的每个二进制数字，这意味着全 0 变成 1，全 1 变成 0。的！运算符对`boolean`值的工作方式类似:它将`boolean`值从`true`反转为`false`，反之亦然。

现在我们用一个例子来理解如何求一个十进制数的补数。

让我们做 value1 = 6 的补码:

```java
@Test
public void givenOneInteger_whenNotOperator_thenNewDecimalNumber() {
    int value1 = 6;
    int result = ~value1;
    assertEquals(-7, result);
}
```

二进制值为:

```java
value1 = 0000 0110
```

通过应用补码运算符，结果将是:

```java
0000 0110 -> 1111 1001
```

这是十进制数 6 的补码。由于第一个(最左边的)二进制位是 1，这意味着所存储的数字的符号是负的。

现在，由于数字存储为二进制补码，首先我们需要找到它的二进制补码，然后将结果二进制数转换为十进制数:

```java
1111 1001 -> 0000 0110 + 1 -> 0000 0111
```

最后，0000 0111 是十进制的 7。如上所述，由于符号位是 1，因此得到的答案是:

```java
result : -7
```

### 3.5.按位运算符表

让我们在一个比较表中总结一下到目前为止我们看到的操作符的结果:

```java
A	B	A|B	A&B;	A^B	~A
0	0	0	0	0	1
1	0	1	0	1	0
0	1	1	0	1	1
1	1	1	1	0	0
```

## 4.按位移位运算符

**二进制移位运算符根据移位运算符将输入值的所有位向左或向右移位。**

让我们看看这些运算符的语法:

```java
value <operator> <number_of_times>
```

表达式的左边是被移位的整数，表达式的右边表示它必须被移位的次数。

按位移位运算符进一步分为按位左移和按位右移运算符。

### 4.1.带符号左移[<

**左移运算符将位向左移动操作数右侧指定的次数。左移后，右边的空白用 0 填充。**

另一个需要注意的要点是，**将一个数移动 1 相当于将它乘以 2，或者，一般来说，将一个数向左移动`n`个位置相当于乘以 2^ `n`。**

让我们把值 12 作为输入值。

现在，我们将它向左移动两个位置(12 <<2)，看看最终的结果是什么。

12 的二进制等效值是 00001100。左移 2 位后，结果是 00110000，相当于十进制的 48:

```java
@Test
public void givenOnePositiveInteger_whenLeftShiftOperator_thenNewDecimalNumber() {
    int value = 12;
    int leftShift = value << 2;
    assertEquals(48, leftShift);
} 
```

对于负值，情况类似:

```java
@Test
public void givenOneNegativeInteger_whenLeftShiftOperator_thenNewDecimalNumber() {
    int value = -12;
    int leftShift = value << 2;
    assertEquals(-48, leftShift);
}
```

### 4.2.有符号右移位[>>]

**右移位运算符将所有位向右移位。**左侧的空白处根据输入的数字填充:

*   **当输入数字为负数时，其中最左边的位是 1，那么空白空间将被填充 1**
*   **当一个输入数为正数时，最左边的位为 0，那么空位将被填充为 0**

让我们继续使用 12 作为输入的例子。

现在，我们将它向右移动 2 个位置(12 >>2)，看看最终的结果会是什么。

输入的数字是正数，所以右移 2 位后，结果是 0011，十进制是 3:

```java
@Test
public void givenOnePositiveInteger_whenSignedRightShiftOperator_thenNewDecimalNumber() {
    int value = 12;
    int rightShift = value >> 2;
    assertEquals(3, rightShift);
}
```

此外，对于负值:

```java
@Test
public void givenOneNegativeInteger_whenSignedRightShiftOperator_thenNewDecimalNumber() {
    int value = -12;
    int rightShift = value >> 2;
    assertEquals(-3, rightShift);
}
```

### 4.3.无符号右移位[>>>]

该运算符非常类似于有符号右移位运算符。唯一的区别是，不管数字是正数还是负数，左边的空格都用 0 填充。因此，结果永远是正整数。

让我们右移 12 的相同值:

```java
@Test
public void givenOnePositiveInteger_whenUnsignedRightShiftOperator_thenNewDecimalNumber() {
    int value = 12;
    int unsignedRightShift = value >>> 2;
    assertEquals(3, unsignedRightShift);
}
```

现在，负值:

```java
@Test
public void givenOneNegativeInteger_whenUnsignedRightShiftOperator_thenNewDecimalNumber() {
    int value = -12;
    int unsignedRightShift = value >>> 2;
    assertEquals(1073741821, unsignedRightShift);
}
```

## 5.按位运算符和逻辑运算符的区别

我们在这里讨论的按位运算符和更常见的逻辑运算符有一些不同。

首先，**逻辑运算符处理`boolean`表达式**并返回`boolean`值(要么是`true`要么是`false),`，而**位运算符处理整数值的二进制数字**(`long, int, short, char,`和`byte`)并返回一个整数。

此外，逻辑运算符总是计算第一个`boolean`表达式，根据其结果和使用的运算符，可能会也可能不会计算第二个。另一方面，**位运算符总是对两个操作数**求值。

最后，逻辑运算符用于根据多个条件做出决策，而按位运算符处理位并执行逐位运算。

## 6。用例

按位运算符的一些潜在使用情况如下:

*   通信堆栈，其中附加到数据的报头中的各个位表示重要信息
*   在嵌入式系统中，只设置/清除/切换特定寄存器的一位，而不修改其余位
*   为了安全起见，使用 XOR 运算符加密数据
*   在数据压缩中，通过将数据从一种表示法转换成另一种表示法来减少所占用的空间

## 7.结论

在本教程中，我们学习了按位运算符的类型以及它们与逻辑运算符的不同之处。我们也看到了它们的一些潜在用例。

GitHub 上的[提供了本文中的所有代码示例。](https://web.archive.org/web/20220820082115/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-lang-operators)