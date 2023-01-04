# Java ArrayIndexOutOfBoundsException

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-arrayindexoutofboundsexception>

## 1.概观

在本教程中，我们将讨论 Java 中的`ArrayIndexOutOfBoundsException`。我们会明白为什么会发生以及如何避免。

## 2.`ArrayIndexOutOfBoundsException` 发生在什么时候？

我们知道，在 Java 中，[数组](/web/20220803133836/https://www.baeldung.com/java-arrays-guide)是一个静态数据结构，我们在创建时就定义了它的大小。

我们使用索引来访问数组的元素。数组中的索引从零开始，并且决不能大于或等于数组的大小。

简而言之，**经验法则是 0 < =索引<(数组大小)。**

**`ArrayIndexOutOfBoundsException`发生在我们访问一个数组，或者一个`Collection`的时候，它是由一个带有无效索引的数组支持的。**这意味着索引要么小于零，要么大于或等于数组的大小。

此外，边界检查发生在运行时。所以，`ArrayIndexOutOfBoundsException`是一个运行时异常。因此，在访问数组的边界元素时，我们需要格外小心。

先来了解一下导致`ArrayIndexOutOfBoundsException`的一些常见操作。

### 2.1.访问数组

访问数组时最常见的错误是忘记了上限和下限。

数组的下限总是 0，而上限比它的长度小 1。

**访问这些边界之外的数组元素会抛出一个`ArrayIndexOutOfBoundsException` :**

```java
int[] numbers = new int[] {1, 2, 3, 4, 5};
int lastNumber = numbers[5];
```

这里，数组的大小是 5，这意味着索引的范围是从 0 到 4。

在这种情况下，访问第 5 个索引会导致一个`ArrayIndexOutOfBoundsException`:

```java
Exception in thread "main" java.lang.ArrayIndexOutOfBoundsException: Index 5 out of bounds for length 5
    at ...
```

类似地，如果我们将一个小于零的值作为索引传递给`numbers.`，我们就会得到`ArrayIndexOutOfBoundsException`

### 2.2.访问由`Arrays.asList()`返回的`List`

静态方法 [`Arrays.asList()`](/web/20220803133836/https://www.baeldung.com/java-arrays-aslist-vs-new-arraylist#arraysaslist) 返回由指定数组支持的固定大小的列表。此外，它还充当了基于数组和基于集合的 API 之间的桥梁。

这个返回的`List`有基于索引访问其元素的方法。此外，与数组类似，索引从零开始，范围是比其大小小一。

**如果我们试图在这个范围之外访问由`Arrays.asList()`返回的`List`的元素，我们将得到一个`ArrayIndexOutOfBoundsException` :**

```java
List<Integer> numbersList = Arrays.asList(1, 2, 3, 4, 5);
int lastNumber = numbersList.get(5);
```

这里，我们再次尝试获取`List`的最后一个元素。最后一个元素的位置是 5，但它的索引是 4(大小-1)。因此，我们得到`ArrayIndexOutOfBoundsException`如下:

```java
Exception in thread "main" java.lang.ArrayIndexOutOfBoundsException: Index 5 out of bounds for length 5
    at java.base/java.util.Arrays$ArrayList.get(Arrays.java:4351)
    at  ...
```

同样，如果我们传递一个负的索引，比如-1，我们会得到类似的结果。

### 2.3.循环迭代

有时，在 for 循环中迭代数组时，我们可能会放入错误的终止表达式。

不是在比数组长度小 1 的位置终止索引，我们可能会迭代到它的长度:

```java
int sum = 0;
for (int i = 0; i <= numbers.length; i++) {
    sum += numbers[i];
}
```

在上面的终止表达式中，循环变量`i `被比较为小于或等于我们现有数组`numbers.` 的长度，因此，在最后一次迭代中，`i `的值将变为 5。

由于索引 5 超出了`numbers,`的范围，它将再次导致`ArrayIndexOutOfBoundsException`:

```java
Exception in thread "main" java.lang.ArrayIndexOutOfBoundsException: Index 5 out of bounds for length 5
    at com.baeldung.concatenate.IndexOutOfBoundExceptionExamples.main(IndexOutOfBoundExceptionExamples.java:22)
```

## 3.如何避免`ArrayIndexOutOfBoundsException`？

现在让我们来了解一些避免`ArrayIndexOutOfBoundsException`的方法。

### 3.1.记住起始索引

我们必须记住在 Java 中数组索引是从 0 开始的。因此，第一个元素总是在索引 0 处，而最后一个元素在比数组长度小一的索引处。

记住这条规则会帮助我们在大多数时候避免`ArrayIndexOutOfBoundsException`。

### 3.2.在循环中正确使用运算符

**错误地将循环变量初始化为索引 1 可能会导致`ArrayIndexOutOfBoundsException`。**

**同样，在循环的终止表达式中不正确地使用运算符<、< =、>或> =也是出现这种异常的常见原因。**

我们应该正确地确定这些运算符在循环中的使用。

### 3.3.使用增强的`for` 循环

如果我们的应用程序运行在 Java 1.5 或更高版本上，我们应该使用一个专门为迭代集合和数组而开发的[增强的`for `循环](/web/20220803133836/https://www.baeldung.com/java-for-loop)语句。此外，它使我们的循环更加简洁易读。

**此外，使用增强的`for`循环有助于我们完全避免`ArrayIndexOutOfBoundsException` ，因为它不涉及索引变量**:

```java
for (int number : numbers) {
    sum += number;
}
```

在这里，我们不必担心索引。在每次迭代中，增强的`for`循环选取一个元素，并将其分配给循环变量`number`。因此，它完全避免了`ArrayIndexOutOfBoundsException`。

## 4.`IndexOutOfBoundsException`对`ArrayIndexOutOfBoundsException`

`IndexOutOfBoundsException`发生在我们试图访问某种类型的索引(`String`、数组、`List`等)时。)超出了它的范围。是`ArrayIndexOutOfBoundsException`和`StringIndexOutOfBoundsException`的超类。

与`ArrayIndexOutOfBoundsException`类似，当我们试图访问一个索引超过长度的`String` 的字符时，`StringIndexOutOfBoundsException` 被抛出。

## 5.结论

在本文中，我们探讨了`ArrayIndexOutOfBoundsException`，一些关于它如何发生的例子，以及一些避免它的常用技术。

和往常一样，所有这些例子的源代码都可以在 GitHub 上找到。