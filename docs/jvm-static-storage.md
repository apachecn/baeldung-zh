# 静态成员的 JVM 存储

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/jvm-static-storage>

## 1.概观

在我们的日常工作中，我们往往并不关心 JVM 的内存分配。

然而，**了解 JVM 内存模型的基础知识对于性能优化和提高代码质量很有用**。

在本文中，我们将探索静态方法和成员的 JVM 存储。

## 2.JVM 的内存分类

在深入研究静态成员的内存分配之前，我们必须重新理解 JVM 的内存结构。

### 2.1.堆内存

[堆内存是所有 JVM 线程共享的运行时数据区](/web/20220524113522/https://www.baeldung.com/java-stack-heap#heap-space-in-java)，为所有类实例和数组分配内存。

Java 将堆内存分为两类——年轻一代和老一代。

JVM 在内部将年轻一代分为伊甸园和幸存者空间。同样，终身空间是老一代的官方称呼。

堆内存中一个对象的生命周期由一个自动内存管理系统[管理，这个系统被称为垃圾收集器T3。](/web/20220524113522/https://www.baeldung.com/jvm-garbage-collectors)

因此，**垃圾收集器可以自动释放一个对象，或者将它移动到堆内存的各个部分**(年轻到老的一代)。

### 2.2.非堆内存

非堆内存主要由**组成，它是一个存储类结构、字段、方法数据和方法/构造函数代码**的方法区域。

与堆内存类似，所有 JVM 线程都可以访问方法区。

方法区域，也称为永久生成(PermGen ),从逻辑上来说被认为是堆内存的一部分，尽管 JVM 的简单实现可能选择不对它进行垃圾收集。

不过， **[Java 8 去掉了 PermGen 空间](https://web.archive.org/web/20220524113522/https://openjdk.java.net/jeps/122)，引入了一个新的原生内存空间，名为 Metaspace** 。

### 2.3.高速缓冲存储器

JVM 为编译和存储本机代码(如 JVM 内部结构和 JIT 编译器生成的本机代码)保留高速缓存区域。

## 3.Java 8 之前的静态成员存储

在 Java 8 之前， **PermGen 像静态方法和静态变量一样存储静态成员**。此外，PermGen 还存储被拘留的字符串。

换句话说，PermGen 空间存储变量及其技术值，这些值可以是原语或引用。

## 4.Java 8 及更高版本的静态成员存储

正如我们已经讨论过的，在 Java 8 中， [PermGen 空间被替换为元空间，导致静态成员的内存分配发生变化。](/web/20220524113522/https://www.baeldung.com/java-permgen-metaspace)

从 Java 8 开始，Metaspace 只存储类元数据，**堆内存保存静态成员**。此外，堆内存还为被拘留的字符串提供存储。

## 5.结论

在这篇短文中，我们探讨了静态成员的 JVM 存储。

首先，我们快速看了一下 JVM 的内存模型。然后，我们讨论了 Java 8 之前和之后静态成员的 JVM 存储。

简单地说，我们知道静态成员在 Java 8 之前是 PermGen 的一部分。然而，**从 Java 8 开始，它们就成了堆内存**的一部分。