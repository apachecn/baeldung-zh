# JVM 垃圾收集器

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/jvm-garbage-collectors>

## 1。概述

在这个快速教程中，我们将演示不同的`JVM Garbage Collection (GC)`实现的基础。然后我们将学习如何在我们的应用程序中启用特定类型的垃圾收集。

## 2。垃圾收集简介

顾名思义，`Garbage Collection`似乎会处理从内存中查找和删除垃圾的工作。然而，实际上，`Garbage Collection`会跟踪 JVM 堆空间中所有可用的对象，并删除未使用的对象。

基本上，`GC`的工作分为两个简单的步骤，即标记和扫描:

*   **Mark–**这是垃圾收集器识别哪些内存正在使用，哪些没有使用的地方。
*   **Sweep–**该步骤删除在“标记”阶段识别的对象。

**优点:**

*   没有手动内存分配/取消分配处理，因为未使用的内存空间由`GC`自动处理
*   无处理开销`[Dangling Pointer](https://web.archive.org/web/20220905193148/https://en.wikipedia.org/wiki/Dangling_pointer)`
*   自动`[Memory Leak](https://web.archive.org/web/20220905193148/https://en.wikipedia.org/wiki/Memory_leak)`管理(`GC`本身不能保证内存泄漏的完全证明解决方案；但是，它处理了很大一部分)

**缺点:**

*   由于`JVM`必须跟踪对象引用的创建/删除，这个活动比原始应用程序需要更多的 CPU 能力。它可能会影响需要大内存的请求的性能。
*   程序员无法控制专门用于释放不再需要的对象的 CPU 时间的调度。
*   使用一些 GC 实现可能会导致应用程序意外停止。
*   自动化的内存管理不如适当的手动内存分配/释放有效。

## 3。GC 实现

JVM 有五种类型的`GC`实现:

*   串行垃圾收集器
*   并行垃圾收集器
*   CMS 垃圾收集器
*   G1 垃圾收集器
*   垃圾收集器

### 3.1。串行垃圾收集器

这是最简单的 GC 实现，因为它基本上是用单线程工作的。因此，**这个`GC`实现在运行**时会冻结所有应用程序线程。因此，在多线程应用程序中使用它并不是一个好主意，比如服务器环境。

然而，[的工程师在 QCon 2012 上做了一个关于`Serial Garbage Collector,`性能的精彩演讲](https://web.archive.org/web/20220905193148/https://www.infoq.com/presentations/JVM-Performance-Tuning-twitter-QCon-London-2012)，这是更好地了解这款采集器的好方法。

对于大多数对暂停时间要求不高并运行在客户端风格的机器上的应用程序来说，串行 GC 是垃圾收集器的首选。要启用`Serial Garbage Collector`，我们可以使用以下参数:

```java
java -XX:+UseSerialGC -jar Application.java
```

### 3.2。并行垃圾收集器

这是`JVM,` 的默认`GC`，有时也称为吞吐量收集器。与`Serial Garbage Collector`不同的是，**使用多线程来管理堆空间**，但是它在执行`GC`的同时也会冻结其他应用程序线程。

如果我们使用这个`GC`，我们可以指定最大垃圾收集`threads and pause time, throughput, and footprint`(堆大小)。

垃圾收集器线程的数量可以通过命令行选项`-XX:ParallelGCThreads=<N>`来控制。

最大暂停时间目标(两个`GC`之间的间隔[毫秒])由命令行选项`-XX:MaxGCPauseMillis=<N>`指定。

花费在垃圾收集上的时间与花费在垃圾收集之外的时间之比称为最大吞吐量目标，可以通过命令行选项`-XX:GCTimeRatio=<N>.`来指定

使用选项`-Xmx<N>.`指定最大堆内存占用量(程序运行时所需的堆内存量)

要启用`Parallel Garbage Collector`，我们可以使用以下参数:

```java
java -XX:+UseParallelGC -jar Application.java
```

### 3.3。CMS 垃圾收集器

**`Concurrent Mark Sweep (CMS)`实现使用多个垃圾收集器线程进行垃圾收集。**它是为那些喜欢更短的垃圾收集暂停的应用程序设计的，并且能够在应用程序运行时与垃圾收集器共享处理器资源。

简单地说，使用这种类型的 GC 的应用程序平均响应较慢，但不会停止响应来执行垃圾收集。

这里需要注意的一点是，因为这个`GC` 是并发的，所以调用显式垃圾收集，比如在并发进程工作时使用`System.gc()`，将导致`Concurrent Mode Failure / Interruption`。

如果总时间的 98%以上花在了`CMS`垃圾收集上，而只有不到 2%的堆被回收，那么`CMS` `collector`就会抛出一个`OutOfMemoryError`。如果有必要，我们可以通过在命令行中添加选项`-XX:-UseGCOverheadLimit`来禁用这个特性。

该收集器还有一种称为增量模式的模式，这种模式在 Java SE 8 中已被弃用，在未来的主要版本中可能会被删除。

要启用`CMS Garbage Collector`，我们可以使用以下标志:

```java
java -XX:+UseParNewGC -jar Application.java
```

**[从 Java 9](https://web.archive.org/web/20220905193148/https://openjdk.java.net/jeps/291) 开始，CMS 垃圾收集器已经被弃用**。因此，如果我们试图使用它，JVM 会打印一条警告消息:

```java
>> java -XX:+UseConcMarkSweepGC --version
Java HotSpot(TM) 64-Bit Server VM warning: Option UseConcMarkSweepGC was deprecated 
in version 9.0 and will likely be removed in a future release.
java version "9.0.1"
```

而且， [Java 14](https://web.archive.org/web/20220905193148/https://openjdk.java.net/jeps/363) 完全放弃了 CMS 支持:

```java
>> java -XX:+UseConcMarkSweepGC --version
OpenJDK 64-Bit Server VM warning: Ignoring option UseConcMarkSweepGC; 
support was removed in 14.0
openjdk 14 2020-03-17
```

### 3.4。G1 垃圾收集器

`G1 (Garbage First) Garbage Collector`专为运行在具有大内存空间的多处理器机器上的应用程序而设计。它可以从`JDK7 Update 4`和以后的版本中获得。

`G1`收集器将取代`CMS`收集器，因为它的性能效率更高。

与其他收集器不同，`G1`收集器将堆划分为一组大小相等的堆区域，每个区域都是一个连续的虚拟内存范围。当执行垃圾收集时，`G1`显示一个并发的全局标记阶段(即阶段 1，称为`Marking)`,用于确定整个堆中对象的活性。

在标记阶段完成后，`G1`知道哪些区域大部分是空的。它首先在这些区域收集，这通常会产生大量的空闲空间(即阶段 2，称为`Sweeping).`,这就是为什么这种垃圾收集方法被称为垃圾优先。

为了启用`G1 Garbage Collector`，我们可以使用下面的参数:

```java
java -XX:+UseG1GC -jar Application.java
```

### 3.5。Java 8 的变化

`Java 8u20`引入了一个额外的`JVM`参数，通过创建相同`String.`的过多实例来减少不必要的内存使用。这通过将重复的`String`值移除到全局单个`char[]`数组来优化堆内存。

我们可以通过添加`**-XX:+UseStringDeduplication**`作为`JVM`参数来启用该参数。

### 3.6。z 垃圾收集器

[`ZGC (Z Garbage Collector)`](/web/20220905193148/https://www.baeldung.com/jvm-zgc-garbage-collector) 是一个可伸缩的低延迟垃圾收集器，在 Java 11 中首次亮相，作为 Linux 的实验选项。`JDK` 14 介绍了 Windows 和 macOS 操作系统下的`ZGC`。`ZGC`已经获得 Java 15 以后的生产状态。

`ZGC`并发执行所有昂贵的工作，**不停止执行应用程序线程超过 10 毫秒**，这使其适合要求低延迟的应用程序。当线程运行时，它使用带有彩色指针的**负载屏障来执行并发操作，它们用于跟踪堆的使用情况。**

引用着色(彩色指针)是`ZGC`的核心概念。意味着`ZGC`使用一些引用的位(元数据位)来标记对象的状态。它还**处理大小从 8MB 到 16TB 的堆**。此外，暂停时间不会随着堆、活动集或根集的大小而增加。

类似于`G1, Z Garbage Collector` 对堆进行分区，除了堆区域可以有不同的大小。

要启用`Z Garbage Collector`，我们可以在低于 15 的`JDK`版本中使用以下参数:

```java
java -XX:+UnlockExperimentalVMOptions -XX:+UseZGC Application.java
```

从版本 15 开始，我们不需要实验模式:

```java
java -XX:+UseZGC Application.java
```

我们应该注意到`ZGC`不是默认的垃圾收集器。

## 4。结论

在本文中，我们查看了不同的`JVM Garbage Collection`实现及其用例。

更详细的文档可以在这里找到[。](https://web.archive.org/web/20220905193148/http://www.oracle.com/technetwork/java/javase/gc-tuning-6-140523.html)