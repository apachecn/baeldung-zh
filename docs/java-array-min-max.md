# 用 Java 查找数组中的最小值/最大值

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-array-min-max>

## 1。简介

在这个简短的教程中，我们将看到如何使用 Java 8 的`Stream` API 找到数组中的最大值和最小值。

我们将从寻找整数数组中的最小值开始，然后我们将寻找对象数组中的最大值。

## 2。概述

有许多方法可以找到无序数组中的最小值或最大值，它们看起来都像:

```java
SET MAX to array[0]
FOR i = 1 to array length - 1
  IF array[i] > MAX THEN
    SET MAX to array[i]
  ENDIF
ENDFOR
```

我们将看看 **Java 8 如何对我们隐藏这些细节**。但是，在 Java 的 API 不适合我们的情况下，我们总是可以回到这个基本算法。

因为我们需要检查数组中的每个值，所以所有的实现都是`O(n)`。

## 3。寻找最小值

**`java.util.stream.IntStream`接口提供了`min`方法**，它将很好地满足我们的目的。

因为我们只处理整数，`min`不需要`Comparator`:

```java
@Test
public void whenArrayIsOfIntegerThenMinUsesIntegerComparator() {
    int[] integers = new int[] { 20, 98, 12, 7, 35 };

    int min = Arrays.stream(integers)
      .min()
      .getAsInt();

    assertEquals(7, min);
}
```

注意我们是如何在`Arrays`中使用`stream` 静态方法创建`Integer`流对象的。每个原始数组类型都有等效的`stream`方法。

因为数组可能是空的，`min`返回一个`Optional,` ，所以为了将它转换成一个`int`，我们使用 *getAsInt* 。

## 4。寻找最大的自定义对象

让我们创建一个简单的 POJO:

```java
public class Car {
    private String model;
    private int topSpeed;

    // standard constructors, getters and setters
}
```

然后我们可以再次使用`Stream` API 在一组`Car`中找到最快的汽车:

```java
@Test
public void whenArrayIsOfCustomTypeThenMaxUsesCustomComparator() {
    Car porsche = new Car("Porsche 959", 319);
    Car ferrari = new Car("Ferrari 288 GTO", 303);
    Car bugatti = new Car("Bugatti Veyron 16.4 Super Sport", 415);
    Car mcLaren = new Car("McLaren F1", 355);
    Car[] fastCars = { porsche, ferrari, bugatti, mcLaren };

    Car maxBySpeed = Arrays.stream(fastCars)
      .max(Comparator.comparing(Car::getTopSpeed))
      .orElseThrow(NoSuchElementException::new);

    assertEquals(bugatti, maxBySpeed);
}
```

在这种情况下，`Arrays`的静态方法`stream`返回接口`java.util.stream.Stream<T>`的**实例，其中方法`max` 需要一个`Comparator`** 。

我们本可以构建自己的定制`Comparator`，但是 [`Comparator.comparing` 要简单得多。](/web/20221208143856/https://www.baeldung.com/java-8-comparator-comparing)

再次注意，`max` 返回一个`Optional`实例，原因和前面一样。

我们可以`get`这个值，或者我们可以用`Optional` s 做任何其他可能的事情，比如`orElseThrow`如果`max`没有返回值就会抛出异常。

## 5。结论

在这篇短文中，我们看到了使用 Java 8 的`Stream` API 在一个数组中找到 max 和 min 是多么简单和紧凑。

有关该库的更多信息，请参考 [Oracle 文档](https://web.archive.org/web/20221208143856/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/stream/package-summary.html)。

所有这些例子和代码片段的实现都可以在 GitHub 上找到[。](https://web.archive.org/web/20221208143856/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-8)