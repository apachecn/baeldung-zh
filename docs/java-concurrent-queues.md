# Java 并发队列指南

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-concurrent-queues>

## 1.概观

在本教程中，我们将介绍 Java 中并发队列的一些主要实现。关于队列的一般介绍，请参考我们的 Java `Queue`接口文章的[指南。](/web/20220824135537/https://www.baeldung.com/java-queue)

## 2.行列

在多线程应用程序中，队列需要处理多个并发的生产者-消费者场景。正确选择并发队列对于实现我们算法的良好性能至关重要。

首先，我们将看到阻塞队列和非阻塞队列之间的一些重要区别。然后，我们将看看一些实现和最佳实践。

## 2.阻塞与非阻塞队列

[`BlockingQueue`](/web/20220824135537/https://www.baeldung.com/java-blocking-queue) 为**提供了一个简单的线程安全机制**。在这个队列中，线程需要等待队列的可用性。生产者将在添加元素之前等待可用容量，而消费者将等到队列为空。在这些情况下，非阻塞队列要么抛出一个异常，要么返回一个特殊值，比如`null`或`false`。

为了实现这种阻塞机制，`BlockingQueue`接口在普通的`Queue`函数之上公开了两个函数:`put`和`take`。这些功能相当于标准`Queue`中的`add`和`remove`。

## 3.并发`Queue`实现

### 3.1.`ArrayBlockingQueue`

顾名思义，这个队列在内部使用一个数组。因此，**是一个有界队列，这意味着它有一个固定的大小**。

一个简单的工作队列就是一个用例。这种情况通常是生产者与消费者的比率较低，我们将耗时的任务分配给多个工作人员。由于该队列不能无限增长，因此如果内存出现问题，大小限制将作为一个安全阈值。

说到内存，需要注意的是队列会预先分配数组。虽然这可能会提高吞吐量，但它**也可能会消耗比必要的**更多的内存。例如，大容量队列可能长时间保持为空。

此外，`ArrayBlockingQueue`对`put`和`take`操作使用同一个锁。这确保不会覆盖条目，但会影响性能。

### 3.2.`LinkedBlockingQueue`

[`LinkedBlockingQueue`](/web/20220824135537/https://www.baeldung.com/java-queue-linkedblocking-concurrentlinked#linkedblockingqueue) 使用了一个 [`LinkedList`](/web/20220824135537/https://www.baeldung.com/java-linkedlist) 变体，其中每个队列项都是一个新节点。虽然这使得队列在原则上是无界的，但它仍然有一个硬限制`Integer.MAX_VALUE`。

另一方面，我们可以通过使用构造函数`LinkedBlockingQueue(int capacity)`来设置队列大小。

这个队列为`put`和`take`操作使用不同的锁。因此，这两个操作可以并行进行，从而提高吞吐量。

既然`LinkedBlockingQueue`可以是有界的也可以是无界的，为什么我们要在这个上面使用`ArrayBlockingQueue`? **`LinkedBlockingQueue`每次在队列**中添加或删除一个项目时，都需要分配和取消分配节点。因此，如果队列快速增长和快速收缩，那么`ArrayBlockingQueue`可能是更好的选择。

`LinkedBlockingQueue`的表现据说无法预测。换句话说，我们总是需要分析我们的场景，以确保我们使用正确的数据结构。

### 3.3.`PriorityBlockingQueue`

[`PriorityBlockingQueue`](/web/20220824135537/https://www.baeldung.com/java-priority-blocking-queue) 是我们需要按照特定顺序消费物品时的首选**。为了实现这一点，`PriorityBlockingQueue`使用了基于数组的二进制堆。**

虽然在内部它使用单锁机制，但`take`操作可以与`put`操作同时发生。简单旋转锁的使用使这成为可能。

一个典型的用例是使用不同优先级的任务。我们不希望低优先级任务取代高优先级任务。

### 3.4.`DelayQueue`

我们使用 [`DelayQueue`](/web/20220824135537/https://www.baeldung.com/java-delay-queue) **当一个消费者只能拿一个过期的商品**时。有趣的是，它在内部使用了一个`PriorityQueue`来根据项目的到期时间进行排序。

因为这不是一个通用的队列，所以它不像`ArrayBlockingQueue`或`LinkedBlockingQueue`那样涵盖很多场景。例如，我们可以使用这个队列实现一个简单的事件循环，类似于 NodeJS 中的事件循环。我们将异步任务放在队列中，以便在它们到期时进行处理。

### 3.5.`LinkedTransferQueue`

[`LinkedTransferQueue`](/web/20220824135537/https://www.baeldung.com/java-transfer-queue) 介绍了一种`transfer`方法。当生产或消费物品时，其他队列通常会阻塞，`LinkedTransferQueue` **允许生产者等待物品的消费**。

当我们需要保证我们放入队列中的特定项目已经被某人取走时，我们使用`LinkedTransferQueue`。此外，我们可以使用这个队列实现一个简单的背压算法。事实上，通过在消费前阻止生产者，**消费者可以推动产生的信息流**。

### 3.6.`SynchronousQueue`

虽然队列通常包含许多项目，但 [`SynchronousQueue`](/web/20220824135537/https://www.baeldung.com/java-synchronous-queue) 最多只会包含一个项目。换句话说，我们需要将`SynchronousQueue`视为**，一种在两个线程**之间交换数据的简单方式。

当我们有两个线程需要访问共享状态时，我们通常用 [`CountDownLatch`](/web/20220824135537/https://www.baeldung.com/java-countdown-latch) 或其他同步机制来同步它们。通过使用一个`SynchronousQueue`，我们可以**避免这种手动线程同步**。

### 3.7.`ConcurrentLinkedQueue`

[`ConcurrentLinkedQueue`](/web/20220824135537/https://www.baeldung.com/java-queue-linkedblocking-concurrentlinked#concurrentlinkedqueue) 是本指南唯一的非阻塞队列。因此，它提供了一种“无等待”算法，其中 **`add`和`poll`保证是线程安全的，并立即返回**。这个队列使用 [CAS(比较和交换)](/web/20220824135537/https://www.baeldung.com/lock-free-programming)而不是锁。

在内部，它是基于 Maged M. Michael 和 Michael L. Scott 的 [`Simple, Fast, and Practical Non-Blocking and Blocking Concurrent Queue Algorithms`](https://web.archive.org/web/20220824135537/https://www.cs.rochester.edu/u/scott/papers/1996_PODC_queues.pdf) 算法。

它是现代[反应式系统](/web/20220824135537/https://www.baeldung.com/java-reactive-systems) 的**完美候选，在这些系统中使用阻塞数据结构通常是被禁止的。**

另一方面，如果我们的消费者最终在循环中等待，我们可能应该选择阻塞队列作为更好的替代方案。

## 4.结论

在本指南中，我们介绍了不同的并发队列实现，讨论了它们的优缺点。考虑到这一点，我们可以更好地开发高效、持久和可用的系统。