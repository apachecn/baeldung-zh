# 无法为对象堆保留足够的空间

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-heap-memory-error>

## 1.概观

在本教程中，我们将学习`“Could not reserve enough space for object heap”` 错误的原因，同时经历一些可能的场景。

## 2.症状

“无法为对象堆保留足够的空间”是一个特定的 JVM 错误，当 **Java 进程由于在运行系统**上遇到内存限制而无法创建 **虚拟机时会引发该错误**

```java
java -Xms4G -Xmx4G -jar HelloWorld.jar

Error occurred during initialization of VM
Could not reserve enough space for object heap
Error: Could not create the Java Virtual Machine.
Error: A fatal exception has occurred. Program will exit.
```

通常，当我们遇到错误时，有两种可能的情况。
首先，当我们用**最大堆大小限制参数(`-Xmx` )** 、值**比进程在操作系统上可以拥有的值** **多一个 Java 进程时。**

堆大小限制因几个约束条件而异:

*   硬件架构(32/64 位)
*   JVM 位版本(32/64 位)
*   我们使用的操作系统

其次，当 Java 进程**由于在同一系统上运行的其他应用程序消耗内存而无法保留指定数量的内存**时。

## 3.堆大小

Java 堆空间是运行时 Java 程序的内存分配池，由 JVM 自己管理。默认情况下，**分配池被限制为初始和最大大小。**要了解更多关于 Java 堆空间的知识，请看[的这篇文章](/web/20221126230938/https://www.baeldung.com/java-stack-heap)。

让我们看看不同环境中的最大堆大小是多少，以及如何设置限制。

### 3.1.最大堆大小

32 位和 64 位 JVM 的最大理论堆限制很容易通过查看可用内存空间来确定，32 位 JVM 的 2^32 (4 GB)和 64 位 JVM 的 2^64(16eb)。

在实践中，由于各种限制，该限制可能会低得多，并且因操作系统而异。例如，**在 32 位 Windows 系统上，最大堆大小范围在 1.4 GB 到 1.6 GB 之间**。相比之下，在 32 位 Linux 系统上，最大堆大小可以扩展到 3 GB。

因此，**如果应用程序需要大堆，我们应该使用 64 位 JVM** 。然而，对于大堆，垃圾收集器将有更多的工作要做，所以在堆大小和性能之间找到一个好的平衡很重要。

### 3.2.如何控制堆大小限制？

我们有两个选项来控制 JVM 的堆大小限制。

首先，通过在每次 JVM 初始化时使用 Java **命令行参数**:

```java
-Xms<size>    Sets initial Java heap size. This value must be a multiple of 1024 and greater than 1 MB.
-Xmx<size>    Sets maximum Java heap size. This value must be a multiple of 1024 and greater than 2 MB.
-Xmn<size>    Sets the initial and maximum size (in bytes) of the heap for the young generation.
```

对于大小值，我们可以附加字母`k`或`K`、`m`或`M`和`g`或`G`来分别表示千字节、兆字节和千兆字节。如果没有指定字母，则使用默认单位(字节)。

```java
-Xmn2g
-Xmn2048m
-Xmn2097152k
-Xmn2147483648
```

其次，通过使用环境变量`JAVA_OPTS`来全局配置上述 Java 命令行参数。因此，系统上的每个 JVM 初始化都将自动使用环境变量中设置的配置。

```java
JAVA_OPTS="-Xms256m -Xmx512m"
```

要了解更多信息，请查看我们全面的 [JVM 参数](/web/20221126230938/https://www.baeldung.com/jvm-parameters)指南。

## 4.结论

在本教程中，我们讨论了 JVM 无法为对象堆保留足够空间的两种可能情况。我们还学习了如何控制堆大小限制来减少这种错误。

接下来，了解更多关于运行时的[潜在内存问题以及如何识别它们](/web/20221126230938/https://www.baeldung.com/java-memory-leaks)。