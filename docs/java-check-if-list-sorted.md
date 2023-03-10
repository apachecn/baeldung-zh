# 在 Java 中检查列表是否排序

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-check-if-list-sorted>

## 1。概述

在本教程中，我们将看到用 Java 检查列表是否排序的不同方法。

## 2。迭代方法

迭代方法是检查排序列表的一种简单而直观的方法。在这种方法中，我们将迭代列表并比较相邻的元素。如果两个相邻元素中的任何一个没有排序，我们可以说这个列表没有排序。

列表可以按自然顺序排序，也可以按自定义顺序排序。我们将使用`Comparable`和`Comparator`接口来涵盖这两种情况。

### 2.1。使用`Comparable`

首先，让我们看一个**列表的例子，它的元素类型是`Comparable`** 。这里，我们将考虑一个包含类型为`String`的对象的列表:

```java
public static boolean isSorted(List<String> listOfStrings) {
    if (isEmpty(listOfStrings) || listOfStrings.size() == 1) {
        return true;
    }

    Iterator<String> iter = listOfStrings.iterator();
    String current, previous = iter.next();
    while (iter.hasNext()) {
        current = iter.next();
        if (previous.compareTo(current) > 0) {
            return false;
        }
        previous = current;
    }
    return true;
}
```

### 2.2。使用`Comparator`

现在，让我们考虑一个没有实现`Comparable`的`Employee`类。所以，在这种情况下，我们需要用一个`Comparator`来比较列表中的相邻元素:

```java
public static boolean isSorted(List<Employee> employees, Comparator<Employee> employeeComparator) {
    if (isEmpty(employees) || employees.size() == 1) {
        return true;
    }

    Iterator<Employee> iter = employees.iterator();
    Employee current, previous = iter.next();
    while (iter.hasNext()) {
        current = iter.next();
        if (employeeComparator.compare(previous, current) > 0) {
            return false;
        }
        previous = current;
    }
    return true;
}
```

以上两个例子类似。唯一的区别是我们如何比较列表的前一个和当前元素。

此外，**我们还可以使用`Comparator`对排序检查**进行精确控制。关于这两个的更多信息可以在我们的[比较器和 Java](/web/20221206015113/https://www.baeldung.com/java-comparator-comparable) 教程中找到。

## 3。递归方法

现在，我们将看到如何使用递归来检查排序列表:

```java
public static boolean isSorted(List<String> listOfStrings) {
    return isSorted(listOfStrings, listOfStrings.size());
}

public static boolean isSorted(List<String> listOfStrings, int index) {
    if (index < 2) {
        return true;
    } else if (listOfStrings.get(index - 2).compareTo(listOfStrings.get(index - 1)) > 0) {
        return false;
    } else {
        return isSorted(listOfStrings, index - 1);
    }
}
```

## 4.用番石榴

使用第三方库而不是编写自己的逻辑往往很好。Guava 库有一些实用程序类，我们可以用它们来检查列表是否已经排序。

### 4.1。番石榴`Ordering`类

在这一节中，我们将看到如何使用 Guava 中的`Ordering`类来检查一个排序列表。

首先，我们将看到一个包含类型为`Comparable`的元素的列表示例:

```java
public static boolean isSorted(List<String> listOfStrings) {
    return Ordering.<String> natural().isOrdered(listOfStrings);
}
```

接下来，我们将看看如何使用一个`Comparator`来检查一列`Employee`对象是否被排序:

```java
public static boolean isSorted(List<Employee> employees, Comparator<Employee> employeeComparator) {
    return Ordering.from(employeeComparator).isOrdered(employees);
}
```

此外，我们可以使用`natural().reverseOrder()`来检查一个列表是否按逆序排序。另外，我们可以使用`natural().nullFirst()`和`natural()`。`nullLast()`检查`null` 是出现在排序列表的第一个还是最后一个。

要了解更多关于番石榴`Ordering`类的知识，我们可以参考我们的[指南，番石榴的订购](/web/20221206015113/https://www.baeldung.com/guava-ordering)文章。

### 4.2。番石榴`Comparators`类

如果我们使用的是 Java 8 或更高版本，Guava 在`Comparators`类方面提供了一个更好的选择。我们将看到一个**使用这个类的`isInOrder`方法**的例子:

```java
public static boolean isSorted(List<String> listOfStrings) {
    return Comparators.isInOrder(listOfStrings, Comparator.<String> naturalOrder());
}
```

正如我们所看到的，在上面的例子中，我们使用了自然排序来检查一个排序列表。我们也可以使用一个`Comparator`来定制分类检查。

## 5。结论

在本文中，我们看到了如何使用简单的迭代方法、递归方法和 Guava 来检查排序列表。我们还简要地提到了`Comparator`和`Comparable` 在决定分类检查逻辑时的用法。

所有这些例子和代码片段的实现都可以在 GitHub 上找到[。](https://web.archive.org/web/20221206015113/https://github.com/eugenp/tutorials/tree/master/algorithms-modules/algorithms-miscellaneous-3)