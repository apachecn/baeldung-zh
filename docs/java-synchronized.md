# Java 中的同步关键字指南

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-synchronized>

## 1。概述

这个快速教程将介绍如何在 Java 中使用`synchronized`块。

简而言之，在多线程环境中，当两个或更多线程试图同时更新可变共享数据时，就会出现[竞争条件](/web/20221006231448/https://www.baeldung.com/cs/race-conditions)。Java 提供了一种机制，通过同步线程对共享数据的访问来避免竞争情况。

一段标有`synchronized`的逻辑变成了一个同步块，**在任何给定时间只允许一个线程执行**。

## 2.为什么要同步？

让我们考虑一个典型的竞争情况，我们计算总和，多个线程执行`calculate()` 方法:

```java
public class BaeldungSynchronizedMethods {

    private int sum = 0;

    public void calculate() {
        setSum(getSum() + 1);
    }

    // standard setters and getters
} 
```

那么让我们写一个简单的测试:

```java
@Test
public void givenMultiThread_whenNonSyncMethod() {
    ExecutorService service = Executors.newFixedThreadPool(3);
    BaeldungSynchronizedMethods summation = new BaeldungSynchronizedMethods();

    IntStream.range(0, 1000)
      .forEach(count -> service.submit(summation::calculate));
    service.awaitTermination(1000, TimeUnit.MILLISECONDS);

    assertEquals(1000, summation.getSum());
}
```

我们使用一个带有 3 线程池的`ExecutorService` 来执行`calculate()` 1000 次。

如果我们串行执行，预期的输出将是 1000，但是**我们的多线程执行几乎每次都失败**,因为实际输出不一致:

```java
java.lang.AssertionError: expected:<1000> but was:<965>
at org.junit.Assert.fail(Assert.java:88)
at org.junit.Assert.failNotEquals(Assert.java:834)
...
```

当然，我们并不觉得这个结果出乎意料。

避免竞争情况的一个简单方法是通过使用`synchronized`关键字使操作线程安全。

## 3。`Synchronized` 关键词

我们可以在不同的层面上使用`synchronized`关键字:

*   实例方法
*   静态方法
*   代码块

当我们使用一个`synchronized`块时，Java 内部使用一个[监视器](/web/20221006231448/https://www.baeldung.com/cs/monitor)，也称为监视器锁或固有锁，来提供同步。这些监视器被绑定到一个对象；因此，同一对象的所有同步块只能有一个线程同时执行它们。

### 3.1。`Synchronized` 实例方法

我们可以在方法声明中添加`synchronized` 关键字，使方法同步:

```java
public synchronized void synchronisedCalculate() {
    setSum(getSum() + 1);
}
```

注意，一旦我们同步了方法，测试用例通过，实际输出为 1000:

```java
@Test
public void givenMultiThread_whenMethodSync() {
    ExecutorService service = Executors.newFixedThreadPool(3);
    SynchronizedMethods method = new SynchronizedMethods();

    IntStream.range(0, 1000)
      .forEach(count -> service.submit(method::synchronisedCalculate));
    service.awaitTermination(1000, TimeUnit.MILLISECONDS);

    assertEquals(1000, method.getSum());
}
```

实例方法`synchronized`在拥有该方法的类的实例之上，这意味着该类的每个实例只有一个线程可以执行该方法。

### 3.2。`Synchronized`统计`c` 方法

静态方法`synchronized` 就像实例方法:

```java
 public static synchronized void syncStaticCalculate() {
     staticSum = staticSum + 1;
 }
```

这些方法是与类相关联的`Class`对象上的`synchronized` 。因为每个 JVM 每个类只有一个`Class`对象，所以每个类只有一个线程可以在`static` `synchronized` 方法中执行，不管它有多少实例。

让我们来测试一下:

```java
@Test
public void givenMultiThread_whenStaticSyncMethod() {
    ExecutorService service = Executors.newCachedThreadPool();

    IntStream.range(0, 1000)
      .forEach(count -> 
        service.submit(BaeldungSynchronizedMethods::syncStaticCalculate));
    service.awaitTermination(100, TimeUnit.MILLISECONDS);

    assertEquals(1000, BaeldungSynchronizedMethods.staticSum);
}
```

### 3.3。`Synchronized`方法中的块

有时我们不想同步整个方法，只同步其中的一些指令。我们可以通过`applying`同步到一个块来实现这一点:

```java
public void performSynchronisedTask() {
    synchronized (this) {
        setCount(getCount()+1);
    }
}
```

然后我们可以测试变化:

```java
@Test
public void givenMultiThread_whenBlockSync() {
    ExecutorService service = Executors.newFixedThreadPool(3);
    BaeldungSynchronizedBlocks synchronizedBlocks = new BaeldungSynchronizedBlocks();

    IntStream.range(0, 1000)
      .forEach(count -> 
        service.submit(synchronizedBlocks::performSynchronisedTask));
    service.awaitTermination(100, TimeUnit.MILLISECONDS);

    assertEquals(1000, synchronizedBlocks.getCount());
}
```

注意，我们将参数`this` 传递给了`synchronized` 块。这是监视器对象。块内的代码在 monitor 对象上同步。简单地说，每个 monitor 对象只有一个线程可以在该代码块中执行。

如果方法是`static`，我们将传递类名来代替对象引用，并且该类将是块同步的监视器:

```java
public static void performStaticSyncTask(){
    synchronized (SynchronisedBlocks.class) {
        setStaticCount(getStaticCount() + 1);
    }
}
```

让我们测试一下`static` 方法中的代码块:

```java
@Test
public void givenMultiThread_whenStaticSyncBlock() {
    ExecutorService service = Executors.newCachedThreadPool();

    IntStream.range(0, 1000)
      .forEach(count -> 
        service.submit(BaeldungSynchronizedBlocks::performStaticSyncTask));
    service.awaitTermination(100, TimeUnit.MILLISECONDS);

    assertEquals(1000, BaeldungSynchronizedBlocks.getStaticCount());
}
```

### 3.4.可重入

**`synchronized`方法和块后面的锁是可重入的。**这意味着当前线程可以在持有锁的同时反复获取同一个`synchronized`锁:

```java
Object lock = new Object();
synchronized (lock) {
    System.out.println("First time acquiring it");

    synchronized (lock) {
        System.out.println("Entering again");

         synchronized (lock) {
             System.out.println("And again");
         }
    }
}
```

如上所示，当我们在一个`synchronized `块中时，我们可以重复获取同一个监视器锁。

## 4。结论

在这篇简短的文章中，我们探索了使用`synchronized`关键字实现线程同步的不同方法。

我们还了解了竞争条件如何影响我们的应用程序，以及同步如何帮助我们避免这种情况。关于在 Java 中使用锁的线程安全的更多信息，请参考我们的`java.util.concurrent.Locks` [文章](/web/20221006231448/https://www.baeldung.com/java-concurrent-locks)。

这篇文章的完整代码可以在 GitHub 的[上找到。](https://web.archive.org/web/20221006231448/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-concurrency-simple)