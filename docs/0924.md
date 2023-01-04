# Java 队列接口指南

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-queue>

## 1.介绍

在本教程中，我们将讨论 Java 的`Queue`接口。

首先，我们将**研究一下`Queue`做什么，以及它的一些核心方法**。接下来，我们将深入研究 Java 作为标准提供的一些**实现。**

最后，在结束之前，我们将讨论线程安全。

## 2.可视化队列

让我们先做一个简单的类比。

想象一下，我们刚刚开了第一家店——一个热狗摊。我们希望为我们的小企业以最有效的方式服务新的潜在客户；一次一个。首先，我们要求他们在我们的摊位前排成有序的队伍，新顾客从后面加入。多亏了我们的组织能力，我们现在可以公平地分发美味的热狗了。

`[Queues](/web/20220706104853/https://www.baeldung.com/cs/types-of-queues) `在 Java 中以类似的方式工作。在我们声明了我们的`Queue, `之后，我们可以在后面添加新元素，并从前面移除它们。

事实上，**我们在 Java 中遇到的大多数`Queues `工作都是以这种先进先出**的方式进行的——通常缩写为 FIFO。

然而，有一个例外，我们将在稍后的中提到[。](#priority_queues)

## 3.核心方法

`Queue `声明了需要由所有实现类编码的[个方法](https://web.archive.org/web/20220706104853/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/Queue.html)。**现在让我们概述几个比较重要的****:**

1.  **`offer()`**–在`Queue`上插入一个新元素
2.  `**poll()** `–从`Queue`的前面移除一个元素
3.  `**peek()** –`检查`Queue, `前部的元件，无需将其移除

## 4.`AbstractQueue`

`[AbstractQueue](https://web.archive.org/web/20220706104853/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/AbstractQueue.html) `是 Java 提供的**最简单的可能`Queue `实现。**它包括一些`Queue `接口方法的框架实现，不包括`[offer](https://web.archive.org/web/20220706104853/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/Queue.html#offer(E))`。

**当我们创建一个扩展了`AbstractQueue`类`,` **的自定义队列**时，我们必须提供一个`offer` 方法**的实现 **，该方法`not` 允许插入空元素。**

另外，**我们必须提供方法`peek, poll, size,` 和`java.util`的`iterator`。**

**让我们用`AbstractQueue.`组装一个简单的`Queue `实现**

首先，让我们用一个`LinkedList `来定义我们的类，以存储我们的`Queue's `元素:

```
public class CustomBaeldungQueue<T> extends AbstractQueue<T> {

    private LinkedList<T> elements;

    public CustomBaeldungQueue() {
      this.elements = new LinkedList<T>();
    }

}
```

接下来，让我们**覆盖所需的方法并提供代码:**

```
@Override
public Iterator<T> iterator() {
    return elements.iterator();
}

@Override
public int size() {
    return elements.size();
}

@Override
public boolean offer(T t) {
    if(t == null) return false;
    elements.add(t);
    return true;
}

@Override
public T poll() {
    Iterator<T> iter = elements.iterator();
    T t = iter.next();
    if(t != null){
        iter.remove();
        return t;
    }
    return null;
}

@Override
public T peek() {
    return elements.getFirst();
}
```

太好了，让我们用一个快速的单元测试来检查它的工作情况:

```
customQueue.add(7);
customQueue.add(5);

int first = customQueue.poll();
int second = customQueue.poll();

assertEquals(7, first);
assertEquals(5, second);
```

## 4.子接口

一般情况下，`Queue `接口由 **3 个主子接口继承。** `Blocking Queues, Transfer Queues`，还有`Deques`。

总的来说，这 3 个接口是由绝大多数可用的 Java 实现的`Queues.`,让我们快速地看一下这些接口的用途。

### 4.1.`Blocking Queues`

`[BlockingQueue](https://web.archive.org/web/20220706104853/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/concurrent/BlockingQueue.html) `接口**支持根据当前状态强制线程等待**T1 的额外操作。当尝试检索时，线程可以**等待`Queue `为非空，或者当添加新元素**时等待其变为空。

标准`Blocking Queues`包括`LinkedBlockingQueue, [SynchronousQueue,](/web/20220706104853/https://www.baeldung.com/java-synchronous-queue) `和`ArrayBlockingQueue`。

更多信息，请阅读我们关于 [`Blocking Queues`](/web/20220706104853/https://www.baeldung.com/java-blocking-queue) 的文章。

### 4.2.`Transfer Queues`

`[TransferQueue](https://web.archive.org/web/20220706104853/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/concurrent/TransferQueue.html) `接口扩展了`BlockingQueue `接口，但是**是针对生产者-消费者模式定制的。它控制着信息从生产者到消费者的流动，在系统中产生反压力。**

Java 附带了一个`TransferQueue `接口的实现`[LinkedTransferQueue](/web/20220706104853/https://www.baeldung.com/java-transfer-queue).`

### 4.3.`Deques`

`Deque`是**D**double-**E**nded`**Que**ue` 的缩写，类似于一副纸牌——元素可以取自`Deque`的起点和终点。与传统的`Queue,` 非常相似， **`Deque `提供了添加、检索和查看顶部和底部`.`** 元素的方法

关于`Deque `如何工作的详细指南，请查看我们的 [`ArrayDeque `](/web/20220706104853/https://www.baeldung.com/java-array-deque) [文章](/web/20220706104853/https://www.baeldung.com/java-array-deque)。

## 5.`Priority Queues`

我们之前看到，我们在 Java 中遇到的大多数`Queues `都遵循 FIFO 原则。

这个规则的一个例外是 [`PriorityQueue`](https://web.archive.org/web/20220706104853/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/PriorityQueue.html) 。**当新元素被插入到`Priority` `Queue,` 中时，它们根据它们的自然排序进行排序，或者由我们构造`Priority` `Queue`时提供的一个定义好的 [`Comparator`](/web/20220706104853/https://www.baeldung.com/java-comparator-comparable)** 进行排序。

让我们通过一个简单的单元测试来看看这是如何工作的:

```
PriorityQueue<Integer> integerQueue = new PriorityQueue<>();

integerQueue.add(9);
integerQueue.add(2);
integerQueue.add(4);

int first = integerQueue.poll();
int second = integerQueue.poll();
int third = integerQueue.poll();

assertEquals(2, first);
assertEquals(4, second);
assertEquals(9, third);
```

尽管我们的整数被添加到`Priority` `Queue`的顺序，我们可以看到**的检索顺序是根据数字的自然顺序改变的。**

我们可以看到，同样的情况也适用于`Strings`:

```
PriorityQueue<String> stringQueue = new PriorityQueue<>();

stringQueue.add("blueberry");
stringQueue.add("apple");
stringQueue.add("cherry");

String first = stringQueue.poll();
String second = stringQueue.poll();
String third = stringQueue.poll();

assertEquals("apple", first);
assertEquals("blueberry", second);
assertEquals("cherry", third);
```

## 6.线程安全

向`Queues`添加项目在多线程环境中特别有用。一个 **`Queue `可以在线程间共享，并用于阻塞进程，直到空间可用**——帮助我们**克服一些常见的多线程问题。**

例如，从多个线程写入单个磁盘会造成资源争用，并可能导致写入时间变慢。**创建一个带`BlockingQueue `的写线程可以缓解这个问题，并大大提高写速度。**

幸运的是，Java 提供了`ConcurrentLinkedQueue, ArrayBlockingQueue`和`ConcurrentLinkedDeque `，它们是线程安全的，非常适合多线程程序。

## 7.结论

在本教程中，我们深入研究了 Java `Queue `接口。

首先，我们**探索了 a `Queue `做什么**，以及 Java 提供的**实现。**

接下来，我们**看了一下`Queue'`通常的 FIFO 原理，以及顺序不同的`PriorityQueue `。**

最后，我们**探讨了线程安全**以及如何在多线程环境中使用`Queues `。

和往常一样，代码可以在 GitHub 的[上获得。](https://web.archive.org/web/20220706104853/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-collections)