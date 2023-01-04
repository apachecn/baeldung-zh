# Java 中的数字类指南

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-number-class>

## 1.概观

在本教程中，我们将讨论 Java 的`[Number](https://web.archive.org/web/20221105011848/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/lang/Number.html) c`类。首先，**我们将学习****`Number class`做什么，它包含哪些方法**。然后，我们将深入这个`abstract` 类的各种实现。

## 2.`Number`阶级

`Number`是`java.lang`包中的一个 a `bstract` 类。各种子类扩展了`Number`类。最常用的有:

*   `Byte`
*   `Short`
*   `Integer`
*   `Long`
*   `Double`
*   `Float`

这个类的主要目的是提供一些方法，将所讨论的数值转换成各种原始类型，如`byte`、`short`、`int`、`long`、`double`和 `float`。

有四种方法可以帮助完成任务:

*   `intValue()`
*   `longValue()`
*   `doubleValue()`
*   `floatValue()`

**`Number`也有两个具体的方法，`byteValue()`和`shortValue()`** ，分别返回指定数字的`byte`值和`short`值。要了解更多关于`Number`类的不同实现，请参考我们关于[包装类](/web/20221105011848/https://www.baeldung.com/java-wrapper-classes)的文章。

在接下来的章节中，我们将学习更多关于这些方法及其用法的知识。

## 3.具体方法

具体方法我们一个一个来讨论吧。

### 3.1.`shortValue`()

**顾名思义，该方法将指定的`Number`对象转换为原语`short`值**。

默认实现将`int`值转换成`short`并返回它。但是，子类有自己的实现，它们将各自的值转换成`short`，然后返回。

下面是如何将一个`Double`值转换成一个`short`原始类型:

```java
@Test
public void givenDoubleValue_whenShortValueUsed_thenShortValueReturned() {
    Double doubleValue = Double.valueOf(9999.999);
    assertEquals(9999, doubleValue.shortValue());
}
```

### 3.2.`byteValue()`

**该方法将指定的`Number`对象的值作为`byte`值**返回。然而，`Number` 类的子类有自己的实现。

下面是如何将`Float`值转换成`byte`值:

```java
@Test
public void givenFloatValue_whenByteValueUsed_thenByteValueReturned() {
    Float floatValue = Float.valueOf(101.99F);
    assertEquals(101, floatValue.byteValue());
}
```

## 4.抽象方法

另外，`Number`类也有几个抽象方法和几个实现它们的子类。

在本节中，让我们快速看一下这些方法是如何使用的。

### 4.1. `intValue()`

**该方法返回上下文中`Number`的`int`表示。**

让我们看看如何将一个`Long`值变成`int`:

```java
@Test
public void givenLongValue_whenInitValueUsed_thenInitValueReturned() {
    Long longValue = Long.valueOf(1000L);
    assertEquals(1000, longValue.intValue());
}
```

**当然，编译器正在通过将`long`值转换成`int`来执行[缩小](/web/20221105011848/https://www.baeldung.com/java-primitive-conversions#narrowing-primitive-conversion)操作。**

### 4.2.`longValue()`

**这个方法将返回指定为`long`的 N `umber`的值。**

在这个例子中，我们看到一个`Integer`值是如何通过`Integer`类转换成一个`long`的:

```java
@Test
public void givenIntegerValue_whenLongValueUsed_thenLongValueReturned() {
    Integer integerValue = Integer.valueOf(100);
    assertEquals(100, integerValue.longValue());
}
```

**与`intValue()`方法相反，`longValue()`在[扩大](/web/20221105011848/https://www.baeldung.com/java-primitive-conversions#widening-primitive-conversions)原始转换后返回`long`值。**

### 4.3.`floatValue()`

**我们可以用这个方法将指定的 N `umber`的值返回为`float.`** 让我们来看看一个`Short`的值如何转换为`float`的值:

```java
@Test
public void givenShortValue_whenFloatValueUsed_thenFloatValueReturned() {
    Short shortValue = Short.valueOf(127);
    assertEquals(127.0F, shortValue.floatValue(), 0);
}
```

**同样，`longValue()`和` floatValue()`也执行扩展原语转换。**

### 4.4.`doubleValue()`

最后，这个方法将给定的`Number` 类的值转换为`double`原始数据类型并返回它。

下面是一个使用这种方法将一个`Byte` 转换成`double`的例子:

```java
@Test
public void givenByteValue_whenDoubleValueUsed_thenDoubleValueReturned() {
    Byte byteValue = Byte.valueOf(120);
    assertEquals(120.0, byteValue.doubleValue(), 0);
}
```

## 5.结论

在这个快速教程中，我们看了一下`Number` 类`.`中一些最重要的方法

最后，我们展示了如何在各种 [`Wrapper`类](/web/20221105011848/https://www.baeldung.com/java-wrapper-classes)中使用这些方法。

和往常一样，GitHub 上的[提供了这篇文章的完整源代码。](https://web.archive.org/web/20221105011848/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-numbers-3)