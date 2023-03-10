# 函数式 Java 简介

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-functional-library>

## 1。概述

在本教程中，我们将提供一个关于 [Functional Java](https://web.archive.org/web/20221129012123/http://www.functionaljava.org/) 库的快速概述以及几个例子。

## 2。Java 函数库

Functional Java 库是一个开源库，旨在促进 Java 中的函数式编程。该库提供了许多在[函数式编程](/web/20221129012123/https://www.baeldung.com/cs/functional-programming)中常用的基本和高级编程抽象。

该库的大部分功能都围绕着`F`接口。**这个`F`接口模拟了一个函数，它接受类型为`A`的输入并返回类型为`B`的输出。**所有这些都建立在 Java 自己的类型系统之上。

## 3。Maven 依赖关系

首先，我们需要将所需的[依赖项](https://web.archive.org/web/20221129012123/https://search.maven.org/search?q=g:org.functionaljava)添加到我们的`pom.xml` 文件中:

```java
<dependency>
    <groupId>org.functionaljava</groupId>
    <artifactId>functionaljava</artifactId>
    <version>4.8.1</version>
</dependency>
<dependency>
    <groupId>org.functionaljava</groupId>
    <artifactId>functionaljava-java8</artifactId>
    <version>4.8.1</version>
</dependency>
<dependency>
    <groupId>org.functionaljava</groupId>
    <artifactId>functionaljava-quickcheck</artifactId>
    <version>4.8.1</version>
</dependency>
<dependency>
    <groupId>org.functionaljava</groupId>
    <artifactId>functionaljava-java-core</artifactId>
    <version>4.8.1</version>
</dependency>
```

## 4。定义功能

让我们从创建一个函数开始，我们可以在后面的例子中使用它。

如果没有函数式 Java，一个基本的乘法方法看起来会像这样:

```java
public static final Integer timesTwoRegular(Integer i) {
    return i * 2;
}
```

使用函数式 Java 库，我们可以更优雅地定义这种功能:

```java
public static final F<Integer, Integer> timesTwo = i -> i * 2;
```

在上面的**中，我们看到了一个`F`接口的例子，它将一个`Integer`作为输入，并将这个`Integer`乘以 2 作为输出。**

下面是另一个基本函数的例子，它将一个`Integer`作为输入，但是在这种情况下，返回一个`Boolean`来指示输入是偶数还是奇数:

```java
public static final F<Integer, Boolean> isEven = i -> i % 2 == 0;
```

## 5。应用功能

现在我们已经有了自己的函数，让我们将它们应用到数据集。

函数式 Java 库提供了管理数据的常用类型集，如列表、集合、数组和映射。要意识到的关键是这些数据类型是不可变的。

此外，如果需要的话，这个库提供了方便的函数在标准 Java 集合类之间进行转换。

在下面的例子中，我们将定义一个整数列表，并对其应用我们的`timesTwo`函数。我们还将使用同一函数的内联定义来调用`map`。当然，我们希望结果是一样的:

```java
public void multiplyNumbers_givenIntList_returnTrue() {
    List<Integer> fList = List.list(1, 2, 3, 4);
    List<Integer> fList1 = fList.map(timesTwo);
    List<Integer> fList2 = fList.map(i -> i * 2);

    assertTrue(fList1.equals(fList2));
}
```

正如我们所见,`map`返回一个相同大小的列表，其中每个元素的值都是应用了函数的输入列表的值。输入列表本身不会改变。

下面是一个使用我们的`isEven`函数的类似例子:

```java
public void calculateEvenNumbers_givenIntList_returnTrue() {
    List<Integer> fList = List.list(3, 4, 5, 6);
    List<Boolean> evenList = fList.map(isEven);
    List<Boolean> evenListTrueResult = List.list(false, true, false, true);

    assertTrue(evenList.equals(evenListTrueResult));
}
```

**由于`map`方法返回一个列表，我们可以对它的输出应用另一个函数。**我们调用`map`函数的顺序会改变我们的结果输出:

```java
public void applyMultipleFunctions_givenIntList_returnFalse() {
    List<Integer> fList = List.list(1, 2, 3, 4);
    List<Integer> fList1 = fList.map(timesTwo).map(plusOne);
    List<Integer> fList2 = fList.map(plusOne).map(timesTwo);

    assertFalse(fList1.equals(fList2));
}
```

上述列表的输出将是:

```java
List(3,5,7,9)
List(4,6,8,10)
```

## 6。使用函数过滤

函数式编程中另一个经常使用的操作是**接受一个输入，并根据一些标准**过滤出数据。您可能已经猜到，这些过滤标准是以函数的形式提供的。该函数将需要返回一个布尔值，以指示数据是否需要包含在输出中。

现在，让我们使用`isEven`函数通过`filter`方法从输入数组中过滤出奇数:

```java
public void filterList_givenIntList_returnResult() {
    Array<Integer> array = Array.array(3, 4, 5, 6);
    Array<Integer> filteredArray = array.filter(isEven);
    Array<Integer> result = Array.array(4, 6);

    assertTrue(filteredArray.equals(result));
}
```

一个有趣的观察是，在这个例子中，我们使用了一个`Array`而不是前面例子中使用的`List`，我们的函数工作得很好。**由于函数被抽象和执行的方式，它们不需要知道使用了什么方法来收集输入和输出。**

在这个例子中，我们也使用了自己的`isEven`函数，但是 Functional Java 自己的`Integer`类也有用于[基本数值比较](https://web.archive.org/web/20221129012123/http://www.functionaljava.org/javadoc/4.8.1/functionaljava/fj/function/Integers.html)的标准函数。

## 7。使用函数应用布尔逻辑

在函数式编程中，我们经常使用这样的逻辑:“只有当所有元素满足某个条件时才这样做”，或者“只有当至少一个元素满足某个条件时才这样做”。

**函数式 Java 库通过 [`exists`](https://web.archive.org/web/20221129012123/http://www.functionaljava.org/javadoc/4.4/functionaljava/fj/data/Option.html#exists-fj.F-) 和 [`forall`](https://web.archive.org/web/20221129012123/http://www.functionaljava.org/javadoc/4.4/functionaljava/fj/data/Option.html#forall-fj.F-) 方法:**为我们提供了这种逻辑的快捷方式

```java
public void checkForLowerCase_givenStringArray_returnResult() {
    Array<String> array = Array.array("Welcome", "To", "baeldung");
    assertTrue(array.exists(s -> List.fromString(s).forall(Characters.isLowerCase)));

    Array<String> array2 = Array.array("Welcome", "To", "Baeldung");
    assertFalse(array2.exists(s -> List.fromString(s).forall(Characters.isLowerCase)));

    assertFalse(array.forall(s -> List.fromString(s).forall(Characters.isLowerCase)));
}
```

在上面的例子中，我们使用了一个字符串数组作为输入。调用`fromString`函数会将数组中的每个字符串转换成一个字符列表。对于每一个列表，我们都应用了`forall(Characters.isLowerCase)`。

正如您可能猜到的，`Characters.isLowerCase`是一个函数，如果字符是小写的，它将返回 true。因此，如果整个列表由小写字符组成，将`forall(Characters.isLowerCase)`应用于字符列表将只返回`true`，这反过来表明原始字符串全部是小写的。

在前两个测试中，我们使用了`exists`,因为我们只想知道是否至少有一个字符串是小写的。第三个测试使用了`forall`来验证是否所有的字符串都是小写的。

## 8。用函数处理可选值

处理代码中的可选值通常需要`== null`或`isNotBlank`检查。Java 8 现在提供了`Optional`类来更优雅地处理这些检查，并且函数式 Java 库提供了类似的构造来通过其[选项](https://web.archive.org/web/20221129012123/http://www.functionaljava.org/javadoc/4.8.1/functionaljava/fj/data/Option.html)类优雅地处理缺失数据:

```java
public void checkOptions_givenOptions_returnResult() {
    Option<Integer> n1 = Option.some(1);
    Option<Integer> n2 = Option.some(2);
    Option<Integer> n3 = Option.none();

    F<Integer, Option<Integer>> function = i -> i % 2 == 0 ? Option.some(i + 100) : Option.none();

    Option<Integer> result1 = n1.bind(function);
    Option<Integer> result2 = n2.bind(function);
    Option<Integer> result3 = n3.bind(function);

    assertEquals(Option.none(), result1);
    assertEquals(Option.some(102), result2);
    assertEquals(Option.none(), result3);
}
```

## 9。使用函数减少集合

最后，我们将看看减少集合的功能。“减少一个集合”是“将它汇总成一个值”的一种花哨说法。

**函数式 Java 库将这种功能称为折叠**。

需要指定一个函数来指示折叠元素的含义。一个例子是`Integers.add`函数，显示数组或列表中需要相加的整数。

根据函数在折叠时执行的操作，结果可能会有所不同，这取决于您是从右侧还是左侧开始折叠。这就是函数式 Java 库提供两个版本的原因:

```java
public void foldLeft_givenArray_returnResult() {
    Array<Integer> intArray = Array.array(17, 44, 67, 2, 22, 80, 1, 27);

    int sumAll = intArray.foldLeft(Integers.add, 0);
    assertEquals(260, sumAll);

    int sumEven = intArray.filter(isEven).foldLeft(Integers.add, 0);
    assertEquals(148, sumEven);
}
```

第一个`foldLeft`简单地将所有的整数相加。而第二种方法将首先应用一个过滤器，然后将剩余的整数相加。

## 10。结论

本文只是对函数式 Java 库的一个简短介绍。

和往常一样，GitHub 上的[提供了这篇文章的完整源代码。](https://web.archive.org/web/20221129012123/https://github.com/eugenp/tutorials/tree/master/libraries-6)