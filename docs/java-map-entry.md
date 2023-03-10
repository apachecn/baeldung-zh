# 使用地图。入门级 Java 类

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-map-entry>

## 1.概观

我们经常使用映射来存储键值对的集合。然后，在某些时候，我们经常需要[迭代它们](/web/20220816155319/https://www.baeldung.com/java-iterate-map)。

在本教程中，我们将比较不同的地图迭代方法，突出使用`Map.Entry`的好处。然后，我们将学习如何使用`Map.Entry`来创建一个元组。最后，我们将创建一个有序的元组列表。

## 2.优化`Map`迭代

假设我们有一个以作者姓名为关键字的书名图:

```java
Map<String, String> map = new HashMap<>();

map.put("Robert C. Martin", "Clean Code");
map.put("Joshua Bloch", "Effective Java");
```

让我们比较两种从地图中获取所有键和值的方法。

### 2.1.使用`Map.keySet`

首先，考虑以下情况:

```java
for (String key : bookMap.keySet()) {
    System.out.println("key: " + key + " value: " + bookMap.get(key));
}
```

这里，循环在`keySet`上迭代。对于每个键，我们使用`Map.get`获得相应的值。虽然这是使用 map 中所有条目的一种显而易见的方法，**但是对于每个条目**它需要两个操作——一个是获取下一个键，一个是用`get`查找值。

如果我们只需要地图中的键， `keySet`是一个很好的选择。然而，有一种更快的方法可以同时获得键和值。

### 2.2.使用`Map.entrySet` 代替

让我们重写我们的迭代以使用`entrySet`:

```java
for (Map.Entry<String, String> book: bookMap.entrySet()) {
    System.out.println("key: " + book.getKey() + " value: " + book.getValue());
}
```

在这个例子中，我们的循环遍历了一组`Map.Entry`对象。由于 **`Map.Entry`将键和值一起存储在一个类中，所以我们在单个操作**中获得它们。

同样的规则也适用于使用 Java 8 流操作的。流过`entrySet`并处理`Entry`对象更有效，需要的代码也更少。

## 3.使用元组

元组是具有固定数量和顺序的元素的数据结构。我们可以认为`Map.Entry`是一个存储两个元素的元组——一个键和一个值。然而，由于`Map.Entry`是一个接口，我们需要一个实现类。在这一节中，我们将探索 JDK 提供的一个实现:`AbstractMap.SimpleEntry`。

### 3.1.创建元组

首先，考虑一下`Book`类:

```java
public class Book {
    private String title;
    private String author;

    public Book(String title, String author) {
        this.title = title;
        this.author = author;
    }
    ...
```

接下来，让我们创建一个以 ISBN 为键，以`Book`对象为值的`Map.Entry`元组:

```java
Map.Entry<String, Book> tuple; 
```

最后，让我们用`AbstractMap.SimpleEntry`实例化我们的元组:

```java
tuple = new AbstractMap.SimpleEntry<>("9780134685991", new Book("Effective Java 3d Edition", "Joshua Bloch")); 
```

### 3.2.创建元组的有序列表

当处理元组时，将它们作为有序列表通常很有用。

首先，我们将定义元组列表:

```java
List<Map.Entry<String, Book>> orderedTuples = new ArrayList<>(); 
```

其次，让我们在列表中添加一些条目:

```java
orderedTuples.add(new AbstractMap.SimpleEntry<>("9780134685991", 
  new Book("Effective Java 3d Edition", "Joshua Bloch")));
orderedTuples.add(new AbstractMap.SimpleEntry<>("9780132350884", 
  new Book("Clean Code","Robert C Martin")));
```

### 3.3.与 a 比较`Map`

为了比较与`Map`的区别，让我们添加一个新条目，它带有一个已经存在的键:

```java
orderedTuples.add(new AbstractMap.SimpleEntry<>("9780132350884", 
  new Book("Clean Code", "Robert C Martin"))); 
```

其次，我们将遍历列表，显示所有的键和值:

```java
for (Map.Entry<String, Book> tuple : orderedTuples) {
    System.out.println("key: " + tuple.getKey() + " value: " + tuple.getValue());
}
```

最后，让我们看看输出:

```java
key: 9780134685991 value: Book{title='Effective Java 3d Edition', author='Joshua Bloch'}
key: 9780132350884 value: Book{title='Clean Code', author='Robert C Martin'}
key: 9780132350884 value: Book{title='Clean Code', author='Robert C Martin'}
```

注意，我们可以有重复的键，不像基本的`Map`，每个键必须是唯一的。这是因为我们使用了一个`List`实现来存储我们的`SimpleEntry`对象，这意味着所有的对象都是相互独立的。

### 3.4.`Entry`对象列表

我们应该注意到`Entry`的目的不是作为一个通用元组。库类经常为[提供一个通用的`Pair`](/web/20220816155319/https://www.baeldung.com/java-pairs) 类。

然而，我们可能会发现，在为`Map`准备数据或从其中提取数据时，我们需要临时处理条目列表。

## 4.结论

在本文中，我们将`Map.entrySet`视为迭代映射键的替代方法。

然后我们看了如何将`Map.Entry`用作一个元组。

最后，我们创建了一个有序元组列表，将差异与基本的`Map`进行比较。

与往常一样，GitHub 上的示例代码[可用。](https://web.archive.org/web/20220816155319/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-collections-maps-5)