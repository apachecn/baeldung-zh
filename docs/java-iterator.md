# Java 迭代器指南

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-iterator>

## 1。简介

一个`Iterator`是我们可以遍历一个集合的许多方式之一，并且作为每一种选择，它都有它的优点和缺点。

它最初是在 Java 1.2 中作为对`Enumerations`的替代而引入的，并且:

*   引入了改进的方法名
*   [使得从我们正在迭代的集合中移除元素成为可能](/web/20220928094931/https://www.baeldung.com/java-concurrentmodificationexception)
*   不保证迭代顺序

在本教程中，我们将回顾简单的`Iterator` 接口，学习如何使用它的不同方法。

我们还将检查更加健壮的`ListIterator`扩展，它增加了一些有趣的功能。

## 2。`Iterator`界面

首先，我们需要从一个`Collection`获得一个`Iterator`；这是通过调用`iterator()`方法来完成的。

为简单起见，我们将从一个列表中获取`Iterator`实例:

```java
List<String> items = ...
Iterator<String> iter = items.iterator();
```

`Iterator`接口有三个核心方法:

### 2.1。`hasNext()`

`hasNext()`方法可用于检查是否还有至少一个元素需要迭代。

它被设计用作`while`循环中的条件:

```java
while (iter.hasNext()) {
    // ...
}
```

### 2.2。`next()`

`next()`方法可用于跳过下一个元素并获得它:

```java
String next = iter.next();
```

**在尝试调用`next()`之前使用`hasNext()`是个好习惯。**

`Iterators` for `Collections`不保证以任何特定的顺序迭代[，除非特定的实现提供它。](https://web.archive.org/web/20220928094931/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/Collection.html#iterator())

### 2.3。`remove()`

最后，如果我们想要**从集合中移除当前元素，**我们可以使用`remove:`

```java
iter.remove();
```

这是一种在迭代集合时移除元素的安全方式，没有 [`ConcurrentModificationException.`](/web/20220928094931/https://www.baeldung.com/java-concurrentmodificationexception) 的风险

### 2.4。完整的`Iterator`例子

现在，我们可以将它们结合起来，看看如何将这三种方法一起用于集合过滤:

```java
while (iter.hasNext()) {
    String next = iter.next();
    System.out.println(next);

    if( "TWO".equals(next)) {
        iter.remove();				
    }
}
```

这就是我们通常如何使用一个`Iterator,` 我们提前检查是否有另一个元素，我们检索它，然后我们对它执行一些动作。

### 2.5。用 Lambda 表达式迭代

正如我们在前面的例子中看到的，当我们只想检查所有的元素并对它们做一些事情时，使用`Iterator`是非常冗长的。

从 Java 8 开始，我们有了允许使用 lambdas 处理剩余元素的`forEachRemaining`方法:

```java
iter.forEachRemaining(System.out::println);
```

## 3。`ListIterator`界面

`ListIterator` 是一个扩展，增加了遍历列表的新功能:

```java
ListIterator<String> listIterator = items.listIterator(items.size());
```

注意我们如何提供一个起始位置，在这个例子中是`List.`的结尾

### 3.1。`hasPrevious()`和 `previous()`

`ListIterator`可用于向后遍历，因此它提供了`hasNext()`和`next()`的等效物:

```java
while(listIterator.hasPrevious()) {
    String previous = listIterator.previous();
}
```

### 3.2.`nextIndex()`和`previousIndex()`

此外，我们可以遍历索引，而不是实际的元素:

```java
String nextWithIndex = items.get(listIterator.nextIndex());
String previousWithIndex = items.get(listIterator.previousIndex());
```

如果我们需要知道我们当前正在修改的对象的索引，或者如果我们想要保留被删除的元素的记录，这可能会非常有用。

### 3.3。`add()`

`add`方法，顾名思义，它允许我们在`next()`返回的项目之前和`previous():` 返回的项目之后添加一个元素

```java
listIterator.add("FOUR");
```

### 3.4。`set()`

最后一个值得一提的方法是`set(),` ，它让我们替换在调用`next()`或`previous()`时返回的元素:

```java
String next = listIterator.next();
if( "ONE".equals(next)) {
    listIterator.set("SWAPPED");
}
```

需要注意的是，**这只能在没有预先调用`add()`或`remove()`的情况下执行。**

### 3.5。完整的`ListIterator`例子

我们现在可以把它们结合起来，构成一个完整的例子:

```java
ListIterator<String> listIterator = items.listIterator();
while(listIterator.hasNext()) {
    String nextWithIndex = items.get(listIterator.nextIndex());		
    String next = listIterator.next();
    if("REPLACE ME".equals(next)) {
        listIterator.set("REPLACED");
    }
}
listIterator.add("NEW");
while(listIterator.hasPrevious()) {
    String previousWithIndex
     = items.get(listIterator.previousIndex());
    String previous = listIterator.previous();
    System.out.println(previous);
}
```

在这个例子中，我们从从`List`获取`ListIterator`开始，然后我们可以通过索引–**(这不会增加迭代器的内部当前元素**)或者通过调用`next`来获取下一个元素。

然后我们可以用`set`替换一个特定的项目，并用`add.`插入一个新的项目

在到达迭代的末尾之后，我们可以回过头来修改额外的元素，或者简单地从下到上打印它们。

## 4。结论

`Iterator`接口允许我们在遍历集合时修改它，这对于简单的 for/while 语句来说更加困难。反过来，这给了我们一个好的模式，我们可以在许多方法中使用，只需要集合处理，同时保持良好的内聚性和低耦合性。

最后，和往常一样，完整的源代码可以在 GitHub 上获得[。](https://web.archive.org/web/20220928094931/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-collections)**