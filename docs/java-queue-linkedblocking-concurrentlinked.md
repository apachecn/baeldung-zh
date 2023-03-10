# LinkedBlockingQueue 与 ConcurrentLinkedQueue

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-queue-linkedblocking-concurrentlinked>

## 1.介绍

**`LinkedBlockingQueue`和`ConcurrentLinkedQueue` 是 Java** 中最常用的两个并发队列。尽管这两个队列经常被用作并发数据结构，但是它们之间存在细微的特征和行为差异。

在这个简短的教程中，我们将讨论这两种队列，并解释它们的相似之处和不同之处。

## 2.`LinkedBlockingQueue`

`LinkedBlockingQueue` **是一个`optionally-bounded` 阻塞队列实现，**意味着如果需要可以指定队列大小。

让我们创建一个可以包含多达 100 个元素的`LinkedBlockingQueue`:

```java
BlockingQueue<Integer> boundedQueue = new LinkedBlockingQueue<>(100);
```

我们还可以通过不指定大小来创建一个无界的`LinkedBlockingQueue `:

```java
BlockingQueue<Integer> unboundedQueue = new LinkedBlockingQueue<>();
```

无限队列意味着在创建时没有指定队列的大小。因此，队列可以随着元素的添加而动态增长。然而，如果没有剩余内存，那么队列抛出一个`java.lang.OutOfMemoryError.`

我们也可以从现有的集合中创建一个`LinkedBlockingQueue `:

```java
Collection<Integer> listOfNumbers = Arrays.asList(1,2,3,4,5);
BlockingQueue<Integer> queue = new LinkedBlockingQueue<>(listOfNumbers);
```

`LinkedBlockingQueue` 类**实现了`BlockingQueue`接口，该接口为其**提供了阻塞特性。

阻塞队列表示如果队列已满(当队列受限时)或变空，队列将阻塞访问线程。如果队列已满，那么添加新元素将阻塞访问线程，除非有空间可用于新元素。类似地，如果队列为空，则访问元素会阻塞调用线程:

```java
ExecutorService executorService = Executors.newFixedThreadPool(1);
LinkedBlockingQueue<Integer> queue = new LinkedBlockingQueue<>();
executorService.submit(() -> {
  try {
    queue.take();
  } 
  catch (InterruptedException e) {
    // exception handling
  }
});
```

在上面的代码片段中，我们正在访问一个空队列。因此，`take` 方法会阻塞调用线程。

`LinkedBlockingQueue` 的阻塞特性与一些成本有关。这个成本是因为每个`put`或`take`操作都在生产者线程或消费者线程之间争用锁。因此，在有许多生产者和消费者的场景中，`put`和采取行动可能会慢一些。

## 3.`ConcurrentLinkedQueue`

`ConcurrentLinkedQueue` **是一个无界的、线程安全的、非阻塞的队列。**

让我们创建一个空的`ConcurrentLinkedQueue`:

```java
ConcurrentLinkedQueue queue = new ConcurrentLinkedQueue();
```

我们也可以从现有的集合中创建一个`ConcurrentLinkedQueue`:

```java
Collection<Integer> listOfNumbers = Arrays.asList(1,2,3,4,5);
ConcurrentLinkedQueue<Integer> queue = new ConcurrentLinkedQueue<>(listOfNumbers);
```

与 T4 不同，T1 是一个非阻塞队列。因此，一旦队列为空，它就不会阻塞线程。相反，它返回`null`。因为它是无界的，如果没有额外的内存来添加新元素，它将抛出一个`java.lang.OutOfMemoryError`。

除了非阻塞之外，`ConcurrentLinkedQueue `还有额外的功能。

在任何生产者-消费者情景中，消费者都不会与生产者竞争；然而，多个生产商将相互竞争:

```java
int element = 1;
ExecutorService executorService = Executors.newFixedThreadPool(2);
ConcurrentLinkedQueue<Integer> queue = new ConcurrentLinkedQueue<>();

Runnable offerTask = () -> queue.offer(element);

Callable<Integer> pollTask = () -> {
  while (queue.peek() != null) {
    return queue.poll().intValue();
  }
  return null;
};

executorService.submit(offerTask);
Future<Integer> returnedElement = executorService.submit(pollTask);
assertThat(returnedElement.get().intValue(), is(equalTo(element))); 
```

第一个任务`offerTask`向队列中添加一个元素，第二个任务`pollTask,`从队列中检索一个元素。轮询任务另外**首先检查队列中的一个元素，因为`ConcurrentLinkedQueue` 是非阻塞的，并且可以返回一个`null`值**。

## 4.类似

`LinkedBlockingQueue` 和`ConcurrentLinkedQueue` 都是队列实现，有一些共同的特征。让我们讨论一下这两个队列的相似之处:

1.  两个**都实现了`Queue`接口**
2.  它们都使用链接节点来存储它们的元素
3.  两个**都适合并发访问场景**

## 5.差异

尽管这两种队列有某些相似之处，但也有本质的特征差异:

| 特征 | linkedblockingqueue(链接阻止队列) | concurrentlinkedqueue 并行伫列 |
| **阻挡性质** | 它是一个阻塞队列，实现了`BlockingQueue`接口 | 它是一个非阻塞队列，不实现`BlockingQueue` 接口 |
| **队列大小** | 它是一个可选的有界队列，这意味着在创建过程中有定义队列大小的规定 | 这是一个无限制的队列，在创建过程中没有指定队列大小的规定 |
| **锁定性质** | 这是一个基于锁的队列 | 这是一个无锁队列 |
| **算法** | 它根据`two-lock queue`算法实现**锁定** | 它依赖于无阻塞、无锁队列的迈克尔 T2 斯科特算法 |
| **实施** | 在`two-lock queue` 算法机制中， `LinkedBlockingQueue` 使用两种不同的锁——即 `putLock` 和 `takeLock` 。 `put/take` 操作使用第一种锁类型，而 `take/poll` 操作使用另一种锁类型 | 它使用 CAS(比较和交换)进行操作 |
| **阻塞行为** | 这是一个阻塞队列。因此，当队列为空时，它会阻塞正在访问的线程 | 当队列为空时，它不会阻塞正在访问的线程，并返回`null` |

## 6.结论

在这篇文章中，我们学习了`LinkedBlockingQueue `和`ConcurrentLinkedQueue.`

首先，我们分别讨论了这两个队列实现及其一些特征。然后，我们看到了这两个队列实现之间的相似之处。最后，我们探讨了这两种队列实现之间的差异。

和往常一样，GitHub 上的[提供了示例的源代码。](https://web.archive.org/web/20220706104912/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-concurrency-collections)