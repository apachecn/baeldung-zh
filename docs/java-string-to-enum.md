# 在 Java 中将字符串转换为枚举

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-string-to-enum>

## 1。简介

在这个简短的教程中，我们将学习如何在 Java 中将字符串快速转换成枚举。

## 2。设置

我们正在处理核心 Java，所以我们不需要添加任何额外的工件。我们还将与来自[枚举指南](/web/20221129014248/https://www.baeldung.com/a-guide-to-java-enums)文章的`PizzaDeliveryStatusEnum`一起工作。

## 3。转换

类似于标准的 Java 类，我们可以使用点符号来访问它们的值。因此，要访问`PizzaDeliveryStatusEnum`的`READY`值，我们可以使用:

```java
PizzaStatusEnum readyStatus = PizzaStatusEnum.READY;
```

这很好，但是如果我们将状态的值存储为一个`String`，并希望将其转换为一个`PizzaStatusEnum`呢？最简单的方法是写一个巨大的`switch`语句，为每个可能的值返回正确的值。但是编写和维护这样的代码是一场噩梦，我们应该不惜一切代价避免它。

另一方面，`enum`类型的**提供了一个`valueOf()`方法，该方法将一个`String`作为参数，并返回相应的`enum`对象:**

```java
PizzaStatusEnum readyStatus = PizzaStatusEnum.valueOf("READY");
```

我们可以通过单元测试来检查这种方法是否有效:

```java
@Test
public void whenConvertedIntoEnum_thenGetsConvertedCorrectly() {

    String pizzaEnumValue = "READY";
    PizzaStatusEnum pizzaStatusEnum
      = PizzaStatusEnum.valueOf(pizzaEnumValue);
    assertTrue(pizzaStatusEnum == PizzaStatusEnum.READY);
}
```

重要的是要记住，`valueOf()` 方法对提供给它的参数进行区分大小写的匹配，因此传递一个与任何原始`enum`值的大小写都不匹配的值将导致一个`IllegalArgumentException`:

```java
@Test(expected = IllegalArgumentException.class)
public void whenConvertedIntoEnum_thenThrowsException() {

    String pizzaEnumValue = "rEAdY";
    PizzaStatusEnum pizzaStatusEnum
      = PizzaStatusEnum.valueOf(pizzaEnumValue);
} 
```

传递一个不是原始`enum`值的一部分的值也会导致一个`IllegalArgumentException`:

```java
@Test(expected = IllegalArgumentException.class)
public void whenConvertedIntoEnum_thenThrowsException() {
    String pizzaEnumValue = "invalid";
    PizzaStatusEnum pizzaStatusEnum = PizzaStatusEnum.valueOf(pizzaEnumValue);
}
```

## 4。结论

在这篇简短的文章中，我们展示了如何将一个`String`转换成一个`enum`。

我们强烈推荐使用`enum`类型的内置`valueOf()`方法，而不是我们自己进行转换。

和往常一样，这篇文章的代码可以在 GitHub 上找到[。](https://web.archive.org/web/20221129014248/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-string-conversions)