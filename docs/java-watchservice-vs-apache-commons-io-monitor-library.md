# Java WatchService API 和 Apache Commons IO Monitor 库之间的区别

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-watchservice-vs-apache-commons-io-monitor-library>

## 1。概述

早在 Java 7 中发布 Java `WatchService` API 之前，Apache Commons IO Monitoring library 就已经解决了监视文件系统位置或目录变化的相同用例。

在本文中，我们将探讨这两个 API 之间的差异。

## 2。Maven 依赖关系

要使用 Apache Commons IO，需要在`pom`中添加以下依赖关系:

```java
<dependency>
    <groupId>commons-io</groupId>
    <artifactId>commons-io</artifactId>
    <version>2.11.0</version>
</dependency>
```

当然，手表服务是 JDK 的一部分，因此它不需要外部依赖。

## 3。特征比较

### 3.1。事件驱动处理

`WatchService` API 由操作系统触发的文件系统改变事件驱动。这种方法使应用程序不必重复轮询文件系统的变化。

另一方面，Apache Commons IO Monitor 库通过调用`File`类的`listFiles()`方法，以可配置的睡眠间隔轮询文件系统位置。这种方法会浪费 CPU 周期，尤其是在没有发生变化的情况下。

### 3.2。回调方法

`WatchService` API 不提供回调方法。相反，它提供了两种类型的轮询方法来检查新的变更事件是否可用于处理:

1.  像`poll()`(带有超时参数)和`take()`这样的阻塞方法
2.  类似`poll()`的非阻塞方法(没有超时参数)

使用阻塞方法，应用程序线程仅在新的更改事件可用时才开始处理。因此，它不需要持续轮询新事件。

这些方法的细节和用法可以在我们的文章[这里](/web/20221013193920/https://www.baeldung.com/java-nio2-watchservice)中找到。

相反，Apache Commons IO 库在`FileAlterationListener`接口上提供了回调方法，当检测到文件系统位置或目录发生变化时，就会调用这些方法。

```java
FileAlterationObserver observer = new FileAlterationObserver("pathToDir");
FileAlterationMonitor monitor = new FileAlterationMonitor(POLL_INTERVAL);
FileAlterationListener listener = new FileAlterationListenerAdaptor() {
    @Override
    public void onFileCreate(File file) {
        // code for processing creation event
    }

    @Override
    public void onFileDelete(File file) {
        // code for processing deletion event
    }

    @Override
    public void onFileChange(File file) {
        // code for processing change event
    }
};
observer.addListener(listener);
monitor.addObserver(observer);
monitor.start();
```

### 3.3。事件溢出

`WatchService` API 由操作系统事件驱动。因此，如果应用程序不能足够快地处理事件，保存事件的操作系统缓冲区就有可能溢出。在这种情况下，事件`StandardWatchEventKinds.OVERFLOW`被触发，表明一些事件在应用程序可以读取它们之前丢失或被丢弃。

这需要正确处理应用程序中的`OVERFLOW`事件，以确保应用程序能够处理任何可能触发`OVERFLOW`事件的突发变更事件。

另一方面，Commons IO 库不基于操作系统事件，因此不存在溢出问题。

在每次轮询中，观察者都会获得目录中的文件列表，并将其与前一次轮询获得的列表进行比较。

1.  如果在上一次轮询中发现了新的文件名，将在监听器上调用`onFileCreate()`
2.  如果在上次轮询中获得的文件列表中缺少在上次轮询中找到的文件名，则在监听器上调用`onFileDelete()`
3.  如果找到匹配，则检查文件的属性是否有任何变化，如最后修改日期、长度等。如果检测到变化，在监听器上调用`onFileChange()`

## 4。结论

在本文中，我们强调了这两种 API 的主要区别。

和往常一样，本文中使用的例子的完整源代码可以在我们的 GitHub 项目中获得。