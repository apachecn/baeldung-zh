# Java 中的数组排序

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-sorting-arrays>

## 1。概述

在本教程中，我们将讨论按升序和降序对[数组](/web/20221208143956/https://www.baeldung.com/java-arrays-guide)进行排序的常用方法。

我们将看看如何使用 Java 的 [`Arrays`](/web/20221208143956/https://www.baeldung.com/java-util-arrays) 类排序方法，以及如何实现我们自己的`Comparator`来排序数组的值。

## 2.对象定义

在开始之前，让我们快速定义几个数组，我们将在本教程中对它们进行排序。首先，我们将创建一个`ints`数组和一个字符串数组:

```java
int[] numbers = new int[] { -8, 7, 5, 9, 10, -2, 3 };
String[] strings = new String[] { "learning", "java", "with", "baeldung" };
```

让我们也创建一个`Employee `对象的数组，其中每个雇员都有一个`id `和一个`name `属性:

```java
Employee john = new Employee(6, "John");
Employee mary = new Employee(3, "Mary");
Employee david = new Employee(4, "David");
Employee[] employees = new Employee[] { john, mary, david };
```

## 3.按升序排序

Java 的 [`util.Arrays.sort`](https://web.archive.org/web/20221208143956/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/Arrays.html#sort(byte%5B%5D)) 方法为我们提供了一种快速简单的方法来对实现`Comparable` 接口的原语或对象数组进行升序排序。

**在对原语进行排序时，`Arrays.sort `方法使用了[快速排序](/web/20221208143956/https://www.baeldung.com/java-quicksort)的双支点实现。然而，当排序对象时，使用了[合并排序](/web/20221208143956/https://www.baeldung.com/java-merge-sort)的迭代实现。**

### 3.1.基元

为了对原始数组进行升序排序，我们将数组传递给`sort `方法:

```java
Arrays.sort(numbers);
assertArrayEquals(new int[] { -8, -2, 3, 5, 7, 9, 10 }, numbers); 
```

### 3.2.实现可比的

对于实现了`Comparable `接口的对象，就像我们的原始数组一样，我们也可以简单地将数组传递给`sort`方法:

```java
Arrays.sort(strings);
assertArrayEquals(new String[] { "baeldung", "java", "learning", "with" }, strings);
```

### 3.3.没有实现 Comparable 的对象

**对没有实现`Comparable `接口的对象进行排序，比如我们的`Employees`数组，需要我们指定自己的比较器。**

在 Java 8 中，我们可以很容易地做到这一点，通过指定我们想要在我们的`Comparator:`中比较我们的`Employee `对象的属性

```java
Arrays.sort(employees, Comparator.comparing(Employee::getName));
assertArrayEquals(new Employee[] { david, john, mary }, employees);
```

在这种情况下，我们已经指定希望根据员工的`name `属性对他们进行排序。

**我们也可以通过使用`Comparator's [thenComparing](/web/20221208143956/https://www.baeldung.com/java-8-comparator-comparing) `方法**将我们的比较链接起来，在不止一个属性上对我们的对象进行排序

```java
Arrays.sort(employees, Comparator.comparing(Employee::getName).thenComparing(Employee::getId));
```

## 4.按降序排序

### 4.1.基元

按降序对原始数组进行排序不像按升序排序那么简单，因为 Java 不支持在原始类型上使用`Comparators `。为了克服这个不足，我们有几个选择。

首先，我们可以按升序对数组进行排序，然后对数组进行就地反转。

第二，可以将我们的数组转换成一个列表，**使用芭乐的 [`Lists.reverse()`](/web/20221208143956/https://www.baeldung.com/guava-lists) 方法**，然后将我们的列表转换回一个数组。

最后，我们可以将数组转换成一个`Stream `数组，然后将其映射回一个`int`数组。它有一个很好的优势，那就是**一行程序，并且只使用核心 Java:**

```java
numbers = IntStream.of(numbers).boxed().sorted(Comparator.reverseOrder()).mapToInt(i -> i).toArray();
assertArrayEquals(new int[] { 10, 9, 7, 5, 3, -2, -8 }, numbers);
```

这样做的原因是`boxed`将每个`int`转换成一个`Integer`，由`does`实现`Comparator.`

### 4.2.实现可比的

按照降序对实现了`Comparable `接口的对象数组进行排序非常简单。我们需要做的就是传递一个`Comparator `作为我们的`sort`方法的第二个参数。

**在 Java 8 中，我们可以使用`Comparator.reverseOrder() `来表示我们希望我们的数组以降序排序:**

```java
Arrays.sort(strings, Comparator.reverseOrder());
assertArrayEquals(new String[] { "with", "learning", "java", "baeldung" }, strings);
```

### 4.3.没有实现 Comparable 的对象

类似于对实现 comparable 的对象进行排序，我们可以通过在比较定义的末尾添加`reversed() `来颠倒自定义`Comparator `的顺序:

```java
Arrays.sort(employees, Comparator.comparing(Employee::getName).reversed());
assertArrayEquals(new Employee[] { mary, john, david }, employees);
```

## 5。结论

在本文中，我们讨论了如何使用`Arrays.sort`方法对原语和对象的数组进行升序和降序排序。

像往常一样，这篇文章的源代码可以在 Github 上找到[。](https://web.archive.org/web/20221208143956/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-arrays-sorting)