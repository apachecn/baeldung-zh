# Java 数组列表指南

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-arraylist>

## 1。概述

在本文中，我们将看看 Java 集合框架中的`ArrayList`类。我们将讨论它的属性、常见用例以及它的优缺点。

驻留在 Java 核心库中，所以你不需要任何额外的库。为了使用它，只需添加以下导入语句:

```java
import java.util.ArrayList;
```

`List`表示值的有序序列，其中某些值可能出现多次。

`ArrayList`是构建在数组之上的`List`实现之一，它能够随着您添加/删除元素而动态增长和收缩。元素可以很容易地通过从零开始的索引来访问。该实现具有以下属性:

*   随机存取需要`O(1)`时间
*   添加元素需要摊余常数时间`O(1)`
*   插入/删除需要`O(n)`时间
*   对于未排序的数组，搜索需要花费`O(n)`时间，而对于已排序的数组，搜索需要花费`O(log n)`时间

## 2。创造一个`ArrayList`

有几个构造函数，我们将在本节中介绍它们。

首先，注意到`ArrayList`是一个泛型类，所以你可以用任何你想要的类型来参数化它，编译器会确保，例如，你不能把`Integer`的值放在`Strings`的集合中。此外，从集合中检索元素时，不需要强制转换元素。

其次，使用通用接口`List`作为变量类型是一个很好的实践，因为它将它从特定的实现中分离出来。

### 2.1。默认无参数构造函数

```java
List<String> list = new ArrayList<>();
assertTrue(list.isEmpty());
```

我们只是创建一个空的`ArrayList`实例。

### 2.2。接受初始容量的构造器

```java
List<String> list = new ArrayList<>(20);
```

在这里，您可以指定基础数组的初始长度。这可以帮助您在添加新项目时避免不必要的大小调整。

### 2.3。建造师接受`Collection`

```java
Collection<Integer> number 
  = IntStream.range(0, 10).boxed().collect(toSet());

List<Integer> list = new ArrayList<>(numbers);
assertEquals(10, list.size());
assertTrue(numbers.containsAll(list));
```

注意，`Collection`实例元素用于填充底层数组。

## 3。`ArrayList`添加元素到

您可以在末尾或特定位置插入元素:

```java
List<Long> list = new ArrayList<>();

list.add(1L);
list.add(2L);
list.add(1, 3L);

assertThat(Arrays.asList(1L, 3L, 2L), equalTo(list));
```

您也可以一次插入一个集合或多个元素:

```java
List<Long> list = new ArrayList<>(Arrays.asList(1L, 2L, 3L));
LongStream.range(4, 10).boxed()
  .collect(collectingAndThen(toCollection(ArrayList::new), ys -> list.addAll(0, ys)));
assertThat(Arrays.asList(4L, 5L, 6L, 7L, 8L, 9L, 1L, 2L, 3L), equalTo(list));
```

## 4。`ArrayList`迭代

有两种类型的迭代器可用:`Iterator`和`ListIterator`。

前者让您有机会单向遍历列表，而后者允许您双向遍历列表。

这里我们将只向您展示`ListIterator`:

```java
List<Integer> list = new ArrayList<>(
  IntStream.range(0, 10).boxed().collect(toCollection(ArrayList::new))
);
ListIterator<Integer> it = list.listIterator(list.size());
List<Integer> result = new ArrayList<>(list.size());
while (it.hasPrevious()) {
    result.add(it.previous());
}

Collections.reverse(list);
assertThat(result, equalTo(list));
```

您也可以使用迭代器搜索、添加或删除元素。

## 5。搜索`ArrayList`

我们将使用一个集合演示搜索是如何工作的:

```java
List<String> list = LongStream.range(0, 16)
  .boxed()
  .map(Long::toHexString)
  .collect(toCollection(ArrayList::new));
List<String> stringsToSearch = new ArrayList<>(list);
stringsToSearch.addAll(list);
```

### 5.1。搜索未排序的列表

为了找到一个元素，你可以使用`indexOf()`或`lastIndexOf()`方法。它们都接受一个对象并返回`int`值:

```java
assertEquals(10, stringsToSearch.indexOf("a"));
assertEquals(26, stringsToSearch.lastIndexOf("a"));
```

如果你想找到满足一个谓词的所有元素，你可以使用 Java 8 `Stream API` (在这里阅读更多关于它的[)像这样使用`Predicate`:](/web/20220529012209/https://www.baeldung.com/java-8-streams)

```java
Set<String> matchingStrings = new HashSet<>(Arrays.asList("a", "c", "9"));

List<String> result = stringsToSearch
  .stream()
  .filter(matchingStrings::contains)
  .collect(toCollection(ArrayList::new));

assertEquals(6, result.size());
```

也可以使用`for`循环或迭代器:

```java
Iterator<String> it = stringsToSearch.iterator();
Set<String> matchingStrings = new HashSet<>(Arrays.asList("a", "c", "9"));

List<String> result = new ArrayList<>();
while (it.hasNext()) {
    String s = it.next();
    if (matchingStrings.contains(s)) {
        result.add(s);
    }
}
```

### 5.2。搜索排序列表

如果你有一个排序的数组，那么你可以使用一个比线性搜索更快的二分搜索法算法:

```java
List<String> copy = new ArrayList<>(stringsToSearch);
Collections.sort(copy);
int index = Collections.binarySearch(copy, "f");
assertThat(index, not(equalTo(-1)));
```

注意，如果没有找到元素，那么将返回-1。

## 6。`ArrayList`从中删除元素

为了删除一个元素，你应该找到它的索引，然后通过`remove()`方法执行删除。此方法的重载版本，它接受一个对象，搜索该对象并移除第一个相等元素:

```java
List<Integer> list = new ArrayList<>(
  IntStream.range(0, 10).boxed().collect(toCollection(ArrayList::new))
);
Collections.reverse(list);

list.remove(0);
assertThat(list.get(0), equalTo(8));

list.remove(Integer.valueOf(0));
assertFalse(list.contains(0));
```

但是在使用像`Integer`这样的装箱类型时要小心。为了删除一个特定的元素，你应该首先框`int`值，否则，一个元素将被它的索引删除。

您也可以使用前面提到的`Stream API`来删除几个项目，但我们不会在这里显示它。为此，我们将使用迭代器:

```java
Set<String> matchingStrings
 = HashSet<>(Arrays.asList("a", "b", "c", "d", "e", "f"));

Iterator<String> it = stringsToSearch.iterator();
while (it.hasNext()) {
    if (matchingStrings.contains(it.next())) {
        it.remove();
    }
}
```

## 7 .**。总结**

在这篇简短的文章中，我们了解了 Java 中的数组列表。

我们展示了如何创建一个`ArrayList`实例，如何使用不同的方法添加、查找或删除元素。

像往常一样，你可以在 GitHub 上找到所有的代码样本[。](https://web.archive.org/web/20220529012209/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-collections-array-list)