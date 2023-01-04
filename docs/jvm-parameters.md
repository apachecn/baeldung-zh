# 最重要的 JVM 参数指南

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/jvm-parameters>

## 1。概述

在这个快速教程中，我们将探索可用于配置 Java 虚拟机的最常见的选项。

## 2。显式堆内存–Xms 和 Xmx 选项

最常见的与性能相关的实践之一是根据应用程序需求初始化堆内存。

这就是为什么我们应该指定最小和最大堆大小。以下参数可用于实现这一目标:

```java
-Xms<heap size>[unit] 
-Xmx<heap size>[unit]
```

这里， **`unit`** 表示存储器(由 **`heap size`** 表示)将被初始化的单位。单位可以标记为 **`‘g'`** 为 GB，`**‘m'**`为 MB，`**‘k'**`为 KB。

例如，如果我们想给 JVM 分配最小 2 GB 和最大 5 GB，我们需要编写:

```java
-Xms2G -Xmx5G
```

从 Java 8 开始，`[Metaspace](https://web.archive.org/web/20221105162029/https://matthung0807.blogspot.com/2019/03/about-g1-garbage-collector-permanent.html)`的大小就没有定义了。一旦达到全局限制，JVM 会自动增加它，然而，为了克服任何不必要的不稳定性，我们可以用以下方式设置`Metaspace`的大小:

```java
-XX:MaxMetaspaceSize=<metaspace size>[unit]
```

这里，`**metaspace size**`表示我们想要分配给`Metaspace`的内存量。

根据 [Oracle 指南](https://web.archive.org/web/20221105162029/https://docs.oracle.com/en/java/javase/11/gctuning/factors-affecting-garbage-collection-performance.html#GUID-189AD425-F9A0-444A-AC89-C967E742B25C)，在总可用内存之后，第二大影响因素是为年轻一代保留的堆的比例。默认情况下，YG 的最小尺寸为 1310 `MB`，最大尺寸为`unlimited`。

我们可以明确地分配它们:

```java
-XX:NewSize=<young size>[unit] 
-XX:MaxNewSize=<young size>[unit]
```

## 3。垃圾收集

为了提高应用程序的稳定性，选择正确的[垃圾收集](https://web.archive.org/web/20221105162029/http://www.oracle.com/webfolder/technetwork/tutorials/obe/java/gc01/index.html)算法至关重要。

JVM 有四种类型的`GC`实现:

*   串行垃圾收集器
*   并行垃圾收集器
*   CMS 垃圾收集器
*   G1 垃圾收集器

这些实现可以用下面的参数来声明:

```java
-XX:+UseSerialGC
-XX:+UseParallelGC
-XX:+USeParNewGC
-XX:+UseG1GC
```

关于 `Garbage Collection`实现的更多细节可以在[这里](/web/20221105162029/https://www.baeldung.com/jvm-garbage-collectors)找到。

## 4。GC 日志记录

为了严格监控应用程序的健康状况，我们应该经常检查 JVM 的`Garbage Collection`性能。最简单的方法是以人类可读的格式记录`GC`活动。

使用以下参数，我们可以记录`GC`活动:

```java
-XX:+UseGCLogFileRotation 
-XX:NumberOfGCLogFiles=< number of log files > 
-XX:GCLogFileSize=< file size >[ unit ]
-Xloggc:/path/to/gc.log
```

**`UseGCLogFileRotation`** 指定了日志文件的滚动策略，很像 log4j、s4lj 等。 **`NumberOfGCLogFiles`** 表示单个应用程序生命周期可以写入的日志文件的最大数量。 **`GCLogFileSize`** 指定文件的最大大小。最后，`**loggc**`表示其位置。

这里要注意的是，还有两个可用的 JVM 参数(`**-XX:+PrintGCTimeStamps**`和`**-XX:+PrintGCDateStamps**`)，它们可以用来在`GC`日志中打印日期时间戳。

例如，如果我们想要分配最多 100 个`GC`日志文件，每个文件的最大大小为 50 MB，并且想要将它们存储在“`/home/user/log/'` 位置，我们可以使用下面的语法:

```java
-XX:+UseGCLogFileRotation  
-XX:NumberOfGCLogFiles=10
-XX:GCLogFileSize=50M 
-Xloggc:/home/user/log/gc.log
```

然而，问题是一个额外的守护线程总是用于在后台监控系统时间。这种行为可能会造成一些性能瓶颈；这就是为什么在生产中不玩这个参数总是比较好的原因。

## 5。处理内存不足

对于大型应用程序来说，面临内存不足错误是很常见的，这反过来会导致应用程序崩溃。这是一个非常关键的场景，很难复制来解决问题。

这就是 JVM 附带一些参数的原因，这些参数将堆内存转储到一个物理文件中，稍后可以使用该文件来查找泄漏:

```java
-XX:+HeapDumpOnOutOfMemoryError 
-XX:HeapDumpPath=./java_pid<pid>.hprof
-XX:OnOutOfMemoryError="< cmd args >;< cmd args >" 
-XX:+UseGCOverheadLimit
```

这里需要注意几点:

*   `**HeapDumpOnOutOfMemoryError**`指示 JVM 在`OutOfMemoryError`的情况下将堆转储到物理文件中
*   **`HeapDumpPath`** 表示要写入文件的路径；可以给出任何文件名；但是，如果 JVM 在文件名中发现一个 **< pid >** 标签，那么导致内存不足错误的当前进程的进程 id 将被附加到文件名的后面，格式为`.hprof`
*   `**OnOutOfMemoryError**`用于发布紧急命令，在出现内存不足错误时执行；应该在 cmd 参数空间中使用正确的命令。例如，如果我们想在出现内存不足时立即重启服务器，我们可以设置参数:

```java
-XX:OnOutOfMemoryError="shutdown -r"
```

*   **`UseGCOverheadLimit`** 是一种策略，在抛出`OutOfMemory`错误之前，限制虚拟机花费在 GC 中的时间比例

## 6。32/64 位

在同时安装了 32 位和 64 位软件包的 OS 环境中，JVM 会自动选择 32 位环境软件包。

如果我们想手动将环境设置为 64 位，我们可以使用以下参数:

```java
-d<OS bit>
```

OS 位可以是 **32** 或 **64** 。更多关于这个的信息可以在[这里](https://web.archive.org/web/20221105162029/http://www.oracle.com/technetwork/java/hotspotfaq-138619.html#64bit_layering)找到。

## 7。杂项

*   `**-server**`:启用“服务器热点虚拟机”；在 64 位 JVM 中，默认情况下使用该参数
*   `**-XX:+UseStringDeduplication**` : `Java 8u20`引入了这个 JVM 参数，通过创建同一个`String;`的过多实例来减少不必要的内存使用。这通过将重复的`String`值减少到单个全局 char[]数组来优化堆内存
*   `**-XX:+UseLWPSynchronization**`:设置基于`LWP` ( `Light Weight Process`)的同步策略，而不是基于线程的同步
*   **`-XX:LargePageSizeInBytes` :** 设置 Java 堆使用的大页面大小；它接受以 GB/MB/KB 为单位的参数；随着页面尺寸的增大，我们可以更好地利用虚拟内存硬件资源；然而，这可能会导致`PermGen`的空间变大，从而迫使 Java 堆空间变小
*   `**-XX:MaxHeapFreeRatio**`:设置`GC`后堆空闲的最大百分比，以避免收缩。
*   `**-XX:MinHeapFreeRatio**`:设置`GC`后堆空闲的最小百分比，以避免扩展；要监控堆的使用情况，您可以使用 JDK 附带的 [VisualVM](https://web.archive.org/web/20221105162029/https://visualvm.github.io/) 。
*   `**-XX:SurvivorRatio**`:`eden`/`survivor space size`的比值——例如`-XX:SurvivorRatio=6`将每个`survivor space`与`eden space`的比值设置为 1:6。
*   `**-XX:+UseLargePages**`:如果系统支持，使用大页面内存；请注意，如果使用这个 JVM 参数，那么`OpenJDK 7`往往会崩溃
*   **`-XX:+UseStringCache` :** 启用缓存`String`池中可用的常用分配字符串
*   `**-XX:+UseCompressedStrings**`:对于可以用纯 ASCII 格式表示的`String`对象，使用`byte[]`类型
*   **`-XX:+OptimizeStringConcat` :** 在可能的情况下优化`String`级联操作

## 8。结论

在这篇简短的文章中，我们了解了一些重要的 JVM 参数——这些参数可用于调优和提高一般应用程序的性能。

其中一些还可以用于调试目的。

如果你想更详细的探究参考参数，可以从[这里](https://web.archive.org/web/20221105162029/http://www.oracle.com/technetwork/articles/java/vmoptions-jsp-140102.html)开始。