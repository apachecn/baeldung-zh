# 历史队列介绍

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-chronicle-queue>

## 1。概述

历史记录队列使用内存映射文件保存每一条消息。这允许我们在进程间共享消息。

它将数据直接存储到堆外内存中，因此没有垃圾收集的开销。它旨在为高性能应用提供低延迟的消息框架。

在这篇简短的文章中，我们将研究一组基本的操作。

## 2。Maven 依赖关系

我们需要在依赖关系之后添加[:](https://web.archive.org/web/20221128111804/https://search.maven.org/classic/#search%7Cgav%7C1%7Cg%3A%22net.openhft%22%20AND%20a%3A%22chronicle%22)

```
<dependency>
    <groupId>net.openhft</groupId>
    <artifactId>chronicle</artifactId>
    <version>3.6.4</version>
</dependency>
```

我们总是可以通过之前提供的链接查看 Maven Central 托管的最新版本。

## 3。积木

历史记录队列有三个概念特征:

*   **摘录—**是一个数据容器
*   **Appender—**Appender 用于写入数据
*   **Trailer—**用于顺序读取数据

我们将使用`Chronicle`接口为`read-write`操作保留部分内存。

以下是创建实例的示例代码:

```
File queueDir = Files.createTempDirectory("chronicle-queue").toFile();
Chronicle chronicle = ChronicleQueueBuilder.indexed(queueDir).build();
```

我们将需要一个基本目录，队列将在其中保存内存映射文件中的记录。

`ChronicleQueueBuilder` 类提供了不同类型的队列。在这种情况下，我们使用`IndexedChronicleQueue`，而`h`使用顺序索引来维护队列中记录的内存偏移量。

## 4。写入队列

要将项目写入队列，我们需要使用`Chronicle`实例创建一个`ExcerptAppender` 类的对象。以下是将消息写入队列的示例代码:

以下是将消息写入队列的示例代码:

```
ExcerptAppender appender = chronicle.createAppender();
appender.startExcerpt();

String stringVal = "Hello World";
int intVal = 101;
long longVal = System.currentTimeMillis();
double doubleVal = 90.00192091d;

appender.writeUTF(stringValue);
appender.writeInt(intValue);
appender.writeLong(longValue);
appender.writeDouble(doubleValue);
appender.finish();
```

创建完 appender 后，我们将使用一个`startExcerpt`方法启动 appender。它启动一个默认消息容量为`128K`的`Excerpt`。我们可以使用过载版本的`startExcerpt`来提供定制的容量。

一旦开始，我们可以使用库提供的各种各样的写方法将任何文字或对象值写入队列。

最后，当我们完成写入时，我们将完成摘录，将数据保存到队列中，稍后保存到磁盘。

## 5。从队列中读取

使用`ExcerptTrailer`实例可以很容易地读取队列的值`from`。

它就像我们在 Java 中用来遍历集合的迭代器。

让我们从队列中读取值:

```
ExcerptTailer tailer = chronicle.createTailer();
while (tailer.nextIndex()) {
    tailer.readUTF();
    tailer.readInt();
    tailer.readLong();
    tailer.readDouble();
}
tailer.finish();
```

在创建预告片之后，我们使用`nextIndex` 方法来检查是否有新的摘录要读取。

一旦`ExcerptTailer`有一个新的`Excerpt`要读取，我们就可以使用一系列用于文字和对象类型值的 *read* 方法从它那里读取消息。

最后，我们用`finish` API 完成读取。

## 6。结论

在本教程中，我们简要介绍了历史队列及其构建块。我们看到了如何创建队列、写入和读取数据。使用它有很多好处，包括低延迟、持久的进程间通信(IPC)以及没有垃圾收集开销。

该解决方案通过内存映射文件提供数据持久性，不会丢失数据。它还允许来自多个进程的并发读写；但是，写入是同步处理的。

和往常一样，所有代码片段都可以在 GitHub 上找到[。](https://web.archive.org/web/20221128111804/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-9)