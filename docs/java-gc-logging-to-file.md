# 用 Java 将垃圾收集日志记录到文件中

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-gc-logging-to-file>

## 1.概观

垃圾收集是 Java 编程语言的奇迹，它为我们提供了自动内存管理。垃圾收集隐藏了必须手动分配和释放内存的细节。虽然这种机制很神奇，但有时它并不按照我们想要的方式工作。在本教程中，我们将探索 Java 的垃圾收集统计数据的**日志选项，并发现如何**将这些统计数据重定向到文件**。**

## 2.Java 8 和更早版本中的 GC 日志标志

首先，让我们探索一下 Java 9 之前的 Java 版本中与 GC 日志记录相关的 JVM 标志。

### 2.1.`-XX:+PrintGC`

`-XX:+PrintGC`标志是`-verbose:gc`的别名，**开启基本的 GC 日志记录**。在这种模式下，为每个年轻一代和每个完整一代系列打印一行。现在让我们把注意力转向提供详细的 GC 信息。

### 2.2.`-XX:+PrintGCDetails`

类似地，我们使用标志`-XX:+PrintGCDetails` 来**激活详细的 GC 日志记录**而不是`-XX:+PrintGC`。

请注意，`-XX:+PrintGCDetails`的输出根据使用的 GC 算法而变化。

接下来，我们将看看用日期和时间信息来注释日志。

### 2.3.`-XX:+PrintGCDateStamps`和`-XX:+PrintGCTimeStamps`

我们可以**通过分别利用标志`-XX:+PrintGCDateStamps`和`-XX:+PrintGCTimeStamps`将日期和时间信息添加到我们的 GC 日志**中。

首先，`-XX:+PrintGCDateStamps `将日志条目的日期和时间添加到每一行的开头。

其次，`-XX:PrintGCTimeStamps`向日志的每一行添加一个时间戳，详细记录自 JVM 启动以来经过的时间(以秒为单位)。

### 2.4.`-Xloggc`

最后，我们来到**，将 GC 日志重定向到一个文件**。该标志使用语法`-Xloggc:file`将可选文件名作为参数，如果没有文件名，GC 日志将被写入标准输出。

此外，该标志还为我们设置了`-XX:PrintGC`和`-XX:PrintGCTimestamps`标志。让我们看一些例子:

如果我们想将 GC 日志写入标准输出，我们可以运行:

```java
java -cp $CLASSPATH -Xloggc mypackage.MainClass
```

或者将 GC 日志写入文件，我们可以运行:

`java -cp $CLASSPATH -Xloggc:/tmp/gc.log mypackage.MainClass`

## 3.Java 9 和更高版本中的 GC 日志标志

在 Java 9+中，`-verbose:gc`的别名`-XX:PrintGC`已经被弃用，取而代之的是**统一日志选项、`-Xlog`、**。上面提到的所有其他 GC 标志在 Java 9+中仍然有效。这个新的日志选项允许我们**指定应该显示哪些消息，设置日志级别，并重定向输出**。

我们可以运行下面的命令来查看日志级别、日志装饰器和标记集的所有可用选项:

```java
java -Xlog:logging=debug -version 
```

例如，如果我们想将所有 GC 消息记录到一个文件中，我们可以运行:

```java
java -cp $CLASSPATH -Xlog:gc*=debug:file=/tmp/gc.log mypackage.MainClass
```

此外，这个新的统一日志标记是可重复的，因此，例如，**可以将所有 GC 消息记录到标准输出和文件**:

```java
java -cp $CLASSPATH -Xlog:gc*=debug:stdout -Xlog:gc*=debug:file=/tmp/gc.log mypackage.MainClass
```

## 4.结论

在本文中，我们展示了如何在 Java 8 和 Java 9+中记录垃圾收集输出，包括如何将输出重定向到文件。