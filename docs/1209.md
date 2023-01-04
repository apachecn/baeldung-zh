# 诊断正在运行的 JVM

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/running-jvm-diagnose>

## 1.概观

Java 虚拟机(JVM)是使计算机能够运行 Java 程序的虚拟机。在本文中，我们将了解如何轻松诊断正在运行的 JVM。

我们在 JDK 有许多可用的[工具](https://web.archive.org/web/20220628054830/https://docs.oracle.com/en/java/javase/11/tools/jconsole.html)，可用于各种开发、监控和故障排除活动。我们来看看`jcmd`，它相当好用，可以提供关于一个正在运行的 JVM 的各种信息。**此外，`jcmd`是从 JDK 7 开始推荐的工具，用于增强 JVM 诊断，没有或只有最小的性能开销。**

## 2.什么是`jcmd`？

这是一个向正在运行的 JVM 发送诊断命令请求的实用程序。然而，它必须在运行 JVM 的同一台机器上使用。更多细节可在其[文档](https://web.archive.org/web/20220628054830/https://docs.oracle.com/en/java/javase/11/tools/jcmd.html#GUID-59153599-875E-447D-8D98-0078A5778F05)中获得。

让我们看看如何将这个实用程序与运行在服务器上的示例 Java 应用程序一起使用。

## 3.如何使用`jcmd`？

让我们使用 [`Spring Initializr`](https://web.archive.org/web/20220628054830/https://start.spring.io/) 和`JDK11`创建一个快速演示 web 应用程序。现在，让我们启动服务器并使用`jcmd`对其进行诊断。

### 3.1.获取 PID

我们知道每个进程都有一个相关的进程 id，称为`PID`。因此，要为我们的应用程序获取关联的`PID`，我们可以使用`jcmd`，它将列出所有适用的 Java 进程，如下所示:

```
[[email protected]](/web/20220628054830/https://www.baeldung.com/cdn-cgi/l/email-protection):/# jcmd
65 jdk.jcmd/sun.tools.jcmd.JCmd
18 /home/pgm/demo-0.0.1-SNAPSHOT.jar
[[email protected]](/web/20220628054830/https://www.baeldung.com/cdn-cgi/l/email-protection):/# 
```

在这里，我们可以看到正在运行的应用程序的 PID 是 18。

### 3.2.获取可能的`jcmd`用法列表

让我们从`jcmd` `PID` `help`命令开始，找出可用的选项:

```
[[email protected]](/web/20220628054830/https://www.baeldung.com/cdn-cgi/l/email-protection):/# jcmd 18 help
18:
The following commands are available:
Compiler.CodeHeap_Analytics
Compiler.codecache
Compiler.codelist
Compiler.directives_add
Compiler.directives_clear
Compiler.directives_print
Compiler.directives_remove
Compiler.queue
GC.class_histogram
GC.class_stats
GC.finalizer_info
GC.heap_dump
GC.heap_info
GC.run
GC.run_finalization
JFR.check
JFR.configure
JFR.dump
JFR.start
JFR.stop
JVMTI.agent_load
JVMTI.data_dump
ManagementAgent.start
ManagementAgent.start_local
ManagementAgent.status
ManagementAgent.stop
Thread.print
VM.class_hierarchy
VM.classloader_stats
VM.classloaders
VM.command_line
VM.dynlibs
VM.flags
VM.info
VM.log
VM.metaspace
VM.native_memory
VM.print_touched_methods
VM.set_flag
VM.stringtable
VM.symboltable
VM.system_properties
VM.systemdictionary
VM.uptime
VM.version
help 
```

在不同版本的 HotSpot VM 中，可用的诊断命令可能会有所不同。

## 4. **`jcmd`命令**

让我们探索一些最有用的`jcmd`命令选项来诊断我们正在运行的 JVM。

### 4.1.`VM.version`

这是为了获得 JVM 的基本细节，如下所示:

```
[[email protected]](/web/20220628054830/https://www.baeldung.com/cdn-cgi/l/email-protection):/# jcmd 18 VM.version
18:
OpenJDK 64-Bit Server VM version 11.0.11+9-Ubuntu-0ubuntu2.20.04
JDK 11.0.11
[[email protected]](/web/20220628054830/https://www.baeldung.com/cdn-cgi/l/email-protection):/# 
```

在这里，我们可以看到我们的示例应用程序使用了 OpenJDK 11。

### 4.2.`VM.system_properties`

这将打印为我们的虚拟机设置的所有系统属性。可能会显示数百行信息:

```
[[email protected]](/web/20220628054830/https://www.baeldung.com/cdn-cgi/l/email-protection):/# jcmd 18 VM.system_properties
18:
#Thu Jul 22 10:56:13 IST 2021
awt.toolkit=sun.awt.X11.XToolkit
java.specification.version=11
sun.cpu.isalist=
sun.jnu.encoding=ANSI_X3.4-1968
java.class.path=/home/pgm/demo-0.0.1-SNAPSHOT.jar
java.vm.vendor=Ubuntu
sun.arch.data.model=64
catalina.useNaming=false
java.vendor.url=https\://ubuntu.com/
user.timezone=Asia/Kolkata
java.vm.specification.version=11
...
```

### 4.3.`VM.flags`

对于我们的示例应用程序，这将打印所有使用的 VM 参数，或者是我们给出的，或者是 JVM 默认使用的。在这里，我们可以注意到各种默认虚拟机参数，如下所示:

```
[[email protected]](/web/20220628054830/https://www.baeldung.com/cdn-cgi/l/email-protection):/# jcmd 18 VM.flags            
18:
-XX:CICompilerCount=3 -XX:CompressedClassSpaceSize=260046848 -XX:ConcGCThreads=1 -XX:G1ConcRefinementThreads=4 -XX:G1HeapRegionSize=1048576 -XX:GCDrainStackTargetSize=64 -XX:InitialHeapSize=536870912 -XX:MarkStackSize=4194304 -XX:MaxHeapSize=536870912 -XX:MaxMetaspaceSize=268435456 -XX:MaxNewSize=321912832 -XX:MinHeapDeltaBytes=1048576 -XX:NonNMethodCodeHeapSize=5830732 -XX:NonProfiledCodeHeapSize=122913754 -XX:ProfiledCodeHeapSize=122913754 -XX:ReservedCodeCacheSize=251658240 -XX:+SegmentedCodeCache -XX:ThreadStackSize=256 -XX:+UseCompressedClassPointers -XX:+UseCompressedOops -XX:+UseFastUnorderedTimeStamps -XX:+UseG1GC 
[[email protected]](/web/20220628054830/https://www.baeldung.com/cdn-cgi/l/email-protection):/#
```

类似地，其他命令，如`VM.command_line`、 `VM.uptime`、`VM.dynlibs,` 也提供了关于所使用的各种其他属性的其他基本和有用的细节。

以上所有命令主要是为了获得不同的`JVM`相关细节。现在让我们看看更多的命令，这些命令可以帮助解决一些与 JVM 相关的问题。

### 4.4.`Thread.print`

该命令用于获取即时线程转储。因此，它将打印所有正在运行的线程的堆栈跟踪。下面是它的使用方法，根据使用的线程数量，它可能会给出很长的输出:

```
[[email protected]](/web/20220628054830/https://www.baeldung.com/cdn-cgi/l/email-protection):/# jcmd 18 Thread.print
18:
2021-07-22 10:58:08
Full thread dump OpenJDK 64-Bit Server VM (11.0.11+9-Ubuntu-0ubuntu2.20.04 mixed mode, sharing):

Threads class SMR info:
_java_thread_list=0x00007f21cc0028d0, length=25, elements={
0x00007f2210244800, 0x00007f2210246800, 0x00007f221024b800, 0x00007f221024d800,
0x00007f221024f800, 0x00007f2210251800, 0x00007f2210253800, 0x00007f22102ae800,
0x00007f22114ef000, 0x00007f21a44ce000, 0x00007f22114e3800, 0x00007f221159d000,
0x00007f22113ce800, 0x00007f2210e78800, 0x00007f2210e7a000, 0x00007f2210f20800,
0x00007f2210f22800, 0x00007f2210f24800, 0x00007f2211065000, 0x00007f2211067000,
0x00007f2211069000, 0x00007f22110d7800, 0x00007f221122f800, 0x00007f2210016000,
0x00007f21cc001000
}

"Reference Handler" #2 daemon prio=10 os_prio=0 cpu=2.32ms elapsed=874.34s tid=0x00007f2210244800 nid=0x1a waiting on condition  [0x00007f221452a000]
   java.lang.Thread.State: RUNNABLE
	at java.lang.ref.Reference.waitForReferencePendingList([[email protected]](/web/20220628054830/https://www.baeldung.com/cdn-cgi/l/email-protection)/Native Method)
	at java.lang.ref.Reference.processPendingReferences([[email protected]](/web/20220628054830/https://www.baeldung.com/cdn-cgi/l/email-protection)/Reference.java:241)
	at java.lang.ref.Reference$ReferenceHandler.run([[email protected]](/web/20220628054830/https://www.baeldung.com/cdn-cgi/l/email-protection)/Reference.java:213)

"Finalizer" #3 daemon prio=8 os_prio=0 cpu=0.32ms elapsed=874.34s tid=0x00007f2210246800 nid=0x1b in Object.wait()  [0x00007f22144e9000]
   java.lang.Thread.State: WAITING (on object monitor)
	at java.lang.Object.wait([[email protected]](/web/20220628054830/https://www.baeldung.com/cdn-cgi/l/email-protection)/Native Method)
	- waiting on <0x00000000f7330898> (a java.lang.ref.ReferenceQueue$Lock)
	at java.lang.ref.ReferenceQueue.remove([[email protected]](/web/20220628054830/https://www.baeldung.com/cdn-cgi/l/email-protection)/ReferenceQueue.java:155)
	- waiting to re-lock in wait() <0x00000000f7330898> (a java.lang.ref.ReferenceQueue$Lock)
	at java.lang.ref.ReferenceQueue.remove([[email protected]](/web/20220628054830/https://www.baeldung.com/cdn-cgi/l/email-protection)/ReferenceQueue.java:176)
	at java.lang.ref.Finalizer$FinalizerThread.run([[email protected]](/web/20220628054830/https://www.baeldung.com/cdn-cgi/l/email-protection)/Finalizer.java:170)

"Signal Dispatcher" #4 daemon prio=9 os_prio=0 cpu=0.40ms elapsed=874.33s tid=0x00007f221024b800 nid=0x1c runnable  [0x0000000000000000]
   java.lang.Thread.State: RUNNABLE 
```

关于使用其他选项捕获线程转储的详细讨论可以在[这里](/web/20220628054830/https://www.baeldung.com/java-thread-dump)找到。

### 4.5.`GC.class_histogram`

让我们使用另一个`jcmd `命令，它将提供关于堆使用的重要信息。此外，这将列出具有许多实例的所有类(外部的或特定于应用程序的)。同样，根据使用的类的数量，该列表可能有数百行:

```
[[email protected]](/web/20220628054830/https://www.baeldung.com/cdn-cgi/l/email-protection):/# jcmd 18 GC.class_histogram
18:
 num     #instances         #bytes  class name (module)
-------------------------------------------------------
   1:         41457        2466648  [B ([[email protected]](/web/20220628054830/https://www.baeldung.com/cdn-cgi/l/email-protection))
   2:         38656         927744  java.lang.String ([[email protected]](/web/20220628054830/https://www.baeldung.com/cdn-cgi/l/email-protection))
   3:          6489         769520  java.lang.Class ([[email protected]](/web/20220628054830/https://www.baeldung.com/cdn-cgi/l/email-protection))
   4:         21497         687904  java.util.concurrent.ConcurrentHashMap$Node ([[email protected]](/web/20220628054830/https://www.baeldung.com/cdn-cgi/l/email-protection))
   5:          6570         578160  java.lang.reflect.Method ([[email protected]](/web/20220628054830/https://www.baeldung.com/cdn-cgi/l/email-protection))
   6:          6384         360688  [Ljava.lang.Object; ([[email protected]](/web/20220628054830/https://www.baeldung.com/cdn-cgi/l/email-protection))
   7:          9668         309376  java.util.HashMap$Node ([[email protected]](/web/20220628054830/https://www.baeldung.com/cdn-cgi/l/email-protection))
   8:          7101         284040  java.util.LinkedHashMap$Entry ([[email protected]](/web/20220628054830/https://www.baeldung.com/cdn-cgi/l/email-protection))
   9:          3033         283008  [Ljava.util.HashMap$Node; ([[email protected]](/web/20220628054830/https://www.baeldung.com/cdn-cgi/l/email-protection))
  10:          2919         257000  [I ([[email protected]](/web/20220628054830/https://www.baeldung.com/cdn-cgi/l/email-protection))
  11:           212         236096  [Ljava.util.concurrent.ConcurrentHashMap$Node; ([[email protected]](/web/20220628054830/https://www.baeldung.com/cdn-cgi/l/email-protection)) 
```

然而，如果这不能给出一个清晰的图像，我们可以得到一个堆转储。接下来我们来看看。

### 4.6.`G**C.heap_dump**`

这个命令将给出一个即时的 JVM 堆转储。因此，我们可以将堆转储提取到一个文件中，以便稍后进行分析，如下所示:

```
[[email protected]](/web/20220628054830/https://www.baeldung.com/cdn-cgi/l/email-protection):/# jcmd 18 GC.heap_dump ./demo_heap_dump
18:
Heap dump file created
[[email protected]](/web/20220628054830/https://www.baeldung.com/cdn-cgi/l/email-protection):/# 
```

这里，`demo_heap_dump` 是堆转储文件名。此外，这将在我们的应用程序 jar 所在的位置创建。

### 4.7.`JFR`命令选项

在我们之前的[文章](/web/20220628054830/https://www.baeldung.com/java-flight-recorder-monitoring)中，我们讨论了使用`JFR`和`JMC`的 Java 应用程序监控。现在，让我们研究一下`jcmd`命令，我们可以用它来分析应用程序的性能问题。

[`JFR`(或 Java 飞行记录器)](https://web.archive.org/web/20220628054830/https://docs.oracle.com/en/java/javase/11/troubleshoot/diagnostic-tools.html#GUID-D38849B6-61C7-4ED6-A395-EA4BC32A9FD6)是内置于 JDK 中的剖析和事件收集框架。`JFR`允许我们收集关于 JVM 和 Java 应用程序如何运行的详细底层信息。另外，我们可以使用`[JMC](https://web.archive.org/web/20220628054830/https://docs.oracle.com/en/java/javase/11/troubleshoot/diagnostic-tools.html#GUID-3FA1CF76-96FA-41A6-8F38-DC83171EE834)`来可视化`JFR`采集的数据。因此，`JFR`和`JMC`一起创建了一个完整的工具链来持续收集底层和详细的运行时信息。

虽然如何使用`JMC`不在本文的讨论范围内，但是我们将看到如何使用`jcmd`创建一个`JFR`文件。`JFR`是商业特性。因此，默认情况下，它是禁用的。然而，这可以使用“`jcmd PID VM.unlock_commercial_features`”来启用。

然而，我们在文章中使用了`OpenJDK`。因此`JFR`为我们启用。现在让我们使用下面的`jcmd`命令生成一个`JFR`文件:

```
[[email protected]](/web/20220628054830/https://www.baeldung.com/cdn-cgi/l/email-protection):/# jcmd 18 JFR.start name=demo_recording settings=profile delay=10s duration=20s filename=./demorecording.jfr
18:
Recording 1 scheduled to start in 10 s. The result will be written to:

/demorecording.jfr
[[email protected]](/web/20220628054830/https://www.baeldung.com/cdn-cgi/l/email-protection):/# jcmd 18 JFR.check
18:
Recording 1: name=demo_recording duration=20s (delayed)
[[email protected]](/web/20220628054830/https://www.baeldung.com/cdn-cgi/l/email-protection):/# jcmd 18 JFR.check
18:
Recording 1: name=demo_recording duration=20s (running)
[[email protected]](/web/20220628054830/https://www.baeldung.com/cdn-cgi/l/email-protection):/# jcmd 18 JFR.check
18:
Recording 1: name=demo_recording duration=20s (stopped) 
```

我们已经在 jar 应用程序所在的相同位置创建了一个名为`demorecording.jfr `的样本`JFR`记录文件。此外，该记录时长 20 秒，根据要求进行配置。

**此外，我们可以使用`JFR.check `命令检查`JFR`记录的状态。而且，我们可以使用`JFR.stop`命令立即停止并丢弃录像。另一方面，`JFR.dump`命令可用于立即停止并转储记录。**

### 4.8.虚拟机本地内存

这是最好的命令之一，可以提供关于 JVM 上的`heap and non-heap `内存的许多有用的细节。因此，这可以用来调整内存使用和检测任何内存泄漏。众所周知，JVM 内存可以大致分为堆内存和非堆内存。为了获得整个 JVM 内存使用的细节，我们可以使用这个实用程序。此外，这对于为基于容器的应用程序定义内存大小也很有用。

**要使用此功能，我们需要使用额外的 VM 参数重启我们的应用程序，即–`XX:NativeMemoryTracking=summary   or  -XX:NativeMemoryTracking=detail`。请注意，启用 NMT 会导致 5% -10%的性能开销。**

这将为我们提供一个新的 PID 来进行诊断:

```
[[email protected]](/web/20220628054830/https://www.baeldung.com/cdn-cgi/l/email-protection):/# jcmd 19 VM.native_memory
19:

Native Memory Tracking:

Total: reserved=1159598KB, committed=657786KB
-                 Java Heap (reserved=524288KB, committed=524288KB)
                            (mmap: reserved=524288KB, committed=524288KB) 

-                     Class (reserved=279652KB, committed=29460KB)
                            (classes #6425)
                            (  instance classes #5960, array classes #465)
                            (malloc=1124KB #15883) 
                            (mmap: reserved=278528KB, committed=28336KB) 
                            (  Metadata:   )
                            (    reserved=24576KB, committed=24496KB)
                            (    used=23824KB)
                            (    free=672KB)
                            (    waste=0KB =0.00%)
                            (  Class space:)
                            (    reserved=253952KB, committed=3840KB)
                            (    used=3370KB)
                            (    free=470KB)
                            (    waste=0KB =0.00%)

-                    Thread (reserved=18439KB, committed=2699KB)
                            (thread #35)
                            (stack: reserved=18276KB, committed=2536KB)
                            (malloc=123KB #212) 
                            (arena=39KB #68)

-                      Code (reserved=248370KB, committed=12490KB)
                            (malloc=682KB #3839) 
                            (mmap: reserved=247688KB, committed=11808KB) 

-                        GC (reserved=62483KB, committed=62483KB)
                            (malloc=10187KB #7071) 
                            (mmap: reserved=52296KB, committed=52296KB) 

-                  Compiler (reserved=146KB, committed=146KB)
                            (malloc=13KB #307) 
                            (arena=133KB #5)

-                  Internal (reserved=460KB, committed=460KB)
                            (malloc=428KB #1421) 
                            (mmap: reserved=32KB, committed=32KB) 

-                     Other (reserved=16KB, committed=16KB)
                            (malloc=16KB #3) 

-                    Symbol (reserved=6593KB, committed=6593KB)
                            (malloc=6042KB #72520) 
                            (arena=552KB #1)

-    Native Memory Tracking (reserved=1646KB, committed=1646KB)
                            (malloc=9KB #113) 
                            (tracking overhead=1637KB)

-        Shared class space (reserved=17036KB, committed=17036KB)
                            (mmap: reserved=17036KB, committed=17036KB) 

-               Arena Chunk (reserved=185KB, committed=185KB)
                            (malloc=185KB) 

-                   Logging (reserved=4KB, committed=4KB)
                            (malloc=4KB #191) 

-                 Arguments (reserved=18KB, committed=18KB)
                            (malloc=18KB #489) 

-                    Module (reserved=124KB, committed=124KB)
                            (malloc=124KB #1521) 

-              Synchronizer (reserved=129KB, committed=129KB)
                            (malloc=129KB #1089) 

-                 Safepoint (reserved=8KB, committed=8KB)
                            (mmap: reserved=8KB, committed=8KB) 
```

在这里，我们可以注意到除了`Java` `Heap Memory`之外的不同内存类型的细节。*类*定义了用于存储类元数据的 JVM 内存。类似地，`Thread`定义了我们的应用程序线程正在使用的内存。并且`Code`给出了用于存储 *JIT-* 生成代码的内存，`Compiler`本身也有一些空间使用，`GC`也占用了一些空间。

此外，`reserved`可以估计我们的应用程序所需的内存。而`committed`显示了分配的最小内存。

## 5.诊断内存泄漏

让我们看看如何识别 JVM 中是否有内存泄漏。因此，首先，我们需要有一个基线。然后需要监视一段时间，以了解在上述任何一种内存类型中，内存是否持续增加。

让我们首先`baseline`一下 JVM 的内存使用情况如下:

```
[[email protected]](/web/20220628054830/https://www.baeldung.com/cdn-cgi/l/email-protection):/# jcmd 19 VM.native_memory baseline
19:
Baseline succeeded 
```

现在，使用该应用程序正常或大量使用一段时间。最后，只需使用`diff `来标识自`baseline `以来的变化，如下所示:

```
[[email protected]](/web/20220628054830/https://www.baeldung.com/cdn-cgi/l/email-protection):/# jcmd 19 VM.native_memory summary.diff
19:

Native Memory Tracking:

Total: reserved=1162150KB +2540KB, committed=660930KB +3068KB

-                 Java Heap (reserved=524288KB, committed=524288KB)
                            (mmap: reserved=524288KB, committed=524288KB)

-                     Class (reserved=281737KB +2085KB, committed=31801KB +2341KB)
                            (classes #6821 +395)
                            (  instance classes #6315 +355, array classes #506 +40)
                            (malloc=1161KB +37KB #16648 +750)
                            (mmap: reserved=280576KB +2048KB, committed=30640KB +2304KB)
                            (  Metadata:   )
                            (    reserved=26624KB +2048KB, committed=26544KB +2048KB)
                            (    used=25790KB +1947KB)
                            (    free=754KB +101KB)
                            (    waste=0KB =0.00%)
                            (  Class space:)
                            (    reserved=253952KB, committed=4096KB +256KB)
                            (    used=3615KB +245KB)
                            (    free=481KB +11KB)
                            (    waste=0KB =0.00%)

-                    Thread (reserved=18439KB, committed=2779KB +80KB)
                            (thread #35)
                            (stack: reserved=18276KB, committed=2616KB +80KB)
                            (malloc=123KB #212)
                            (arena=39KB #68)

-                      Code (reserved=248396KB +21KB, committed=12772KB +213KB)
                            (malloc=708KB +21KB #3979 +110)
                            (mmap: reserved=247688KB, committed=12064KB +192KB)

-                        GC (reserved=62501KB +16KB, committed=62501KB +16KB)
                            (malloc=10205KB +16KB #7256 +146)
                            (mmap: reserved=52296KB, committed=52296KB)

-                  Compiler (reserved=161KB +15KB, committed=161KB +15KB)
                            (malloc=29KB +15KB #341 +34)
                            (arena=133KB #5)

-                  Internal (reserved=495KB +35KB, committed=495KB +35KB)
                            (malloc=463KB +35KB #1429 +8)
                            (mmap: reserved=32KB, committed=32KB)

-                     Other (reserved=52KB +36KB, committed=52KB +36KB)
                            (malloc=52KB +36KB #9 +6)

-                    Symbol (reserved=6846KB +252KB, committed=6846KB +252KB)
                            (malloc=6294KB +252KB #76359 +3839)
                            (arena=552KB #1)

-    Native Memory Tracking (reserved=1727KB +77KB, committed=1727KB +77KB)
                            (malloc=11KB #150 +2)
                            (tracking overhead=1716KB +77KB)

-        Shared class space (reserved=17036KB, committed=17036KB)
                            (mmap: reserved=17036KB, committed=17036KB)

-               Arena Chunk (reserved=186KB, committed=186KB)
                            (malloc=186KB)

-                   Logging (reserved=4KB, committed=4KB)
                            (malloc=4KB #191)

-                 Arguments (reserved=18KB, committed=18KB)
                            (malloc=18KB #489)

-                    Module (reserved=124KB, committed=124KB)
                            (malloc=124KB #1528 +7)

-              Synchronizer (reserved=132KB +3KB, committed=132KB +3KB)
                            (malloc=132KB +3KB #1111 +22)

-                 Safepoint (reserved=8KB, committed=8KB)
                            (mmap: reserved=8KB, committed=8KB) 
```

随着时间的推移，随着 GC 的工作，我们会注意到内存使用的增加和减少。但是，如果内存使用出现不受控制的增长，那么这可能是一个内存泄漏问题。因此，我们可以识别内存泄漏区域，如`Heap`、`Thread, Code, Class,`等。从这些数据来看。如果我们的应用程序需要更多的内存，我们可以分别调整相应的 VM 参数。

**如果内存泄漏发生在`Heap`中，我们可以进行堆转储(如前所述)或者只是调优`Xmx`。同样，如果内存泄漏是在`Thread, we`可以寻找未处理的递归指令或调优`Xss`。**

## 6.结论

在本文中，我们介绍了一个实用程序，用于针对不同的场景诊断 JVM。

我们还介绍了`jcmd`命令及其各种用法，以获得堆转储、线程转储和 JFR 记录，用于各种与性能相关的分析。最后，我们还研究了一种使用`jcmd`诊断内存泄漏的方法。