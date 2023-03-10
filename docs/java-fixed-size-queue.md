# Java 中固定大小队列的实现

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-fixed-size-queue>

## 1.概观

在本文中，我们将了解 Java 对队列提供了什么。首先，我们将探索来自 [Java 集合框架](https://web.archive.org/web/20221112171232/https://docs.oracle.com/en/java/javase/18/docs/api/java.base/java/util/package-summary.html#CollectionsFramework)的队列的基础知识。

接下来，我们将探索 Java 集合框架中固定大小队列的实现。最后，我们将创建一个固定大小的队列实现。

## 2.Java 集合框架队列

`Java Collections Framework` 提供了不同的[队列](/web/20221112171232/https://www.baeldung.com/java-queue)实现，我们可以根据需要使用它们。例如，如果我们需要一个线程安全的实现，我们可以使用 [`ConcurrentLinkedQueue`](/web/20221112171232/https://www.baeldung.com/java-concurrent-queues) 。同样，如果我们需要指定元素在队列中必须如何排序，我们可以使用 [`PriorityQueue`](/web/20221112171232/https://www.baeldung.com/cs/priority-queue) 。

**Java 集合框架提供了一些不同的固定大小队列实现**。一个这样的实现是[`ArrayBlockingQueue`](https://web.archive.org/web/20221112171232/https://docs.oracle.com/javase/7/docs/api/java/util/concurrent/ArrayBlockingQueue.html)——一个使用固定数组存储元素的 FIFO 有界队列。队列的大小一旦创建就不能修改。

再比如 [`LinkedBlockingQueue`](https://web.archive.org/web/20221112171232/https://docs.oracle.com/javase/7/docs/api/java/util/concurrent/LinkedBlockingQueue.html) 。这个实现也是一个 FIFO 队列，但是它是可选有界的，所以如果没有设置容量，那么队列的限制将是`Integer.MAX_VALUE`个元素。**当我们不在并发环境中时，链接队列比数组队列提供更好的吞吐量**。

我们可以在`Java Collections Framework is` [`LinkedBlockingDeque`](https://web.archive.org/web/20221112171232/https://docs.oracle.com/javase/7/docs/api/java/util/concurrent/LinkedBlockingDeque.html)实现中找到最后一个固定大小的队列。然而，让我们注意这个类实现了`Deque`接口，而不仅仅是`Queue`接口。

尽管该框架提供了许多队列实现，但我们可能并不总能找到适合我们解决方案的实现。在这个场景中，**我们可以创建自己的实现**。

## 3.`Queue`Java 界面

为了更好地理解示例，让我们仔细看看`Queue` 界面。**`Queue`接口扩展了`Collection`接口****增加了具体的插入、提取、检查操作** 。每个操作都有两种形式，一种是在操作失败时抛出异常，另一种是返回特定值。

让我们来看看在`Queue`接口中定义的**插入操作:**

```java
boolean add(E e);
boolean offer(E e);
```

如果队列未满，两者都会在队列尾部添加一个新元素。当队列已满时，`offer()`方法会返回`false`，元素无法插入，但`add()`方法会抛出`IllegalStateException`异常。

接下来，我们有在`Queue`接口中定义的**提取操作:**

```java
E remove();
E poll();
```

这些方法将返回队列的头，并将从队列中移除元素。它们之间的区别在于，如果队列为空，`poll()`方法将返回`null`，而`remove()`方法将抛出`NoSuchElementException`异常。

最后，`Queue`界面中定义的**检验方法:**

```java
E element();
E peek();
```

注意`Queue`接口的所有方法中的类型`E`。这意味着**`Queue interface`被[仿制](/web/20221112171232/https://www.baeldung.com/java-generics)** :

```java
public interface Queue<E> extends Collection<E> {
    // ...
}
```

## 4.FIFO 固定大小队列实现，当满时删除最旧的元素

让我们创建我们的队列实现，一个固定大小的 FIFO 队列，当它满了的时候会删除最老的元素。姑且称之为`FifoFixedSizeQueue`。

`AbstractQueue` [抽象类](/web/20221112171232/https://www.baeldung.com/java-abstract-class)是一个很好的起点，因为它定义了一个 FIFO 队列，并实现了来自`Queue`和`Collection`接口的大多数方法。

让我们将固定大小的队列实现为一个元素数组。我们将使用一个数组`Object`来存储我们的数据，这允许我们插入任何类型的对象。此外，我们将保留一个`count`属性来指示队列中元素的数量:

```java
public class FifoFixedSizeQueue<E> extends AbstractQueue<E> {
    final Object[] items;
    int count;

    public FifoFixedSizeQueue(int capacity) {
        super();

        items = new Object[capacity];
        count = 0;
    }

    ...
} 
```

上面，我们可以看到构造函数初始化了保存队列元素和`count`属性的数组。因为队列是空的，所以数组的所有元素都是`null,`并且`count`属性是零。

接下来，让我们实现所有的抽象方法。

### 4.1.o `ffer()`方法实现

主要规则是**所有的** **元素都应该被插入，但是如果队列满了，最老的元素必须在之前被删除。**我们还需要确保**队列不允许插入`null`元素:**

```java
public boolean offer(E e) {
    if (e == null) {
        throw new NullPointerException("Queue doesn't allow nulls");
    }
    if (count == items.length) {
        this.poll();
    }
    this.items[count] = e;
    count++;
    return true;
} 
```

首先，由于不允许使用`nulls`，我们检查元素是否为`null`。如果是这种情况，我们抛出一个`NullPointerException`:

```java
if (e == null) {
    throw new NullPointerException("Queue doesn't allow nulls");
} 
```

接下来，我们检查队列是否已满，如果是，我们将从队列中删除最老的元素:

```java
while (count >= items.length) {
    this.poll();
} 
```

最后，我们将元素插入队列的尾部，并更新元素的队列编号:

```java
this.items[count] = e;
count++; 
```

### 4.2.`poll()`方法实现

`poll()`方法将返回队列的 head 元素，并将其从队列中移除。**如果队列为空， *poll()* 方法将返回`null`** :

```java
@Override
public E poll() {
    if (count <= 0) {
        return null;
    }
    E item = (E) items[0];
    shiftLeft();
    count--;
    return item;
} 
```

首先，我们检查队列是否为空，如果为空，则返回`null` :

```java
if (count <= 0) {
    return null;
} 
```

如果没有，我们将队列的头部存储在一个局部变量中，该变量将在最后返回，并调用`shiftLeft()`方法将队列的所有元素向队列的头部移动一步:

```java
E item = (E) items[0];
shiftLeft();
```

最后，我们更新元素的队列号，并返回队列头。

在继续之前，让我们检查一下方法`shiftLeft()`。此方法从队列中的第二个元素开始，一直到末尾，将每个元素向队列头移动一个位置:

```java
private void shiftLeft() {
    int i = 1;
    while (i < items.length) {
        if (items[i] == null) {
            break;
        }
        items[i - 1] = items[i];
        i++;
    }
} 
```

### 4.3.`Peek`方法实现

`peek()`方法像`poll()`方法一样工作，但是不从队列中删除元素:

```java
public E peek() {
    if (count <= 0) {
        return null;
    }
    return (E) items[0];
}
```

## 5.结论

在本文中，我们已经介绍了 Java 集合框架队列的基础知识。我们已经看到了框架必须提供的固定大小的队列，并学习了如何创建我们自己的队列。

和往常一样，代码可以在 GitHub 的[上获得。](https://web.archive.org/web/20221112171232/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-collections-4)