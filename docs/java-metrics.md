# 用 Java 监控磁盘使用和其他指标

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-metrics>

## 1.概观

在这个快速教程中，我们将讨论如何在 Java 中监控关键指标。我们将关注**磁盘空间、内存使用和线程数据——仅使用核心 Java API**。

在我们的第一个例子中，我们将利用 [`File`](https://web.archive.org/web/20220628155531/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/io/File.html) 类来查询特定的磁盘信息。

然后，我们将深入到 [`ManagementFactory`](https://web.archive.org/web/20220628155531/https://docs.oracle.com/en/java/javase/11/docs/api/java.management/java/lang/management/ManagementFactory.html) 类来分析内存使用和处理器信息。

最后，我们将谈到**如何使用 Java profiler**在运行时监控这些关键指标。

## 2.`File`类简介

简单地说， **`File`类代表一个文件**或目录的抽象。它可以用来**获取关于文件系统的关键信息，并维护关于文件路径的** **操作系统独立性**。在本教程中，我们将使用这个类来检查 Windows 和 Linux 机器上的根分区。

## 3.`ManagementFactory`

**Java 提供了`ManagementFactory `类作为工厂，用于获取包含关于 JVM** 的 **特定信息的受管 bean**(MX beans)**。我们将在下面的代码示例中检查两个:**

### 3.1.`MemoryMXBean`

**MemoryMXBean 代表 JVM 内存系统的管理接口**。在运行时，JVM 创建这个接口的单个实例，我们可以使用`ManagementFactory`的`getMemoryMXBean()`方法来检索它。

### 3.2.`ThreadMXBean`

与`MemoryMXBean`类似，`ThreadMXBean`是 JVM 线程系统的管理接口。可以使用`getThreadMXBean()`方法调用它，它保存关于线程的关键数据。

在下面的例子中，我们将使用`ThreadMXBean`来获取 JVM 的 **`ThreadInfo `类——它包含了关于在 JVM 上运行的线程的特定信息。**

## 3.监控磁盘使用情况

在这个代码示例中，我们将使用 File 类来包含关于分区的关键信息。以下示例将返回 Windows 计算机上 C:驱动器的可用空间、总空间和可用空间:

```
File cDrive = new File("C:");
System.out.println(String.format("Total space: %.2f GB",
  (double)cDrive.getTotalSpace() /1073741824));
System.out.println(String.format("Free space: %.2f GB", 
  (double)cDrive.getFreeSpace() /1073741824));
System.out.println(String.format("Usable space: %.2f GB", 
  (double)cDrive.getUsableSpace() /1073741824)); 
```

类似地，我们可以为 Linux 机器的**根目录返回相同的信息:**

```
File root = new File("/");
System.out.println(String.format("Total space: %.2f GB", 
  (double)root.getTotalSpace() /1073741824));
System.out.println(String.format("Free space: %.2f GB", 
  (double)root.getFreeSpace() /1073741824));
System.out.println(String.format("Usable space: %.2f GB", 
  (double)root.getUsableSpace() /1073741824)); 
```

上面的代码打印出已定义文件的总空间、空闲空间和可用空间。默认情况下，上述方法提供字节数。我们已经将这些字节转换成千兆字节，使结果更易于阅读。

## 4.监控内存使用情况

我们现在将使用 **`ManagementFactory `类来** **通过调用`MemoryMXBean`** 查询 JVM 可用的内存。

在这个例子中，我们将主要关注堆内存的查询。值得注意的是，非堆内存也可以使用`MemoryMXBean:`进行查询

```
MemoryMXBean memoryMXBean = ManagementFactory.getMemoryMXBean();
System.out.println(String.format("Initial memory: %.2f GB", 
  (double)memoryMXBean.getHeapMemoryUsage().getInit() /1073741824));
System.out.println(String.format("Used heap memory: %.2f GB", 
  (double)memoryMXBean.getHeapMemoryUsage().getUsed() /1073741824));
System.out.println(String.format("Max heap memory: %.2f GB", 
  (double)memoryMXBean.getHeapMemoryUsage().getMax() /1073741824));
System.out.println(String.format("Committed memory: %.2f GB", 
  (double)memoryMXBean.getHeapMemoryUsage().getCommitted() /1073741824)); 
```

上面的示例分别返回初始、已用、最大和提交的内存。下面简单解释一下[是什么意思](https://web.archive.org/web/20220628155531/https://docs.oracle.com/en/java/javase/11/docs/api/java.management/java/lang/management/MemoryUsage.html):

*   Initial:JVM 在启动时向操作系统请求的初始内存
*   used:JVM 当前使用的内存量
*   maximum 可用的最大内存。如果达到这个限制，可能会抛出一个`[OutOfMemoryException](https://web.archive.org/web/20220628155531/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/lang/OutOfMemoryError.html) `
*   Committed:保证 JVM 可用的内存量

## 5.CPU 使用率

接下来，我们将使用 **`ThreadMXBean`来获得`ThreadInfo`** 对象的完整列表，并通过**查询它们来获得关于 JVM 上运行的** **当前线程**的 **有用信息。**

```
ThreadMXBean threadMXBean = ManagementFactory.getThreadMXBean();

for(Long threadID : threadMXBean.getAllThreadIds()) {
    ThreadInfo info = threadMXBean.getThreadInfo(threadID);
    System.out.println("Thread name: " + info.getThreadName());
    System.out.println("Thread State: " + info.getThreadState());
    System.out.println(String.format("CPU time: %s ns", 
      threadMXBean.getThreadCpuTime(threadID)));
  } 
```

首先，代码使用`getAllThreadIds `方法获得当前线程的列表。然后，对于每个线程，它输出线程的名称和状态，后跟该线程的 CPU 时间(以纳秒为单位)。

## 6.使用分析器监控指标

最后，值得一提的是**我们可以在不使用任何 Java 代码的情况下监控这些关键指标**。Java Profilers 密切监视 JVM 级别的关键构造和操作，并提供对内存、线程等的实时分析。

VisualVM 就是这样一个 Java 分析器的例子，从 Java 6 开始，它就和 JDK 捆绑在一起了。许多集成开发环境(IDE)都包含插件，在开发新代码时利用分析器。你可以在这里了解更多关于 Java Profilers 和 VisualVM [的信息。](/web/20220628155531/https://www.baeldung.com/java-profilers)

## 7.结论

在本文中，我们谈到了使用核心 Java APIs 来查询关于磁盘使用、内存管理和线程信息的关键信息。

我们已经查看了多个使用`File `和`ManagmentFactory `类来获取这些指标的例子。