# Java 中递增和递减一元运算符指南

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-unary-operators>

## 1.概观

在本教程中，我们将简要讨论 Java 中的递增和递减一元运算符。

我们将从语法开始，然后是用法。

## 2.Java 中的递增和递减操作

在 Java 中，递增一元运算符**将变量的值**增加 1，而递减一元运算符**将变量的值**减少 1。

两个**都将操作数的值**更新为新值。

所需的操作数应该是一个不恒定的变量，因为我们不能修改它的值。此外，操作数不能是表达式，因为我们不能更新它们。

递增和递减一元运算符有两种形式，即前缀和后缀。

## 3.预递增和预递减一元运算符

在前缀形式中，递增和递减一元运算符出现在操作数之前。

使用前缀形式时，我们首先更新操作数的值，然后在表达式中使用新值。

首先，让我们来看一个使用前递增一元运算符的代码片段:

```java
int operand = 1;
++operand; // operand = 2
int number = ++operand; // operand = 3, number = 3
```

接下来，让我们看看使用预减量的代码片段:

```java
int operand = 2;
--operand; // operand = 1
int number = --operand; // operand = 0, number = 0
```

正如我们看到的，前缀运算符首先改变操作数的值，然后计算表达式的其余部分。如果嵌入到复杂的表达式中，这很容易导致混淆。建议**我们在它们自己的行**上使用它们，而不是在更大的表达式中。

## 4.后递增和后递减一元运算符

在后缀形式中，运算符出现在操作数之后。

**在使用后缀形式时，我们首先使用表达式中操作数的值**，然后更新它。

让我们来看一个使用后递增运算符的示例代码片段:

```java
int operand = 1;
operand++; // operand = 2
int number = operand++; // operand = 3, number = 2
```

另外，我们来看看后减量的那个:

```java
int operand = 2;
operand--; //operand = 1
int number = operand--; // operand = 0, number 1
```

类似地，后递增和后递减一元运算符应该在它们自己的行上，而不是包含在更大的表达式中。

## 5.结论

在这个快速教程中，我们学习了 Java 中的递增和递减一元运算符。此外，我们看了它们的两种形式:前缀和后缀。最后，我们看了它的语法和示例代码片段。

我们这里例子的完整源代码一如既往地在 GitHub 的[上。](https://web.archive.org/web/20220912083422/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-lang-operators)