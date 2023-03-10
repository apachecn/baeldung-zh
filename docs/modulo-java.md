# Java 中的模运算符

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/modulo-java>

## 1.概观

在这个简短的教程中，我们将展示什么是模操作符，以及我们如何在 Java 中使用它来处理一些常见的用例。

## 2.模运算符

先说 Java 中简单除法的缺点。

**如果除法运算符两边的操作数都有类型`int`，则运算的结果是另一个`int:`**

```java
@Test
public void whenIntegerDivision_thenLosesRemainder() {
    assertThat(11 / 4).isEqualTo(2);
}
```

当至少一个操作数的类型为`float`或`double:`时，相同的除法运算会产生不同的结果

```java
@Test
public void whenDoubleDivision_thenKeepsRemainder() {
    assertThat(11 / 4.0).isEqualTo(2.75);
}
```

我们可以观察到，在对整数进行除法运算时，我们丢失了除法运算的余数。

模运算符正好给出了这个余数:

```java
@Test
public void whenModulo_thenReturnsRemainder() {
    assertThat(11 % 4).isEqualTo(3);
}
```

余数是 11(被除数)除以 4(除数)后的余数——在这个例子中是 3。

由于同样的原因，不能被零除，所以当右边的参数为零时，不能使用模运算符。

当我们试图使用零作为右边的操作数时，除法和模运算都会抛出一个`ArithmeticException`:

```java
@Test(expected = ArithmeticException.class)
public void whenDivisionByZero_thenArithmeticException() {
    double result = 1 / 0;
}

@Test(expected = ArithmeticException.class)
public void whenModuloByZero_thenArithmeticException() {
    double result = 1 % 0;
}
```

## 3.常见使用案例

模运算符最常见的用例是找出一个给定的数字是奇数还是偶数。

如果任意数和 2 之间的模运算结果等于 1，则它是奇数:

```java
@Test
public void whenDivisorIsOddAndModulusIs2_thenResultIs1() {
    assertThat(3 % 2).isEqualTo(1);
}
```

另一方面，如果结果为零(即没有余数)，则它是一个偶数:

```java
@Test
public void whenDivisorIsEvenAndModulusIs2_thenResultIs0() {
    assertThat(4 % 2).isEqualTo(0);
}
```

模运算的另一个好用途是跟踪环形数组中下一个自由点的索引。

在针对`int` 值的循环队列的简单实现中，元素保存在固定大小的数组中。

每当我们想要将一个元素推入循环队列时，我们只需通过计算我们已经插入的项目数加 1 的模和队列容量来计算下一个空闲位置:

```java
@Test
public void whenItemsIsAddedToCircularQueue_thenNoArrayIndexOutOfBounds() {
    int QUEUE_CAPACITY= 10;
    int[] circularQueue = new int[QUEUE_CAPACITY];
    int itemsInserted = 0;
    for (int value = 0; value < 1000; value++) {
        int writeIndex = ++itemsInserted % QUEUE_CAPACITY;
        circularQueue[writeIndex] = value;
    }
}
```

使用模操作符，我们可以防止`writeIndex`落在数组的边界之外，因此，我们永远不会得到`ArrayIndexOutOfBoundsException`。

然而，一旦我们插入超过`QUEUE_CAPACITY`个项目，下一个项目将覆盖第一个项目。

## 4.结论

模运算符用于计算整数除法的余数，否则会丢失余数。

这对于做一些简单的事情很有用，比如计算一个给定的数字是偶数还是奇数，对于更复杂的任务也很有用，比如跟踪一个循环数组中的下一个书写位置。

示例代码可以在 [GitHub 库](https://web.archive.org/web/20220930182435/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-lang-operators)中找到。