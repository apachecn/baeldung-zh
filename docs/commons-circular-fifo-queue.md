# Apache Commons 循环队列指南

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/commons-circular-fifo-queue>

[This article is part of a series:](javascript:void(0);)[• Apache Commons Collections Bag](/web/20221208143854/https://www.baeldung.com/apache-commons-bag)
[• Apache Commons Collections SetUtils](/web/20221208143854/https://www.baeldung.com/apache-commons-setutils)
[• Apache Commons Collections OrderedMap](/web/20221208143854/https://www.baeldung.com/apache-commons-ordered-map)
[• Apache Commons Collections BidiMap](/web/20221208143854/https://www.baeldung.com/commons-collections-bidi-map)
[• A Guide to Apache Commons Collections CollectionUtils](/web/20221208143854/https://www.baeldung.com/apache-commons-collection-utils)
[• Apache Commons Collections MapUtils](/web/20221208143854/https://www.baeldung.com/apache-commons-map-utils)
• Guide to Apache Commons CircularFifoQueue (current article)

## 1。概述

在这个快速教程中，我们将看看 Apache Commons Collections 库`.`的`collections4.queue` 包中提供的`CircularFifoQueue`数据结构

`CircularFifoQueue<E>`实现了`Queue<E>`接口，并且是一个**固定大小的**、**非阻塞队列** — **当你向一个已满的队列中添加一个元素时，最老的元素将被移除，以便为新元素**腾出空间。

## 2。Maven 依赖关系

对于 Maven 项目，我们需要添加所需的依赖项:

```java
<dependency>
    <groupId>org.apache.commons</groupId>
    <artifactId>commons-collections4</artifactId>
    <version>4.1</version>
</dependency>
```

你可以在 [Maven Central](https://web.archive.org/web/20221208143854/https://search.maven.org/classic/#search%7Cga%7C1%7Cg%3A%22org.apache.commons%22%20AND%20a%3A%22commons-collections4%22) 上找到这个库的最新版本。

## 3。构造函数

要创建一个`CircularFifoQueue`对象，我们可以使用默认的构造函数，它创建一个默认大小为 32:

```java
CircularFifoQueue<String> bits = new CircularFifoQueue();
```

如果我们知道期望的队列最大大小，我们可以使用构造函数，该函数将一个`int`作为参数来指定大小:

```java
CircularFifoQueue<String> colors = new CircularFifoQueue<>(5);
```

还有一个选项是通过给构造函数一个集合作为参数来创建一个`CircularFifoQueue`对象。

在这种情况下，队列将被集合的元素填充，其大小将与集合的大小相同:

```java
CircularFifoQueue<String> daysOfWeek = new CircularFifoQueue<>(days);
```

注意:由于这个队列在构造时已经满了，任何添加都会导致第一个创建的元素被丢弃。

## 4。添加元素

与任何`Queue`实现一样，我们可以通过使用`add`和`offer`方法来添加元素。 [`Queue` JavaDoc](https://web.archive.org/web/20221208143854/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/Queue.html) 指定了`offer`方法用于处理容量受限的队列。

然而，因为`CircularFifoQueue`是非阻塞的，所以插入不会失败。因此，它的`add`和`offer`方法表现出相同的行为。

让我们看看如何使用`add`方法向我们的`colors`队列添加元素:

```java
colors.add("Red");
colors.add("Blue");
colors.add("Green");
```

让我们使用`offer`方法添加一些元素:

```java
colors.offer("White");
colors.offer("Black");
```

## 5。移除和检索元素

当我们需要操作队列的元素时,`CircularFifoQueue`类提供了一些有用的方法。一些方法用于从队列中获取元素，一些方法用于移除元素，一些方法用于同时执行这两种操作。

### 5.1。`Peek`法

`peek`方法是非破坏性的，并且**返回队列的头**。

只要在两次调用之间队列中的元素没有任何变化，这个方法将总是返回相同的元素。**如果队列为空，`peek`将返回`null:`**

```java
String colorsHead = colors.peek();
```

### 5.2。`Element`法

`element`方法类似于`peek` —它**返回当前队列头**。

然而，如果队列为空，`element`方法会抛出一个异常:

```java
colorsHead = colors.element();
```

### 5.3。`Get`法

当我们需要从队列中获取某个元素时，可以使用`get`方法。该方法将所需元素的索引作为参数。队列的索引是从零开始的。

让我们从前面填充了元素的`colors`队列中获取一个元素:

```java
String color = colors.get(1);
```

这将返回“`Blue`”。

现在让我们向队列中添加三个元素，并再次检查这个结果:

```java
colors.add("Orange");
colors.add("Violet");
colors.add("Pink");

color = colors.get(1);
```

这一次，`get`方法返回“`Black`”。这是因为我们的队列被创建为有限的五个，前三个元素(“`Red`”、“`Blue`”、“`Green`”)随着新元素的添加而被删除。

### 5.4。`Poll`法

`poll`方法**移除队列的头元素并返回该元素**。如果队列没有元素，`poll`方法返回`null:`

```java
colorsHead = colors.poll();
```

### 5.5。`Remove`法

**`remove`方法** **的操作与`poll`方法**非常相似——它返回队列的头部并移除返回的元素。然而**，如果队列为空，`remove`将抛出一个异常**:

```java
colorsHead = colors.remove();
```

### 5.6。`Clear`法

当我们想清空队列时，可以使用`clear`方法:

```java
colors.clear();
```

## 6。检查方法

在了解了如何添加、移除和检索队列元素之后，让我们看看这个类在检查其大小和容量方面提供了什么。在我们的示例中，我们将使用前面几节中创建的队列。

一般来说，我们有两种方法来检查队列的大小——一种用于获取对象的最大大小，另一种用于检查其当前元素计数。

`maxSize`方法将返回队列最大大小的`integer`值:

```java
int maxSize = bits.maxSize();
```

这将返回`32`，因为`bits`队列是用默认构造函数创建的。

`size`方法将返回当前存储在队列中的元素数量:

```java
int size = colors.size();
```

为了检查队列对象的容量，我们可以使用`isEmpty`和`isAtFullCapacity`方法。

`isEmpty`方法将返回一个`boolean`值，指示队列是否为空:

```java
boolean isEmpty = bits.isEmpty();
```

为了检查我们的队列是否已满，我们可以使用**的`isAtFullCapacity`方法**。这个方法**只有在队列中的元素达到最大值**时才返回`true`:

```java
boolean isFull = daysOfWeek.isAtFullCapacity();
```

你应该注意到这个方法是从版本 4.1 开始提供的**。**

另一个我们可以用来检查队列是否已满的方法是`isFull`方法。对于`CircularFifoQueue`，**，`isFull`方法将总是返回`false,`，因为队列总是可以接受新元素**:

```java
boolean isFull = daysOfWeek.isFull();
```

## 7。结论

在本文中，我们看到了如何使用 Apache Commons `CircularFifoQueue`。我们看到了一些例子，说明了如何实例化一个队列对象，如何填充它，如何清空它，如何从中获取和移除元素，以及如何检查它的大小和容量。

您可以在我们的 [GitHub 项目](https://web.archive.org/web/20221208143854/https://github.com/eugenp/tutorials/tree/master/libraries-apache-commons-collections)中找到本文中使用的完整示例代码。这是一个 Maven 项目，所以您应该能够导入它并按原样运行它。

**«** Previous[Apache Commons Collections MapUtils](/web/20221208143854/https://www.baeldung.com/apache-commons-map-utils)