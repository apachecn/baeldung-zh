# Java 中的弱引用

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-weak-reference>

## 1。概述

在本文中，我们将了解 Java 语言中弱引用的概念。

我们将解释这些是什么，它们的用途，以及如何正确使用它们。

## 2。弱引用

当弱引用对象弱可及时，它会被垃圾收集器清除。

弱可达性意味着一个**对象既没有强引用也没有[软引用指向它](/web/20220820044725/https://www.baeldung.com/java-soft-references)**。只有通过遍历弱引用才能到达该对象。

首先，垃圾收集器清除一个弱引用，因此 referent 不再可访问。然后，引用被放入一个引用队列(如果有关联的话),我们可以从那里获取它。

同时，以前弱可及的对象将被终结。

### 2.1.弱参考与软参考

有时，弱引用和软引用之间的区别并不明显。软引用基本上是一个大的 LRU 缓存。也就是说，**当引用对象在不久的将来很有可能被重用时，我们使用软引用**。

由于软引用充当缓存，所以即使被引用对象本身不可访问，它也可以继续被访问。事实上，当且仅当满足以下条件时，软引用才符合收集条件:

*   引用对象不是强可及的
*   软引用最近没有被访问

因此，在引用对象变得不可达之后，软引用可能在几分钟甚至几小时内可用。另一方面，弱引用只有在它的引用对象还在的时候才可用。

## 3。 **用例**

如 Java 文档所述，**弱引用最常用于实现规范化映射**。如果一个映射只包含一个特定值的实例，则称之为规范化的。它不是创建一个新对象，而是在映射中查找现有的对象并使用它。

当然，**最广为人知的使用这些引用的是 [`WeakHashMap`](https://web.archive.org/web/20220820044725/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/WeakHashMap.html) 级**。它是 [`Map`](https://web.archive.org/web/20220820044725/https://docs.oracle.com/en/java/javase/12/docs/api/java.base/java/util/Map.html) 接口的实现，其中每个键都存储为给定键的弱引用。当垃圾收集器移除一个键时，与这个键相关联的实体也被删除。

更多信息，请查看我们的 WeakHashMap 指南。

另一个可以使用它们的领域是**失效的监听器问题**。

发布者(或主题)持有对所有订阅者(或侦听器)的强引用，以通知他们所发生的事件。当听众无法成功退订发布者时，问题就出现了。

因此，侦听器不能被垃圾收集，因为对它的强引用对于发布者仍然是可用的。因此，可能会发生内存泄漏。

这个问题的解决方案可以是一个对观察者持有弱引用的主题，允许对前者进行垃圾收集，而不需要取消订阅(注意，这不是一个完整的解决方案，它引入了一些其他问题，这些问题在这里没有涉及)。

## 4。使用弱引用

弱引用由 [`java.lang.ref.WeakReference`](https://web.archive.org/web/20220820044725/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/lang/ref/WeakReference.html) 类表示。我们可以通过传递一个 referent 作为参数来初始化它。可选地，我们可以提供一个 [`java.lang.ref.ReferenceQueue`](https://web.archive.org/web/20220820044725/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/lang/ref/ReferenceQueue.html) :

```java
Object referent = new Object();
ReferenceQueue<Object> referenceQueue = new ReferenceQueue<>();

WeakReference weakReference1 = new WeakReference<>(referent);
WeakReference weakReference2 = new WeakReference<>(referent, referenceQueue); 
```

引用的被引用对象可以通过 [`get`](https://web.archive.org/web/20220820044725/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/lang/ref/Reference.html#get()) 方法获取，通过 [`clear`](https://web.archive.org/web/20220820044725/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/lang/ref/Reference.html#clear()) 方法手动移除:

```java
Object referent2 = weakReference1.get();
weakReference1.clear(); 
```

这种参考的安全工作模式与[软参考](/web/20220820044725/https://www.baeldung.com/java-soft-references)相同:

```java
Object referent3 = weakReference2.get();
if (referent3 != null) {
    // GC hasn't removed the instance yet
} else {
    // GC has cleared the instance
}
```

## 5。结论

在这个快速教程中，我们了解了 Java 中弱引用的低级概念——并重点关注了使用这些概念的最常见场景。