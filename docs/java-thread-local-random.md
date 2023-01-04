# Java 中的 ThreadLocalRandom 指南

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-thread-local-random>

## 1。概述

生成随机值是一项非常常见的任务。这也是 Java 提供`java.util.Random`类的原因。

然而，这个类在多线程环境中表现不佳。

简单来说，`Random`在多线程环境中性能不佳的原因是由于争用——假设多个线程共享同一个`Random`实例。

为了解决这个限制， **Java 在 JDK 7 中引入了`java.util.concurrent.ThreadLocalRandom`类——用于在多线程环境中生成随机数**。

让我们看看`ThreadLocalRandom`表现如何，如何在现实应用中使用。

## 2。`ThreadLocalRandom`超过`Random`超过

**`ThreadLocalRandom`是`[ThreadLocal](/web/20220625171946/https://www.baeldung.com/java-threadlocal)`和`Random`类的组合(稍后会详细介绍)，并且独立于当前线程。因此，通过简单地避免对`Random`实例的任何并发访问，它在多线程环境中实现了更好的性能。**

一个线程获得的随机数不受另一个线程的影响，而`java.util.Random`提供全局随机数。

另外，不像`Random,` `ThreadLocalRandom`不支持显式设置种子。相反，它覆盖了从`Random`继承的`setSeed(long seed)`方法，在被调用时总是抛出一个`UnsupportedOperationException`。

### 2.1.线程竞争

到目前为止，我们已经确定了`Random `类在高度并发的环境中表现不佳。为了更好地理解这一点，让我们看看它的主要操作之一 [`next(int)`](https://web.archive.org/web/20220625171946/https://github.com/openjdk/jdk/blob/a8a2246158bc53414394b007cbf47413e62d942e/src/java.base/share/classes/java/util/Random.java#L198) 是如何实现的:

```java
private final AtomicLong seed;

protected int next(int bits) {
    long oldseed, nextseed;
    AtomicLong seed = this.seed;
    do {
        oldseed = seed.get();
        nextseed = (oldseed * multiplier + addend) & mask;
    } while (!seed.compareAndSet(oldseed, nextseed));

    return (int)(nextseed >>> (48 - bits));
}
```

这是一个线性同余生成器算法的 Java 实现。很明显，所有线程都共享同一个`seed` 实例变量。

为了生成下一组随机位，它首先试图通过`compareAndSet`或简称为`CAS `自动改变共享的`seed `值。

**当多个线程试图使用 CAS 并发更新`seed `时，一个线程获胜并更新`seed, `，其余线程失败。失败的线程将一遍又一遍地尝试相同的过程，直到它们有机会更新值并且** **最终生成随机数。**

这种算法是无锁的，不同的线程可以并发执行。然而，当争用很高时，CAS 失败和重试的次数将严重损害整体性能。

另一方面，`ThreadLocalRandom`完全消除了这种争用，因为每个线程都有自己的`Random `实例，因此也有自己的受限`seed.`

现在让我们来看看产生随机`int, long`和`double`值的一些方法。

## 3。使用`ThreadLocalRandom` 生成随机值

根据 Oracle 文档，**我们只需要调用`ThreadLocalRandom.current()`方法，它就会为当前线程**返回`ThreadLocalRandom`的实例。然后，我们可以通过调用该类的可用实例方法来生成随机值。

让我们生成一个没有任何界限的随机`int`值:

```java
int unboundedRandomValue = ThreadLocalRandom.current().nextInt());
```

接下来，让我们看看如何生成一个随机有界的`int`值，意思是一个在给定的下限和上限之间的值。

这里有一个生成 0 到 100 之间的随机`int`值的例子:

```java
int boundedRandomValue = ThreadLocalRandom.current().nextInt(0, 100);
```

请注意，0 是包含下限，100 是不包含上限。

我们可以通过调用`nextLong()`和`nextDouble()`方法为`long`和`double`生成随机值，方法与上面的例子类似。

Java 8 还添加了`nextGaussian()`方法来生成下一个正态分布值，其平均值为 0.0，与生成器序列的标准差为 1.0。

与`Random`类一样，我们也可以使用`doubles(), ints()`和`longs()`方法来生成随机值流。

## 4。使用 JMH 比较`ThreadLocalRandom`和`Random`

让我们看看如何在多线程环境中使用这两个类生成随机值，然后使用 JMH 比较它们的性能。

首先，让我们创建一个例子，其中所有线程共享一个`Random.`实例。这里，我们将使用`Random`实例生成随机值的任务提交给一个`ExecutorService:`

```java
ExecutorService executor = Executors.newWorkStealingPool();
List<Callable<Integer>> callables = new ArrayList<>();
Random random = new Random();
for (int i = 0; i < 1000; i++) {
    callables.add(() -> {
         return random.nextInt();
    });
}
executor.invokeAll(callables);
```

让我们使用 JMH 基准测试来检查上面代码的性能:

```java
# Run complete. Total time: 00:00:36
Benchmark                                            Mode Cnt Score    Error    Units
ThreadLocalRandomBenchMarker.randomValuesUsingRandom avgt 20  771.613 ± 222.220 us/op
```

类似地，现在让我们使用`ThreadLocalRandom` 而不是`Random`实例，它为池中的每个线程使用一个`ThreadLocalRandom`实例:

```java
ExecutorService executor = Executors.newWorkStealingPool();
List<Callable<Integer>> callables = new ArrayList<>();
for (int i = 0; i < 1000; i++) {
    callables.add(() -> {
        return ThreadLocalRandom.current().nextInt();
    });
}
executor.invokeAll(callables);
```

下面是使用`ThreadLocalRandom:`的结果

```java
# Run complete. Total time: 00:00:36
Benchmark                                                       Mode Cnt Score    Error   Units
ThreadLocalRandomBenchMarker.randomValuesUsingThreadLocalRandom avgt 20  624.911 ± 113.268 us/op
```

最后，通过比较上面的`Random`和`ThreadLocalRandom`的 JMH 结果，我们可以清楚地看到，使用`Random`生成 1000 个随机值的平均时间是 772 微秒，而使用`ThreadLocalRandom`大约是 625 微秒。

因此，我们可以得出结论， **`ThreadLocalRandom`在高度并发的环境**中效率更高。

要了解更多关于 JMH 的信息，请点击这里查看我们之前的文章。

## 5.实施细节

将`ThreadLocalRandom`看作是`ThreadLocal`和`Random `类的组合是一个很好的心理模型。事实上，在 Java 8 之前，这个心理模型与实际实现是一致的。

**然而，从 Java 8 开始，随着`ThreadLocalRandom `变成了单例**，这种一致性完全被打破了。下面是 Java 8+中的 [`current()`](https://web.archive.org/web/20220625171946/https://github.com/openjdk/jdk14u/blob/89deef4dd8b7aac7c3cea6e13c494a438d34d4c4/src/java.base/share/classes/java/util/concurrent/ThreadLocalRandom.java#L176) 方法:

```java
static final ThreadLocalRandom instance = new ThreadLocalRandom();

public static ThreadLocalRandom current() {
    if (U.getInt(Thread.currentThread(), PROBE) == 0)
        localInit();

    return instance;
}
```

共享一个全局`Random `实例确实会导致高争用情况下的次优性能。然而，每个线程使用一个专用实例也是多余的。

**每个线程不需要一个专用的`Random`实例，每个线程只需要维护自己的`seed `值**。从 Java 8 开始，`[Thread](https://web.archive.org/web/20220625171946/https://github.com/openjdk/jdk14u/blob/d48548f5b7713e0d51b107a5e2dfd60383edbd88/src/java.base/share/classes/java/lang/Thread.java#L2059) `类本身已经被改进以保持`seed `值:

```java
public class Thread implements Runnable {
    // omitted

    @jdk.internal.vm.annotation.Contended("tlr")
    long threadLocalRandomSeed;

    @jdk.internal.vm.annotation.Contended("tlr")
    int threadLocalRandomProbe;

    @jdk.internal.vm.annotation.Contended("tlr")
    int threadLocalRandomSecondarySeed;
}
```

`threadLocalRandomSeed `变量负责维护`ThreadLocalRandom. `的当前种子值。此外，二级种子`threadLocalRandomSecondarySeed`通常由`ForkJoinPool.`等内部使用

这个实现结合了一些优化，使`ThreadLocalRandom `的性能更高:

*   通过使用`@Contented `注释来避免[错误共享](https://web.archive.org/web/20220625171946/https://alidg.me/blog/2020/4/24/thread-local-random#false-sharing)，这基本上是添加了足够的填充来将竞争的变量隔离在它们自己的缓存行中
*   使用`sun.misc.Unsafe`来更新这三个变量，而不是使用反射 API
*   避免与`ThreadLocal `实现相关的额外哈希表查找

## 6。结论

这篇文章阐述了`java.util.Random`和`java.util.concurrent.ThreadLocalRandom`的区别。

我们还看到了在多线程环境中`ThreadLocalRandom`相对于`Random`的优势，以及性能和我们如何使用该类生成随机值。

`ThreadLocalRandom`是 JDK 的一个简单补充，但它可以在高度并发的应用程序中产生显著的影响。

和往常一样，所有这些例子的实现都可以在 GitHub 上找到[。](https://web.archive.org/web/20220625171946/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-concurrency-advanced-2)