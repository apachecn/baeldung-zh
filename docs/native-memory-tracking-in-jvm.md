# JVM 中的本机内存跟踪

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/native-memory-tracking-in-jvm>

## 1。概述

有没有想过为什么 Java 应用程序会通过众所周知的`-Xms`和`-Xmx`调优标志消耗比指定数量多得多的内存？出于各种原因和可能的优化，JVM 可能会分配额外的本机内存。这些额外的分配最终会使消耗的内存超过`-Xmx`限制。

在本教程中，我们将列举 JVM 中本机内存分配的一些常见来源，以及它们的大小调整标志，然后学习如何使用 `Native Memory Tracking`来监控它们。

## 2。本地分配

堆通常是 Java 应用程序中最大的内存消耗者，但是还有其他的。除了堆之外，JVM 还从本机内存中分配一个相当大的块来维护它的类元数据、应用程序代码、JIT 生成的代码、内部数据结构等。在接下来的章节中，我们将探讨其中的一些分配。

### 2.1\. Metaspace

**为了维护一些关于加载的类的元数据，JVM 使用了一个专用的非堆区域，称为`Metaspace`** 。在 Java 8 之前，这个对等物被称为`PermGen`或`Permanent Generation`。Metaspace 或 PermGen 包含关于已加载类的元数据，而不是保存在堆中的实例。

这里重要的是**堆大小配置不会影响元空间大小**，因为元空间是堆外数据区。为了限制元空间的大小，我们使用其他调优标志:

*   ` -XX:MetaspaceSize`和`-XX:MaxMetaspaceSize`设置最小和最大元空间大小
*   在 Java 8 之前，`-XX:PermSize`和`-XX:MaxPermSize`设置最小和最大 PermGen 大小

### 2.2.线

JVM 中最消耗内存的数据区域之一是堆栈，它与每个线程同时创建。堆栈存储局部变量和部分结果，在方法调用中起着重要的作用。

默认的线程堆栈大小取决于平台，但是在大多数现代 64 位操作系统中，它大约是 1 MB。该尺寸可通过`-Xss `调谐标志进行配置。

与其他数据区域相比，当线程数量没有限制时，分配给堆栈的总内存实际上是无限的。同样值得一提的是，JVM 本身需要一些线程来执行其内部操作，如 GC 或实时编译。

### 2.3.代码缓存

为了在不同平台上运行 JVM 字节码，需要将其转换成机器指令。JIT 编译器负责程序执行时的编译。

**当 JVM 将字节码编译成汇编指令时，它将这些指令存储在一个名为`Code Cache. `** 的特殊非堆数据区。代码缓存可以像 JVM 中的其他数据区一样进行管理。`-XX:InitialCodeCacheSize `和`-XX:ReservedCodeCacheSize `调优标志决定代码缓存的初始和最大可能大小。

### 2.4.碎片帐集

JVM 附带了一些 GC 算法，每种算法适合不同的用例。所有这些 GC 算法都有一个共同的特点:它们需要使用一些堆外数据结构来执行任务。这些内部数据结构消耗更多的本机内存。

### 2.5.标志

让我们从应用程序和库代码中最常用的数据类型之一`Strings, `开始。因为它们无处不在，所以它们通常占据堆的很大一部分。如果大量的字符串包含相同的内容，那么堆的很大一部分就会被浪费掉。

为了节省一些堆空间，我们可以存储每个`String `的一个版本，让其他的引用存储的版本。**这个过程被称为`String Interning.`** ，因为 JVM 只能对`Compile Time String Constants, `进行实习，所以我们可以在我们打算实习的字符串上手动调用`intern() `方法。

**JVM 将托管的字符串存储在一个特殊的本地固定大小的散列表中，称为** `**String Table,**` **，也称为 [`String Pool`](/web/20220918082910/https://www.baeldung.com/java-string-pool)** `**.**`我们可以通过`-XX:StringTableSize `调优标志来配置表的大小(即桶的数量)。

除了字符串表，还有另一个称为`Runtime Constant Pool. `的本地数据区。JVM 使用这个池来存储常量，如编译时的数字文字或方法，以及运行时必须解析的字段引用。

### 2.6.本机字节缓冲区

JVM 通常是大量本机内存分配的嫌疑人，但有时开发人员也可以直接分配本机内存。最常见的方法是 JNI 的`malloc `调用和 NIO 的直接`ByteBuffers.`

### 2.7.其他调整标志

在本节中，我们针对不同的优化场景使用了一些 JVM 调优标志。使用下面的技巧，我们可以找到几乎所有与特定概念相关的调优标志:

```java
$ java -XX:+PrintFlagsFinal -version | grep <concept>
```

`PrintFlagsFinal` 打印 JVM 中所有的–`XX `选项。例如，要查找所有与元空间相关的标志:

```java
$ java -XX:+PrintFlagsFinal -version | grep Metaspace
      // truncated
      uintx MaxMetaspaceSize                          = 18446744073709547520                    {product}
      uintx MetaspaceSize                             = 21807104                                {pd product}
      // truncated
```

## 3。原生记忆追踪(NMT)

现在我们知道了 JVM 中本机内存分配的常见来源，是时候了解如何监控它们了。**首先，我们应该使用另一个 JVM 调优标志来启用本机内存跟踪:`-XX:NativeMemoryTracking=off|sumary|detail. `** 默认情况下，NMT 是关闭的，但是我们可以启用它来查看其观察结果的摘要或详细视图。

假设我们想要跟踪一个典型的 Spring Boot 应用程序的本机分配:

```java
$ java -XX:NativeMemoryTracking=summary -Xms300m -Xmx300m -XX:+UseG1GC -jar app.jar
```

这里，我们启用了 NMT，同时分配了 300 MB 的堆空间，使用 G1 作为我们的 GC 算法。

### 3.1.即时快照

启用 NMT 后，我们可以随时使用`jcmd `命令获取本机内存信息:

```java
$ jcmd <pid> VM.native_memory
```

为了找到 JVM 应用程序的 PID，我们可以使用*jps**命令:*

```java
$ jps -l                    
7858 app.jar // This is our app
7899 sun.tools.jps.Jps
```

现在，如果我们将`jcmd `与适当的`pid`一起使用，`VM.native_memory `会让 JVM 打印出关于本机分配的信息:

```java
$ jcmd 7858 VM.native_memory
```

让我们一节一节地分析 NMT 的输出。

### 3.2.分配总额

NMT 报告保留和提交的总内存如下:

```java
Native Memory Tracking:
Total: reserved=1731124KB, committed=448152KB
```

**保留内存代表我们的应用程序可能使用的内存总量。相反，提交的内存等于我们的应用程序现在使用的内存量。**

尽管分配了 300 MB 的堆，我们的应用程序的总保留内存几乎是 1.7 GB，比这多得多。类似地，提交的内存大约为 440 MB，也比 300 MB 多得多。

在 total 部分之后，NMT 报告每个分配源的内存分配。所以，让我们深入探究每一个来源。

### 3.3.许多

NMT 报告了我们预期的堆分配:

```java
Java Heap (reserved=307200KB, committed=307200KB)
          (mmap: reserved=307200KB, committed=307200KB)
```

300 MB 的保留和提交内存，这与我们的堆大小设置相匹配。

### 3.4\. Metaspace

以下是 NMT 对加载类的类元数据的描述:

```java
Class (reserved=1091407KB, committed=45815KB)
      (classes #6566)
      (malloc=10063KB #8519) 
      (mmap: reserved=1081344KB, committed=35752KB)
```

几乎保留了 1 GB，45 MB 用于加载 6566 个类。

### 3.5.线

这是 NMT 关于线程分配的报告:

```java
Thread (reserved=37018KB, committed=37018KB)
       (thread #37)
       (stack: reserved=36864KB, committed=36864KB)
       (malloc=112KB #190) 
       (arena=42KB #72)
```

总共有 36 MB 的内存分配给 37 个线程的堆栈——几乎每个堆栈 1 MB。JVM 在创建时就将内存分配给线程，因此保留的和提交的分配是相等的。

### 3.6.代码缓存

让我们看看 NMT 对 JIT 生成和缓存的汇编指令怎么说:

```java
Code (reserved=251549KB, committed=14169KB)
     (malloc=1949KB #3424) 
     (mmap: reserved=249600KB, committed=12220KB)
```

目前，大约有 13 MB 的代码被缓存，这个数量有可能达到大约 245 MB。

### 3.7.车底距地高(Ground Clearance)

以下是 NMT 关于 G1 GC 内存使用的报告:

```java
GC (reserved=61771KB, committed=61771KB)
   (malloc=17603KB #4501) 
   (mmap: reserved=44168KB, committed=44168KB)
```

正如我们所看到的，几乎有 60 MB 被保留下来，用于帮助 G1。

让我们看看一个简单得多的 GC 的内存使用情况，比如串行 GC:

```java
$ java -XX:NativeMemoryTracking=summary -Xms300m -Xmx300m -XX:+UseSerialGC -jar app.jar
```

串行 GC 几乎不使用 1 MB:

```java
GC (reserved=1034KB, committed=1034KB)
   (malloc=26KB #158) 
   (mmap: reserved=1008KB, committed=1008KB)
```

显然，我们不应该仅仅因为一个 GC 算法的内存使用量就选择它，因为串行 GC 的停止世界性质可能会导致性能下降。然而，[有几个 GC 可以从](/web/20220918082910/https://www.baeldung.com/jvm-garbage-collectors)中选择，它们各自以不同的方式平衡内存和性能。

### 3.8.标志

以下是 NMT 关于符号分配的报告，如字符串表和常量池:

```java
Symbol (reserved=10148KB, committed=10148KB)
       (malloc=7295KB #66194) 
       (arena=2853KB #1)
```

将近 10 MB 被分配给符号。

### 3.9.NMT 随着时间的推移

NMT 允许我们跟踪内存分配如何随时间变化。首先，我们应该将应用程序的当前状态标记为基线:

```java
$ jcmd <pid> VM.native_memory baseline
Baseline succeeded
```

然后，过一会儿，我们可以将当前的内存使用情况与基线进行比较:

```java
$ jcmd <pid> VM.native_memory summary.diff
```

NMT 使用+和–符号，告诉我们这段时间内存使用的变化情况:

```java
Total: reserved=1771487KB +3373KB, committed=491491KB +6873KB
-             Java Heap (reserved=307200KB, committed=307200KB)
                        (mmap: reserved=307200KB, committed=307200KB)

-             Class (reserved=1084300KB +2103KB, committed=39356KB +2871KB)
// Truncated
```

保留和提交的总内存分别增加了 3 MB 和 6 MB。内存分配的其他波动也很容易被发现。

### 3.10.详细的 NMT

NMT 可以提供关于整个存储空间的地图的非常详细的信息。为了启用这个详细的报告，我们应该使用`-XX:NativeMemoryTracking=detail `调优标志。

## 4.结论

在本文中，我们列举了 JVM 中本机内存分配的不同贡献者。然后，我们学习了如何检查一个正在运行的应用程序来监控它的本机分配。有了这些见解，我们可以更有效地调优我们的应用程序和调整我们的运行时环境。