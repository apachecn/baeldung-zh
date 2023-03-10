# Java 基本语法介绍

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-syntax>

## 1。概述

Java 是一种静态类型的、面向对象的编程语言。它还是独立于平台的——Java 程序可以在一种类型的机器上编写和编译，如 Windows 系统，并在另一种机器上执行，如 MacOS，而无需对源代码进行任何修改。

在本教程中，我们将学习并理解 Java 语法的基础。

## 2。数据类型

Java 中有两大类数据类型:[原始类型和对象/引用类型](/web/20221001005349/https://www.baeldung.com/java-primitives-vs-objects)。

[**原语类型**](/web/20221001005349/https://www.baeldung.com/java-primitives) **是存储简单数据**的基本数据类型，是数据操作的基础。例如，Java 具有用于整数值(`int, long,` `byte, short` ) `, `、浮点值(`float``double`)`, `、字符值(`char`)和逻辑值(`boolean`)的原始类型。

另一方面，**引用类型是包含对值和/或其他对象**的引用的对象，或者是对特殊值`null`的引用，以表示没有值。

`String`类是引用类型的一个很好的例子。该类的一个实例称为对象，表示一系列字符，如“Hello World”。

## 3。在 Java 中声明变量

要在 Java 中声明一个变量，我们必须**指定它的名称(也称为标识符)并键入**。让我们看一个简单的例子:

```java
int a;
int b;
double c;
```

在上面的例子中，变量将基于它们声明的类型接收缺省的初始值。由于我们将变量声明为`int`和`double`，它们的默认值分别为 0 和 0.0。

或者，**我们可以在声明期间使用赋值操作符(=)来初始化变量:**

```java
int a = 10;
```

在上面的例子中，我们将带有**标识符** `**a** `的变量声明为**类型`int`** 并且**使用赋值操作符(=)将值 10** 赋给它**，并用分号(；)**。在 Java 中，所有的语句都必须以分号结束。

**标识符是任意长度的名称，由字母、数字、下划线和美元符号组成，**符合以下规则:

*   以字母、下划线(_)或美元符号($)开头
*   不能是保留关键字
*   不能是`true`、`false,`或者`null`

让我们扩展上面的代码片段，加入一个简单的算术运算:

```java
int a = 10;
int b = 5;
double c = a + b;
System.out.println( a + " + " + b + " = " + c);
```

我们可以将上面代码片段的前三行读作**“将值 10 赋给`a,`将值 5 赋给`b, `将`a `和`b`的值求和并将结果赋给** **c"** 。在最后一行，我们将操作的结果输出到控制台:

```java
10 + 5 = 15.0
```

其他类型变量的声明和初始化遵循我们上面展示的相同语法。例如，让我们声明`String`、`char`和`boolean`变量:

```java
String name = "Baeldung Blog";
char toggler = 'Y';
boolean isVerified = true;
```

为了强调起见**，表示文字值`char `和`String`的主要区别在于值**周围的引号数量。因此，`‘a'`是一个`char`，而`“a”`是一个`String.`

## 4。数组

数组是一种引用类型，可以存储特定类型的值的集合。在 Java 中声明数组的一般语法是:

`**type[] identifier = new type[length];**`

该类型可以是任何基元或引用类型。

例如，让我们看看如何声明一个最多可以容纳 100 个整数的数组:

```java
int[] numbers = new int[100];
```

为了引用数组中的特定元素，或者给元素赋值，我们使用变量名及其索引:

```java
numbers[0] = 1;
numbers[1] = 2;
numbers[2] = 3;
int thirdElement = numbers[2];
```

在 Java 中，**数组索引从零开始。**数组的第一个元素在索引 0 处，第二个元素在索引 1 处，依此类推。

另外，我们可以通过调用`numbers.length`来获得数组的长度:

```java
int lengthOfNumbersArray = numbers.length;
```

## 5。Java 关键词

**关键字是在 Java 中有特殊含义的保留字。**

例如，`public, static, class, main, new, instanceof`，是 Java 中的[关键字，同样，**我们不能把它们作为标识符(变量名)**。](/web/20221001005349/https://www.baeldung.com/tag/java-keyword/)

## 6。Java 中的运算符

现在我们已经看到了上面的赋值操作符(=)，让我们探索一下 Java 语言中其他类型的[操作符:](https://web.archive.org/web/20221001005349/https://docs.oracle.com/javase/tutorial/java/nutsandbolts/opsummary.html)

### 6.1。算术运算符

Java 支持以下[算术运算符](https://web.archive.org/web/20221001005349/https://docs.oracle.com/javase/tutorial/java/nutsandbolts/op1.html)，可用于编写数学、计算逻辑:

*   +(加号或加法；也用于字符串连接)
*   –(减或减)
*   *(乘法)
*   /(除法)
*   %(模数或余数)

我们在前面的代码示例中使用了加号(+)操作符来执行两个变量的加法。其他算术运算符的用法类似。

加号(+)的另一个用途是连接字符串以形成一个全新的字符串:

```java
String output =  a + " + " + b + " = " + c;
```

### 6.2。逻辑运算符

除了算术运算符，Java 还支持以下用于计算布尔表达式的[逻辑运算符](https://web.archive.org/web/20221001005349/https://docs.oracle.com/javase/tutorial/java/nutsandbolts/op2.html):

*   &&(和)
*   ||(或)
*   ！(不是)

让我们考虑下面的代码片段，它演示了逻辑 AND 和 OR 操作符。第一个示例显示了当`number`变量可被 2 和 3 整除时执行的打印语句:

```java
int number = 6;

if (number % 2 == 0 && number % 3 == 0) {
    System.out.println(number + " is divisible by 2 AND 3");
}
```

而当`number`被 2 或 5 整除时执行第二个:

```java
if (number % 2 == 0 || number % 5 == 0) {
    System.out.println(number + " is divisible by 2 OR 5");
}
```

### 6.3。比较运算符

当我们需要比较一个变量的值和另一个变量的值时，我们可以使用 Java 的[比较运算符](https://web.archive.org/web/20221001005349/https://docs.oracle.com/javase/tutorial/java/nutsandbolts/op2.html):

*   `<`(小于)
*   < =(小于或等于)
*   >(大于)
*   > =(大于或等于)
*   ==(等于)
*   ！=(不等于)

例如，我们可以使用比较运算符来确定投票人的资格:

```java
public boolean canVote(int age) {
    if(age < 18) {
        return false;
    }
    return true;
}
```

## 7。Java 程序结构

既然我们已经学习了数据类型、变量和一些基本操作符，那么让我们看看如何将这些元素放在一个简单的可执行程序中。

**Java 程序的基本单位是一个`Class`。**一个`Class `可以有一个或多个字段(有时称为属性)`, `方法，甚至其他的类成员称为内部类。

**要使`Class`可执行，它必须有一个`main`方法。**`main`方法表示程序的入口点。

让我们编写一个简单的、可执行的`Class`来练习我们之前考虑过的一个代码片段:

```java
public class SimpleAddition {

    public static void main(String[] args) {
        int a = 10;
        int b = 5;
        double c = a + b;
        System.out.println( a + " + " + b + " = " + c);
    }
}
```

这个类的名字是`SimpleAddition`，在它的内部，我们有一个`main`方法来存放我们的逻辑。**在左花括号和右花括号之间的代码段被称为代码块。**

Java 程序的源代码存储在扩展名为`.java`的文件中。

## 8。编译和执行程序

为了执行我们的源代码，我们首先需要编译它。该过程将生成一个扩展名为`the .class`的二进制文件。我们可以在任何安装了 Java 运行时环境(JRE)的机器上执行二进制文件。

让我们将上面示例中的源代码保存到一个名为`SimpleAddition.java`的文件中，并从保存该文件的目录中运行该命令:

```java
javac SimpleAddition.java
```

要执行程序，我们只需运行:

```java
java SimpleAddition
```

这将向控制台产生与上面所示相同的输出:

```java
10 + 5 = 15.0
```

## 9。结论

在本教程中，我们已经了解了 Java 的一些基本语法。就像任何其他编程语言一样，通过不断的练习，它变得越来越简单。

本教程的完整源代码可以在 Github 上找到。