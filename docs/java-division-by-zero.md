# Java 中被零除:异常、无穷大或非数字

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-division-by-zero>

## 1.概观

被零除是一个在普通算术中没有意义的运算，因此是未定义的。然而，在编程中，虽然 it **经常与错误相关联，但情况并不总是如此**。

在这篇文章中，我们将讨论当 Java 程序中出现被零除的情况时会发生什么。

根据除法运算的 [Java 规范，](https://web.archive.org/web/20220712165834/https://docs.oracle.com/javase/specs/jls/se14/html/jls-15.html#jls-15.17.2)我们可以识别两种不同的被零除的情况:整数和浮点数。

## 2.整数

首先，对于整数，事情非常简单。**整数除以零将得到`ArithmeticException` :**

```java
assertThrows(ArithmeticException.class, () -> {
    int result = 12 / 0;
});
```

```java
assertThrows(ArithmeticException.class, () -> {
    int result = 0 / 0;
});
```

## 3.浮点类型

然而，当用浮点数`,` 处理**时，不会抛出异常**:

```java
assertDoesNotThrow(() -> {
    float result = 12f / 0;
});
```

为了处理这样的情况，Java 使用了一些特殊的数值来表示这种运算的结果:`NaN`、`POSITIVE_INFINITY`和`NEGATIVE_INFINITY.`

### 3.1.圆盘烤饼

让我们从用零除浮点零值开始**:**

```java
assertEquals(Float.NaN, 0f / 0);
assertEquals(Double.NaN, 0d / 0);
```

这些情况下的结果是 [`NaN`](/web/20220712165834/https://www.baeldung.com/java-not-a-number) (不是数字)。

### 3.2.无穷

接下来，让我们**将一些非零值除以零**:

```java
assertEquals(Float.POSITIVE_INFINITY, 12f / 0);
assertEquals(Double.POSITIVE_INFINITY, 12d / 0);
assertEquals(Float.NEGATIVE_INFINITY, -12f / 0);
assertEquals(Double.NEGATIVE_INFINITY, -12d / 0);
```

正如我们所见，结果是`INFINITY,`，其符号取决于操作数的符号。

此外，我们还可以使用负零的概念来得到`NEGATIVE_INFINITY`:

```java
assertEquals(Float.NEGATIVE_INFINITY, 12f / -0f);
assertEquals(Double.NEGATIVE_INFINITY, 12f / -0f);
```

### 3.3.记忆表征

那么，为什么整数被零除会抛出异常，而浮点被零除不会呢？

让我们从内存表示的角度来看这个问题。**对于整数，没有可以用来存储这种运算结果**的位模式，而**浮点数有类似`NaN`或`INFINITY` 的值，可以用在这种情况下。**

现在，让我们把浮点数的二进制表示看作是Seeeeee EFFFFFFF FFFFFFFF FFFFFFFF，其中一位(S)表示符号，8 位(E)表示指数，其余(F)表示尾数。

在三个值`NaN`、`POSITIVE_INFINITY,`和 NEGATIVE_INFINITY、**的每一个中，指数部分中的所有位都被设置为 1。**

`INFINITY`具有全部设置为 0 的尾数位，而`NaN`具有非零尾数:

```java
assertEquals(Float.POSITIVE_INFINITY, Float.intBitsToFloat(0b01111111100000000000000000000000));
assertEquals(Float.NEGATIVE_INFINITY, Float.intBitsToFloat(0b11111111100000000000000000000000));
assertEquals(Float.NaN, Float.intBitsToFloat(0b11111111100000010000000000000000));
assertEquals(Float.NaN, Float.intBitsToFloat(0b11111111100000011000000000100000));
```

## 4.摘要

总而言之，在本文中，我们看到了被零除在 Java 中是如何工作的。

像 **`INFINITY`和`NaN`这样的值适用于浮点数，但不适用于整数**。因此，将整数除以零将导致异常。但是，对于一个`float`或`double`，Java 允许操作。

完整的代码可以在 GitHub 的[上找到。](https://web.archive.org/web/20220712165834/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-numbers-3)**