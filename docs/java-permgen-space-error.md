# 处理“Java . lang . out of memory Error:perm gen space”错误

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-permgen-space-error>

## 1.概观

PermGen(永久生成)是为运行基于 JVM 的应用程序而分配的一块特殊内存。 [PermGen 错误](/web/20221016100023/https://www.baeldung.com/java-memory-leaks)是来自`java.lang.OutOfMemoryError`家族的一个错误，它表示资源(内存)耗尽。

在这个快速教程中，我们将看看是什么导致了`java.lang.OutOfMemoryError` : Permgen 空间错误以及如何解决它。

## 2.Java 内存类型

JVM 使用两种类型的内存:堆栈和堆。堆栈仅用于存储基本类型和对象地址。相反，堆包含对象的值。**当我们谈论内存错误时，我们总是指堆。事实上，PermGen 是堆内存的一部分，但是与 JVM 的主内存是分开的，处理方式也不同。**需要掌握的最重要的概念是，堆中可能还有大量可用空间，但 perm gen 内存仍然不足。

PermGen 的主要作用域是存储 Java 应用程序运行时的静态内容:尤其是，它包含静态方法、静态变量、对静态对象的引用和类文件。

## 3.`java.lang.OutOfMemoryError`:借过

简单地说，当为 PermGen 分配的空间不再能够存储对象时，就会出现此错误。这是因为 PermGen 不是动态分配的，并且具有固定的最大容量。64 位版本 JVM 的默认大小是 82 Mb，旧的 32 位 JVM 的默认大小是 64 Mb。

PemGen 耗尽的最常见原因之一是与类加载器相关的内存泄漏。事实上，PermGen 包含类文件，类加载器负责加载 Java 类。类加载器问题在应用服务器中很常见，在应用服务器中，多个类加载器被实例化以实现各种应用程序的独立部署。

当应用程序被取消部署，并且服务器容器保存一个或多个类的引用时，问题就出现了。如果发生这种情况，类装入器本身就不能被垃圾收集，从而使 PermGen 的类文件充满了内存。PermGen 崩溃的另一个常见原因是应用程序线程在应用程序取消部署后继续运行，从而维护了内存中分配的几个对象。

## 4.处理错误

### 4.1.调整正确的 JVM 参数

关于有限的内存空间，首先要做的是尽可能增加空间。通过使用特定的标志，可以增加 PermGen 空间的默认大小。拥有数千个类或大量 Java 字符串的大型应用程序通常需要更大的 PermGen 空间。通过使用 [JVM 参数](/web/20221016100023/https://www.baeldung.com/jvm-parameters)–`XX:MaxPermSize`可以指定一个更大的空间分配给这个内存区域。

**既然我们提到了 JVM 标志，也值得提一下一个不常用的标志，它会触发这个错误。**–`Xnoclassgc`JVM 参数，当在 JVM 开始时指定时，从要丢弃的实体列表中显式删除类文件。在应用服务器和现代框架中，每个应用程序的生命周期要加载和卸载类数千次，这会导致 PermGen 空间非常快地耗尽。

在旧版本的 Java 中，类是堆的永久组成部分，这意味着一旦被加载，它们就保留在内存中。通过指定`CMSClassUnloadingEnabled`(对于 Java 1.5 或 `CMSPermGenSweepingEnabled`对于 Java 1.6) JVM 参数，可以启用类的垃圾收集。如果我们碰巧使用 Java 1.6，`UseConcMarkSweepGC`也必须设置为`true.` ，否则`CMSClassUnloadingEnabled`参数将被忽略。

### 4.2.升级到 JVM 8+版本

修复这种错误的另一种方法是升级到 Java 的新版本。从 Java 版本 8 开始， [Permgen 已经完全被 Metaspace](/web/20221016100023/https://www.baeldung.com/java-permgen-metaspace) 所取代，它拥有一个可自动调整大小的空间和一个支持清理死类的高级特性。

### 4.3.堆积分析

应该注意到，在内存泄漏的情况下，没有一种解决方案是足够的。记忆会结束，不管它有多大。即使 Metaspace 的可用内存也是有限的。深度堆分析有时是唯一的解决方案，可以使用 VisualGC 或 JPROFILER 之类的工具进行。

## 5.摘要

在这篇快速的文章中，我们已经看到了 PermGen 内存的用途以及它与堆内存的主要区别。接下来，我们已经看到了`java.lang.OutOfMemoryError` : Permgen 错误意味着什么，以及在什么特殊情况下会被触发。在上一节中，我们重点介绍了在尝试解决这个特殊问题时可以使用的各种解决方案。