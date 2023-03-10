# 用 Java 打印二进制格式的整数

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-print-integer-binary>

## 1.概观

在本教程中，我们将简要介绍在 Java 中以二进制格式打印整数的不同方法。

首先，我们来看一下概念。然后，我们将学习一些用于转换的内置 Java 函数。

## 2.使用整数到二进制转换

在这一节中，我们将编写自定义方法，用 Java 将整数转换成二进制格式的字符串。在写代码之前，我们先来了解一下如何将整数转换成二进制格式。

要将整数`n`转换成二进制格式，我们需要:

1.  存储数字`n`被 2 除时的余数，并用商的值更新数字`n`
2.  重复步骤 1，直到数字 n 大于零
3.  最后，以相反的顺序打印余数

让我们看一个将 7 转换成等效二进制格式的例子:

1.  首先，将 7 除以 2:余数 1，商 3
2.  第二，用 2 除 3:余数 1，商 1
3.  然后，用 1 除以 2:余数 1，商 0
4.  最后，以相反的顺序打印余数，因为上一步的商是 0: 111

接下来，让我们实现上面的算法:

```java
public static String convertIntegerToBinary(int n) {
    if (n == 0) {
        return "0";
    }
    StringBuilder binaryNumber = new StringBuilder();
    while (n > 0) {
        int remainder = n % 2;
        binaryNumber.append(remainder);
        n /= 2;
    }
    binaryNumber = binaryNumber.reverse();
    return binaryNumber.toString();
}
```

## 3.使用`Integer` # `toBinaryString`方法

Java 的 [`Integer`](https://web.archive.org/web/20220924213613/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/lang/Integer.html) 类有一个名为 [`toBinaryString`](https://web.archive.org/web/20220924213613/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/lang/Integer.html#toBinaryString(int)) 的方法将一个整数转换成它的二进制等价字符串。

我们来看看`Integer` # `toBinaryString`方法的签名:

```java
public static String toBinaryString(int i)
```

它接受一个整数参数，并返回该整数的二进制字符串表示形式:

```java
int n = 7;
String binaryString = Integer.toBinaryString(n);
assertEquals("111", binaryString);
```

## 4.使用`Integer` # `toString`方法

现在，让我们来看看`[Integer](https://web.archive.org/web/20220924213613/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/lang/Integer.html#toString(int,int))` [#](https://web.archive.org/web/20220924213613/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/lang/Integer.html#toString(int,int)) `[toString](https://web.archive.org/web/20220924213613/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/lang/Integer.html#toString(int,int))`的签名方法:

```java
public static String toString(int i, int radix)
```

`Integer` # `toString`方法是 Java 中的内置方法，它有两个参数。首先，它接受一个要转换成字符串的整数。第二，在将整数转换成字符串表示形式时，需要用到基数。

它以基数指定的基数返回整数输入的字符串表示。

让我们使用这个方法将一个整数转换成二进制格式，使用基数值 2:

```java
int n = 7;
String binaryString = Integer.toString(n, 2);
assertEquals("111", binaryString);
```

正如我们可以看到的，我们在调用`Integer#toString`方法将整数`n`转换为二进制字符串表示时传递了基数值 2。

## 5.结论

总之，我们看了整数到二进制的转换。此外，我们还看到了几个内置的 Java 方法，用于将整数转换成二进制格式的字符串。

和往常一样，所有这些代码示例都可以在 GitHub 上的[处获得。](https://web.archive.org/web/20220924213613/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-numbers-3)