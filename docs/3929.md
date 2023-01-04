# Java 中的 Permgen 与元空间

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-permgen-metaspace>

## 1。简介

在这个快速教程中，**我们将研究 Java 环境中 PermGen 和元空间内存区域**之间的差异。

重要的是要记住，从 Java 8 开始，元空间取代了 perm gen——带来了一些实质性的变化。

## 2 .准许〔t1〕

**PermGen(永久生成)是从主存堆中分离出来的特殊堆空间**。

JVM 跟踪 PermGen 中加载的类元数据。此外，JVM 将所有静态内容存储在这个内存部分。这包括所有静态方法、原始变量和对静态对象的引用。

此外，**它包含关于字节码、名称和 JIT 信息的数据**。在 Java 7 之前，字符串池也是这种内存的一部分。在我们的[报道](/web/20220708032032/https://www.baeldung.com/java-string-pool)中列出了固定池大小的缺点。

32 位 JVM 的默认最大内存大小是 64 MB，64 位版本是 82 MB。

但是，我们可以使用 JVM 选项更改默认大小:

*   `-XX:PermSize=[size]`是 PermGen 空间的初始或最小大小
*   `-XX:MaxPermSize=[size]`是最大尺寸

最重要的是， **Oracle 在 JDK 8 版本中完全移除了这个内存空间。**因此，如果我们在 Java 8 和更新版本中使用这些调优标志，我们将得到以下警告:

```
>> java -XX:PermSize=100m -XX:MaxPermSize=200m -version
OpenJDK 64-Bit Server VM warning: Ignoring option PermSize; support was removed in 8.0
OpenJDK 64-Bit Server VM warning: Ignoring option MaxPermSize; support was removed in 8.0
...
```

**由于内存大小有限，PermGen 参与生成了著名的`OutOfMemoryError`** 。简单地说，[类加载器](/web/20220708032032/https://www.baeldung.com/java-classloaders)没有被正确地垃圾收集，结果产生了内存泄漏。

因此，我们收到一个[内存空间错误](/web/20220708032032/https://www.baeldung.com/java-gc-overhead-limit-exceeded)；这主要发生在开发环境中创建新的类装入器时。

## 3。元空间

简单来说，元空间是一个新的内存空间——从 Java 8 版本开始；**它取代了旧的 PermGen 内存空间**。最显著的区别是它如何处理内存分配。

具体来说，**这个原生内存区域默认自动增长**。

我们也有新的标志来调整内存:

*   `MetaspaceSize`和`MaxMetaspaceSize –`我们可以设置元空间的上限。
*   `MinMetaspaceFreeRatio – `是在[垃圾收集](/web/20220708032032/https://www.baeldung.com/jvm-garbage-collectors)之后空闲的类元数据容量的最小百分比
*   `MaxMetaspaceFreeRatio `–为避免空间量减少，垃圾收集后可用的类元数据容量的最大百分比

此外，垃圾收集过程也从这一变化中获得了一些好处。现在，一旦类元数据的使用达到其最大元空间大小，垃圾收集器就会自动触发死类的清理。

因此，**有了这个改进，JVM 减少了得到`OutOfMemory`错误**的机会。

尽管有了所有这些改进，我们仍然需要监控和[调优元空间](/web/20220708032032/https://www.baeldung.com/jvm-parameters)以避免内存泄漏。

## 4。总结

在这篇快速的文章中，我们简要描述了 PermGen 和元空间内存区域。此外，我们解释了它们之间的主要区别。

PermGen 在 JDK 7 和更老的版本中仍然存在，但是 Metaspace 为我们的应用程序提供了更加灵活和可靠的内存使用。