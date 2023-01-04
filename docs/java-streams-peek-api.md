# Java 8 Streams peek() API

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-streams-peek-api>

## 1.介绍

Java 流 API 向我们介绍了一种强大的处理数据的替代方法。

在这个简短的教程中，我们将关注`peek()`，一个经常被误解的方法。

## 2.快速示例

让我们把手弄脏，尝试使用`peek()`。我们有一串名字，我们想把它们打印到控制台上。

因为`peek()`期望一个`Consumer<T>`作为它唯一的参数，这看起来很合适，所以让我们试一试:

```java
Stream<String> nameStream = Stream.of("Alice", "Bob", "Chuck");
nameStream.peek(System.out::println);
```

但是，上面的代码片段不产生任何输出。为了理解为什么，让我们快速回顾一下流生命周期的各个方面。

## 3.中间与终端操作

回想一下，流有三个部分:一个数据源、零个或多个中间操作以及零个或一个终端操作。

源向管道提供元素。

中间操作逐个获取元素并处理它们。所有中间操作都是懒惰的，因此，在管道开始工作之前，任何操作都不会有任何效果。

终端操作意味着流生命周期的结束。对我们的场景最重要的是，他们**启动管道**中的工作。

## 4.`peek()`用法

在我们的第一个例子中,`peek()`不起作用的原因是,**是一个`intermediate`操作，我们没有对流水线应用`terminal`操作**。或者，我们可以使用带有相同参数的`forEach()`来获得期望的行为:

```java
Stream<String> nameStream = Stream.of("Alice", "Bob", "Chuck");
nameStream.forEach(System.out::println);
```

`peek()`的 [Javadoc 页面](https://web.archive.org/web/20221005211523/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/stream/Stream.html#peek(java.util.function.Consumer))说:“**这个方法的存在主要是为了支持调试，当你想看到元素流过管道**的某个点时。

让我们考虑来自同一个 Javadoc 页面的这个片段:

```java
Stream.of("one", "two", "three", "four")
  .filter(e -> e.length() > 3)
  .peek(e -> System.out.println("Filtered value: " + e))
  .map(String::toUpperCase)
  .peek(e -> System.out.println("Mapped value: " + e))
  .collect(Collectors.toList());
```

它演示了我们如何观察通过每个操作的元素。

除此之外，`peek()`在另一个场景中也很有用:当**我们想要改变元素**的内部状态时。例如，假设我们想在打印之前将所有用户名转换成小写:

```java
Stream<User> userStream = Stream.of(new User("Alice"), new User("Bob"), new User("Chuck"));
userStream.peek(u -> u.setName(u.getName().toLowerCase()))
  .forEach(System.out::println);
```

或者，我们可以使用`map()`，但是`peek()`更方便，因为我们不想替换元素。

## 5.结论

在这个简短的教程中，我们看到了流生命周期的总结，以理解`peek()`是如何工作的。我们还看到了两个日常用例，使用`peek()`是最直接的选择。

和往常一样，这些例子可以在 GitHub 的[上找到。](https://web.archive.org/web/20221005211523/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-streams-2)