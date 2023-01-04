# 在 Java 中检查数组是否排序

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-check-sorted-array>

## 1.概观

在本教程中，我们将看到不同的方法来检查一个数组是否排序。

不过，在开始之前，检查一下[如何在 Java](/web/20221127130125/https://www.baeldung.com/java-sorting-arrays) 中对数组进行排序会很有趣。

## 2.有一个环

一种检查方法是使用`for` 循环。我们可以**逐一迭代数组的所有值。**

我们来看看怎么做。

### 2.1.原始数组

简单地说，我们将迭代除最后一个位置以外的所有位置。这是因为我们要比较一个位置和下一个位置。

如果它们中的一些没有排序，该方法将返回`false.`如果没有一个比较返回`false,`，这意味着一个数组已经排序:

```
boolean isSorted(int[] array) {
    for (int i = 0; i < array.length - 1; i++) {
        if (array[i] > array[i + 1])
            return false;
    }
    return true;
}
```

### 2.2.实现`Comparable`的对象

我们可以对实现`Comparable.`的对象做类似的事情，而不是使用大于号**，我们将使用`compareTo` :**

```
boolean isSorted(Comparable[] array) {
    for (int i = 0; i < array.length - 1; ++i) {
        if (array[i].compareTo(array[i + 1]) > 0)
            return false;
    }
    return true;
}
```

### 2.3.不实现`Comparable`的对象

但是，如果我们的对象没有实现`Comparable`呢？在这种情况下，我们可以改为**创建一个`Comparator.`**

在这个例子中，我们将使用`Employee`对象。这是一个简单的 POJO，有三个字段:

```
public class Employee implements Serializable {
    private int id;
    private String name;
    private int age;

    // getters and setters
}
```

然后，我们需要选择我们想要排序的字段。这里，让我们按`age` 字段排序:

```
Comparator<Employee> byAge = Comparator.comparingInt(Employee::getAge);
```

然后，我们可以改变我们的方法，取一个`Comparator`:

```
boolean isSorted(Object[] array, Comparator comparator) {
    for (int i = 0; i < array.length - 1; ++i) {
        if (comparator.compare(array[i], (array[i + 1])) > 0)
            return false;
    }

    return true;
}
```

## 3.递归地

当然，我们可以使用[递归](/web/20221127130125/https://www.baeldung.com/java-recursion)来代替。这里的想法是，我们将检查数组中的两个位置，然后递归，直到我们检查完每个位置。

### 3.1.原始数组

在这个方法中，**我们检查最后两个位置。如果它们被排序了，我们将再次调用这个方法，但是使用先前的位置。**如果其中一个位置没有排序，该方法将返回`false:`

```
boolean isSorted(int[] array, int length) {
    if (array == null || length < 2) 
        return true; 
    if (array[length - 2] > array[length - 1])
        return false;
    return isSorted(array, length - 1);
}
```

### 3.2.实现`Comparable`的对象

现在，让我们再次看看实现了`Comparable. `的对象，我们会看到同样的方法对`compareTo`也有效:

```
boolean isSorted(Comparable[] array, int length) {
    if (array == null || length < 2) 
        return true; 
    if (array[length - 2].compareTo(array[length - 1]) > 0)
        return false;
    return isSorted(array, length - 1);
}
```

### 3.3.不实现`Comparable`的对象

最近，让我们再次尝试我们的`Employee` 对象，添加`Comparator `参数:

```
boolean isSorted(Object[] array, Comparator comparator, int length) {
    if (array == null || length < 2)
        return true;
    if (comparator.compare(array[length - 2], array[length - 1]) > 0)
        return false;
    return isSorted(array, comparator, length - 1);
}
```

## 4.结论

在本教程中，我们看到了如何检查一个数组是否排序。我们看到了迭代和递归的解决方案。

我们建议使用循环解决方案。更干净，更容易阅读。

像往常一样，本教程的源代码可以在 GitHub 上找到[。](https://web.archive.org/web/20221127130125/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-arrays-sorting)