# System.gc 指南()

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-system-gc>

## 1.概观

在本教程中，我们将研究位于`java.lang`包中的`System.gc()` 方法。

众所周知，显式调用`System.gc()`是一种不好的做法。让我们试着理解为什么调用这个方法可能有用，以及是否有任何用例。

## 2.碎片帐集

当有指示时，Java 虚拟机决定执行垃圾收集。这些指示在不同的 GC 实现中有所不同。它们基于不同的试探法。然而，有几个时刻 GC 肯定会被执行:

*   旧代(终身空间)已满，这将触发主要/完整垃圾收集
*   新一代(Eden+survivir 0+survivir 1 空间)已满，这将触发较小的垃圾收集

唯一独立于 GC 实现的是对象被垃圾收集的资格。

现在，我们来看看`System.gc()`方法本身。

## 3.`System.gc()`

方法的调用很简单:

```java
System.gc()
```

甲骨文官方文档称:

> 调用`gc`方法`suggests`，Java 虚拟机花费精力回收未使用的对象，以使它们当前占用的内存可用于快速重用。

**不能保证实际的垃圾收集器会被触发**。

`System.gc()` [触发重大 GC](https://web.archive.org/web/20220926184331/https://www.oracle.com/java/technologies/javase/gc-tuning-6.html#other_considerations) 。因此，根据您的垃圾收集器实现，在停止世界阶段花费一些时间是有风险的。结果，**我们有了一个不可靠的工具，潜在的严重性能损失**。

显式垃圾收集调用的存在对每个人来说都是一个严重的危险信号。

我们可以通过使用`-XX:DisableExplicitGC` JVM 标志来阻止`System.gc()`做任何工作。

### 3.1.性能调整

值得注意的是，就在抛出`OutOfMemoryError,`之前，JVM 将执行一次完整的 GC。因此，显式调用 **`System.gc() `不会将我们从失败**中拯救出来。

如今的垃圾收集者真的很聪明。他们了解内存使用和其他统计数据，能够做出正确的决策。因此，我们应该信任他们。

如果出现内存问题，我们可以更改一系列设置来调整我们的应用——从选择不同的垃圾收集器开始，通过设置所需的应用时间/GC 时间比率，最后，为内存段设置固定的大小。

还有一些方法可以[减轻由显式调用](https://web.archive.org/web/20220926184331/https://docs.oracle.com/javase/8/docs/technotes/guides/vm/cms-6.html)引起的完全 GC 的影响。我们可以使用其中一个标志:

```java
-XX:+ExplicitGCInvokesConcurrent
```

或者:

```java
-XX:+ExplicitGCInvokesConcurrentAndUnloadsClasses
```

如果我们真的希望我们的应用程序正常工作，我们应该解决真正的底层内存问题。

在下一章，我们将看到一个显式调用`System.gc()` 有用的实际例子。

## 4.用法示例

### 4.1.方案

我们来写一个测试 app。**我们希望找到一种情况，这时调用`System.gc()`可能会有用**。

次要垃圾收集比主要垃圾收集发生得更频繁。所以，我们可能应该关注后者。如果一个对象在几次收集中“幸存”下来，并且仍然可以从 GC 根访问，那么这个对象就被移动到永久空间。

让我们想象一下，我们有一大堆活了一段时间的物品。然后，在某个时候，我们清除对象集合。也许现在是跑`System.gc()`的好时机？

### 4.2.演示应用程序

我们将创建一个简单的控制台应用程序来模拟这个场景:

```java
public class DemoApplication {

    private static final Map<String, String> cache = new HashMap<String, String>();

    public static void main(String[] args) {
        Scanner scanner = new Scanner(System.in);

        while (scanner.hasNext()) {
            final String next = scanner.next();
            if ("fill".equals(next)) {
                for (int i = 0; i < 1000000; i++) { 
                    cache.put(randomUUID().toString(), randomUUID().toString()); 
                } 
            } else if ("invalidate".equals(next)) {
                cache.clear();
            } else if ("gc".equals(next)) {
                System.gc();
            } else if ("exit".equals(next)) {
                System.exit(0);
            } else {
                System.out.println("unknown");
            }
        }
    }
}
```

### 4.3.运行演示

让我们用几个附加标志来运行我们的应用程序:

```java
-XX:+PrintGCDetails -Xloggc:gclog.log -Xms100M -Xmx500M -XX:+UseConcMarkSweepGC
```

记录 GC 信息需要前两个标志。接下来的两个标志是设置初始堆大小和最大堆大小。我们希望保持较低的堆大小，以迫使 GC 更加活跃。最后，我们决定使用 CMS——并发标记和清除垃圾收集器。是时候运行我们的应用程序了！

首先，让我们**尝试填补任期空间**。类型`fill.`

我们可以[调查我们的`gclog.log`](/web/20220926184331/https://www.baeldung.com/java-verbose-gc) 文件，看看发生了什么。我们会看到大约 15 个系列。为单个集合记录的行如下所示:

```java
197.057: [GC (Allocation Failure) 197.057: [ParNew: 67498K->40K(75840K), 0.0016945 secs] 
  168754K->101295K(244192K), 0.0017865 secs] [Times: user=0.01 sys=0.00, real=0.00 secs] secs]
```

如我们所见，内存已满。

接下来，让我们**通过键入`gc`来强制`System.gc()`** 。我们可以看到内存使用没有显著变化:

```java
238.810: [Full GC (System.gc()) 238.810: [CMS: 101255K->101231K(168352K); 0.2634318 secs] 
  120693K->101231K(244192K), [Metaspace: 32186K->32186K(1079296K)], 0.2635908 secs] 
  [Times: user=0.27 sys=0.00, real=0.26 secs]
```

再运行几次后，我们会看到内存大小保持在相同的水平。

让我们通过键入`invalidate`来**清除缓存**。我们应该不会在`gclog.log`文件中看到更多的日志行。

我们可以尝试多次填充缓存，但是没有发生 GC。这是一个我们可以智取垃圾收集器的时刻。现在，在强制 GC 之后，我们将看到这样一行:

```java
262.124: [Full GC (System.gc()) 262.124: [CMS: 101523K->14122K(169324K); 0.0975656 secs] 
  103369K->14122K(245612K), [Metaspace: 32203K->32203K(1079296K)], 0.0977279 secs]
  [Times: user=0.10 sys=0.00, real=0.10 secs]
```

我们已经释放了数量惊人的内存！但是现在真的有必要吗？怎么回事？

根据这个例子，当我们释放大对象或使缓存无效时，调用`System.gc() `可能看起来很诱人。

## 5.其他用法

显式调用`System.gc()`方法可能有用的原因很少。

一个可能的原因是**在服务器启动后清理内存**——我们正在启动一个做了大量准备工作的服务器或应用程序。之后还有很多对象要敲定。然而，做好准备后的清洁不应该是我们的责任。

另一个是**内存泄漏分析** `— `这更像是一种调试实践，而不是我们希望保留在产品代码中的东西。调用`System.gc()`并看到堆空间仍然很高可能是一个[内存泄漏](/web/20220926184331/https://www.baeldung.com/java-memory-leaks)的指示。

## 6.摘要

在本文中，我们研究了`System.gc() `方法以及它在什么情况下可能有用。

当涉及到我们应用程序的正确性时，我们不应该依赖它。 GC 在大多数情况下都比我们聪明，万一出现什么内存问题，我们应该考虑调优虚拟机，而不是进行这样的显式调用。

像往常一样，本文中使用的代码可以在 GitHub 上找到[。](https://web.archive.org/web/20220926184331/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-jvm)