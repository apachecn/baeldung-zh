# Java 中的自守数

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-automorphic-numbers>

## 1.概观

在这个简短的教程中，我们将讨论自同构数，并学习几种在 Java 程序中找到它们的方法。

## 2。什么是自守数？

一个自同构数是这样一个数，它的平方最后的位数与该数本身的位数相同。

例如，25 是一个自守数，因为 25 的平方是 625，以 25 结尾。类似地，76 是一个自守数，因为 76 的平方是 5776，同样以 76 结尾。

**在数学中，一个自同构数也被称为循环数。**

自守数的更多例子有 0、1、5、6、25、76、376、625、9376 等。

0 和 1 被称为平凡自同构数，因为它们在每个基中都是自同构数。

## 3.确定一个数是否是自同构的

有许多算法可以用来判断一个数是否是自同构的。接下来，我们将看到几种方法。

### 3.1.循环数字并进行比较

这里有一种方法来确定一个数是否是自同构的:

1.  得到数字并计算它的平方
2.  得到正方形的最后一位数字，并将其与数字的最后一位数字进行比较
    *   如果最后几位不相等，则该数不是自同构数
    *   如果最后几位数字相等，请继续下一步
3.  删除数字和平方的最后一个数字
4.  重复步骤 2 和 3，直到数字的所有位数都比较完毕

上述方法以相反的方式循环输入号码的数字。

让我们编写一个 Java 程序来实现这种方法。`isAutomorphicUsingLoop()`方法接受一个整数作为输入，并检查它是否是自同构的:

```java
public boolean isAutomorphicUsingLoop(int number) {
    int square = number * number;

    while (number > 0) {
        if (number % 10 != square % 10) {
            return false;
        }
        number /= 10;
        square /= 10;
    }

    return true;
}
```

这里我们先算一下`number`的平方。然后，我们迭代`number`的数字，并逐个比较它的最后一个数字和`square`的数字。

在任何阶段，如果最后的数字不相等，我们返回`false`并退出该方法。否则，我们去掉相等的最后一位数，并对`number`的剩余位数重复该过程。

让我们来测试一下:

```java
assertTrue(AutomorphicNumber.isAutomorphicUsingLoop(76));
assertFalse(AutomorphicNumber.isAutomorphicUsingLoop(9));
```

### 3.2.直接比较数字

我们还可以用一种更直接的方式来确定一个数是否是自同构的:

1.  获取数字并计算位数(`n`)
2.  计算数字的平方
3.  从方块中获取最后一个`n `数字
    *   如果正方形的最后一个`n `数字构成原始数字，则该数字是自同构的
    *   否则它不是一个自守数

在这种情况下，我们不需要遍历数字。相反，我们可以使用现有的编程框架库。

**我们可以使用 [`Math`](/web/20221208143830/https://www.baeldung.com/java-lang-math) 类来执行数字运算，例如计算给定数字的位数，并从它的平方中得到最后的位数:**

```java
public boolean isAutomorphicUsingMath(int number) {
    int square = number * number;

    int numberOfDigits = (int) Math.floor(Math.log10(number) + 1);
    int lastDigits = (int) (square % (Math.pow(10, numberOfDigits)));

    return number == lastDigits;
}
```

与第一种方法类似，我们从计算`number`的平方开始。然后，我们不用逐一比较`number`和`square`的最后几位，而是用 [`Math.floor()`](/web/20221208143830/https://www.baeldung.com/java-lang-math#floor) 一次性得到`number`中的`numberOfDigits`总数。之后，我们通过使用 [`Math.pow()`](https://web.archive.org/web/20221208143830/https://docs.oracle.com/en/java/javase/12/docs/api/java.base/java/lang/Math.html#pow(double,double)) 从`square`中提取尽可能多的数字。最后，我们将输入的`number` 与提取的数字`lastDigits`进行比较。

如果`number`和`lastDigits`相等，这个数是自同构的，我们返回`true,` ，否则，我们返回`false`。

让我们来测试一下:

```java
assertTrue(AutomorphicNumber.isAutomorphicUsingMath(76));
assertFalse(AutomorphicNumber.isAutomorphicUsingMath(9));
```

## 4.结论

在本文中，我们探讨了自同构数。我们还研究了几种确定一个数是否是自同构数的方法，以及相应的 Java 程序。

和往常一样，这些例子的代码可以在 GitHub 的[上找到。](https://web.archive.org/web/20221208143830/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-numbers-4)