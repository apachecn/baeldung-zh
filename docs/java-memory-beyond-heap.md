# Java 应用程序可以使用比堆大的内存吗？

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-memory-beyond-heap>

## 1.概观

我们可能都已经注意到，当谈到内存消耗时，我们的 Java 应用程序的内存使用并不遵循我们基于`-Xmx`(最大堆大小)选项的严格指令。事实上，JVM 拥有比堆更多的内存区域。

为了限制总的内存使用，需要注意一些额外的内存设置，所以让我们从 Java 应用程序的内存结构和内存分配的来源开始。

## 2.Java 进程的内存结构

Java 虚拟机(JVM) 的[内存分为两大类:堆和非堆。](https://web.archive.org/web/20221107202825/https://spring.io/blog/2019/03/11/memory-footprint-of-the-jvm)

堆内存是 JVM 内存中最广为人知的部分。它存储由应用程序创建的对象。JVM 在启动时启动堆。当应用程序创建一个对象的新实例时，该对象驻留在堆中，直到应用程序释放该实例。然后，垃圾收集器(GC)释放实例占用的内存。因此，堆大小根据负载而变化，尽管我们可以使用`-Xmx`选项配置最大 JVM 堆大小。

非堆内存构成了其余部分。它允许我们的应用程序使用比配置的堆大小更多的内存。JVM 的非堆内存分为几个不同的区域。JVM 代码和内部结构之类的数据、加载的 profiler 代理代码、常量池之类的每个类的结构、字段和方法的元数据、方法和构造函数的代码以及内部字符串都被归类为非堆内存。

值得一提的是，我们可以用`-XX`选项调优非堆内存的某些区域，比如`-XX:MaxMetaspaceSize`(相当于 Java 7 和更早版本中的`–XX:MaxPermSize`)。我们将在本教程中看到更多的标志。

除了 JVM 本身，Java 进程还会在其他领域消耗内存。例如，我们有堆外技术，通常使用直接的[`ByteBuffer`](/web/20221107202825/https://www.baeldung.com/java-bytebuffer)来处理大内存，并使其不受 GC 的控制。另一个来源是本地库使用的内存。

## 3.JVM 的非堆内存区域

让我们继续讨论 JVM 的非堆内存区域。

### 3.1\. Metaspace

元空间是存储类元数据的本地内存区域。当加载一个类时，JVM 将类的[元数据(它的运行时表示)分配到元空间中。每当从堆中移除类加载器及其所有类时，它们在元空间中的分配可以被认为是由 GC 释放的。](https://web.archive.org/web/20221107202825/https://docs.oracle.com/javase/specs/jvms/se8/html/jvms-4.html)

然而，释放的元空间不一定返回给操作系统。JVM 仍然可以保留全部或部分内存，以便在将来加载类时重用。

在早于 8 的 Java 版本中，元空间被称为永久生成(PermGen)。然而，与 Metaspace 不同，Metaspace 是一个堆外内存区域， [PermGen 驻留在一个特殊的堆区域](/web/20221107202825/https://www.baeldung.com/java-permgen-metaspace)。

### 3.2。代码缓存

[实时(JIT)编译器](/web/20221107202825/https://www.baeldung.com/graal-java-jit-compiler#jit-compiler)将其输出存储在[代码缓存](/web/20221107202825/https://www.baeldung.com/jvm-code-cache)区域。JIT 编译器将字节码编译成本机代码，用于频繁执行的部分，也就是热点。[Java 7 中引入的分层编译](/web/20221107202825/https://www.baeldung.com/jvm-tiered-compilation)，是客户端编译器(C1)使用指令编译代码，然后服务器编译器(C2)使用概要数据以优化的方式编译代码的方法。

分层编译的目标是混合 C1 和 C2 编译器，以获得快速的启动时间和良好的长期性能。**分层编译将需要缓存在内存中的代码量增加了四倍。**从 Java 8 开始，JIT 默认启用了这一功能，尽管我们仍然可以禁用分层编译。

### 3.3.线

[线程堆栈](/web/20221107202825/https://www.baeldung.com/java-stack-heap#stack-memory-in-java)包含每个已执行方法的所有局部变量，以及线程调用以到达当前执行点的方法。线程堆栈只能由创建它的线程访问。

理论上，由于线程堆栈内存是正在运行的线程数量的函数，并且线程数量没有限制，因此线程区域是无限的，可以占用很大一部分内存。实际上，操作系统限制线程的数量，JVM 根据平台对每个线程的堆栈内存大小有一个默认值。

### 3.4.碎片帐集

JVM 附带了一组 GC 算法，可以根据我们的应用程序的用例来选择。无论我们使用什么算法，都会给 GC 进程分配一定量的本地内存，但是使用的内存量会根据使用哪个[垃圾收集器](/web/20221107202825/https://www.baeldung.com/jvm-garbage-collectors)而变化。

### 3.5.标志

JVM 使用符号区来存储诸如字段名、方法签名和内部字符串之类的符号。在 Java 开发工具包(JDK)中，**符号存储在三个不同的表**中:

*   系统字典像 Java 类一样包含所有加载的类型信息。
*   常量池使用 符号表数据结构为类、方法、字段和可枚举类型保存加载的符号。JVM 维护着一个按类型划分的常量池，称为[运行时常量池](/web/20221107202825/https://www.baeldung.com/jvm-constant-pool)，其中包含几种常量，从编译时的数字文字到运行时方法，甚至是字段引用。
*   字符串表包含对所有常量字符串的引用，也称为内部字符串。

To understand the String Table, we need to know a bit more about the [String Pool.](/web/20221107202825/https://www.baeldung.com/java-string-pool) The String Pool is the JVM mechanism that optimizes the amount of memory allocated to a `String` by storing only one copy of each literal `String` in the pool, by a process called interning. The String Pool has two parts:

*   被拘留的字符串的内容作为常规的`String`对象存在于 Java 堆中。
*   哈希表，也就是所谓的字符串表，被分配到堆外，包含对内部字符串的引用。

换句话说，字符串池既有堆上部分，也有堆外部分。堆外部分是字符串表。虽然它通常比较小，但是当我们有更多的内部字符串时，它仍然会占用大量的额外内存。

### 3.6.竞技场

arena 是 JVM 自己实现的[基于 Arena 的内存管理](https://web.archive.org/web/20221107202825/https://en.wikipedia.org/wiki/Region-based_memory_management)，与 glibc 的 Arena 内存管理截然不同。它由 JVM 的一些子系统使用，如编译器和符号，或者当本机代码使用依赖于 JVM 领域的内部对象时。

### 3.7.其他的

不能归入本机内存区域的所有其他内存使用都属于这一部分。例如，`DirectByteBuffer`的用法在本部分中是间接可见的。

## 4.内存监控工具

既然我们已经发现 Java 内存使用不仅限于堆，我们将研究跟踪总内存使用的方法。发现可以在分析和内存监控工具的帮助下完成，然后，我们可以通过一些特定的调整来调整总的使用情况。

让我们快速浏览一下 JDK 附带的用于 JVM 内存监控的工具:

*   [`jmap`](https://web.archive.org/web/20221107202825/https://docs.oracle.com/javase/8/docs/technotes/guides/troubleshoot/tooldescr014.html) 是一个命令行实用程序，用于打印正在运行的虚拟机或核心文件的内存映射。我们也可以使用`jmap`来查询远程机器上的进程。但是，在 JDK8 中引入`jcmd`后，建议使用`jcmd`而不是`jmap`，以增强诊断并降低性能开销。
*   [`jcmd`](/web/20221107202825/https://www.baeldung.com/running-jvm-diagnose) 用于向 JVM 发送诊断命令请求，这些请求对于控制 Java [飞行记录](https://web.archive.org/web/20221107202825/https://docs.oracle.com/javacomponents/jmc-5-4/jfr-runtime-guide/about.htm#JFRUH170)、故障排除以及诊断 JVM 和 Java 应用程序非常有用。 **`jcmd`不支持远程进程**。我们将在本文中看到`jcmd`的一些具体用法。
*   [`jhat`](https://web.archive.org/web/20221107202825/https://docs.oracle.com/javase/7/docs/technotes/tools/share/jhat.html) 通过启动本地 web 服务器来可视化堆转储文件。有几种方法可以创建堆转储，比如使用`jmap -dump` 或 `jcmd GC.heap_dump filename.`
*   [`hprof`](https://web.archive.org/web/20221107202825/https://docs.oracle.com/javase/7/docs/technotes/samples/hprof.html) 能够显示 CPU 使用情况、堆分配统计信息，并监控争用配置文件。根据所请求的分析类型，`hprof`指示虚拟机收集相关的 JVM 工具接口(JVM TI)事件，并将事件数据处理成分析信息。

除了 JVM 自带的工具，还有特定于操作系统的命令来检查进程的内存。`[pmap](https://web.archive.org/web/20221107202825/http://man7.org/linux/man-pages/man1/pmap.1.html)`是一个适用于 Linux 发行版的工具，它提供了 Java 进程使用的内存的完整视图。

## 5.本机内存跟踪

[本地内存跟踪](https://web.archive.org/web/20221107202825/https://docs.oracle.com/javase/8/docs/technotes/guides/vm/nmt-8.html) (NMT)是一个 JVM 特性，我们可以用它来跟踪 VM 的内部内存使用情况。NMT 不会像[第三方本机代码内存分配](https://web.archive.org/web/20221107202825/https://docs.oracle.com/javase/8/docs/technotes/guides/vm/nmt-8.html)那样跟踪所有本机内存使用情况，但是，对于一大类典型应用程序来说，这已经足够了。

从 NMT 开始，我们必须为我们的应用程序启用它:

```java
java -XX:NativeMemoryTracking=summary -jar app.jar
```

`-XX:NativeMemoryTracking`的其他可用值是`off`和`detail`。请注意，启用 NMT [会产生间接成本，这会影响性能](https://web.archive.org/web/20221107202825/https://docs.oracle.com/javase/8/docs/technotes/guides/troubleshoot/tooldescr007.html)。此外，NMT 将两个机器字作为 malloc 头添加到所有 malloc 内存中。

然后我们可以使用不带参数的 `[jps](https://web.archive.org/web/20221107202825/https://docs.oracle.com/javase/7/docs/technotes/tools/share/jps.html)`或`jcmd`来查找我们的应用程序的进程 id (pid ):

```java
jcmd
<pid> <our.app.main.Class>
```

找到我们的应用程序 pid 后，我们可以继续使用 `jcmd`，它提供了一长串要监控的选项。让我们向`jcmd`寻求帮助，看看有哪些可用的选项:

```java
jcmd <pid> help
```

从输出中，我们看到`jcmd`支持不同的类别，如编译器、GC、 [JFR](https://web.archive.org/web/20221107202825/https://docs.oracle.com/javacomponents/jmc-5-4/jfr-runtime-guide/about.htm) 、JVMTI、ManagementAgent 和 VM。像`VM.metaspace`、`VM.native_memory`这样的选项可以帮助我们进行记忆追踪。让我们来探索其中的一些。

### 5.1.本机内存摘要报告

最顺手的是`VM.native_memory`。我们可以使用它来查看应用程序的虚拟机内部本机内存使用情况的摘要:

```java
jcmd <pid> VM.native_memory summary
```

```java
<pid>:

Native Memory Tracking:

Total: reserved=1779287KB, committed=503683KB
- Java Heap (reserved=307200KB, committed=307200KB)
  ...
- Class (reserved=1089000KB, committed=44824KB)
  ...
- Thread (reserved=41139KB, committed=41139KB)
  ...
- Code (reserved=248600KB, committed=17172KB)
  ...
- GC (reserved=62198KB, committed=62198KB)
  ...
- Compiler (reserved=175KB, committed=175KB)
  ...
- Internal (reserved=691KB, committed=691KB)
  ...
- Other (reserved=16KB, committed=16KB)
  ...
- Symbol (reserved=9704KB, committed=9704KB)
  ...
- Native Memory Tracking (reserved=4812KB, committed=4812KB)
  ...
- Shared class space (reserved=11136KB, committed=11136KB)
  ...
- Arena Chunk (reserved=176KB, committed=176KB)
  ... 
- Logging (reserved=4KB, committed=4KB)
  ... 
- Arguments (reserved=18KB, committed=18KB)
  ... 
- Module (reserved=175KB, committed=175KB)
  ... 
- Safepoint (reserved=8KB, committed=8KB)
  ... 
- Synchronization (reserved=4235KB, committed=4235KB)
  ... 
```

查看输出，我们可以看到 JVM 内存区域的概要，如 Java 堆、GC 和 thread。术语“保留”内存是指通过`malloc`或`mmap`预映射的总地址范围，因此它是该区域的最大可寻址内存。术语“已提交”意味着正在使用的内存。

[在这里](/web/20221107202825/https://www.baeldung.com/native-memory-tracking-in-jvm#nmt)，我们可以找到输出的详细解释。要查看内存使用的变化，我们可以依次使用`VM.native_memory baseline`和`VM.native_memory``summary.diff` `.`

### 5.2.元空间和字符串表的报告

我们可以尝试使用`jcmd`的其他 VM 选项来了解本机内存的一些特定区域，比如元空间、符号和内部字符串。

让我们试试 Metaspace:

```java
jcmd <pid> VM.metaspace
```

我们的输出看起来像:

```java
<pid>:
Total Usage - 1072 loaders, 9474 classes (1176 shared):
...
Virtual space:
  Non-class space:       38.00 MB reserved,      36.67 MB ( 97%) committed 
      Class space:        1.00 GB reserved,       5.62 MB ( <1%) committed 
             Both:        1.04 GB reserved,      42.30 MB (  4%) committed 
Chunk freelists:
   Non-Class: ...
       Class: ...
Waste (percentages refer to total committed size 42.30 MB):
              Committed unused:    192.00 KB ( <1%)
        Waste in chunks in use:      2.98 KB ( <1%)
         Free in chunks in use:      1.05 MB (  2%)
     Overhead in chunks in use:    232.12 KB ( <1%)
                In free chunks:     77.00 KB ( <1%)
Deallocated from chunks in use:    191.62 KB ( <1%) (890 blocks)
                       -total-:      1.73 MB (  4%)
MaxMetaspaceSize: unlimited
CompressedClassSpaceSize: 1.00 GB
InitialBootClassLoaderMetaspaceSize: 4.00 MB
```

现在，让我们看看应用程序的字符串表:

```java
jcmd <pid> VM.stringtable 
```

让我们看看输出:

```java
<pid>:
StringTable statistics:
Number of buckets : 65536 = 524288 bytes, each 8
Number of entries : 20046 = 320736 bytes, each 16
Number of literals : 20046 = 1507448 bytes, avg 75.000
Total footprint : = 2352472 bytes
Average bucket size : 0.306
Variance of bucket size : 0.307
Std. dev. of bucket size: 0.554
Maximum bucket size : 4
```

## 6.JVM 内存调优

我们知道，Java 应用程序使用的总内存是 JVM 或第三方库分配的堆内存和一堆非堆内存的总和。

非堆内存在负载下不太可能改变大小。通常，一旦加载了所有正在使用的类并且 JIT 完全预热，我们的应用程序就有稳定的非堆内存使用。然而，我们可以使用一些标志来指示 JVM 如何管理某些区域的内存使用。

**`jcmd`提供了一个`VM.flag`选项来查看我们的 Java 进程已经有了哪些标志，包括默认值，因此我们可以使用它作为工具来检查默认配置，并了解 JVM 是如何配置的:**

```java
jcmd <pid> VM.flags
```

在这里，我们看到使用的标志及其值:

```java
<pid>:
-XX:CICompilerCount=4 
-XX:ConcGCThreads=2 
-XX:G1ConcRefinementThreads=8 
-XX:G1HeapRegionSize=1048576 
-XX:InitialHeapSize=314572800 
...
```

让我们来看看一些用于不同区域内存调优的 [VM 标志](/web/20221107202825/https://www.baeldung.com/jvm-parameters)。

### 6.1。堆

我们有一些用于调优 JVM 堆的标志。为了配置最小和最大堆大小，我们有`-Xms` ( `-XX:InitialHeapSize`)和`-Xmx` ( `-XX:MaxHeapSize`)。如果我们喜欢将堆大小设置为物理内存的百分比，我们可以使用 [`-XX:MinRAMPercentage`和`-XX:MaxRAMPercentage`](/web/20221107202825/https://www.baeldung.com/java-jvm-parameters-rampercentage) 。**重要的是要知道，当我们分别使用`-Xms`和`-Xmx` 选项时，JVM 会忽略这两个选项。**

另一个可能影响内存分配模式的选项是`XX:+AlwaysPreTouch.` 默认情况下，JVM 最大堆分配在虚拟内存中，而不是物理内存中。只要没有写操作，操作系统就可能决定不分配内存。为了避免这一点(特别是对于巨大的`DirectByteBuffers`，重新分配可能需要一些时间来重新安排 OS 内存页面)，我们可以启用-XX:+AlwaysPreTouch。预触摸会在所有页面上写入“0 ”,并强制操作系统分配内存，而不仅仅是保留内存。**预触导致 JVM 启动延迟，因为它在单线程中工作。**

### 6.2。线程堆栈

线程堆栈是每个执行的方法的所有局部变量的每个线程的存储。我们使用 **[-Xss 或者`XX:ThreadStackSize`](/web/20221107202825/https://www.baeldung.com/jvm-configure-stack-sizes)** 选项来配置每个线程的堆栈大小。默认的线程堆栈大小取决于平台，但是在大多数现代 64 位操作系统中，它最大为 1 MB。

### 6.3.碎片帐集

我们可以用这些标志之一来设置应用程序的 GC 算法:`-XX:+UseSerialGC`、`-XX:+UseParallelGC`、`-XX:+UseParallelOldGC`、`-XX:+UseConcMarkSweepGC`或`-XX:+UseG1GC`。

如果我们选择 G1 作为 GC，我们可以选择通过`-XX:+UseStringDeduplication`启用[字符串重复数据删除](https://web.archive.org/web/20221107202825/https://openjdk.org/jeps/192)。它可以节省大量内存。字符串重复数据删除仅适用于长期实例。为了避免这一点，我们可以用`-XX:StringDeduplicationAgeThreshold.` 配置实例的有效年龄,`-XX:StringDeduplicationAgeThreshold` 的值表示 GC 周期存活的数量。

### 6.4。代码缓存

从 Java 9 开始，JVM 将代码缓存分成三个区域。因此，JVM 提供了特定的选项来调整它们:

*   `-XX:NonNMethodCodeHeapSize` 配置非方法段，是 JVM 内部相关代码。默认情况下，大约为 5 MB。
*   `-XX:ProfiledCodeHeapSize` 配置分析代码段，它是 C1 编译的代码，具有潜在的短生命周期。默认大小约为 122 MB。
*   `-XX:NonProfiledCodeHeapSize` 设置非概要段的大小，这是 C2 编译的代码，具有潜在的长生命周期。默认大小约为 122 MB。

### 6.5。分配器

JVM 从[保留内存](https://web.archive.org/web/20221107202825/https://github.com/corretto/corretto-11/blob/3b31d243a19774bebde63df21cc84e994a89439a/src/src/hotspot/os/linux/os_linux.cpp#L3421-L3444)开始，然后通过[使用 glibc 的 malloc 和 mmap 修改内存映射](https://web.archive.org/web/20221107202825/https://github.com/corretto/corretto-11/blob/3b31d243a19774bebde63df21cc84e994a89439a/src/src/hotspot/os/linux/os_linux.cpp#L3517-L3531)，部分“保留”将变得可用。保留和释放内存块的行为会导致碎片。已分配内存中的碎片会导致内存中大量未使用的区域。

除了 malloc，我们还可以使用其他的分配器，比如 [jemalloc](https://web.archive.org/web/20221107202825/http://jemalloc.net/) 或者 [tcmalloc](https://web.archive.org/web/20221107202825/https://google.github.io/tcmalloc/overview.html) 。jemalloc 是一个通用的 malloc 实现，强调避免碎片和可伸缩的并发支持，因此它通常比普通 glibc 的 malloc 更聪明。此外，jemalloc 还可以用于[泄漏检查](https://web.archive.org/web/20221107202825/https://github.com/jemalloc/jemalloc/wiki/Use-Case:-Leak-Checking)和堆分析。

### 6.6\. Metaspace

像堆一样，我们也有配置元空间大小的选项。为了配置元空间的下限和上限**、**，我们可以分别使用`-XX:MetaspaceSize`和`-XX:`、`MaxMetaspaceSize`。

`-XX:InitialBootClassLoaderMetaspaceSize is also useful to` 配置初始引导类加载程序的大小。

`There are -XX:MinMetaspaceFreeRatio` 和 `-XX:MaxMetaspaceFreeRatio options` 配置 GC 后可用的类元数据容量的最小和最大百分比。

我们还能够配置元空间扩展的最大大小，而无需使用`-XX:MaxMetaspaceExpansion`进行完全垃圾收集。

### 6.7.其他非堆内存区域

还有一些标志用于调优本机内存其他区域的使用。

我们可以使用`-XX:StringTableSize`来指定字符串池的映射大小，其中映射大小表示不同的内部字符串的最大数量。对于 JDK7+，默认的映射大小是`600013`，这意味着默认情况下池中有 600，013 个不同的字符串。

为了控制`DirectByteBuffers`的内存使用，我们可以使用 [`-XX:MaxDirectMemorySize.`](https://web.archive.org/web/20221107202825/https://www.eclipse.org/openj9/docs/xxmaxdirectmemorysize/) 通过这个选项，我们限制了可以保留给所有`DirectByteBuffers`的内存量。

对于需要加载更多类的应用程序，我们可以使用`-XX:PredictedLoadedClassCount.` 这个选项从 JDK8 开始就可用，它允许我们设置系统字典的存储桶大小。

## 7.结论

在本文中，我们探索了 Java 进程的不同内存区域和一些监控内存使用的工具。我们已经看到 Java 内存使用不仅仅局限于堆，所以我们使用`jcmd`来检查和跟踪 JVM 的内存使用。最后，我们回顾了一些 JVM 标志，它们可以帮助我们调优 Java 应用程序的内存使用。