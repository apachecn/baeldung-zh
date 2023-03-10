# Java 中的详细垃圾收集

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-verbose-gc>

## 1。概述

在本教程中，**我们将看看如何在 Java 应用程序**中打开详细的垃圾收集。我们将从介绍什么是详细垃圾收集以及它为什么有用开始。

接下来，我们将看几个不同的示例，并了解可用的不同配置选项。此外，**我们还将关注如何解释详细日志的输出。**

要了解更多关于垃圾收集(GC)和可用的不同实现，请查看我们关于 Java 垃圾收集器的文章。

## 2。详细垃圾收集简介

**在调优和调试许多问题**时，尤其是内存问题时，经常需要打开详细的垃圾收集日志记录。事实上，有些人会认为，为了严格监控我们的应用程序健康状况，我们应该始终监控 JVM 的垃圾收集性能。

正如我们将看到的，GC 日志是揭示应用程序的堆和 GC 配置的潜在改进的非常重要的工具。**对于每次发生的垃圾收集，垃圾收集日志提供了关于其结果和持续时间的准确数据。**

随着时间的推移，对这些信息的分析可以帮助我们更好地理解应用程序的行为，并帮助我们调整应用程序的性能。此外，通过指定最佳堆大小、其他 JVM 选项和替代的 GC 算法，它可以帮助优化 GC 频率和收集时间。

### 2.1.一个简单的 Java 程序

我们将使用一个简单的 Java 程序来演示如何启用和解释我们的 GC 日志:

```java
public class Application {

    private static Map<String, String> stringContainer = new HashMap<>();

    public static void main(String[] args) {
        System.out.println("Start of program!");
        String stringWithPrefix = "stringWithPrefix";

        // Load Java Heap with 3 M java.lang.String instances
        for (int i = 0; i < 3000000; i++) {
            String newString = stringWithPrefix + i;
            stringContainer.put(newString, newString);
        }
        System.out.println("MAP size: " + stringContainer.size());

        // Explicit GC!
        System.gc();

        // Remove 2 M out of 3 M
        for (int i = 0; i < 2000000; i++) {
            String newString = stringWithPrefix + i;
            stringContainer.remove(newString);
        }

        System.out.println("MAP size: " + stringContainer.size());
        System.out.println("End of program!");
    }
}
```

正如我们在上面的例子中看到的，这个简单的程序将 300 万个*字符串*实例加载到一个*地图*对象中。然后，我们使用 *System.gc()* 显式调用垃圾收集器。

最后，我们从*图*中移除 200 万个*字符串*实例。我们还显式地使用了 *System.out.println* 来简化对输出的解释。

在下一节中，我们将看到如何激活 GC 日志记录。

## 3。激活“简单”GC 日志记录

让我们首先运行我们的程序，并通过 JVM 启动参数启用详细 GC:

```java
-XX:+UseSerialGC -Xms1024m -Xmx1024m -verbose:gc
```

**这里重要的参数是 *-verbose:gc* ，它以最简单的形式激活垃圾收集信息的日志**。默认情况下，GC 日志被写入`stdout`，并且应该为每个年轻的 GC 和每个完整的 GC 输出一行。

出于示例的目的，我们通过参数 *-XX:+UseSerialGC* 指定了串行垃圾收集器，这是最简单的 GC 实现。

我们还设置了 1024mb 的最小和最大堆大小，但是当然，我们还可以调优更多的 JVM 参数。

### 3.1.对详细输出的基本理解

现在让我们来看看这个简单程序的输出:

```java
Start of program!
[GC (Allocation Failure)  279616K->146232K(1013632K), 0.3318607 secs]
[GC (Allocation Failure)  425848K->295442K(1013632K), 0.4266943 secs]
MAP size: 3000000
[Full GC (System.gc())  434341K->368279K(1013632K), 0.5420611 secs]
[GC (Allocation Failure)  647895K->368280K(1013632K), 0.0075449 secs]
MAP size: 1000000
End of program!
```

在上面的输出中，我们已经可以看到很多关于 JVM 内部情况的有用信息。

起初，这个输出看起来非常令人生畏，但是现在让我们一步一步地来看它。

首先，**我们可以看到发生了四次收集，一次全 GC，三次清洗年轻代。**

### 3.2.更详细的详细输出

让我们更详细地分解输出行，以准确理解发生了什么:

1.  `GC`或`Full GC`–**垃圾收集的类型，用`GC`或`Full GC`来区分少量垃圾收集或完全垃圾收集**
2.  `(Allocation Failure)`或`(System.gc())`——收集的原因—**分配失败表明在 Eden 中没有剩余的空间来分配我们的对象**
3.  `279616K->146232K`–分别在 GC 之前和之后占用的堆内存(用箭头分隔)
4.  `(1013632K)`–堆的当前容量
5.  `0.3318607 secs`–GC 事件的持续时间，以秒为单位

因此，如果我们取第一行，`279616K->146232K(1013632K)`意味着 GC 将占用的堆内存从`279616K`减少到了`146232K`。GC 时的堆容量是`1013632K`，GC 用了`0.3318607`秒。

然而，尽管简单的 GC 日志记录格式可能有用，但它提供的细节有限。**例如，我们无法判断 GC 是否将任何对象从年轻代转移到了老代，或者年轻代在每次收集前后的总大小是多少**。

因此，详细的 GC 日志记录比简单的更有用。

## 4。激活“详细”GC 记录

**为了激活详细的 GC 日志记录，我们使用参数`-XX:+PrintGCDetails`。**这将为我们提供每个 GC 的更多详细信息，例如:

*   每次 GC 前后年轻一代和老一代的规模
*   在年轻一代和老一代中发生 GC 所需要的时间
*   每次垃圾收集时提升的对象的大小
*   总堆大小的摘要

在下一个例子中，我们将看到如何将`-verbose:gc`与这个额外的参数结合起来，在我们的日志中捕获更详细的信息。

请注意，`-XX:+PrintGCDetails`标志在 Java 9 中已被弃用，取而代之的是新的统一日志记录机制(稍后将详细介绍)。无论如何，**是`-XX:+PrintGCDetails`的新等价物，是`-Xlog:gc*`期权。**

## 5.解释“详细”的详细输出

让我们再次运行我们的示例程序:

```java
-XX:+UseSerialGC -Xms1024m -Xmx1024m -verbose:gc -XX:+PrintGCDetails
```

这一次输出更加详细:

```java
Start of program!
[GC (Allocation Failure) [DefNew: 279616K->34944K(314560K), 0.3626923 secs] 279616K->146232K(1013632K), 0.3627492 secs] [Times: user=0.33 sys=0.03, real=0.36 secs] 
[GC (Allocation Failure) [DefNew: 314560K->34943K(314560K), 0.4589079 secs] 425848K->295442K(1013632K), 0.4589526 secs] [Times: user=0.41 sys=0.05, real=0.46 secs] 
MAP size: 3000000
[Full GC (System.gc()) [Tenured: 260498K->368281K(699072K), 0.5580183 secs] 434341K->368281K(1013632K), [Metaspace: 2624K->2624K(1056768K)], 0.5580738 secs] [Times: user=0.50 sys=0.06, real=0.56 secs] 
[GC (Allocation Failure) [DefNew: 279616K->0K(314560K), 0.0076722 secs] 647897K->368281K(1013632K), 0.0077169 secs] [Times: user=0.01 sys=0.00, real=0.01 secs] 
MAP size: 1000000
End of program!
Heap
 def new generation   total 314560K, used 100261K [0x00000000c0000000, 0x00000000d5550000, 0x00000000d5550000)
  eden space 279616K,  35% used [0x00000000c0000000, 0x00000000c61e9370, 0x00000000d1110000)
  from space 34944K,   0% used [0x00000000d3330000, 0x00000000d3330188, 0x00000000d5550000)
  to   space 34944K,   0% used [0x00000000d1110000, 0x00000000d1110000, 0x00000000d3330000)
 tenured generation   total 699072K, used 368281K [0x00000000d5550000, 0x0000000100000000, 0x0000000100000000)
   the space 699072K,  52% used [0x00000000d5550000, 0x00000000ebcf65e0, 0x00000000ebcf6600, 0x0000000100000000)
 Metaspace       used 2637K, capacity 4486K, committed 4864K, reserved 1056768K
  class space    used 283K, capacity 386K, committed 512K, reserved 1048576K
```

我们应该能够识别简单 GC 日志中的所有元素。但是有几个新项目。

现在让我们考虑输出中的新项目，它们在下一部分中以蓝色突出显示:

### 5.1.解读《年轻一代》中的一个小 GC

我们将从分析一个小 GC 中的新部件开始:

*   【GC(分配失败)【def new:279616k->34944k(314560k)，0.3626923 秒】279616k->146232k(1013632k)，0.3627492 秒】【Times:user = 0.33 sys = 0.03，real=0.36 秒】【T5

像以前一样，我们将把这些行分成几部分:

1.  `DefNew`–使用的垃圾收集器的名称。**这个不太明显的名字代表单线程标记复制停止世界垃圾收集器，它被用来清理年轻一代**
2.  `279616K->34944K`–收集前后年轻一代的使用情况
3.  年轻一代的总人数
4.  0.3626923 秒–以秒为单位的持续时间
5.  `[Times: user=0.33 sys=0.03, real=0.36 secs`]–GC 事件的持续时间，在不同类别中测量

现在让我们来解释不同的类别:

*   `user`–垃圾收集器消耗的总 CPU 时间
*   `sys`–花费在操作系统调用或等待系统事件上的时间
*   `real`–这是所有经过的时间，包括其他进程使用的时间片

由于我们正在使用串行垃圾收集器运行我们的示例，它总是只使用一个线程，所以实时性等于用户和系统时间的总和。

### 5.2.解释完整的 GC

在这个倒数第二个例子中，我们看到对于一个由系统调用触发的主要收集(Full GC)，使用的收集器是`Tenured`。

我们看到的最后一条附加信息是按照与`Metaspace`相同的模式细分的:

```java
[Metaspace: 2624K->2624K(1056768K)], 0.5580738 secs]
```

**`Metaspace`是 Java 8 中新引入的[内存空间](/web/20221129001719/https://www.baeldung.com/java-permgen-metaspace)，是原生内存的一个区域。**

### 5.3.Java 堆分解分析

**输出的最后一部分包括堆的分解，包括每个内存部分的内存占用摘要**。

我们可以看到，Eden space 占 35 %, Tenured 占 52%。还包括元数据空间和类空间的总结。

从上面的例子中，**我们现在可以准确地理解在 GC 事件期间 JVM 内部的内存消耗发生了什么。**

## 6。添加日期和时间信息

没有日期和时间信息的好日志是不完整的。

当我们需要将 GC 日志数据与其他来源的数据相关联时，这些额外的信息可能非常有用，或者它可以简单地帮助搜索。

当我们运行应用程序来获取出现在日志中的日期和时间信息时，我们可以添加以下两个参数:

```java
-XX:+PrintGCTimeStamps -XX:+PrintGCDateStamps
```

现在，每一行都以写入时的绝对日期和时间开始，后跟一个时间戳，反映自 JVM 启动以来经过的实际时间(秒):

```java
2018-12-11T02:55:23.518+0100: 2.601: [GC (Allocation ...
```

请注意，Java 9 中已经删除了这些调优标志。新的选择是:

```java
-Xlog:gc*::time
```

## 7 .**。记录到文件**

正如我们已经看到的，默认情况下，GC 日志被写到`stdout`。更实际的解决方案是指定一个输出文件。

**我们可以通过使用参数`-Xloggc:<file>`来做到这一点，其中`file`是输出文件**的绝对路径

```java
-Xloggc:/path/to/file/gc.log
```

与其他调优标志类似，Java 9 弃用了-Xloggc 标志，转而支持新的统一日志记录。更具体地说，现在记录到文件的替代方法是:

```java
-Xlog:gc:/path/to/file/gc.log
```

## 8。Java 9:统一 JVM 日志

从 Java 9 开始，大多数与 GC 相关的调优标志都被弃用，取而代之的是统一日志选项`-Xlog:gc`。然而，`–` `verbose:gc`选项在 Java 9 和更新版本中仍然有效。

例如，从 Java 9 开始，新的统一日志系统中的`-verbose:gc`标志的等价形式是:

```java
-Xlog:gc
```

这将把所有信息级别的 GC 日志记录到标准输出中。也可以使用`-Xlog:gc=<level>`语法来改变日志级别。例如，要查看所有调试级别日志:

```java
-Xlog:gc=debug
```

正如我们前面看到的，我们可以通过`-Xlog:gc=<level>:<output>`语法改变输出目的地。默认情况下，`output`是`stdout`，但是我们可以把它改成`stderr`甚至是一个文件:

```java
-Xlog:gc=debug:file=gc.txt
```

此外，还可以使用 decorators 向输出添加一些字段。例如:

```java
-Xlog:gc=debug::pid,time,uptime
```

这里，我们在每个日志语句中打印进程 id、正常运行时间和当前时间戳。

要查看统一 JVM 日志的更多示例，请参见 [JEP 158 标准](https://web.archive.org/web/20221129001719/https://openjdk.java.net/jeps/158)。

## 9。一个用于分析 GC 日志的工具

使用文本编辑器分析 GC 日志可能会非常耗时且乏味。根据 JVM 版本和使用的 GC 算法，GC 日志格式可能会有所不同。

有一个非常好的免费图形分析工具，它可以分析垃圾收集日志，提供许多关于潜在垃圾收集问题的指标，甚至提供这些问题的潜在解决方案。

绝对要看看[通用 GC 日志分析器](https://web.archive.org/web/20221129001719/http://gceasy.io/)！

## 10。结论

总而言之，在本教程中，我们已经详细探讨了 Java 中的详细垃圾收集。

首先，我们从介绍什么是详细垃圾收集以及为什么我们可能想要使用它开始。然后我们看了几个使用简单 Java 应用程序的例子。我们首先以最简单的形式启用 GC 日志记录，然后探究几个更详细的示例以及如何解释输出。

最后，我们探讨了记录时间和日期信息的几个额外选项，以及如何将信息写入日志文件。

代码示例可以在 GitHub 的[中找到。](https://web.archive.org/web/20221129001719/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-perf)