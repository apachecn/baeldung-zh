# Java 中的“流已经被操作或关闭”异常

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-stream-operated-upon-or-closed-exception>

## 1。概述

在这篇简短的文章中，我们将讨论在 Java 8 中使用`Stream`类时可能会遇到的一个常见的`Exception` :

```java
IllegalStateException: stream has already been operated upon or closed.
```

我们将发现这种异常发生时的场景，以及避免这种异常的可能方法，并提供实际例子。

## 2。起因

在 Java 8 中，每个`Stream`类代表一个一次性的数据序列，并支持多个 I/O 操作。

> 一个`Stream`应该只被操作一次(调用一个中间或终端流操作)。如果检测到`Stream`正在被重用，流实现可能会抛出`IllegalStateException` 。

每当在一个`Stream`对象上调用终端操作时，实例就会被消耗和关闭。

因此，**我们只被允许执行一个消耗了** `**Stream**,`的操作，否则，我们将得到一个异常，表明`Stream`已经被操作或关闭。

让我们看看如何将其转化为一个实际的例子:

```java
Stream<String> stringStream = Stream.of("A", "B", "C", "D");
Optional<String> result1 = stringStream.findAny(); 
System.out.println(result1.get()); 
Optional<String> result2 = stringStream.findFirst();
```

因此:

```java
A
Exception in thread "main" java.lang.IllegalStateException: 
  stream has already been operated upon or closed
```

调用`#findAny()`方法后，`stringStream`关闭，因此，对`Stream`的任何进一步操作都会抛出`IllegalStateException`，这就是调用`#findFirst()`方法后的情况。

## 3。解决方案

简而言之，解决方案就是在我们需要的时候创建一个新的`Stream`。

当然，我们可以手动操作，但这正是`Supplier`功能界面变得非常方便的地方:

```java
Supplier<Stream<String>> streamSupplier 
  = () -> Stream.of("A", "B", "C", "D");
Optional<String> result1 = streamSupplier.get().findAny();
System.out.println(result1.get());
Optional<String> result2 = streamSupplier.get().findFirst();
System.out.println(result2.get());
```

因此:

```java
A
A
```

我们定义了类型为`Stream<String>`的`streamSupplier`对象，这与`#get()`方法返回的类型完全相同。`Supplier`基于一个 lambda 表达式，该表达式不接受输入并返回一个新的`Stream`。

调用`Supplier`上的函数方法`get()`会返回一个新创建的`Stream` 对象，我们可以在其上安全地执行另一个`Stream`操作。

## 5。结论

在这个快速教程中，我们已经看到了如何在一个`Stream`上多次执行终端操作，同时避免在`Stream`已经关闭或正在操作时抛出著名的`IllegalStateException`。

你可以在 GitHub 上找到本文[的完整源代码和所有代码片段。](https://web.archive.org/web/20221012100327/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-streams)