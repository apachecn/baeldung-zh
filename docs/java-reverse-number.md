# 用 Java 反转一个数字

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-reverse-number>

## 1.概观

在本教程中，我们将看到如何使用 Java 中的数学方法来反转一个数字。首先，我们将了解做这件事需要哪些数学运算，然后我们将通过三种不同的方式来实现它。

## 2.解决方法

首先，让我们举个例子，看看到底会发生什么。例如，我们希望数字 1234 变成 4321。这可以通过以下方法实现:

1.  获取数字的最后一位数
    *   我们可以应用模数来得到最后一位数字
    *   第一次迭代–1234% 10 = 4
    *   第二次迭代–123% 10 = 3
2.  将反转的数字乘以 10，并加上上一步中找到的数字
    *   第一次迭代–0 * 10+4 = 4(因为开始时没有倒数，所以我们在第一次迭代中乘以 0)
    *   第二次迭代–4 * 10+3 = 43
3.  将原始数字除以 10，从第 1 步开始重复，直到数字不为 0
    *   第一次迭代–1234/10 = 123
    *   第二次迭代–123/10 = 12

## 3.数学实现

我们想把上面的数学运算翻译成代码。这可以通过三种不同的方式实现:使用一个`while`循环、一个`for`循环或者递归。

下面的方法也适用于负值，即使用要反转的数字的绝对值，如果原始数字为负，则将反转的数字乘以-1。

### 3.1.`while`循环

[`while`](/web/20221128042630/https://www.baeldung.com/java-while-loop) 循环是列表中的第一个，因为它是翻译上述数学运算的最清晰的方式:

```java
int reversedNumber = 0;
int numberToReverse = Math.abs(number);

while (numberToReverse > 0) {
    int mod = numberToReverse % 10;
    reversedNumber = reversedNumber * 10 + mod;
    numberToReverse /= 10;
}

return number < 0 ? reversedNumber * -1 : reversedNumber;
```

### 3.2.`for`循环

使用 [`for`](/web/20221128042630/https://www.baeldung.com/java-for-loop) 循环，其逻辑与前面相同。我们跳过`for`循环的初始化语句，使用在终止条件下反转的数字:

```java
int reversedNumber = 0;
int numberToReverse = Math.abs(number);

for (; numberToReverse > 0; numberToReverse /= 10) {
    int mod = numberToReverse % 10;
    reversedNumber = reversedNumber * 10 + mod;
}

return number < 0 ? reversedNumber * -1 : reversedNumber;
```

### 3.3.递归

对于[递归](/web/20221128042630/https://www.baeldung.com/java-recursion)，我们可以使用一个包装器方法，该方法调用返回反转数的递归方法:

```java
int reverseNumberRecWrapper(int number) {
    int output = reverseNumberRec(Math.abs(number), 0);
    return number < 0 ? output * -1 : output;
}
```

递归方法以与前面示例相同的方式实现数学运算:

```java
int reverseNumberRec(int numberToReverse, int recursiveReversedNumber) {

    if (numberToReverse > 0) {
        int mod = numberToReverse % 10;
        recursiveReversedNumber = recursiveReversedNumber * 10 + mod;
        numberToReverse /= 10;
        return reverseNumberRec(numberToReverse, recursiveReversedNumber);
    }

    return recursiveReversedNumber;
}
```

**递归方法在每次迭代中返回当前反转的数字，并且要反转的数字随着每次迭代而缩短。这种情况一直发生，直到要反转的数字为 0，此时我们返回完全反转的数字。**

## 4.结论

在本文中，我们探索了三种不同的反转数字的实现，分别使用了一个`while`循环、一个`for`循环和递归。

与往常一样，GitHub 上的[提供了示例的源代码。](https://web.archive.org/web/20221128042630/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-numbers-4)