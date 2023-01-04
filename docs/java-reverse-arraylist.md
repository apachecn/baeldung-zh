# 在 Java 中反转数组列表

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-reverse-arraylist>

## 1.概观

[`ArrayList`](/web/20221208143817/https://www.baeldung.com/java-arraylist) 是 Java 中常用的`List`实现。

在本教程中，我们将探索如何反转一个`ArrayList`。

## 2.问题简介

和往常一样，我们通过一个例子来理解问题。假设我们有一个`Integer`的`List`:

```java
​List<Integer> aList = new ArrayList<>(Arrays.asList(1, 2, 3, 4, 5, 6, 7));
```

反转后，我们期待会有结果:

```java
List<Integer> EXPECTED = new ArrayList<>(Arrays.asList(7, 6, 5, 4, 3, 2, 1));
```

因此，需求看起来非常简单。然而，这个问题可能有几个变种:

*   将一个`List`反转到位
*   反转一个`List`并将结果作为一个新的`List`对象返回

我们将在本教程中讨论这两种情况。

Java 标准库提供了一个助手方法来完成这项工作。我们将看到如何使用这种方法快速解决问题。

此外，考虑到我们中的一些人可能正在学习 Java，我们将讨论反转`List`的两个有趣但有效的实现。

接下来，让我们看看他们的行动。

## 3.使用标准`Collections.reverse`方法

**Java 标准库已经提供了`[Collections.reverse](https://web.archive.org/web/20221208143817/https://docs.oracle.com/en/java/javase/17/docs/api/java.base/java/util/Collections.html#reverse(java.util.List))`方法来反转给定`List`中元素的顺序。**

这个方便的方法进行就地反转，这将反转它接收到的原始列表中的顺序。但是，首先，让我们创建一个单元测试方法来理解它:

```java
List<Integer> aList = new ArrayList<>(Arrays.asList(1, 2, 3, 4, 5, 6, 7));
Collections.reverse(aList);
assertThat(aList).isEqualTo(EXPECTED); 
```

当我们执行上面的测试时，它通过了。正如我们所见，我们已经将`aList`对象传递给了`reverse`方法，然后`aList`对象中元素的顺序被颠倒。

如果我们不想改变原来的`List`，并且希望得到一个新的`List`对象来包含逆序的元素，我们可以向`reverse`方法传递一个新的`List `对象:

```java
List<Integer> originalList = new ArrayList<>(Arrays.asList(1, 2, 3, 4, 5, 6, 7));
List<Integer> aNewList = new ArrayList<>(originalList);
Collections.reverse(aNewList);

assertThat(aNewList).isNotEqualTo(originalList).isEqualTo(EXPECTED); 
```

这样，我们保持`originalList`不变，而`aNewList`中元素的顺序是相反的。

从上面的两个例子中我们可以看到，标准的`Collections.reverse`方法对于反转一个`List`非常方便。

然而，如果我们正在学习 Java，我们可能希望自己练习实现一个“反向”方法。

接下来，让我们探索几个好的实现:一个使用递归，另一个使用简单的循环。

## 4.使用递归反转一个`List`

首先，让我们使用[递归](/web/20221208143817/https://www.baeldung.com/java-recursion)技术实现我们自己的列表反转方法。首先，让我们看一下实现:

```java
public static <T> void reverseWithRecursion(List<T> list) {
    if (list.size() > 1) {
        T value = list.remove(0);
        reverseWithRecursion(list);
        list.add(value);
    }
} 
```

正如我们所看到的，上面的实现看起来非常紧凑。现在，让我们了解它是如何工作的。

我们递归逻辑中的停止条件是`list.size() <=1`。换句话说，**如果`list`对象是空的或者只包含一个元素，我们就停止递归**。

在每个递归调用中，我们执行“`T value = list.remove(0)`”，从列表中弹出第一个元素。它是这样工作的:

```java
recursion step 0: value = null, list = (1, 2, 3, ... 7)
   |_ recursion step 1: value = 1, list = (2, 3, 4,...7)
      |_ recursion step 2: value = 2, list = (3, 4, 5, 6, 7)
         |_ recursion step 3: value = 3, list = (4, 5, 6, 7)
            |_ ...
               |_ recursion step 6: value = 6, list = (7) 
```

正如我们所看到的，当`list`对象只包含一个元素(7)时，我们停止递归，然后从底部开始执行`list.add(value) `。也就是说，我们首先在列表末尾添加 6，然后是 5，然后是 4，以此类推。最后，列表中元素的顺序被就地颠倒了。此外，**该方法在线性时间**内运行。

接下来，让我们创建一个测试来验证我们的递归实现是否按预期工作:

```java
List<Integer> aList = new ArrayList<>(Arrays.asList(1, 2, 3, 4, 5, 6, 7));
ReverseArrayList.reverseWithRecursion(aList);
assertThat(aList).isEqualTo(EXPECTED); 
```

如果我们进行测试，就会通过。所以，我们的递归实现解决了这个问题。

## 5.使用迭代反转一个`List`

我们刚刚用递归颠倒了列表。或者，我们可以使用迭代来解决这个问题。

首先，让我们看一下实现:

```java
public static <T> void reverseWithLoop(List<T> list) {
    for (int i = 0, j = list.size() - 1; i < j; i++) {
        list.add(i, list.remove(j));
    }
} 
```

正如我们所见，迭代实现也非常简洁。然而，我们只有一个`for`循环，并且在循环体中，我们只有一条语句。

接下来，我们来了解一下它的工作原理。

我们在给定的列表上定义了两个指针，`i`和`j`。指针`j`总是指向列表中的最后一个元素。但是点`i `从 0 增加到 T4

我们在每个迭代步骤中移除最后一个元素，并使用`list.add(i, list.remove(j))`将其填充到`i-th`位置。当`i`到达`j-1`时，循环结束，我们颠倒了列表:

```java
Iteration step 0: i = j = null, list = (1, 2, 3,...7)
Iteration step 1: i = 0; j = 6 
                  |_ list.add(0, list.remove(6))
                  |_ list = (7, 1, 2, 3, 4, 5, 6)
Iteration step 2: i = 1; j = 6 
                  |_ list.add(1, list.remove(6))
                  |_ list = (7, 6, 1, 2, 3, 4, 5)
...
Iteration step 5: i = 4; j = 6 
                  |_ list.add(4, list.remove(6))
                  |_ list = (7, 6, 5, 4, 3, 1, 2)
Iteration step 6: i = 5; j = 6 
                  |_ list.add(5, list.remove(6))
                  |_ list = (7, 6, 5, 4, 3, 2, 1)
```

**该方法也在线性时间**内运行。

最后，让我们测试我们的方法，看看它是否如预期的那样工作:

```java
List<Integer> aList = new ArrayList<>(Arrays.asList(1, 2, 3, 4, 5, 6, 7));
ReverseArrayList.reverseWithLoop(aList);
assertThat(aList).isEqualTo(EXPECTED); 
```

当我们运行上面的测试时，它通过了。

## 6.结论

在本文中，我们已经通过例子解决了如何反转一个`ArrayList`。标准的`Collections.reverse`方法很容易解决这个问题。

然而，如果我们想创建自己的反转实现，我们已经学习了两种有效的原地反转方法。

像往常一样，这篇文章的完整代码可以在 GitHub 上找到[。](https://web.archive.org/web/20221208143817/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-collections-list-4)