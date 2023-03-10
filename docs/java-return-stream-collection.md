# 返回流与集合

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-return-stream-collection>

## 1.概观

Java 8 流 API 提供了一种比 Java 集合[更有效的方法来呈现或处理结果集。然而，决定何时使用哪一个是一个常见的难题。](/web/20220628065735/https://www.baeldung.com/java-collections)

在本文中，我们将探索`Stream`和`Collection`，并讨论适合它们各自用途的各种场景。

## 2.`Collection`对`Stream`

Java `Collection`通过提供 [`List`](/web/20220628065735/https://www.baeldung.com/java-linkedlist) 、 [`Set`](/web/20220628065735/https://www.baeldung.com/java-hashset) 、 [`Map`](/web/20220628065735/https://www.baeldung.com/java-hashmap) 等数据结构，提供了高效的数据存储和处理机制。

然而，流 API 对于在不需要中间存储的情况下对数据执行各种操作是有用的。因此，`Stream`的工作方式类似于直接从底层存储中访问数据，如集合和 [I/O 资源](/web/20220628065735/https://www.baeldung.com/java-io)。

此外，集合主要关注提供对数据的访问和修改数据的方法。另一方面，流与有效传输数据有关。

尽管 Java 允许从`Collection`到`Stream`的简单转换，反之亦然，但是知道哪种机制是呈现/处理结果集的最佳机制还是很方便的。

例如，我们可以使用`stream`和 `parallelStream`方法将`Collection`转换成`Stream`:

```java
public Stream<String> userNames() {
    ArrayList<String> userNameSource = new ArrayList<>();
    userNameSource.add("john");
    userNameSource.add("smith");
    userNameSource.add("tom");
    return userNames.stream();
} 
```

类似地，我们可以使用流 API 的`collect`方法将`Stream`转换成`Collection`:

```java
public List<String> userNameList() {
    return userNames().collect(Collectors.toList());
}
```

这里，我们使用`Collectors.toList()`方法将 [a `Stream`转换成了`List`。同样，我们可以将](/web/20220628065735/https://www.baeldung.com/java-8-collectors#1-collectorstolist)[中的`Stream`转换成`Set`中的](/web/20220628065735/https://www.baeldung.com/java-8-collectors#2-collectorstoset)或将[转换成`Map`中的](/web/20220628065735/https://www.baeldung.com/java-8-collectors#4-collectorstomap):

```java
public static Set<String> userNameSet() {
    return userNames().collect(Collectors.toSet());
}

public static Map<String, String> userNameMap() {
    return userNames().collect(Collectors.toMap(u1 -> u1.toString(), u1 -> u1.toString()));
} 
```

## 3.什么时候回一个`Stream`？

### 3.1.物化成本高

Stream API 提供了惰性执行和动态结果过滤，这是降低物化成本的最有效方法。

例如，[Java NIO`Files`类](/web/20220628065735/https://www.baeldung.com/reading-file-in-java#read-file-with-path-readalllines)中的`readAllLines`方法呈现一个文件的所有行，为此 JVM 必须在内存中保存整个文件内容。因此，这个方法在返回行列表时会有很高的物化成本。

然而，[`Files`类也提供了返回`Stream`](/web/20220628065735/https://www.baeldung.com/reading-file-in-java#%20id=) 的`lines`方法，我们可以用它来呈现所有的行，或者使用`limit`方法更好地限制结果集的大小——两者都是延迟执行:

```java
Files.lines(path).limit(10).collect(toList());
```

同样，`Stream`不会执行中间操作，直到我们对它调用[终端操作，如`forEach`](/web/20220628065735/https://www.baeldung.com/java-collection-stream-foreach) :

```java
userNames().filter(i -> i.length() >= 4).forEach(System.out::println);
```

因此，`Stream`避免了与过早具体化相关的成本。

### 3.2.大型或无限的结果

设计用于获得较大或无限结果的更好性能。因此，对于这样的用例，使用一个`Stream`总是一个好主意。

此外，在无限结果的情况下，我们通常不会处理整个结果集。因此，Stream API 的内置特性如`filter`和`limit`证明在处理期望的结果集时非常方便，使得`Stream`成为更好的选择。

### 3.3.灵活性

非常灵活，允许以任何形式或顺序处理结果。

当我们不想向消费者强加一致的结果集时，`Stream`是一个显而易见的选择。此外，当我们想为消费者提供急需的灵活性时，`Stream`是一个很好的选择。

例如，我们可以使用流 API 上可用的各种操作来过滤/排序/限制结果:

```java
public static Stream<String> filterUserNames() {
    return userNames().filter(i -> i.length() >= 4);
}

public static Stream<String> sortUserNames() {
    return userNames().sorted();
}

public static Stream<String> limitUserNames() {
    return userNames().limit(3);
}
```

### 3.4.功能行为

A `Stream`是功能性的。当以不同的方式处理时，它不允许对源进行任何修改。因此，呈现不可变的结果集是更好的选择。

例如，让我们从初选`Stream`得到`filter`和`limit`一组结果:

```java
userNames().filter(i -> i.length() >= 4).limit(3).forEach(System.out::println);
```

这里，`Stream`上的`filter`和`limit` 等操作每次都返回一个新的`Stream`，并且不修改`userNames`方法提供的源`Stream`。

## 4.什么时候回一个`Collection`？

### 4.1.物化成本低

当呈现或处理涉及低物化成本的结果时，我们可以选择集合而不是流。

换句话说，Java 通过一开始就计算所有的元素，急切地构造了一个`Collection`。因此，结果集很大的`Collection`在物化时会给堆内存带来很大压力。

因此，我们应该考虑用一个`Collection`来呈现一个结果集，这个结果集不会给堆内存带来太大的物化压力。

### 4.2.固定格式

我们可以使用`Collection`为用户强制执行一致的结果集。例如，`Collection` s 就像 [`TreeSet`](/web/20220628065735/https://www.baeldung.com/java-tree-set) 和 [`TreeMap`](/web/20220628065735/https://www.baeldung.com/java-treemap) 返回自然排序的结果。

换句话说，通过使用`Collection`，我们可以确保每个消费者以相同的顺序接收和处理相同的结果集。

### 4.3.可重用结果

当结果以`Collection`的形式返回时，可以很容易地多次遍历它。但是， [a `Stream`穿越一次视为消耗，重用](/web/20220628065735/https://www.baeldung.com/java-stream-operated-upon-or-closed-exception)时抛出`IllegalStateException` :

```java
public static void tryStreamTraversal() {
    Stream<String> userNameStream = userNames();
    userNameStream.forEach(System.out::println);

    try {
        userNameStream.forEach(System.out::println);
    } catch(IllegalStateException e) {
        System.out.println("stream has already been operated upon or closed");
    }
}
```

因此，当消费者显然会多次遍历结果时，返回一个`Collection`是更好的选择。

### 4.4.修正

与`Stream`不同的是，`Collection`允许修改元素，比如从结果源中添加或删除元素。因此，我们可以考虑使用集合来返回结果集，以允许消费者进行修改。

例如，我们可以使用`add` / `remove`方法修改一个`ArrayList`:

```java
userNameList().add("bob");
userNameList().add("pepper");
userNameList().remove(2);
```

类似地，像`put`和`remove`这样的方法允许修改地图:

```java
Map<String, String> userNameMap = userNameMap();
userNameMap.put("bob", "bob");
userNameMap.remove("alfred");
```

### 4.5.内存中的结果

此外，当集合形式的物化结果已经存在于内存中时，使用`Collection`是一个显而易见的选择。

## 5.结论

在本文中，我们比较了`Stream`和`Collection`，并研究了适合它们的各种场景。

我们可以得出结论,`Stream`是呈现大型或无限结果集的绝佳选择，具有惰性初始化、急需的灵活性和函数行为等优点。

然而，当我们需要结果的一致形式时，或者当涉及低物化时，我们应该选择一个`Collection`而不是一个`Stream`。

和往常一样，源代码可以在 GitHub 的[上找到。](https://web.archive.org/web/20220628065735/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-streams-3)