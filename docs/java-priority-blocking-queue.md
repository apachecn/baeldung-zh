# Java 中的 PriorityBlockingQueue 指南

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-priority-blocking-queue>

## 1。简介

在本文中，我们将重点关注`[PriorityBlockingQueue](https://web.archive.org/web/20220706104850/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/concurrent/PriorityBlockingQueue.html)` 类，并查看一些实际例子。

从假设我们已经知道什么是`[Queue](https://web.archive.org/web/20220706104850/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/Queue.html)` 开始，我们将首先演示`PriorityBlockingQueue`中的**元素是如何按照优先级**排序的。

接下来，我们将演示如何使用这种类型的队列来阻塞线程。

最后，我们将展示在多线程中处理数据时，结合使用这两个特性是如何有用的。

## 2。元素的优先级

与标准队列不同，您不能将任何类型的元素添加到`PriorityBlockingQueue.` 中，有两个选项:

1.  添加实现`[Comparable](https://web.archive.org/web/20220706104850/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/lang/Comparable.html)`的元素
2.  添加没有实现`Comparable`的元素，条件是你也提供一个`[Comparator](https://web.archive.org/web/20220706104850/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/Comparator.html)`

通过使用`Comparator` 或`Comparable`实现来比较元素， `PriorityBlockingQueue` 将总是被排序。

目的是以这样一种方式实现比较逻辑，即最高优先级的元素总是首先排序。然后，当我们从队列中删除一个元素时，它将总是具有最高优先级的元素。

首先，让我们在单线程中使用我们的队列，而不是在多个线程中使用它。通过这样做，可以很容易地证明单元测试中的元素是如何排序的:

```java
PriorityBlockingQueue<Integer> queue = new PriorityBlockingQueue<>();
ArrayList<Integer> polledElements = new ArrayList<>();

queue.add(1);
queue.add(5);
queue.add(2);
queue.add(3);
queue.add(4);

queue.drainTo(polledElements);

assertThat(polledElements).containsExactly(1, 2, 3, 4, 5);
```

正如我们所看到的，尽管以随机的顺序将元素添加到队列中，但是当我们开始轮询它们时，它们将是有序的。这是因为`[Integer](https://web.archive.org/web/20220706104850/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/lang/Integer.html)` 类实现了`Comparable,` ,而后者将被用来确保我们以升序将它们从队列中取出。

同样值得注意的是，当两个元素被比较并且相同时，不能保证它们将如何排序。

## 3。用`Queue`来阻挡

如果我们正在处理一个标准队列，我们将调用`poll()` 来检索元素。然而，如果队列是空的，对`poll()` 的调用将返回`null.`

`PriorityBlockingQueue`实现了`BlockingQueue` 接口，它给了我们一些额外的方法，允许我们在从空队列中移除时**阻塞。让我们尝试使用`take()` 方法，它应该可以做到这一点:**

```java
PriorityBlockingQueue<Integer> queue = new PriorityBlockingQueue<>();

new Thread(() -> {
  System.out.println("Polling...");

  try {
      Integer poll = queue.take();
      System.out.println("Polled: " + poll);
  } catch (InterruptedException e) {
      e.printStackTrace();
  }
}).start();

Thread.sleep(TimeUnit.SECONDS.toMillis(5));
System.out.println("Adding to queue");
queue.add(1);
```

虽然使用`sleep()`是一种有点脆弱的演示方式，但是当我们运行这段代码时，我们会看到:

```java
Polling...
Adding to queue
Polled: 1 
```

这证明`take()` 被阻止，直到添加了一个项目:

1.  线程将打印“Polling”来证明它已经启动
2.  测试将暂停大约五秒钟，以证明线程此时已经调用了`take()`
3.  我们添加到队列中，应该或多或少地立即看到“Polled: 1 ”,以证明`take()` 在一个元素变得可用时就返回了它

另外值得一提的是，`BlockingQueue` 接口还为我们提供了在添加到满队列时进行阻塞的方法。

但是，a `PriorityBlockingQueue`是无界的。这意味着它永远不会满，因此总是有可能添加新元素。

## 4。一起使用阻止和优先化

既然我们已经解释了 a `PriorityBlockingQueue,` 的两个关键概念，让我们一起使用它们。我们可以简单地扩展上一个例子，但是这次向队列中添加更多的元素:

```java
Thread thread = new Thread(() -> {
    System.out.println("Polling...");
    while (true) {
        try {
            Integer poll = queue.take();
            System.out.println("Polled: " + poll);
        } 
        catch (InterruptedException e) { 
            e.printStackTrace();
        }
    }
});

thread.start();

Thread.sleep(TimeUnit.SECONDS.toMillis(5));
System.out.println("Adding to queue");

queue.addAll(newArrayList(1, 5, 6, 1, 2, 6, 7));
Thread.sleep(TimeUnit.SECONDS.toMillis(1));
```

同样，虽然由于使用了`sleep(),` 这有点脆弱，但它仍然向我们展示了一个有效的用例。我们现在有一个阻塞的队列，等待添加元素。然后我们一次添加许多元素，然后显示它们将按优先级顺序处理。输出将如下所示:

```java
Polling...
Adding to queue
Polled: 1
Polled: 1
Polled: 2
Polled: 5
Polled: 6
Polled: 6
Polled: 7
```

## 5。结论

在本指南中，我们已经演示了如何使用`PriorityBlockingQueue` 来阻塞一个线程，直到一些项目被添加到它，并且我们能够根据它们的优先级来处理这些项目。

这些例子的实现可以在 GitHub 的[中找到。这是一个基于 Maven 的项目，所以应该很容易运行。](https://web.archive.org/web/20220706104850/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-concurrency-collections)