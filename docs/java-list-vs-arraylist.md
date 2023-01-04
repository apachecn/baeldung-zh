# Java 中的 List 与 ArrayList

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-list-vs-arraylist>

## 1.概观

在本文中，我们将研究使用 [`List`](/web/20220524051922/https://www.baeldung.com/tag/java-list) 和 [`ArrayList`](/web/20220524051922/https://www.baeldung.com/java-arraylist) 类型之间的区别。

首先，我们将看到一个使用`ArrayList`的示例实现。然后，我们将切换到`List`界面，比较不同之处。

## 2.使用`ArrayList`

**T0 是 Java 中最常用的`List`实现**之一。它建立在一个数组之上，可以随着我们添加/删除元素而动态地增长和收缩。当我们知道列表会变大时，用初始容量初始化列表是很好的:

```java
ArrayList<String> list = new ArrayList<>(25);
```

通过使用`ArrayList`作为引用类型，我们可以在`ArrayList` API 中使用不在`List` API 中的方法——例如，`ensureCapacity, trimToSize`或`removeRange`。

### 2.1.快速示例

让我们编写一个基本的乘客处理应用程序:

```java
public class ArrayListDemo {
    private ArrayList<Passenger> passengers = new ArrayList<>(20);

    public ArrayList<Passenger> addPassenger(Passenger passenger) {
        passengers.add(passenger);
        return passengers;
    }

    public ArrayList<Passenger> getPassengersBySource(String source) {
        return new ArrayList<Passenger>(passengers.stream()
            .filter(it -> it.getSource().equals(source))
            .collect(Collectors.toList()));
    }

    // Few other functions to remove passenger, get by destination, ... 
}
```

这里，我们使用了`ArrayList`类型来存储和返回乘客列表。因为乘客的最大数量是 20，所以列表的初始容量被设置为 20。

### 2.2.可变大小数据的问题是

只要我们不需要改变我们正在使用的`List`的类型，上面的实现工作得很好。在我们的例子中，我们选择了`ArrayList`，并认为它满足了我们的需求。

然而，让我们假设随着应用程序的成熟，乘客的数量变化很大。例如，如果只有五个预订的乘客，初始容量为 20，那么[内存浪费](/web/20220524051922/https://www.baeldung.com/java-list-capacity-array-size#2-building-small-multiple-arraylists)是 75%。假设我们决定切换到一个更有内存效率的`List`。

### 2.3.更改实现类型

Java 提供了另一个名为`[LinkedList](/web/20220524051922/https://www.baeldung.com/java-linkedlist)`的`List`实现来存储可变大小的数据`.` **`LinkedList`使用链接节点的集合来存储和检索元素。**如果我们决定将基础实现从`ArrayList`更改为`LinkedList`会怎么样:

```java
private LinkedList<Passenger> passengers = new LinkedList<>();
```

**这一变化影响了应用程序的更多部分，因为演示应用程序中的所有函数都期望与`ArrayList`类型的**一起工作。

## 3.切换到`List`

让我们看看如何通过使用`List`接口类型来处理这种情况:

```java
private List<Passenger> passengers = new ArrayList<>(20);
```

这里，我们使用`List`接口作为引用类型，而不是更具体的`ArrayList`类型。我们可以将相同的原则应用于所有的函数调用和返回类型。例如:

```java
public List<Passenger> getPassengersBySource(String source) {
    return passengers.stream()
        .filter(it -> it.getSource().equals(source))
        .collect(Collectors.toList());
}
```

现在，让我们考虑同一个问题陈述，并将基本实现更改为`LinkedList`类型。`ArrayList`和`LinkedList`类都是`List`接口的实现。因此，我们现在可以安全地更改基本实现，而不会对应用程序的其他部分造成任何干扰。该类仍然可以像以前一样编译和运行。

## 4.比较这些方法

如果我们在整个程序中使用一个具体的列表类型，那么我们所有的代码都不必要地与那个列表类型相关联。这使得将来更难更改列表类型。

此外，Java 中可用的实用程序类返回抽象类型，而不是具体类型。例如，下面的实用函数返回`List` 类型:

```java
Collections.singletonList(...), Collections.unmodifiableList(...)
```

```java
Arrays.asList(...), ArrayList.sublist(...)
```

具体来说，`ArrayList.sublist`返回`List`类型，即使原始对象是`ArrayList`类型。因此，`List` API 中的方法不保证返回相同类型的列表。

## 5.结论

在本文中，我们研究了使用`List`和`ArrayList`类型的区别和最佳实践。

我们看到了引用一个特定的类型是如何使应用程序在以后的某个时间点容易被更改的。具体来说，当底层实现发生变化时，它会影响应用程序的其他层。因此，使用最抽象的类型(顶级类/接口)通常比使用特定的引用类型更好。

与往常一样，GitHub 上的[提供了示例的源代码。](https://web.archive.org/web/20220524051922/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-collections-list-3)