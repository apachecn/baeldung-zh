# Java 中常见的并发陷阱

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-common-concurrency-pitfalls>

## 1.介绍

在本教程中，我们将看到 Java 中一些最常见的并发问题。我们还将学习如何避免它们以及它们的主要原因。

## 2.使用线程安全的对象

### 2.1.共享对象

线程主要通过[共享对相同对象的访问](/web/20220926183938/https://www.baeldung.com/java-thread-safety)来进行通信。因此，在对象改变时读取会产生意想不到的结果。此外，同时更改对象会使其处于损坏或不一致的状态。

我们可以避免这种并发性问题并构建可靠代码的主要方法是使用不可变对象。这是因为它们的状态不能被多线程的干扰所修改。

然而，我们不能总是使用不可变的对象。在这些情况下，我们必须想办法让我们的可变对象线程安全。

### 2.2.使集合线程安全

像任何其他对象一样，集合在内部维护状态。这可能会因多个线程同时更改集合而改变。因此，**我们在多线程环境中安全处理集合的一种方法是[同步它们](/web/20220926183938/https://www.baeldung.com/java-synchronized-collections)** :

```java
Map<String, String> map = Collections.synchronizedMap(new HashMap<>());
List<Integer> list = Collections.synchronizedList(new ArrayList<>());
```

一般来说，同步有助于我们实现互斥。更具体地说，**这些集合一次只能被一个线程访问。因此，我们可以避免集合处于不一致的状态。**

### 2.3.专家多线程集合

现在让我们考虑一个场景，在这个场景中，我们需要更多的读操作而不是写操作。通过使用同步集合，我们的应用程序可能会受到严重的性能影响。如果两个线程想同时读取集合，其中一个必须等到另一个完成。

为此，Java 提供了可以被多线程同时访问的并发集合，如[`CopyOnWriteArrayList`](/web/20220926183938/https://www.baeldung.com/java-copy-on-write-arraylist)[`ConcurrentHashMap`](/web/20220926183938/https://www.baeldung.com/java-concurrent-map):

```java
CopyOnWriteArrayList<String> list = new CopyOnWriteArrayList<>();
Map<String, String> map = new ConcurrentHashMap<>();
```

`CopyOnWriteArrayList`通过为像添加或删除这样的变异操作创建底层数组的单独副本来实现线程安全。虽然它的写操作性能比`Collections.synchronizedList,`差，但当我们需要的读操作比写操作多得多时，它能为我们提供更好的性能。

`ConcurrentHashMap`基本上是线程安全的，比非线程安全的`Map`周围的`Collections.synchronizedMap`包装器性能更好。它实际上是线程安全映射的线程安全映射，允许不同的活动在其子映射中同时发生。

### 2.4.使用非线程安全类型

我们经常使用像 [`SimpleDateFormat`](/web/20220926183938/https://www.baeldung.com/java-simple-date-format) 这样的内置对象来解析和格式化日期对象。`SimpleDateFormat`类在执行操作时会改变其内部状态。

我们需要非常小心，因为它们不是线程安全的。在多线程应用程序中，它们的状态可能会因为竞争条件等原因而变得不一致。

那么，如何才能安全使用`SimpleDateFormat`？我们有几个选择:

*   每次使用时创建一个新的`SimpleDateFormat`实例
*   限制使用`ThreadLocal<SimpleDateFormat>`对象创建的对象数量。它保证每个线程都有自己的`SimpleDateFormat`实例
*   用`synchronized`关键字或锁同步多线程的并发访问

`SimpleDateFormat `只是这方面的一个例子。我们可以对任何非线程安全类型使用这些技术。

## 3.竞赛条件

当两个或多个线程同时访问共享数据并试图改变它时，就会发生[竞争情况](/web/20220926183938/https://www.baeldung.com/cs/race-conditions)。因此，竞争条件可能导致运行时错误或意外结果。

### 3.1.竞争条件示例

让我们考虑下面的代码:

```java
class Counter {
    private int counter = 0;

    public void increment() {
        counter++;
    }

    public int getValue() {
        return counter;
    }
}
```

`Counter`类被设计成每次调用 increment 方法都会给`counter`加 1。然而，如果一个`Counter`对象被多个线程引用，线程间的干扰可能会阻止这种情况的发生。

我们可以将`counter++`语句分解成 3 个步骤:

*   检索`counter`的当前值
*   将检索到的值增加 1
*   将增量值存储回`counter`

现在，让我们假设两个线程`thread1`和`thread2`同时调用 increment 方法。它们的交错操作可能遵循以下顺序:

*   `thread1`读取`counter`的当前值；0
*   `thread2`读取`counter`的当前值；0
*   `thread1`增加检索值；结果是 1
*   `thread2`增加检索值；结果是 1
*   `thread1`将结果存储在`counter`中；结果现在是 1
*   `thread2`将结果存储在`counter`中；结果现在是 1

我们期望`counter`的值是 2，但是它是 1。

### 3.2。基于同步的解决方案

我们可以通过同步关键代码来解决这种不一致:

```java
class SynchronizedCounter {
    private int counter = 0;

    public synchronized void increment() {
        counter++;
    }

    public synchronized int getValue() {
        return counter;
    }
}
```

任何时候都只允许一个线程使用一个对象的`synchronized`方法，所以这就强制了对`counter`的读写的一致性。

### 3.3。内置解决方案

我们可以用内置的`AtomicInteger`对象替换上面的代码。这个类提供了增加整数的原子方法，这是比我们自己写代码更好的解决方案。因此，我们可以直接调用它的方法，而不需要同步:

```java
AtomicInteger atomicInteger = new AtomicInteger(3);
atomicInteger.incrementAndGet();
```

在这种情况下，SDK 为我们解决了问题。否则，我们也可以编写自己的代码，将关键部分封装在自定义的线程安全类中。这种方法帮助我们最小化复杂性，最大化代码的可重用性。

## 4.集合周围的竞争条件

### 4.1。问题

我们可能陷入的另一个陷阱是认为同步收集比它们实际提供的保护更多。

让我们检查下面的代码:

```java
List<String> list = Collections.synchronizedList(new ArrayList<>());
if(!list.contains("foo")) {
    list.add("foo");
}
```

我们的列表的每个操作都是同步的，但是多个方法调用的任何组合都不是同步的。更具体地说，在这两个操作之间，另一个线程可能会修改我们的集合，导致不希望的结果。

例如，两个线程可以同时进入`if`块，然后更新列表，每个线程将`foo`值添加到列表中。

### 4.2。列表的解决方案

我们可以使用同步保护代码不被多个线程同时访问:

```java
synchronized (list) {
    if (!list.contains("foo")) {
        list.add("foo");
    }
}
```

我们没有在函数中添加关键字`synchronized`，而是创建了一个关于`list,`的临界区，它一次只允许一个线程执行这个操作。

我们应该注意，我们可以在列表对象的其他操作上使用`synchronized(list)`，以提供一个**保证一次只有一个线程可以在这个对象上执行我们的任何操作**。

### 4.3。`ConcurrentHashMap` 内置解决方案

现在，出于同样的原因，让我们考虑使用一个 map，即仅在条目不存在的情况下添加条目。

`ConcurrentHashMap`为这类问题提供了更好的解决方案。我们可以用它的原子`putIfAbsent`方法:

```java
Map<String, String> map = new ConcurrentHashMap<>();
map.putIfAbsent("foo", "bar");
```

或者，如果我们想计算值，它的原子`computeIfAbsent`方法`:`

```java
map.computeIfAbsent("foo", key -> key + "bar");
```

我们应该注意，这些方法是`Map`接口的一部分，它们提供了一种方便的方式来避免围绕插入编写条件逻辑。当试图使多线程调用原子化时，它们真的帮了我们大忙。

## 5.内存一致性问题

当多个线程对相同数据的看法不一致时，就会出现内存一致性问题。

除了主内存之外，大多数现代计算机体系结构都使用缓存层次结构(L1、L2 和 L3 缓存)来提高整体性能。因此，任何线程都可以缓存变量，因为与主存储器相比，它提供了更快的访问。

### 5.1.问题是

让我们回忆一下我们的`Counter`例子:

```java
class Counter {
    private int counter = 0;

    public void increment() {
        counter++;
    }

    public int getValue() {
        return counter;
    }
}
```

让我们考虑这样一个场景，其中`thread1`递增`counter`，然后`thread2`读取它的值。可能会发生以下一系列事件:

*   `thread1`从自己的缓存中读取计数器值；计数器为 0
*   t `hread1`递增计数器并将其写回自己的缓存；计数器为 1
*   `thread2`从自己的缓存中读取计数器值；计数器为 0

当然，预期的事件序列也可能发生，`t` `hread2`将读取正确的值(1)，但是**不能保证一个线程所做的更改每次都能被其他线程看到。**

### 5.2.解决方案

为了避免内存一致性错误，**我们需要建立一个发生前关系**。这种关系只是保证一个特定语句的内存更新对另一个特定语句是可见的。

有几种策略可以创造先发制人的关系。其中之一是同步，我们已经讨论过了。

**同步既保证互斥，又保证内存一致性。**然而，这是有性能成本的。

我们还可以通过使用`volatile`关键字来避免内存一致性问题。简单地说，**对一个 volatile 变量的每一次改变对其他线程总是可见的。**

让我们用`volatile`重写我们的`Counter`例子:

```java
class SyncronizedCounter {
    private volatile int counter = 0;

    public synchronized void increment() {
        counter++;
    }

    public int getValue() {
        return counter;
    }
}
```

我们应该注意到**我们仍然需要同步增量操作，因为`volatile`不保证我们互斥。**使用简单的原子变量访问比通过同步代码访问这些变量更有效。

### 5.3.非原子`long`和`double`值

因此，如果我们在没有正确同步的情况下读取变量，我们可能会看到一个过时的值。 **F** **或`long `和`double `值，非常令人惊讶的是，除了陈旧的值之外，甚至还可以看到完全随机的值。**

**根据 [JLS-17](https://web.archive.org/web/20220926183938/https://docs.oracle.com/javase/specs/jls/se14/html/jls-17.html#jls-17.7) ，JVM 可能会将 64 位操作视为两个独立的 32 位操作**。因此，当读取一个`long `或`double `值时，有可能读取一个更新的 32 位和一个陈旧的 32 位。因此，我们可能会在并发上下文中观察到随机出现的`long `或`double`值。

另一方面，volatile `long`和`double`值的写入和读取总是原子的。

## 6.误用同步

[同步机制](/web/20220926183938/https://www.baeldung.com/java-thread-safety)是实现线程安全的强大工具。它依赖于内部锁和外部锁的使用。我们还要记住这样一个事实，每个对象都有一个不同的锁，一次只有一个线程可以获得一个锁。

然而，如果我们不注意并仔细地为我们的关键代码选择正确的锁，意外的行为就会发生。

### 6.1.在`this`参考上同步

方法级同步是许多并发问题的解决方案。但是，如果过度使用，也会导致其他并发问题。这种同步方法依赖于作为锁的`this`引用，也称为固有锁。

在下面的例子中，我们可以看到如何将方法级同步转换成块级同步，并将`this`引用作为锁。

这些方法是等效的:

```java
public synchronized void foo() {
    //...
}
```

```java
public void foo() {
    synchronized(this) {
      //...
    }
}
```

当线程调用这样的方法时，其他线程不能同时访问该对象。这可能会降低并发性能，因为所有东西最终都是单线程运行的。当一个对象被读取的次数多于更新的次数时，这种方法尤其糟糕。

此外，我们代码的客户端也可能获得`this`锁。在最坏的情况下，这个操作会导致死锁。

### 6.2.僵局

**[死锁](/web/20220926183938/https://www.baeldung.com/java-dining-philoshophers)描述了两个或多个线程互相阻塞**的情况，每个线程都在等待获取其他线程持有的资源。

让我们考虑这个例子:

```java
public class DeadlockExample {

    public static Object lock1 = new Object();
    public static Object lock2 = new Object();

    public static void main(String args[]) {
        Thread threadA = new Thread(() -> {
            synchronized (lock1) {
                System.out.println("ThreadA: Holding lock 1...");
                sleep();
                System.out.println("ThreadA: Waiting for lock 2...");

                synchronized (lock2) {
                    System.out.println("ThreadA: Holding lock 1 & 2...");
                }
            }
        });
        Thread threadB = new Thread(() -> {
            synchronized (lock2) {
                System.out.println("ThreadB: Holding lock 2...");
                sleep();
                System.out.println("ThreadB: Waiting for lock 1...");

                synchronized (lock1) {
                    System.out.println("ThreadB: Holding lock 1 & 2...");
                }
            }
        });
        threadA.start();
        threadB.start();
    }
}
```

在上面的代码中我们可以清楚的看到，第一个`threadA`获得`lock1`，`threadB`获得`lock2`。然后，`threadA`试图获取已经被`threadB`获取的`lock2`，而`threadB`试图获取已经被`threadA`获取的`lock1`。因此，他们都不会进行，这意味着他们陷入了僵局。

我们可以通过改变其中一个线程的锁的顺序来轻松解决这个问题。

我们应该注意，这只是一个例子，还有许多其他例子会导致死锁。

## 7.结论

在本文中，我们探讨了在多线程应用程序中可能遇到的几个并发问题的例子。

首先，我们了解到我们应该选择不可变的或者线程安全的对象或者操作。

然后，我们看到了几个竞争条件的例子，以及如何使用同步机制来避免它们。此外，我们还了解了与记忆相关的竞态条件以及如何避免它们。

尽管同步机制有助于我们避免许多并发问题，但我们很容易误用它并产生其他问题。出于这个原因，我们研究了当这种机制被滥用时我们可能面临的几个问题。

和往常一样，本文中使用的所有例子都可以在 GitHub 上找到。