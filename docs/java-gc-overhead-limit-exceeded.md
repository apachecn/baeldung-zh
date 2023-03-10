# OutOfMemoryError 错误:超出了 GC 开销限制

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-gc-overhead-limit-exceeded>

## 1。概述

简单地说，当对象不再被使用时，JVM 负责释放内存。这个过程叫做垃圾收集( [GC](/web/20220921234532/https://www.baeldung.com/jvm-garbage-collectors) )。

`GC Overhead Limit Exceeded` 错误是来自`java.lang.OutOfMemoryError` 系列的一个错误，它表示资源(内存)耗尽。

在这个快速教程中，我们将看看是什么导致了`java.lang.OutOfMemoryError: GC Overhead Limit Exceeded` 错误，以及如何解决它。

## 2。GC 开销限制超出错误

`OutOfMemoryError` 是`java.lang.VirtualMachineError`的子类。当 JVM 遇到与利用资源相关的问题时，就会抛出这个问题。更具体地说，**当 JVM 花费太多时间执行垃圾收集**并且只能回收很少的堆空间时，就会出现错误。

根据 Java 文档，默认情况下，如果 Java 进程花费超过 98%的时间进行 GC，并且每次运行只恢复不到 2%的堆，JVM 就会抛出这个错误。换句话说，这意味着我们的应用程序已经耗尽了几乎所有的可用内存，垃圾收集器花费了太多的时间试图清理它，并且反复失败。

在这种情况下，用户会感觉应用程序非常慢。某些通常在几毫秒内完成的操作需要更多的时间来完成。这是因为 CPU 将其全部容量用于垃圾收集，因此无法执行任何其他任务。

## 3。动作错误

我们来看一段抛出`java.lang.OutOfMemoryError: GC Overhead Limit Exceeded`的代码。

例如，我们可以通过在一个无限循环中添加键值对来实现这一点:

```java
public class OutOfMemoryGCLimitExceed {
    public static void addRandomDataToMap() {
        Map<Integer, String> dataMap = new HashMap<>();
        Random r = new Random();
        while (true) {
            dataMap.put(r.nextInt(), String.valueOf(r.nextInt()));
        }
    }
}
```

当调用这个方法时，JVM 参数为`-Xmx100m -XX:+UseParallelGC` (Java 堆大小设置为 100 MB，GC 算法为 ParallelGC)，我们得到一个`java.lang.OutOfMemoryError: GC Overhead Limit Exceeded` 错误。为了更好地理解不同的垃圾收集算法，请查看 Oracle 的 [Java 垃圾收集基础知识](https://web.archive.org/web/20220921234532/http://www.oracle.com/webfolder/technetwork/tutorials/obe/java/gc01/index.html)教程。

通过从[项目](https://web.archive.org/web/20220921234532/https://github.com/eugenp/tutorials/tree/master/core-java-modules)的根目录运行以下命令，我们将很快得到一个`java.lang.OutOfMemoryError: GC Overhead Limit Exceeded` 错误:

```java
mvn exec:exec
```

还应该注意，在某些情况下，我们可能会在遇到`GC Overhead Limit Exceeded` 错误之前遇到堆空间错误。

## 4。解决 GC 开销限制超出错误

理想的解决方案是通过检查代码中的任何内存泄漏来找到应用程序的潜在问题。

这些问题需要解决:

*   应用程序中占据大部分堆的对象是什么？
*   这些对象是在源代码的哪个部分被分配的？

我们也可以使用自动化的图形工具，如 [JConsole](https://web.archive.org/web/20220921234532/https://docs.oracle.com/en/java/javase/11/management/using-jconsole.html#GUID-77416B38-7F15-4E35-B3D1-34BFD88350B5) ，它有助于检测包括`java.lang.OutOfMemoryErrors`在内的代码中的性能问题。

最后一种方法是通过改变 JVM 启动配置来增加堆的大小。

例如，这为 Java 应用程序提供了 1 GB 的堆空间:

```java
java -Xmx1024m com.xyz.TheClassName
```

但是，如果实际应用程序代码中存在内存泄漏，这并不能解决问题。相反，我们将只是推迟错误。因此，更明智的做法是彻底重新评估应用程序的内存使用情况。

## 5。结论

在本文中，我们研究了`java.lang.OutOfMemoryError: GC Overhead Limit Exceeded` 及其背后的原因。

和往常一样，与本文相关的源代码可以在 GitHub 上找到[。](https://web.archive.org/web/20220921234532/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-perf)