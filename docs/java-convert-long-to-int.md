# 在 Java 中将 long 转换为 int 类型

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-convert-long-to-int>

## 1.概观

在本教程中，我们将看到如何在 Java 中将一个`long`值转换成一个`int`类型。在开始编码之前，我们需要指出关于这种数据类型的一些细节。

首先，在 Java 中，`long`值是用有符号的 64 位数字表示的。另一方面，`int`值由带符号的 32 位数字表示。因此，**将较高的数据类型转换为较低的数据类型称为窄化类型转换**。作为这些转换的结果，当`long`值大于 `Integer.MAX_VALUE`和`Integer.MIN_VALUE`时，一些位会丢失。

此外，我们将展示对于每个转换变量，如何使`long` 值等于`Integer.MAX_VALUE`加 1。

## 2.数据变换

### 2.1.铸造值

首先，在 Java 中对值进行强制转换是最常见的类型转换方式——这很简单:

```java
public int longToIntCast(long number) {
    return (int) number;
}
```

### 2.2.Java 8

从 Java 8 开始，我们可以使用另外两种方法来进行类型转换:使用`[Math](/web/20221208143815/https://www.baeldung.com/java-lang-math)` 包或者使用 lambda 函数。对于`Math` 包，我们可以使用`toIntExact` 方法:

```java
public int longToIntJavaWithMath(long number) {
return Math.toIntExact(number);
}
```

### 2.3.包装类

另一方面，我们可以使用包装类`Long`来获取`int`值:

```java
public int longToIntBoxingValues(long number) {
    return Long.valueOf(number).intValue();
}
```

### 2.4.使用`BigDecimal`

此外，我们可以使用`BigDecimal`类来完成这种转换:

```java
public static int longToIntWithBigDecimal(long number) {
    return new BigDecimal(number).intValueExact();
}
```

### 2.5.用番石榴

接下来，我们将展示使用[谷歌番石榴](/web/20221208143815/https://www.baeldung.com/guava-guide)的`Ints`类的类型转换:

```java
public int longToIntGuava(long number) {
    return Ints.checkedCast(number);
}
```

另外，[的谷歌番石榴](/web/20221208143815/https://www.baeldung.com/guava-guide)的`Ints`类提供了一个`saturatedCast` 方法:

```java
public int longToIntGuavaSaturated(long number) {
    return Ints.saturatedCast(number);
}
```

### 2.6.整数上下界

最后，我们需要考虑一个整数值有一个上下界。这些限值由`Integer.MAX_VALUE` 和`Integer.MIN_VALUE`定义。对于超出这些限制的值，不同方法的结果是不同的。

在下一段代码中，我们将测试 int 值无法保存 long 值的情况:

```java
@Test
public void longToIntSafeCast() {
    long max = Integer.MAX_VALUE + 10L;
    int expected = -2147483639;
    assertEquals(expected, longToIntCast(max));
    assertEquals(expected, longToIntJavaWithLambda(max));
    assertEquals(expected, longToIntBoxingValues(max));
}
```

使用直接强制转换、使用 lambda 或使用装箱值会产生负值。在这些情况下，long 值大于`Integer.MAX_VALUE`，这就是结果值用负数包装的原因。如果长值小于`Integer.MIN_VALUE` ，则结果值为正数。

另一方面，本文描述的三种方法可能抛出不同类型的异常:

```java
@Test
public void longToIntIntegerException() {
    long max = Integer.MAX_VALUE + 10L;
    assertThrows(ArithmeticException.class, () -> ConvertLongToInt.longToIntWithBigDecimal(max));
    assertThrows(ArithmeticException.class, () -> ConvertLongToInt.longToIntJavaWithMath(max));
    assertThrows(IllegalArgumentException.class, () -> ConvertLongToInt.longToIntGuava(max));
}
```

对于第一个和第二个，抛出一个`ArithmeticException` 。对于后者，抛出一个`IllegalArgumentException` 。在这种情况下，`Ints.checkedCast` 检查整数是否超出范围。

最后，从 Guava 开始，`saturatedCast` 方法，首先检查整数极限并返回极限值是大于还是小于整数上限和下限的传递数:

```java
@Test
public void longToIntGuavaSaturated() {
    long max = Integer.MAX_VALUE + 10L;
    int expected = 2147483647;
    assertEquals(expected, ConvertLongToInt.longToIntGuavaSaturated(max));
}
```

## 3.结论

在本文中，我们通过一些例子展示了如何在 Java 中将 long 类型转换为 int 类型。使用原生 Java 造型和一些库。

像往常一样，本文中使用的所有片段都可以在 GitHub 上获得[。](https://web.archive.org/web/20221208143815/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-exceptions-4)