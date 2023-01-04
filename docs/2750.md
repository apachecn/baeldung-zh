# 线程安全 LIFO 数据结构实现

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-lifo-thread-safe>

## 1.介绍

在本教程中，**我们将讨论线程安全 LIFO 数据结构实现的各种选择**。

在 LIFO 数据结构中，根据后进先出原则插入和检索元素。这意味着首先检索最后插入的元素。

**在计算机科学中，`stack`是用来指代这种数据结构的术语。**

A `stack`可以方便地处理一些有趣的问题，如表达式求值、实现撤销操作等。因为它可以在并发执行环境中使用，我们可能需要使它是线程安全的。

## 2。`Stacks`了解

基本上，一个 **`Stack`必须实现以下方法:**

1.  `push()`–在顶部添加一个元素
2.  `pop()`–取出并移除顶部元素
3.  `peek()`–获取元素而不从底层容器中移除

如前所述，假设我们需要一个命令处理引擎。

在这个系统中，撤销已执行的命令是一个重要的特性。

一般来说，所有的命令都被推送到堆栈上，然后可以简单地实现撤销操作:

*   `pop()`方法获取最后执行的命令
*   在弹出的命令对象上调用`undo()`方法

## 3。在`Stacks` 中了解线程安全

**如果一个数据结构不是线程安全的，当并发访问时，它可能会出现竞争情况**。

简而言之，当代码的正确执行依赖于线程的时间和顺序时，就会出现竞争情况。这种情况主要发生在多个线程共享数据结构并且该结构不是为此目的而设计的情况下。

下面让我们来看看 Java 集合类中的一个方法，`ArrayDeque`:

```
public E pollFirst() {
    int h = head;
    E result = (E) elements[h];
    // ... other book-keeping operations removed, for simplicity
    head = (h + 1) & (elements.length - 1);
    return result;
}
```

为了解释上述代码中潜在的竞争情况，让我们假设有两个线程按以下顺序执行该代码:

*   第一个线程执行第三行:用索引“head”处的元素设置结果对象
*   第二个线程执行第三行:用索引“head”处的元素设置结果对象
*   第一个线程执行第五行:将索引“head”重置为后备数组中的下一个元素
*   第二个线程执行第五行:将索引“head”重置为后备数组中的下一个元素

哎呀！现在，两次执行都将返回相同的结果对象`. `

为了避免这种竞争情况，在这种情况下，一个线程不应该执行第一行，直到另一个线程完成重置第五行的“head”索引。换句话说，对于一个线程，访问索引“head”处的元素和重置索引“head”应该自动发生。

显然，在这种情况下，代码的正确执行依赖于线程的计时，因此它不是线程安全的。

## 4.使用锁的线程安全堆栈

在这一节中，我们将讨论线程安全`stack. `的具体实现的两种可能选择

特别是，我们将讨论 Java `Stack `和线程安全修饰的`ArrayDeque. `

两者都使用[锁](/web/20220625230737/https://www.baeldung.com/java-concurrent-locks)进行互斥访问`.`

### 4.1.使用 Java `Stack`

**Java Collections 有一个线程安全 [`Stack`](/web/20220625230737/https://www.baeldung.com/java-stack) 的遗留实现，基于`Vector`，它基本上是`ArrayList.`** 的同步变体

不过官方 doc 本身建议考虑使用`ArrayDeque`。因此，我们不会进入太多的细节。

尽管 Java `Stack`是线程安全的，使用起来也很简单，但是这个类有一些主要的缺点:

*   它不支持设置初始容量
*   它对所有操作都使用锁。这可能会损害单线程执行的性能。

### 4.2.使用`ArrayDeque`

**使用`Deque`接口是 LIFO 数据结构最方便的方法，因为它提供了所有需要的堆栈操作。** [`ArrayDeque`](/web/20220625230737/https://www.baeldung.com/java-array-deque) 就是这样一个具体的实现`.  `

由于它不使用锁来执行操作，单线程执行会工作得很好。但是对于多线程执行，这是有问题的。

然而，我们可以为`ArrayDeque.` 实现一个同步装饰器，尽管它的表现类似于 Java 集合框架的 `Stack` 类，解决了`Stack`类的重要问题，即缺少初始容量设置。

让我们来看看这个类:

```
public class DequeBasedSynchronizedStack<T> {

    // Internal Deque which gets decorated for synchronization.
    private ArrayDeque<T> dequeStore;

    public DequeBasedSynchronizedStack(int initialCapacity) {
        this.dequeStore = new ArrayDeque<>(initialCapacity);
    }

    public DequeBasedSynchronizedStack() {
        dequeStore = new ArrayDeque<>();
    }

    public synchronized T pop() {
        return this.dequeStore.pop();
    }

    public synchronized void push(T element) {
        this.dequeStore.push(element);
    }

    public synchronized T peek() {
        return this.dequeStore.peek();
    }

    public synchronized int size() {
        return this.dequeStore.size();
    }
}
```

注意，为了简单起见，我们的解决方案没有实现`Deque`本身，因为它包含了更多的方法。

此外，Guava 包含了`[SynchronizedDeque](https://web.archive.org/web/20220625230737/https://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/Queues.html#synchronizedDeque-java.util.Deque-) `，它是修饰过的`ArrayDequeue.`的生产就绪实现

## 5.无锁线程安全堆栈

[`ConcurrentLinkedDeque`](https://web.archive.org/web/20220625230737/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/concurrent/ConcurrentLinkedDeque.html) 是`Deque`接口的无锁实现。**这个实现是完全线程安全的**，因为它使用了[高效的无锁算法。](https://web.archive.org/web/20220625230737/http://www.cs.rochester.edu/~scott/papers/1996_PODC_queues.pdf)

与基于锁的实现不同，无锁实现不受以下问题的影响。

*   [**优先级反转**](https://web.archive.org/web/20220625230737/https://www.semanticscholar.org/paper/Avoidance-of-Priority-Inversion-in-Real-Time-Based-Helmy-Jafri/d286108f62af8f65ad8acad184a5360e3acbc112)–当低优先级线程持有高优先级线程所需的锁时，就会发生这种情况。这可能会导致高优先级线程阻塞
*   **死锁**–当不同的线程以不同的顺序锁定同一组资源时，就会出现这种情况。

最重要的是，无锁实现的一些特性使它们非常适合在单线程和多线程环境中使用。

*   对于非共享数据结构和**单线程访问，性能与`ArrayDeque`** 相当
*   对于共享数据结构，性能**根据同时访问它的线程数量**而变化。

就可用性而言，它与`ArrayDeque`没有什么不同，因为两者都实现了 `Deque` 接口。

## 6.结论

在本文中，我们讨论了`stack `数据结构及其在设计命令处理引擎和表达式求值器等系统中的好处。

此外，我们分析了 Java 集合框架中的各种堆栈实现，并讨论了它们的性能和线程安全的细微差别。

像往常一样，代码示例可以在 GitHub 上找到[。](https://web.archive.org/web/20220625230737/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-collections)