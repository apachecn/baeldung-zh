# 虚假分享与@争鸣指南

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-false-sharing-contended>

## 1.概观

在本文中，我们将会看到有时假共享是如何让多线程对我们不利的。

首先，我们将从缓存和空间局部性的理论开始。然后我们将重写`LongAdder `并发实用程序，并根据`java.util.concurrent `实现对其进行基准测试。在整篇文章中，我们将使用不同级别的基准测试结果来调查虚假共享的影响。

文章中与 Java 相关的部分非常依赖于对象的内存布局。由于这些布局细节不是 JVM 规范的一部分，而是留给实现者的[判断，我们将只关注一个特定的 JVM 实现:HotSpot JVM。在整篇文章中，我们可能还会互换使用 JVM 和 HotSpot JVM 这两个术语。](https://web.archive.org/web/20220625084255/https://docs.oracle.com/javase/specs/jvms/se14/html/jvms-2.html)

## 2.高速缓存行和一致性

处理器使用不同级别的缓存—当处理器从主内存中读取一个值时，它可能会缓存该值以提高性能。

事实证明，**大多数现代处理器不仅缓存所请求的值，还缓存一些更接近的值**。这种优化基于空间局部性的思想，可以显著提高应用程序的整体性能。简而言之，处理器缓存是根据缓存行工作的，而不是单个可缓存的值。

**当多个处理器在相同或邻近的存储器位置上操作时，它们可能最终共享相同的高速缓存线**。在这种情况下，保持不同内核中的重叠缓存相互一致至关重要。维护这种一致性的行为称为缓存一致性。

有相当多的协议来维护 CPU 内核之间的缓存一致性。在本文中，我们将讨论 MESI 协议。

### 2.1.《MESI 议定书》

在 MESI 协议中，每个高速缓存行可以处于这四种不同状态之一:修改、独占、共享或无效。【these 这个词是这些州的首字母缩写。

为了更好地理解这个协议是如何工作的，让我们看一个例子。假设两个内核将从附近的内存位置读取数据:

[![false-sharing-exclusive](img/670ab2f59e2c5443fc5595409c1f1324.png)](/web/20220625084255/https://www.baeldung.com/wp-content/uploads/2020/07/false-sharing-exclusive-1.png)

内核`A `从主存储器中读取`a `的值。如上所示，这个内核从内存中获取更多的值，并将它们存储到缓存行中。**然后，它将该高速缓存行标记为`exclusive`，因为内核`A `是在该高速缓存行**上操作的唯一内核。从现在起，如果可能，该内核将通过从高速缓存行读取来避免低效的存储器访问。

过了一会儿，内核`B` 也决定从主存储器中读取`b `的值:

[![false-sharing-shared](img/95e3beffc758dc2744ef21da2ac65dda.png)](/web/20220625084255/https://www.baeldung.com/wp-content/uploads/2020/07/false-sharing-shared.png)

由于`a `和`b `彼此非常接近，并且驻留在同一个缓存行中，**两个内核都会将其缓存行标记为`shared`** 。

现在，让我们假设内核`A `决定改变`a`的值: [![false-sharing-invalid](img/c88af588aed0539a04dbac477ab55817.png)](/web/20220625084255/https://www.baeldung.com/wp-content/uploads/2020/07/false-sharing-invalid.png)

**内核`A`仅在其存储缓冲区中存储这一变化，并将其缓存线标记为`modified`。此外，它将这一变化传达给内核`B, `，并且该内核将依次将其缓存线标记为`invalid`。**

这就是不同的处理器如何确保它们的缓存相互一致。

## 3.虚假分享

现在，让我们看看当内核`B `决定重新读取`b`的值时会发生什么。由于这个值最近没有改变，我们可能期望从缓存行快速读取。然而，共享多处理器体系结构的本质使这种期望在现实中无效。

如前所述，整个缓存线由两个内核共享。**由于内核`B `的缓存线现在是`invalid`，它应该再次从主内存中读取值`b `**:

[![false-sharing-flush](img/e7e3b98e6372f344d3ee4925c95e4949.png)](/web/20220625084255/https://www.baeldung.com/wp-content/uploads/2020/07/false-sharing-flush.png)

如上图所示，从主内存读取相同的`b `值并不是这里唯一的低效。**这种内存访问将迫使内核`A `刷新其存储缓冲区，因为内核`B `需要获得最新值**。在刷新和获取值之后，两个内核将再次以标记为`shared`状态的最新高速缓存行版本结束:

[![false-sharing-shared-again](img/4975b1301ff79eecea51497a9bbf8b9e.png)](/web/20220625084255/https://www.baeldung.com/wp-content/uploads/2020/07/false-sharing-shared-again.png)

**因此，这会对一个内核造成高速缓存未命中，并对另一个内核造成早期缓冲区刷新，即使两个内核不在同一内存位置运行**。这种现象称为假共享，会损害整体性能，尤其是当缓存未命中率很高时。更具体地说，当这个速率很高时，处理器会不断地访问主内存，而不是从缓存中读取。

## 4.示例:动态条带化

为了演示错误共享如何影响应用程序的吞吐量或延迟，我们将在这一部分作弊。让我们定义两个空类:

```java
abstract class Striped64 extends Number {}
public class LongAdder extends Striped64 implements Serializable {}
```

当然，空类没那么有用，所以让我们复制粘贴一些逻辑到它们里面。

对于我们的`Striped64 `类，我们可以从`[java.util.concurrent.atomic.Striped64](https://web.archive.org/web/20220625084255/https://github.com/openjdk/jdk/blob/master/src/java.base/share/classes/java/util/concurrent/atomic/Striped64.java) `类中复制所有内容并粘贴到我们的类中。请确保也复制`import `报表。此外，如果使用 Java 8，我们应该确保将对`[sun.misc.Unsafe.getUnsafe()](https://web.archive.org/web/20220625084255/https://github.com/openjdk/jdk/blob/faf4d7ccb792b16092c791c0ac77acdd440dbca1/src/java.base/share/classes/jdk/internal/misc/Unsafe.java#L91) `方法的任何调用替换为自定义调用:

```java
private static Unsafe getUnsafe() {
    try {
        Field field = Unsafe.class.getDeclaredField("theUnsafe");
        field.setAccessible(true);

        return (Unsafe) field.get(null);
    } catch (Exception e) {
        throw new RuntimeException(e);
    }
}
```

我们不能从我们的应用程序类加载器中调用`sun.misc.Unsafe.getUnsafe() `，所以我们不得不用这个静态方法再次作弊。[然而，从 Java 9](https://web.archive.org/web/20220625084255/https://github.com/openjdk/jdk/blob/10e6a6a19a2743b16e7b36ec4329f656c28090a9/src/java.base/share/classes/java/util/concurrent/atomic/Striped64.java#L141) 开始，相同的逻辑是使用 [`VarHandles`](/web/20220625084255/https://www.baeldung.com/java-variable-handles) 实现的，所以我们不需要在那里做任何特殊的事情，简单的复制粘贴就足够了。

对于`LongAdder `类，让我们复制`[java.util.concurrent.atomic.LongAdder](https://web.archive.org/web/20220625084255/https://github.com/openjdk/jdk/blob/master/src/java.base/share/classes/java/util/concurrent/atomic/LongAdder.java) `类中的所有内容并粘贴到我们的类中。同样，我们也应该照搬`import `的说法。

现在，让我们对比这两个类:我们的自定义`LongAdder `和`java.util.concurrent.atomic.LongAdder.`

### 4.1.基准

为了对比这些类，让我们编写一个简单的 [JMH](/web/20220625084255/https://www.baeldung.com/java-microbenchmark-harness) 基准:

```java
@State(Scope.Benchmark)
public class FalseSharing {

    private java.util.concurrent.atomic.LongAdder builtin = new java.util.concurrent.atomic.LongAdder();
    private LongAdder custom = new LongAdder();

    @Benchmark
    public void builtin() {
        builtin.increment();
    }

    @Benchmark
    public void custom() {
        custom.increment();
    }
}
```

如果我们在吞吐量基准测试模式下用两个分支和 16 个线程运行这个基准测试(相当于传递`“`–`-bm thrpt -f 2 -t 16″ `参数)，那么 JMH 将打印这些统计数据:

```java
Benchmark              Mode  Cnt          Score          Error  Units
FalseSharing.builtin  thrpt   40  523964013.730 ± 10617539.010  ops/s
FalseSharing.custom   thrpt   40  112940117.197 ±  9921707.098  ops/s
```

结果根本说不通。**JDK 的内置实现比我们的复制粘贴解决方案高出近 360%的吞吐量**。

让我们来看看延迟之间的区别:

```java
Benchmark             Mode  Cnt   Score   Error  Units
FalseSharing.builtin  avgt   40  28.396 ± 0.357  ns/op
FalseSharing.custom   avgt   40  51.595 ± 0.663  ns/op
```

如上所示，内置解决方案还具有更好的延迟特性。

为了更好地理解这些看似相同的实现有什么不同，让我们检查一些低级性能监视计数器。

## 5.性能事件

为了检测低级 CPU 事件，如周期、停止周期、每个周期的指令、缓存加载/未命中或内存加载/存储，我们可以在处理器上编程特殊的硬件寄存器。

事实证明，像`perf`或`eBPF`这样的工具已经在使用这种方法来公开有用的度量。从 Linux 2.6.31 开始， [perf](https://web.archive.org/web/20220625084255/https://github.com/torvalds/linux/tree/master/tools/perf) 是标准的 Linux 探查器，能够公开有用的性能监视计数器或 PMC。

因此，我们可以使用 perf 事件来查看在运行这两个基准测试时 CPU 级别发生了什么。例如，如果我们运行:

```java
perf stat -d java -jar benchmarks.jar -f 2 -t 16 --bm thrpt custom
```

Perf 将让 JMH 针对复制粘贴的解决方案运行基准测试，并打印统计数据:

```java
161657.133662      task-clock (msec)         #    3.951 CPUs utilized
         9321      context-switches          #    0.058 K/sec
          185      cpu-migrations            #    0.001 K/sec
        20514      page-faults               #    0.127 K/sec
            0      cycles                    #    0.000 GHz
 219476182640      instructions
  44787498110      branches                  #  277.052 M/sec
     37831175      branch-misses             #    0.08% of all branches
  91534635176      L1-dcache-loads           #  566.227 M/sec
   1036004767      L1-dcache-load-misses     #    1.13% of all L1-dcache hits
```

`L1-dcache-load-misses `字段表示 L1 数据高速缓存的高速缓存未命中的数量。如上所示，该解决方案遇到了大约 10 亿次缓存未命中(准确地说是 1，036，004，767 次)。如果我们为内置方法收集相同的统计数据:

```java
161742.243922      task-clock (msec)         #    3.955 CPUs utilized
         9041      context-switches          #    0.056 K/sec
          220      cpu-migrations            #    0.001 K/sec
        21678      page-faults               #    0.134 K/sec
            0      cycles                    #    0.000 GHz
 692586696913      instructions
 138097405127      branches                  #  853.812 M/sec
     39010267      branch-misses             #    0.03% of all branches
 291832840178      L1-dcache-loads           # 1804.308 M/sec
    120239626      L1-dcache-load-misses     #    0.04% of all L1-dcache hits
```

我们会看到，与定制方法相比，它遇到的缓存未命中要少得多(120，239，626 ~ 120 百万)。因此，大量的高速缓存未命中可能是造成性能差异的罪魁祸首。

让我们更深入地挖掘`LongAdder `的内部表示，找到真正的罪魁祸首。

## 6.动态条带化再探

`java.util.concurrent.atomic.LongAdder `是一个高吞吐量的原子计数器实现。它不是只使用一个计数器，而是使用一组计数器来分配它们之间的内存争用。这样，它将在竞争激烈的应用程序中胜过简单的原子，如`AtomicLong `。

`Striped64 `类负责这种内存争用的分配，这就是这个类如何实现那些计数器的[数组:](https://web.archive.org/web/20220625084255/https://github.com/openjdk/jdk/blob/faf4d7ccb792b16092c791c0ac77acdd440dbca1/src/java.base/share/classes/java/util/concurrent/atomic/Striped64.java#L124)

```java
@jdk.internal.vm.annotation.Contended 
static final class Cell {
    volatile long value;
    // omitted
}
transient volatile Cell[] cells;
```

每个`Cell`封装了每个计数器的细节。这种实现使得不同的线程可以更新不同的内存位置。由于我们使用的是状态数组(也就是条带),这种想法被称为动态条带化。有趣的是，`Striped64 `就是以这种思想和它适用于 64 位数据类型的事实命名的。

无论如何，JVM 可能会在堆中彼此靠近地分配这些计数器。也就是说，这些计数器中的一些将在相同的高速缓存行中。因此，**更新一个计数器可能会使附近计数器**的缓存无效。

这里的关键要点是，动态条带化的简单实现将遭受错误的共享。然而，**通过在每个计数器周围添加足够的填充符，我们可以确保每个计数器都驻留在其缓存行上，从而防止错误共享**:

[![false-sharing-padding](img/2511b7b19c81ee2abe3ca1115d694fd1.png)](/web/20220625084255/https://www.baeldung.com/wp-content/uploads/2020/07/false-sharing-padding.png)

事实证明，`@` `jdk.internal.vm.annotation.Contended `注释负责添加这些填充。

唯一的问题是，为什么这个注释在复制粘贴的实现中不起作用？

## 7.见面`@Contended`

**Java 8 引入了`sun.misc.Contended `注释(Java 9 将其重新打包在`jdk.internal.vm.annotation `包下)防止虚假分享**。

基本上，当我们用这个注释来注释一个字段时，HotSpot JVM 会在被注释的字段周围添加一些填充[。这样，它可以确保字段驻留在自己的缓存行上。此外，如果我们用这个注释来注释整个类，HotSopt JVM 将在所有字段](https://web.archive.org/web/20220625084255/https://github.com/openjdk/jdk/blob/319b4e71e1400f8a482f0ab42377d40056c6f0ac/src/hotspot/share/classfile/classFileParser.cpp#L4454)之前添加相同的填充符[。](https://web.archive.org/web/20220625084255/https://github.com/openjdk/jdk/blob/319b4e71e1400f8a482f0ab42377d40056c6f0ac/src/hotspot/share/classfile/classFileParser.cpp#L4236)

`@Contended `注释旨在由 JDK 自己在内部使用。**所以默认情况下，不影响非内部对象的内存布局**。这就是为什么我们的复制粘贴加法器的性能不如内置加法器的原因。

为了[消除这种仅限内部的限制](https://web.archive.org/web/20220625084255/https://github.com/openjdk/jdk/blob/985061ac28af56eb4593c6cd7d69d6556b5608f9/src/hotspot/share/classfile/classFileParser.cpp#L2118)，我们可以在重新运行基准测试时使用`[-XX:-RestrictContended](https://web.archive.org/web/20220625084255/https://github.com/openjdk/jdk/blob/195c45a0e11207e15c277e7671b2a82b8077c5fb/src/hotspot/share/runtime/globals.hpp#L777) `调优标志:

```java
Benchmark              Mode  Cnt          Score          Error  Units
FalseSharing.builtin  thrpt   40  541148225.959 ± 18336783.899  ops/s
FalseSharing.custom   thrpt   40  546022431.969 ± 16406252.364  ops/s
```

如上所示，现在基准测试的结果更接近了，差别可能只是一点点噪声。

### 7.1.填充尺寸

默认情况下，`@Contended `注释添加了 128 字节的填充。**这主要是因为许多现代处理器中的高速缓存线大小约为 64/128 字节**。

然而，该值可通过`[-XX:ContendedPaddingWidth](https://web.archive.org/web/20220625084255/https://github.com/openjdk/jdk/blob/7436ef236e4826f93df1af53c4aa73429afde41f/src/hotspot/share/runtime/globals.hpp#L769) `调谐标志进行配置。截至本文撰写之时，该标志只接受 0 到 8192 之间的值。

### 7.2.禁用`@Contended`

也可以通过`[-XX:-EnableContended](https://web.archive.org/web/20220625084255/https://github.com/openjdk/jdk/blob/7436ef236e4826f93df1af53c4aa73429afde41f/src/hotspot/share/runtime/globals.hpp#L774) `调谐禁用`@Contended `效果。当内存非常珍贵，我们可以承受损失一点(有时是很多)性能时，这可能是有用的。

### 7.3.用例

在第一次发布之后，`@Contended `注释被广泛用于防止 JDK 内部数据结构中的错误共享。以下是这种实现的几个值得注意的例子:

*   实现高吞吐量的[计数器和累加器](/web/20220625084255/https://www.baeldung.com/java-longadder-and-longaccumulator#dynamic-striping)的`[Striped64](https://web.archive.org/web/20220625084255/https://github.com/openjdk/jdk/blob/195c45a0e11207e15c277e7671b2a82b8077c5fb/src/java.base/share/classes/java/util/concurrent/atomic/Striped64.java#L124) `类
*   `[Thread](https://web.archive.org/web/20220625084255/https://github.com/openjdk/jdk/blob/b0e1ee4b3b345b729d14b897d503777ff779d573/src/java.base/share/classes/java/lang/Thread.java#L2059) `类便于实现[高效随机数生成器](/web/20220625084255/https://www.baeldung.com/java-thread-local-random#implementation-details)
*   `[ForkJoinPool](https://web.archive.org/web/20220625084255/https://github.com/openjdk/jdk/blob/1e8806fd08aef29029878a1c80d6ed39fdbfe182/src/java.base/share/classes/java/util/concurrent/ForkJoinPool.java#L774) `偷工作的队列
*   `[ConcurrentHashMap](https://web.archive.org/web/20220625084255/https://github.com/openjdk/jdk/blob/f29d1d172b82a3481f665999669daed74455ae55/src/java.base/share/classes/java/util/concurrent/ConcurrentHashMap.java#L2565) `实现
*   在`[Exchanger](https://web.archive.org/web/20220625084255/https://github.com/openjdk/jdk/blob/4d1445f42ee5fd98609cb9977a648bf58ec2c6c7/src/java.base/share/classes/java/util/concurrent/Exchanger.java#L305) `类中使用的[双重数据结构](https://web.archive.org/web/20220625084255/http://www.cs.rochester.edu/research/synchronization/pseudocode/duals.html)

## 8.结论

在本文中，我们看到了有时假共享可能会对多线程应用程序的性能产生反作用。

更具体地说，我们确实将 Java 中的`LongAdder `实现与其副本进行了基准测试，并使用其结果作为我们性能调查的起点。

此外，我们使用了`perf `工具来收集一些关于 Linux 上正在运行的应用程序的性能指标的统计数据。要查看更多关于`perf, `的例子，强烈推荐阅读[布兰登·格雷戈](https://web.archive.org/web/20220625084255/http://www.brendangregg.com/perf.html)的博客。此外，从 [Linux 内核版本 4.4](https://web.archive.org/web/20220625084255/https://github.com/torvalds/linux/tree/master/tools/bpf) 开始可用的 [eBPF](https://web.archive.org/web/20220625084255/http://www.brendangregg.com/perf.html#eBPF) ，也可以在许多跟踪和分析场景中使用。

像往常一样，所有的例子都可以在 GitHub 上找到[。](https://web.archive.org/web/20220625084255/https://github.com/eugenp/tutorials/tree/master/jmh)