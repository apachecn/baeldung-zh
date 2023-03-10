# 移除数组的第一个元素

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-array-remove-first-element>

## 1.概观

在本教程中，**我们将看看如何移除数组**的第一个元素。

此外，我们还将看到如何使用来自 [Java Collections Framework](https://web.archive.org/web/20221208143956/https://docs.oracle.com/javase/8/docs/technotes/guides/collections/index.html) 的数据结构使事情变得更加简单。

## 2.使用`Arrays.copyOfRange()`

首先，**在 Java** 中移除数组的元素在技术上是不可能的。引用[官方文件](https://web.archive.org/web/20221208143956/https://docs.oracle.com/javase/tutorial/java/nutsandbolts/arrays.html):

数组是一个容器对象，它保存固定数量的单一类型的值。数组的长度是在创建数组时确定的。创作后，其长度是固定的。”

这意味着**只要我们直接使用一个数组，我们所能做的就是创建一个更小的新数组，它不包含第一个元素**。

幸运的是，JDK 提供了一个我们可以使用的方便的静态帮助函数，叫做`Arrays.copyOfRange()`:

```java
String[] stringArray = {"foo", "bar", "baz"};
String[] modifiedArray = Arrays.copyOfRange(stringArray, 1, stringArray.length);
```

**注意，该操作的成本为`O(n)`，因为它每次都会创建一个新数组。**

当然，这是一种从数组中移除元素的麻烦方法，如果您经常进行这样的操作，那么使用 Java Collections 框架可能更明智。

## 3.使用`List`实现

为了保持大致相同的数据结构语义(可通过索引访问的有序元素序列)，使用 [`List`接口](https://web.archive.org/web/20221208143956/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/List.html)的实现是有意义的。

两个最常见的实现是`ArrayList`和`LinkedList`。

假设我们有以下`List` s:

```java
List<String> arrayList = new ArrayList<>();
// populate the ArrayList

List<String> linkedList = new LinkedList<>();
// populate the LinkedList
```

由于两个类实现了相同的接口，因此移除第一个元素的示例代码看起来是相同的:

```java
arrayList.remove(0);
linkedList.remove(0);
```

**在`ArrayList`的情况下，移除的成本是`O(n)`，而`LinkedList`的成本是`O(1)`。**

现在，这并不意味着我们应该在任何地方使用一个`LinkedList`作为缺省值，因为检索一个对象的成本正好相反。调用`get(i)`的成本在`ArrayList`的情况下是`O(1)`，在`LinkedList`的情况下是`O(n)`。

## 4.结论

我们已经看到了如何在 Java 中移除数组的第一个元素。此外，我们已经了解了如何使用 Java Collections 框架实现相同的结果。

你可以在 GitHub 上找到示例代码[。](https://web.archive.org/web/20221208143956/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-arrays-operations-basic)