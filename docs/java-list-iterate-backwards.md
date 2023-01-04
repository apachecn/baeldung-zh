# 向后遍历一个列表

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-list-iterate-backwards>

## 1。概述

在这个快速教程中，我们将学习在 Java 中向后遍历列表的各种方法。

## 2。`Iterator`在爪哇

一个 [`Iterator`](https://web.archive.org/web/20220628064207/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/Iterator.html) 是 [Java 集合框架](https://web.archive.org/web/20220628064207/https://docs.oracle.com/javase/8/docs/technotes/guides/collections/index.html)中的一个接口，它允许我们迭代集合中的元素。它是在 Java 1.2 中作为`[Enumeration](https://web.archive.org/web/20220628064207/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/Enumeration.html). `的替代而引入的

## 3。使用核心 Java 向后迭代

### 3.1。反向`for` 循环

最简单的实现是使用一个`for` 循环来从列表的最后一个元素开始**，并且当我们到达列表的开始时递减索引**:

```java
for (int i = list.size(); i-- > 0; ) {
    System.out.println(list.get(i));
}
```

### 3.2。`ListIterator`

**我们可以用一个 [`ListIterator`](https://web.archive.org/web/20220628064207/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/ListIterator.html) 来迭代列表中的元素。**

将列表的大小作为索引提供给`ListIterator`会给我们一个指向列表末尾的迭代器:

```java
ListIterator listIterator = list.listIterator(list.size());
```

这个迭代器现在允许我们以相反的方向遍历列表:

```java
while (listIterator.hasPrevious()) {
    System.out.println(listIterator.previous());
}
```

### 3.3。`Collections.reverse()`

**Java 中的 [`Collections`](https://web.archive.org/web/20220628064207/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/Collections.html#reverse(java.util.List)) 类提供了一个静态方法来反转指定列表中元素的顺序:**

```java
Collections.reverse(list);
```

然后，反向列表可用于向后迭代原始元素:

```java
for (String item : list) {
    System.out.println(item);
}
```

然而，**这个方法通过改变元素的顺序来颠倒实际的列表，在很多情况下可能并不理想。**

## 4。使用 Apache 的`ReverseListIterator` 向后迭代

**`Apache Commons Collections`库有一个很好的`[ReverseListIterator](https://web.archive.org/web/20220628064207/https://commons.apache.org/proper/commons-collections/javadocs/api-3.2.2/org/apache/commons/collections/iterators/ReverseListIterator.html)`类，允许我们循环遍历一个列表中的元素，而不需要实际反转它。**

在开始之前，我们需要从 [Maven Central](https://web.archive.org/web/20220628064207/https://search.maven.org/classic/#search%7Cgav%7C1%7Cg%3A%22org.apache.commons%22%20AND%20a%3A%22commons-collections4%22) 导入最新的依赖项:

```java
<dependency>
    <groupId>org.apache.commons</groupId>
    <artifactId>commons-collections4</artifactId>
    <version>4.1</version>
</dependency>
```

我们可以通过将原始列表作为构造函数参数传递来创建一个新的`ReverseListIterator`:

```java
ReverseListIterator reverseListIterator = new ReverseListIterator(list);
```

然后我们可以使用这个迭代器反向遍历列表:

```java
while (reverseListIterator.hasNext()) {
    System.out.println(reverseListIterator.next());
}
```

## 5。使用番石榴的`Lists.reverse()` 向后迭代

类似地， **Google Guava 库也在其`[Lists](https://web.archive.org/web/20220628064207/https://google.github.io/guava/releases/snapshot-jre/api/docs/com/google/common/collect/Lists.html#reverse-java.util.List-)`类**中提供了一个静态的`reverse()`方法，该方法返回所提供列表的反向视图。

最新的番石榴版本可以在 [Maven Central](https://web.archive.org/web/20220628064207/https://search.maven.org/classic/#search%7Cgav%7C1%7Cg%3A%22com.google.guava%22%20AND%20a%3A%22guava%22) 上找到:

```java
<dependency>
    <groupId>com.google.guava</groupId>
    <artifactId>guava</artifactId>
    <version>31.0.1-jre</version>
</dependency>
```

调用`Lists`类上的静态方法`reverse()`以相反的方式给出了列表:

```java
List<String> reversedList = Lists.reverse(list);
```

然后，反向列表可用于在原始列表上向后迭代:

```java
for (String item : reversedList) {
    System.out.println(item);
}
```

这个方法**返回一个新的列表，其中原始列表的元素以相反的顺序排列**。

## 6。结论

在本文中，我们研究了 Java 中向后遍历列表的不同方法。我们使用核心 Java 以及流行的第三方库浏览了一些例子。

GitHub 上的[提供了本文的源代码和相关的测试用例。](https://web.archive.org/web/20220628064207/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-collections-list)