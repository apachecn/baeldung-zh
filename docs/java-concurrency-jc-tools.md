# 带有 JCTools 的 Java 并发实用程序

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-concurrency-jc-tools>

## 1。概述

在本教程中，我们将介绍 [JCTools](https://web.archive.org/web/20190209024432/https://github.com/JCTools/JCTools) (Java 并发工具)库。

简单地说，这提供了许多适合在多线程环境中工作的实用数据结构。

## 2。非阻塞算法

传统上，在可变共享状态下工作的多线程代码使用锁来确保数据一致性和发布(一个线程所做的改变对另一个线程是可见的)。

这种方法有许多缺点:

*   线程在试图获取锁时可能会被阻塞，在另一个线程的操作完成之前不会有任何进展——这有效地防止了并行性
*   锁争用越严重，JVM 花在调度线程、管理争用和等待线程队列上的时间就越多，它所做的实际工作就越少
*   如果涉及不止一个锁，并且它们以错误的顺序被获取/释放，那么死锁是可能的
*   一个[优先级反转](https://web.archive.org/web/20190209024432/https://en.wikipedia.org/wiki/Priority_inversion)的危险是可能的——一个高优先级线程被锁定，试图获得一个由低优先级线程持有的锁
*   大多数情况下使用粗粒度锁，这大大损害了并行性——细粒度锁需要更仔细的设计，增加了锁定开销，并且更容易出错

另一种方法是使用**非阻塞算法，即任何线程的故障或暂停不会导致另一个线程**的故障或暂停。

如果所涉及的线程中至少有一个能够保证在任意时间内取得进展，即在处理过程中不会出现死锁，那么非阻塞算法就是`lock-free`。

此外，如果有保证的每线程进度，这些算法就是`wait-free`。

这里有一个来自优秀的 [Java 并发实践](https://web.archive.org/web/20190209024432/https://www.amazon.com/Java-Concurrency-Practice-Brian-Goetz/dp/0321349601/ref=sr_1_1)一书的非阻塞`Stack`例子；它定义了基本状态:

```java
public class ConcurrentStack<E> {

    AtomicReference<Node<E>> top = new AtomicReference<Node<E>>();

    private static class Node <E> {
        public E item;
        public Node<E> next;

        // standard constructor
    }
}
```

还有几个 API 方法:

```java
public void push(E item){
    Node<E> newHead = new Node<E>(item);
    Node<E> oldHead;

    do {
        oldHead = top.get();
        newHead.next = oldHead;
    } while(!top.compareAndSet(oldHead, newHead));
}

public E pop() {
    Node<E> oldHead;
    Node<E> newHead;
    do {
        oldHead = top.get();
        if (oldHead == null) {
            return null;
        }
        newHead = oldHead.next;
    } while (!top.compareAndSet(oldHead, newHead));

    return oldHead.item;
}
```

我们可以看到，该算法使用细粒度的比较和交换( [CAS](https://web.archive.org/web/20190209024432/https://en.wikipedia.org/wiki/Compare-and-swap) )指令，并且是`lock-free`(即使多个线程同时调用`top.compareAndSet()`，其中一个也能保证成功)而不是`wait-free`，因为不能保证 CAS 最终对任何特定线程都成功。

## 3。依赖性

首先，让我们将 JCTools 依赖项添加到我们的`pom.xml`:

```java
<dependency>
    <groupId>org.jctools</groupId>
    <artifactId>jctools-core</artifactId>
    <version>2.1.2</version>
</dependency>
```

请注意，最新版本可在 [Maven Central](https://web.archive.org/web/20190209024432/https://search.maven.org/classic/#search%7Cgav%7C1%7Cg%3A%22org.jctools%22%20AND%20a%3A%22jctools-core%22) 上获得。

## 4。JCTools 队列

该库提供了许多在多线程环境中使用的队列，即一个或多个线程以线程安全无锁的方式写入队列，一个或多个线程从队列中读取。

所有`Queue`实现的公共接口是`org.jctools.queues.MessagePassingQueue`。

### 4.1。队列类型

所有队列都可以根据它们的生产者/消费者策略进行分类:

*   **单一生产者、单一消费者—**这些类别使用前缀`Spsc`命名，例如`SpscArrayQueue`
*   **单个生产商，多个消费者—**使用`Spmc`前缀，例如`SpmcArrayQueue`
*   **多个生产商，单个消费者—**使用`Mpsc`前缀，例如`MpscArrayQueue`
*   **多个生产商，多个消费者—**使用`Mpmc`前缀，例如`MpmcArrayQueue`

重要的是要注意到**内部没有策略检查，也就是说，在不正确使用**的情况下，队列可能会悄悄地发生故障。

例如，下面的测试从两个线程填充了一个`single-producer`队列，并通过了测试，尽管不能保证消费者能看到来自不同生产者的数据:

```java
SpscArrayQueue<Integer> queue = new SpscArrayQueue<>(2);

Thread producer1 = new Thread(() -> queue.offer(1));
producer1.start();
producer1.join();

Thread producer2 = new Thread(() -> queue.offer(2));
producer2.start();
producer2.join();

Set<Integer> fromQueue = new HashSet<>();
Thread consumer = new Thread(() -> queue.drain(fromQueue::add));
consumer.start();
consumer.join();

assertThat(fromQueue).containsOnly(1, 2);
```

### 4.2.队列实现

总结上面的分类，下面是 JCTools 队列的列表:

*   **`SpscArrayQueue`–**单个生产者，单个消费者，内部使用一个阵列，容量有限
*   **`SpscLinkedQueue`–**单个生产者，单个消费者，内部使用链表，无限制容量
*   **`SpscChunkedArrayQueue`–**单个生产商，单个消费者，从初始产能开始，增长到最大产能
*   **`SpscGrowableArrayQueue`–**单个生产商，单个消费者，从初始产能开始，增长到最大产能。这和`SpscChunkedArrayQueue`是一样的契约，唯一的区别是内部组块管理。推荐使用`SpscChunkedArrayQueue`,因为它有一个简化的实现
*   **`SpscUnboundedArrayQueue`–**单个生产者，单个消费者，内部使用一个阵列，无限制容量
*   **`SpmcArrayQueue`–**单个生产者，多个消费者，内部使用一个阵列，容量有限
*   **`MpscArrayQueue`–**多个生产者，单个消费者，内部使用一个阵列，容量受限
*   **`MpscLinkedQueue`–**多个生产商，单个消费者，内部使用链表，容量不受限制
*   **`MpmcArrayQueue`–**多个生产者，多个消费者，内部使用一个阵列，容量受限

### 4.3。原子队列

前面提到的所有队列都使用`[sun.misc.Unsafe](/web/20190209024432/https://www.baeldung.com/java-unsafe)`。然而，随着 Java 9 和 [JEP-260](https://web.archive.org/web/20190209024432/http://openjdk.java.net/jeps/260) 的出现，这个 API 在默认情况下变得不可访问。

因此，存在使用`java.util.concurrent.atomic.AtomicLongFieldUpdater`(公共 API，性能较低)而不是`sun.misc.Unsafe`的替代队列。

它们是从上面的队列中产生的，它们的名字中间插入了单词`Atomic`，例如`SpscChunkedAtomicArrayQueue`或`MpmcAtomicArrayQueue`。

建议尽可能使用“常规”队列，只有在像 HotSpot Java9+和 JRockit 这样禁止/无效使用`sun.misc.Unsafe`的环境下才求助于`AtomicQueues`。

### 4.4。容量

所有 JCTools 队列也可能有一个最大容量或者是未绑定的。当一个队列已满并且受到容量限制时，它将停止接受新元素。

在下面的例子中，我们:

*   填满队列
*   确保它在那之后停止接受新元素
*   从中汲取营养，并确保以后可以添加更多的元素

请注意，为了可读性，删除了一些代码语句。完整的实现可以在 GitHub 上的[中找到:](https://web.archive.org/web/20190209024432/https://github.com/eugenp/tutorials/blob/master/libraries/src/test/java/com/baeldung/jctools/JCToolsUnitTest.java#L45)

```java
SpscChunkedArrayQueue<Integer> queue = new SpscChunkedArrayQueue<>(8, 16);
CountDownLatch startConsuming = new CountDownLatch(1);
CountDownLatch awakeProducer = new CountDownLatch(1);

Thread producer = new Thread(() -> {
    IntStream.range(0, queue.capacity()).forEach(i -> {
        assertThat(queue.offer(i)).isTrue();
    });
    assertThat(queue.offer(queue.capacity())).isFalse();
    startConsuming.countDown();
    awakeProducer.await();
    assertThat(queue.offer(queue.capacity())).isTrue();
});

producer.start();
startConsuming.await();

Set<Integer> fromQueue = new HashSet<>();
queue.drain(fromQueue::add);
awakeProducer.countDown();
producer.join();
queue.drain(fromQueue::add);

assertThat(fromQueue).containsAll(
  IntStream.range(0, 17).boxed().collect(toSet()));
```

## 5。其他 JCTools 数据结构

JCTools 还提供了一些非队列数据结构。

所有这些都列在下面:

*   **`NonBlockingHashMap`–**一种无锁的`ConcurrentHashMap`替代方案，具有更好的可伸缩性，通常变异成本更低。它是通过`sun.misc.Unsafe`实现的，因此，不建议在 HotSpot Java9+或 JRockit 环境中使用该类
*   **`NonBlockingHashMapLong`–**与`NonBlockingHashMap`相似，但使用原始的`long`键
*   **`NonBlockingHashSet`–**像 JDK 的`java.util.Collections.newSetFromMap()`一样简单的包装`NonBlockingHashMap`
*   **`NonBlockingIdentityHashMap`–**与`NonBlockingHashMap`相似，但按身份比较键。
*   **`NonBlockingSetInt`–**一个多线程位向量集，实现为一个原语数组`longs`。在静默自动装箱的情况下无效

## 6。性能测试

让我们用 [JMH](https://web.archive.org/web/20190209024432/http://openjdk.java.net/projects/code-tools/jmh/) 来比较 JDK`ArrayBlockingQueue`和 JCTools 队列的性能。JMH 是来自 Sun/Oracle JVM 大师的开源微基准框架，它保护我们免受编译器/jvm 优化算法的不确定性的影响。请随时在这篇文章的[中了解更多细节。](/web/20190209024432/https://www.baeldung.com/java-microbenchmark-harness)

注意，为了提高可读性，下面的代码片段省略了几个语句。请在 [GitHub:](https://web.archive.org/web/20190209024432/https://github.com/eugenp/tutorials/blob/master/libraries/src/main/java/com/baeldung/jctools/MpmcBenchmark.java) 上找到完整的源代码

```java
public class MpmcBenchmark {

    @Param({PARAM_UNSAFE, PARAM_AFU, PARAM_JDK})
    public volatile String implementation;

    public volatile Queue<Long> queue;

    @Benchmark
    @Group(GROUP_NAME)
    @GroupThreads(PRODUCER_THREADS_NUMBER)
    public void write(Control control) {
        // noinspection StatementWithEmptyBody
        while (!control.stopMeasurement && !queue.offer(1L)) {
            // intentionally left blank
        }
    }

    @Benchmark
    @Group(GROUP_NAME)
    @GroupThreads(CONSUMER_THREADS_NUMBER)
    public void read(Control control) {
        // noinspection StatementWithEmptyBody
        while (!control.stopMeasurement && queue.poll() == null) {
            // intentionally left blank
        }
    }
}
```

结果(摘录自第 95 个百分位数，纳秒每操作):

```java
MpmcBenchmark.MyGroup:MyGroup·p0.95 MpmcArrayQueue sample 1052.000 ns/op
MpmcBenchmark.MyGroup:MyGroup·p0.95 MpmcAtomicArrayQueue sample 1106.000 ns/op
MpmcBenchmark.MyGroup:MyGroup·p0.95 ArrayBlockingQueue sample 2364.000 ns/op
```

我们可以看到 **`MpmcArrayQueue`比`MpmcAtomicArrayQueue`稍好一点，而`ArrayBlockingQueue`慢了两倍。**

## 7.使用 JCTools 的缺点

使用 JCTools 有一个重要的缺点—**不可能强制正确使用库类。**例如，考虑当我们在大型成熟项目中开始使用`MpscArrayQueue`时的情况(注意，必须有一个消费者)。

不幸的是，由于项目很大，有人可能会犯编程或配置错误，现在队列是从多个线程中读取的。该系统似乎像以前一样工作，但现在消费者有可能错过一些信息。这是一个真正的问题，可能会产生很大的影响，并且很难调试。

理想情况下，应该可以运行一个具有特定系统属性的系统，该系统属性强制 JCTools 确保线程访问策略。例如，本地/测试/试运行环境(而非生产环境)可能会打开它。遗憾的是，JCTools 没有提供这样的属性。

另一个考虑是，即使我们确保 JCTools 明显快于 JDK 的同类产品，这并不意味着我们的应用程序获得了与我们开始使用定制队列实现相同的速度。大多数应用程序不会在线程间交换大量对象，并且大多受限于 I/O。

## 8。结论

我们现在对 JCTools 提供的实用程序类有了一个基本的了解，并且看到了与 JDK 的同类产品相比，它们在高负载下的性能有多好。

总之，只有当我们在线程间交换大量对象时，才值得使用这个库，即使这样，也有必要非常小心地保留线程访问策略。

和往常一样，以上示例的完整源代码可以在 GitHub 上找到[。](https://web.archive.org/web/20190209024432/https://github.com/eugenp/tutorials/tree/master/libraries)