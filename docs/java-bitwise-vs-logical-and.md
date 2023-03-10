# 按位& vs 逻辑&运算符

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-bitwise-vs-logical-and>

## 1.介绍

在 Java 中，我们有两种方式来表达“和”。但是用哪个呢？

在本教程中，我们将看看&和&&之间的区别。同时，我们将学习按位运算和短路。

## 2.使用按位`AND`

**按位 AND ( &)运算符比较两个整数的每个二进制数字，如果两者都是 1，则返回 1，否则返回 0。**

让我们来看看两个整数:

```java
int six = 6;
int five = 5;
```

接下来，让我们对这些数字应用按位 AND 运算符:

```java
int resultShouldBeFour = six & five;
assertEquals(4, resultShouldBeFour);
```

为了理解这个操作，让我们来看看每个数字的二进制表示:

```java
Binary of decimal 4: 0100
Binary of decimal 5: 0101
Binary of decimal 6: 0110
```

&运算符对每个位执行逻辑 AND 运算，并返回一个新的二进制数:

```java
0110
0101
-----
0100
```

最后，我们的结果—`0100 –` 可以转换回十进制数—`4`。

让我们看看测试 Java 代码:

```java
int six = 6;
int five = 5;
int resultShouldBeFour = six & five;
assertEquals(4, resultShouldBeFour);
```

### 2.1.将&与布尔值一起使用

此外，我们可以对操作数使用按位 AND ( `&`)运算符。**只有当两个操作数都是`true`时才返回`true`，否则返回`false.`**

让我们来看三个`boolean`变量:

```java
boolean trueBool = true;
boolean anotherTrueBool = true;
boolean falseBool = false;
```

接下来，让我们对变量`trueBool`和`anotherTrueBool`应用按位 AND 运算符:

```java
boolean trueANDtrue = trueBool & anotherTrueBool;
```

然后，结果会是`true`。

接下来，让我们对`trueBool`和`falseBool`应用按位 AND 运算符:

```java
boolean trueANDFalse = trueBool & falseBool;
```

在这种情况下，结果将是`false`。

让我们看看测试 Java 代码:

```java
boolean trueBool = true;
boolean anotherTrueBool = true;
boolean falseBool = false;

boolean trueANDtrue= trueBool & anotherTrueBool;
boolean trueANDFalse = trueBool & falseBool;

assertTrue(trueANDtrue);
assertFalse(trueANDFalse);
```

## 3.使用逻辑`AND`

**和`&`一样，逻辑 AND ( `&&`)运算符比较两个布尔变量或表达式的值。**并且，只有当两个操作数都是`true`时，它也返回`true`，否则，它返回`false`。

让我们来看三个`boolean`变量:

```java
boolean trueBool = true;
boolean anotherTrueBool = true;
boolean falseBool = false;
```

接下来，让我们对变量`trueBool`和`anotherTrueBool`应用逻辑 AND 运算符:

```java
boolean trueANDtrue = trueBool && anotherTrueBool;
```

然后，结果会是`true`。

接下来，让我们对`trueBool`和`falseBool`应用逻辑 AND 运算符:

```java
boolean trueANDFalse = trueBool && falseBool;
```

在这种情况下，结果将是`false`。

让我们看看测试 Java 代码:

```java
boolean trueBool = true;
boolean anotherTrueBool = true;
boolean falseBool = false;
boolean anotherFalseBool = false;

boolean trueANDtrue = trueBool && anotherTrueBool;
boolean trueANDFalse = trueBool && falseBool;
boolean falseANDFalse = falseBool && anotherFalseBool;

assertTrue(trueANDtrue);
assertFalse(trueANDFalse);
assertFalse(falseANDFalse);
```

### 3.1.短路

那么，有什么区别呢？嗯，**`&&`操作员短路了。**这意味着当左边的操作数或表达式为`false`时，它不计算右边的操作数或表达式。

让我们假设两个表达式的值为假:

```java
First Expression: 2<1
Second Expression: 4<5 
```

当我们对表达式`2<1` 和 `4<5,` 应用逻辑 AND 运算符时，它只计算第一个表达式`2<1`并返回 `false.`

```java
boolean shortCircuitResult = (2<1) && (4<5);
assertFalse(shortCircuitResult);
```

### 3.2.对整数使用&&

我们可以对布尔或数字类型使用&运算符，但&&只能用于布尔操作数。对整数操作数使用它会导致编译错误:

```java
int five = 2;
int six = 4;
int result = five && six;
```

## 4.比较

1.  &运算符总是计算两个表达式，而&&运算符仅在第一个表达式为`true`时才计算第二个表达式
2.  &按位比较每个操作数，而&&只对布尔值进行运算

## 5.结论

在本文中，我们使用了按位运算符`&`来比较两个数字的位，从而得到一个新的数字。此外，我们使用逻辑操作符`&&`来比较两个布尔值，得到一个布尔值。

我们还看到了两个运营商之间的一些关键差异。

和往常一样，你可以在 GitHub 上找到本教程的代码[。](https://web.archive.org/web/20221207022317/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-lang-operators)