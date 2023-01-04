# Epsilon GC 简介:一个无操作的实验性垃圾收集器

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/jvm-epsilon-gc-garbage-collector>

## 1.介绍

Java 11 引入了一个名为 Epsilon 的[无操作垃圾收集器](https://web.archive.org/web/20220628163455/https://openjdk.java.net/jeps/318)，它**承诺了最低的 GC 开销**。

在这个简短的教程中，我们将探索 Epsilon 是如何工作的，我们将提到常见的用例。

## 2.快速动手

让我们从弄脏手开始，带着 Epsilon GC 兜一圈！

我们首先需要一个会产生垃圾的应用程序:

```java
class MemoryPolluter {

    static final int MEGABYTE_IN_BYTES = 1024 * 1024;
    static final int ITERATION_COUNT = 1024 * 10;

    static void main(String[] args) {
        System.out.println("Starting pollution");

        for (int i = 0; i < ITERATION_COUNT; i++) {
            byte[] array = new byte[MEGABYTE_IN_BYTES];
        }

        System.out.println("Terminating");
    }
}
```

这段代码在一个循环中创建了 1mb 的数组。由于我们重复循环 10240 次，这意味着我们分配了 10gb 的内存，这可能高于可用的最大堆大小。

我们还提供了一些助手打印来查看应用程序何时终止。

**要启用 Epsilon GC，我们需要传递以下 VM 参数:**

```java
-XX:+UnlockExperimentalVMOptions -XX:+UseEpsilonGC
```

当我们运行应用程序时，我们会得到以下错误:

```java
Starting pollution
Terminating due to java.lang.OutOfMemoryError: Java heap space
```

但是，当我们使用标准虚拟机选项运行相同的应用程序时，它运行良好:

```java
Starting pollution
Terminating
```

为什么第一次运行失败了？看起来就连最基本的垃圾收集员也能清理掉我们刚刚演示的这种小儿科游戏。

那么，让我们来看看 Epsilon GC 背后的概念，以了解刚刚发生了什么。

## 3.Epsilon GC 的工作原理

Epsilon 是一个无操作垃圾收集器。

[JEP 318](https://web.archive.org/web/20220628163455/https://openjdk.java.net/jeps/318) 解释说“**【ε】…处理内存分配，但不实现任何实际的内存回收机制。一旦可用的 Java 堆耗尽，JVM 就会关闭。**

所以，这解释了为什么我们的应用程序以`OutOfMemoryError.`结束

但是，这提出了一个问题:为什么我们需要一个不收集任何垃圾的垃圾收集器？

有些情况下**我们知道可用堆足够了，所以我们不希望 JVM 使用资源来运行 GC 任务。**

这种情况的一些例子(也来自相关的 JEP):

*   性能试验
*   记忆压力测试
*   虚拟机接口测试
*   寿命极短的工作
*   最后一滴延迟改进
*   最后一滴吞吐量提高

## 4.结论

在这篇短文中，我们了解了 Epsilon，它是 Java 11 中的一个无操作 GC。我们了解了使用它的含义，并回顾了一些它可能有用的案例。

像往常一样，这些例子可以在 GitHub 的[上找到。](https://web.archive.org/web/20220628163455/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-11)