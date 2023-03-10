# Java 中的软引用

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-soft-references>

## 1。概述

在这篇简短的文章中，我们将讨论 Java 中的软引用。

我们将解释它们是什么，为什么我们需要它们，以及如何创建它们。

## 2。什么是软引用？

响应于存储器需求，垃圾收集器可以清除软引用对象(或软可及对象)。一个软可及的对象没有指向它的强引用。

当垃圾收集器被调用时，它开始遍历堆中的所有元素。GC 将引用类型对象存储在一个特殊的队列中。

检查完堆中的所有对象后，GC 通过从上述队列中删除对象来确定应该删除哪些实例。

这些规则在不同的 JVM 实现之间有所不同，但是文档声明在 JVM 抛出`OutOfMemoryError.` 之前，保证清除所有对软可及对象的软引用

但是，不保证清除软引用的时间，也不保证清除一组对不同对象的这种引用的顺序。

通常，JVM 实现会选择清除最近创建的引用或最近使用的引用。

软可及对象将在最后一次被引用后的一段时间内保持活动状态。默认值是堆中每兆空闲字节一秒钟的生存期。该值可以使用*[-XX:SoftRefLRUPolicyMSPerMB](https://web.archive.org/web/20221122092843/http://www.oracle.com/technetwork/java/hotspotfaq-138619.html#gc_softrefs)*标志进行调整。

例如，要将该值更改为 2.5 秒(2500 毫秒)，我们可以使用:

```java
-XX:SoftRefLRUPolicyMSPerMB=2500
```

与弱引用相比，软引用可以有更长的生存期，因为它们会继续存在，直到需要额外的内存。

因此，如果我们需要尽可能长时间地在内存中保存对象，那么它们是更好的选择。

## 3。软引用的用例

**软引用可用于实现内存敏感型缓存**，其中内存管理是一个非常重要的因素。

只要软引用的 referent 是强可达的，也就是说–实际上正在使用，引用就不会被清除。

例如，高速缓存可以通过保存对那些条目的强引用来防止其最近使用的条目被丢弃，而将剩余的条目留给垃圾收集器自行丢弃。

## 4。使用软参照

在 Java 中，软引用由[*Java . lang . ref . soft reference*](https://web.archive.org/web/20221122092843/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/lang/ref/SoftReference.html)类表示。

我们有两个选项来初始化它。

第一种方法是只传递一个 referent:

```java
StringBuilder builder = new StringBuilder();
SoftReference<StringBuilder> reference1 = new SoftReference<>(builder);
```

第二个选项意味着传递对 [`java.lang.ref.ReferenceQueue`](https://web.archive.org/web/20221122092843/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/lang/ref/ReferenceQueue.html) 的引用以及对 referent 的引用。**引用队列旨在让我们了解垃圾收集器执行的操作。**当它决定删除这个引用的被引用对象时，它把一个引用对象附加到一个引用队列。

下面是如何用一个 *ReferenceQueue:* 初始化一个*软引用*

```java
ReferenceQueue<StringBuilder> referenceQueue = new ReferenceQueue<>();
SoftReference<StringBuilder> reference2
 = new SoftReference<>(builder, referenceQueue);
```

作为一个 [`java.lang.ref.Reference`](https://web.archive.org/web/20221122092843/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/lang/ref/Reference.html) ，它包含了 [`get`](https://web.archive.org/web/20221122092843/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/lang/ref/Reference.html#get()) 和 [`clear`](https://web.archive.org/web/20221122092843/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/lang/ref/Reference.html#clear()) 分别获取和重置一个引用对象的方法:

```java
StringBuilder builder1 = reference2.get();
reference2.clear();
StringBuilder builder2 = reference2.get(); // null 
```

每次我们使用这种类型的引用时，我们需要确保由`get`返回的 referent 存在:

```java
StringBuilder builder3 = reference2.get();
if (builder3 != null) {
    // GC hasn't removed the instance yet
} else {
    // GC has cleared the instance
}
```

## 5。结论

在本教程中，我们熟悉了软引用的概念及其用例。

此外，我们还学习了如何创建一个并以编程方式使用它。