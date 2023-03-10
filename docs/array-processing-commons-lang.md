# 用 Apache Commons Lang 3 处理数组

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/array-processing-commons-lang>

## 1。概述

Apache Commons Lang 3 库提供了对 Java APIs 核心类的操作支持。这种支持包括处理字符串、数字、日期、并发性、对象反射等的方法。

在这个快速教程中，我们将关注使用非常有用的`ArrayUtils`实用程序类进行数组处理。

## 2。Maven 依赖关系

为了使用 Commons Lang 3 库，只需使用以下依赖项从中央 Maven 存储库中取出它:

```java
<dependency>
    <groupId>org.apache.commons</groupId>
    <artifactId>commons-lang3</artifactId>
    <version>3.12.0</version>
</dependency>
```

你可以在这里找到这个库的最新版本。

## 3。`ArrayUtils`

`ArrayUtils`类提供了使用数组的实用方法。这些方法试图通过防止在传入一个`null`值时抛出异常来优雅地处理输入。

本节说明了在`ArrayUtils`类中定义的一些方法。请注意，所有这些方法都可以用于任何元素类型。

为了方便起见，它们的重载风格也被定义用于处理包含基元类型的数组。

## 4。`add`和`addAll`

方法复制一个给定的数组，并在新数组的给定位置插入一个给定的元素。如果未指定位置，新元素将添加到数组的末尾。

以下代码片段在`oldArray`数组的第一个位置插入数字 0，并验证结果:

```java
int[] oldArray = { 2, 3, 4, 5 };
int[] newArray = ArrayUtils.add(oldArray, 0, 1);
int[] expectedArray = { 1, 2, 3, 4, 5 };

assertArrayEquals(expectedArray, newArray);
```

如果未指定位置，则在`oldArray`的末尾添加附加元素:

```java
int[] oldArray = { 2, 3, 4, 5 };
int[] newArray = ArrayUtils.add(oldArray, 1);
int[] expectedArray = { 2, 3, 4, 5, 1 };

assertArrayEquals(expectedArray, newArray);
```

方法将所有元素加到给定数组的末尾。以下片段说明了这种方法并确认了结果:

```java
int[] oldArray = { 0, 1, 2 };
int[] newArray = ArrayUtils.addAll(oldArray, 3, 4, 5);
int[] expectedArray = { 0, 1, 2, 3, 4, 5 };

assertArrayEquals(expectedArray, newArray);
```

## 5。`remove`和`removeAll`

方法从给定的数组中移除指定位置的元素。所有后续元素都向左移动。请注意，这适用于所有移除操作。

此方法返回一个新数组，而不是对原始数组进行更改:

```java
int[] oldArray = { 1, 2, 3, 4, 5 };
int[] newArray = ArrayUtils.remove(oldArray, 1);
int[] expectedArray = { 1, 3, 4, 5 };

assertArrayEquals(expectedArray, newArray);
```

方法从给定的数组中删除指定位置的所有元素:

```java
int[] oldArray = { 1, 2, 3, 4, 5 };
int[] newArray = ArrayUtils.removeAll(oldArray, 1, 3);
int[] expectedArray = { 1, 3, 5 };

assertArrayEquals(expectedArray, newArray);
```

## 6。`removeElement`和`removeElements`

方法从给定的数组中删除指定元素的第一个匹配项。

如果给定数组中不存在这样的元素，移除操作将被忽略，而不是引发异常:

```java
int[] oldArray = { 1, 2, 3, 3, 4 };
int[] newArray = ArrayUtils.removeElement(oldArray, 3);
int[] expectedArray = { 1, 2, 3, 4 };

assertArrayEquals(expectedArray, newArray);
```

方法从给定的数组中删除第一次出现的指定元素。

如果给定数组中不存在指定的元素，移除操作将被忽略，而不是引发异常:

```java
int[] oldArray = { 1, 2, 3, 3, 4 };
int[] newArray = ArrayUtils.removeElements(oldArray, 2, 3, 5);
int[] expectedArray = { 1, 3, 4 };

assertArrayEquals(expectedArray, newArray);
```

## 7。`removeAllOccurences` API

方法从给定的数组中删除指定元素的所有出现。

如果给定数组中不存在这样的元素，移除操作将被忽略，而不是引发异常:

```java
int[] oldArray = { 1, 2, 2, 2, 3 };
int[] newArray = ArrayUtils.removeAllOccurences(oldArray, 2);
int[] expectedArray = { 1, 3 };

assertArrayEquals(expectedArray, newArray);
```

## 8。`contains` API

方法检查一个值是否存在于一个给定的数组中。下面是一个代码示例，包括结果验证:

```java
int[] array = { 1, 3, 5, 7, 9 };
boolean evenContained = ArrayUtils.contains(array, 2);
boolean oddContained = ArrayUtils.contains(array, 7);

assertEquals(false, evenContained);
assertEquals(true, oddContained);
```

## 9。`reverse` API

方法在给定数组的指定范围内反转元素顺序。此方法对传入的数组进行更改，而不是返回新的数组。

让我们快速看一下:

```java
int[] originalArray = { 1, 2, 3, 4, 5 };
ArrayUtils.reverse(originalArray, 1, 4);
int[] expectedArray = { 1, 4, 3, 2, 5 };

assertArrayEquals(expectedArray, originalArray);
```

如果未指定范围，则所有元素的顺序相反:

```java
int[] originalArray = { 1, 2, 3, 4, 5 };
ArrayUtils.reverse(originalArray);
int[] expectedArray = { 5, 4, 3, 2, 1 };

assertArrayEquals(expectedArray, originalArray);
```

## 10。`shift` API

方法将给定数组中的一系列元素移动若干位置。此方法对传入的数组进行更改，而不是返回新的数组。

以下代码片段将索引 1(含)和索引 4(不含)处的元素之间的所有元素向右移动一个位置，并确认结果:

```java
int[] originalArray = { 1, 2, 3, 4, 5 };
ArrayUtils.shift(originalArray, 1, 4, 1);
int[] expectedArray = { 1, 4, 2, 3, 5 };

assertArrayEquals(expectedArray, originalArray);
```

如果未指定范围边界，数组的所有元素都将移动:

```java
int[] originalArray = { 1, 2, 3, 4, 5 };
ArrayUtils.shift(originalArray, 1);
int[] expectedArray = { 5, 1, 2, 3, 4 };

assertArrayEquals(expectedArray, originalArray);
```

## 11。`subarray` API

方法创建一个新的数组，其中包含给定数组的指定范围内的元素。以下是结果断言的示例:

```java
int[] oldArray = { 1, 2, 3, 4, 5 };
int[] newArray = ArrayUtils.subarray(oldArray, 2, 7);
int[] expectedArray = { 3, 4, 5 };

assertArrayEquals(expectedArray, newArray);
```

请注意，当传入的索引大于数组长度时，它被降级为数组长度，而不是让方法引发异常。同样，如果传入一个负索引，它将被提升为零。

## 12。`swap` API

方法交换给定数组中指定位置的一系列元素。

以下代码片段交换从索引 0 和 3 开始的两组元素，每组包含两个元素:

```java
int[] originalArray = { 1, 2, 3, 4, 5 };
ArrayUtils.swap(originalArray, 0, 3, 2);
int[] expectedArray = { 4, 5, 3, 1, 2 };

assertArrayEquals(expectedArray, originalArray);
```

如果没有传入长度参数，则每个位置只交换一个元素:

```java
int[] originalArray = { 1, 2, 3, 4, 5 };
ArrayUtils.swap(originalArray, 0, 3);
int[] expectedArray = { 4, 2, 3, 1, 5 };
assertArrayEquals(expectedArray, originalArray);
```

## 13。结论

本教程介绍了 Apache Commons Lang 3-`ArrayUtils`中的核心数组处理实用程序。

与往常一样，上面给出的所有示例和代码片段的实现都可以在 GitHub 项目中找到。