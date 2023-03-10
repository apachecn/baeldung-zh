# Java 中的 XOR 运算符

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-xor-operator>

## 1.概观

在这个快速教程中，我们将学习 Java `XOR`操作符。我们将讨论一些关于`XOR`操作的理论，然后我们将看到如何在 Java 中实现它们。

## 2.`XOR`操作员

让我们从提醒一下`XOR`操作的语义开始。**`XOR`逻辑运算`exclusive or`接受两个布尔操作数，当且仅当操作数不同时返回 true。**相反，如果两个操作数的值相同，则返回 false。

例如，当我们必须检查两个不能同时为真的条件时，可以使用`XOR`操作符。

让我们考虑两个条件，`A`和`B.`。下表显示了`A XOR B`的可能值:

[![XOR Operator Table](img/19bec01f76fd6fb86586c80e9e6a2529.png)](/web/20221108032923/https://www.baeldung.com/wp-content/uploads/2019/08/xor_operator_table-1.png)

**`A XOR B`操作相当于`(A AND !B) OR (!A AND B)`。**为了清楚起见，括号被包括在内，但是可选的，因为`AND`操作符优先于`OR`操作符。

## 3.用 Java 怎么做？

现在我们来看看如何用 Java 表达`XOR`操作。**当然，我们可以选择使用 *& &* 和`||` 操作符，但是这可能有点罗嗦，正如我们将要看到的**。

假设一个`Car`类有两个`boolean`属性:`diesel`和`manual`。现在假设我们想知道汽车是柴油的还是手动的，但不是两者都是。

让我们使用`&&`和`||`操作符来检查这一点:

```java
Car car = Car.dieselAndManualCar();
boolean dieselXorManual = (car.isDiesel() && !car.isManual()) || (!car.isDiesel() && car.isManual());
```

**这有点长，特别是考虑到我们有一个替代方案，由 `^ symbol`代表的 Java `XOR`操作符。**这是一个[位运算符](/web/20221108032923/https://www.baeldung.com/java-bitwise-operators)，意思是它是一个比较两个值的匹配位以返回结果的运算符。在`XOR`的情况下，如果相同位置的两位具有相同的值，则结果位将为 0。否则，就是 1。

因此，我们可以直接使用`^`操作符，而不是繁琐的`XOR`实现:

```java
Car car = Car.dieselAndManualCar();
boolean dieselXorManual = car.isDiesel() ^ car.isManual();
```

正如我们所见，`^`操作符允许我们更简洁地表达`XOR`操作。

**最后，值得一提的是,`XOR `操作符和其他位操作符一样，适用于所有基本类型。**例如，让我们考虑两个整数 1 和 3，它们的二进制表示分别是 000000001 和 00000011。在它们之间使用`XOR`运算符将得到整数 2:

```java
assertThat(1 ^ 3).isEqualTo(2);
```

这两个数中只有第二位不同；因此，该位上的`XOR`运算符的结果将为 1。所有其他位都是相同的，所以它们的按位`XOR`结果是 0，给我们一个最终值 00000010，整数 2 的二进制表示。

## 4.结论

在本文中，我们学习了 Java `XOR`操作符。我们展示了它如何提供一种简洁的方式来表达`XOR `操作。

和往常一样，这篇文章的完整代码可以在 GitHub 上找到[。](https://web.archive.org/web/20221108032923/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-lang-operators)