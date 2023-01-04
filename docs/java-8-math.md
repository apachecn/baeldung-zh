# Java 8 数学新方法

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-8-math>

## 1。简介

通常，当我们想到 Java 第 8 版的新特性时，首先想到的是函数式编程和 lambda 表达式。

然而，除了这些大的特性之外，还有其他一些特性，可能影响较小，但也很有趣，而且很多时候并不为人所知，甚至没有被任何评论所涵盖。

在本教程中，我们将列举并给出一个添加到语言核心类之一的新方法的小例子:`[java.lang.Math](https://web.archive.org/web/20221208143919/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/lang/Math.html)`。

## 2。新`*exact()`方法

首先，我们有一组新方法，它们扩展了一些现有的和最常见的算术运算。

正如我们将看到的，它们是不言而喻的，因为它们与派生它们的方法具有完全相同的功能，但是增加了在结果值溢出它们类型的最大值或最小值时抛出异常的功能。

我们可以将`integers`和`longs`作为参数使用这些方法。

### 2.1。`addExact()`

添加了两个参数，**在溢出时抛出`ArithmeticException`(这适用于加法的所有`*Exact()`方法)**:

```java
Math.addExact(100, 50);               // returns 150
Math.addExact(Integer.MAX_VALUE, 1);  // throws ArithmeticException
```

### 2.2。`substractExact()`

从第一个参数中减去第二个参数的值，在减法溢出的情况下抛出`ArithmeticException`:

```java
Math.subtractExact(100, 50);           // returns 50
Math.subtractExact(Long.MIN_VALUE, 1); // throws ArithmeticException
```

### 2.3。`incrementExact()`

将参数增加 1，在溢出的情况下抛出`ArithmeticException`:

```java
Math.incrementExact(100);               // returns 101
Math.incrementExact(Integer.MAX_VALUE); // throws ArithmeticException
```

### 2.4。`decrementExact()`

将参数减 1，在溢出的情况下抛出`ArithmeticException`:

```java
Math.decrementExact(100);            // returns 99
Math.decrementExact(Long.MIN_VALUE); // throws ArithmeticException
```

### 2.5。`multiplyExact()`

将两个参数相乘，如果乘积溢出，抛出`ArithmeticException`:

```java
Math.multiplyExact(100, 5);            // returns 500
Math.multiplyExact(Long.MAX_VALUE, 2); // throws ArithmeticException
```

### 2.6。`negateExact()`

改变参数的符号，在溢出的情况下抛出一个`ArithmeticException`。

在这种情况下，我们必须考虑内存中值的内部表示，以理解为什么会出现溢出，因为这不像其他“精确”方法那样直观:

```java
Math.negateExact(100);               // returns -100
Math.negateExact(Integer.MIN_VALUE); // throws ArithmeticException
```

第二个例子需要一个解释，因为它并不明显:**溢出是因为`Integer.MIN_VALUE`是 2.147.483.648，另一方面`Integer.MAX_VALUE`是 2.147.483.647** ，所以返回值不适合一个单位的`Integer`。

## 3。其他方法

### 3.1。`floorDiv()`

将第一个参数除以第二个参数，然后对结果执行`floor()`运算，返回小于或等于商的`Integer`:

```java
Math.floorDiv(7, 2));  // returns 3 
```

精确的商是 3.5 所以`floor(3.5)` == 3。

让我们看另一个例子:

```java
Math.floorDiv(-7, 2)); // returns -4 
```

精确的商是-3.5 所以`floor(-3.5)` == -4。

### 3.2。`modDiv()`

这种方法与前面的方法`floorDiv()`相似，但是对除法的模或余数而不是商应用了`floor()`运算:

```java
Math.modDiv(5, 3));  // returns 2 
```

我们可以看到，两个正数的 **`modDiv()`与%运算符**相同。让我们看一个不同的例子:

```java
Math.modDiv(-5, 3));  // returns 1 
```

它返回 1 而不是 2，因为`floorDiv(-5, 3)`是-2 而不是-1。

### 3.3。`nextDown()`

返回参数的下一个值(支持`float`或`double`参数):

```java
float f = Math.nextDown(3);  // returns 2.9999998
double d = Math.nextDown(3); // returns 2.999999761581421
```

## 4。结论

在本文中，我们已经简要描述了 Java 平台版本 8 中添加到类 **`java.lang.Math`** 中的所有新方法的功能，并且还看到了一些如何使用它们的示例。

和往常一样，完整的源代码可以在 GitHub 上找到[。](https://web.archive.org/web/20221208143919/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-lang-math)