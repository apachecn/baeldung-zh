# Java IndexOutOfBoundsException“源不适合目标”

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-indexoutofboundsexception>

## 1.概观

在 Java 中，制作一个`List `的副本有时会产生一个`IndexOutOfBoundsException: “Source does not fit in dest”.`。在这个简短的教程中，我们将看看为什么在使用`[Collections.copy](/web/20221208143854/https://www.baeldung.com/java-copy-list-to-another#collectionscopy) `方法时会出现这个错误，以及如何解决它。我们还将寻找`Collections.copy `的替代方法来复制列表。

## 2.重现问题

让我们从使用`Collections.copy`方法创建一个`List `副本的方法开始:

```java
static List<Integer> copyList(List<Integer> source) {
    List<Integer> destination = new ArrayList<>(source.size());
    Collections.copy(destination, source);
    return destination;
}
```

这里，`copyList`方法创建一个新的列表[，其初始容量](/web/20221208143854/https://www.baeldung.com/java-arraylist#2-constructor-accepting-initial-capacity)等于源列表的大小。然后，它尝试将源列表的元素复制到目标列表:

```java
List<Integer> source = Arrays.asList(1, 2, 3, 4, 5);
List<Integer> copy = copyList(source);
```

然而，一旦我们调用了`copyList` 方法，它就会抛出一个异常`java.lang.IndexOutOfBoundsException: Source does not fit in dest`。

## 3.`Exception`的起因

让我们试着理解哪里出了问题。根据`Collections.copy` 方法的文档:

> 目标列表必须至少与源列表一样长。如果更长，目标列表中的其余元素不受影响。

在我们的例子中，我们使用一个初始容量等于源列表大小的构造函数创建了一个新的`List`。**它只是分配足够的内存，并没有真正定义元素。**新列表的大小保持为零，因为容量和大小是`List`的不同属性。

因此，当`Collections.copy `方法试图将源列表复制到目的列表时，它抛出`java.lang.IndexOutOfBoundsException.`

## 4.解决方法

### 4.1.`Collections.copy`

让我们看一个使用`Collections.copy `方法将一个`List` 复制到另一个`List`的工作示例:

```java
List<Integer> destination = Arrays.asList(1, 2, 3, 4, 5);
List<Integer> source = Arrays.asList(11, 22, 33);
Collections.copy(destination, source);
```

在本例中，我们将源列表的所有三个元素复制到目标列表。`Arrays.asList`方法用元素而不仅仅是大小来初始化列表，因此，我们能够成功地将源列表复制到目标列表。

如果我们只是交换`Collections.copy `方法的参数，它将抛出`java.lang.IndexOutOfBoundsException `，因为源列表的大小小于目的列表的大小`.`

完成此复制操作后，目标列表如下所示:

```java
[11, 22, 33, 4, 5]
```

除了`Collections.copy`方法，Java 中还有其他方法可以复制`List`。让我们来看看其中的一些。

### 4.2.`ArrayList`构造器

复制一个`List `的最简单的方法是使用一个带`Collection`参数的[构造函数:](/web/20221208143854/https://www.baeldung.com/java-arraylist#3-constructor-accepting-collection)

```java
List<Integer> source = Arrays.asList(11, 22, 33);
List<Integer> destination = new ArrayList<>(source);
```

这里，我们简单地将源列表传递给目的列表的构造函数，它创建了源列表的一个浅层副本。

目标列表将只是源列表所引用的同一对象的另一个引用。所以，任何引用所做的每一个改变都会影响同一个对象。

因此，对于复制像`Integers` 和 `Strings.`这样的不可变对象，使用构造函数是一个很好的选择

### 4.3.`addAll`

另一个简单的方法是使用 [`addAll `的方法`List`](/web/20221208143854/https://www.baeldung.com/java-arraylist#Adding) :

```java
List<Integer> destination = new ArrayList<>();
destination.addAll(source);
```

`The addAll` 方法将把源列表中的所有元素复制到目的列表中。

关于这种方法，有几点需要注意:

1.  它创建了源列表的浅层副本。
2.  源列表的元素被附加到目标列表。

### 4.4.Java 8 `Streams`

Java 8 引入了[流 API](/web/20221208143854/https://www.baeldung.com/java-8-streams) ，这是使用 Java `Collections.`的一个很好的工具

使用`stream()`方法，我们使用流 API `:`制作列表的副本

```java
List<Integer> copy = source.stream()
  .collect(Collectors.toList());
```

### 4.5.Java 10

在 Java 10 中复制一个`List`甚至更简单。使用`copyOf()`方法允许我们创建一个包含给定`Collection`元素的不可变列表:

```java
List<Integer> destination = List.copyOf(sourceList);
```

如果我们想采用这种方法，我们需要确保输入`List `不是`null`，并且它不包含任何`null`元素。

## 5.结论

在本文中，我们研究了`Collections.copy` 方法抛出`IndexOutOfBoundException “Source does not file in dest”`的方式和原因。与此同时，我们还探索了将一个`List `复制到另一个`List.`的不同方法

Java-10 之前的[示例](https://web.archive.org/web/20221208143854/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-exceptions-3)和 [Java 10 示例](https://web.archive.org/web/20221208143854/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-10)都可以在 GitHub 上找到。