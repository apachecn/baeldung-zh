# Java 枚举和迭代器的区别

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-enumeration-vs-iterator>

## 1.概观

在本教程中，我们将学习 Java 中的 [`Enumeration`](https://web.archive.org/web/20221208143854/https://docs.oracle.com/javase/7/docs/api/java/util/Enumeration.html) 和 [`Iterator`](/web/20221208143854/https://www.baeldung.com/java-iterator) 。我们还将学习如何在代码中使用它们，以及它们之间的区别。

## 2.`Enumeration`和`Iterator`简介

在本节中，我们将从概念上了解`Enumeration`和`Iterator`及其用法。

### 2.1.`Enumeration`

[`Enumeration`](https://web.archive.org/web/20221208143854/https://docs.oracle.com/javase/7/docs/api/java/util/Enumeration.html) 从 1.0 版本开始就出现在 Java 中。这是一个接口，任何实现**都允许一个接一个地访问元素**。简单来说，它用于迭代一组对象，如 [`Vector`](https://web.archive.org/web/20221208143854/https://docs.oracle.com/javase/8/docs/api/java/util/Vector.html) 和 [`Hashtable`](/web/20221208143854/https://www.baeldung.com/java-hash-table) 。

让我们来看一个`Enumeration`的例子:

```
Vector<Person> people = new Vector<>(getPersons());
Enumeration<Person> enumeration = people.elements();
while (enumeration.hasMoreElements()) {
    System.out.println("First Name = " + enumeration.nextElement().getFirstName());
}
```

这里，我们使用`Enumeration`打印一个`Person` 的`firstName`。`elements()` 方法提供了对`Enumeration`的引用，通过使用它我们可以逐个访问元素。

### 2.2.`Iterator`

[`Iterator`](https://web.archive.org/web/20221208143854/https://docs.oracle.com/javase/8/docs/api/java/util/Iterator.html) 从 1.2 开始就出现在 Java **中，用于迭代同样在同一版本中引入的[集合](/web/20221208143854/https://www.baeldung.com/java-collections)** 。

接下来，让我们使用`Iterator`打印一个`Person` 的`firstName`。`iterator()` 提供了对`Iterator`的引用，通过它我们可以逐个访问元素:

```
List<Person> persons = getPersons();
Iterator<Person> iterator = persons.iterator();
while (iterator.hasNext()) {
    System.out.println("First Name = " + iterator.next().getFirstName());
}
```

因此，我们可以看到 **`Enumeration`和`Iterator`分别从 1.0 和 1.2 开始出现在 Java 中，并且用于一次迭代一个对象集合**。

## 3.`Enumeration`和`Iterator`的区别

在这张表中，我们将了解`Enumeration`和`Iterator:`之间的区别

| **枚举** | **迭代器** |
| 现在从 Java 1.0 开始枚举`Vector`和`Hashtable` | 从 Java 1.2 开始，迭代集合，如`List`、`Set`、`Map,`等 |
| 包含两种方法:`hasMoreElements()`和`nextElement()` | 包含三种方法:`hasNext(), next()` 和 `remove()` |
| 方法有很长的名字 | 方法有简短的名字 |
| 没有方法在迭代时移除元素 | 迭代时必须`remove() `删除元素 |
| 在 Java 9 中添加的`asIterator()`在`Enumeration`之上给出了`Iterator`。然而，`remove()` 在这种特殊情况下抛出了`UnsupportedOperationException` | `forEachRemaining()` Java 8 中添加的对剩余元素执行动作 |

## 4.结论

在本文中，我们已经了解了`Enumeration`和`Iterator`，如何通过代码示例使用它们，以及它们之间的各种差异。

本文中使用的所有代码示例都可以从 GitHub 上的[处获得。](https://web.archive.org/web/20221208143854/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-collections-4)