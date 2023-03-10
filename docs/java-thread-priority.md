# Java 中线程的优先级

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-thread-priority>

## 1.介绍

在本教程中，我们将讨论**Java 线程调度器如何基于优先级执行线程**。此外，我们将介绍 Java 中线程优先级的类型。

## 2.优先权的类型

在 Java 中，线程的优先级是 1 到 10 之间的整数。整数越大，优先级越高。线程调度器使用来自每个线程的这个整数来确定应该允许哪个线程执行。**`Thread`类定义了三种类型的优先级**:

*   最低优先级
*   正常优先级
*   最高优先级

`Thread`类将这些优先级类型定义为常量`MIN_PRIORITY`、`NORM_PRIORITY`和`MAX_PRIORITY`，值分别为 1、5 和 10。 **`NORM_PRIORITY`是新`Thread`** 的默认优先级。

## 3.`Thread`执行概述

JVM 支持一种被称为固定优先级优先调度的调度算法。所有 Java 线程都有一个优先级，JVM 首先服务于优先级最高的线程。

当我们创建一个`Thread`时，它继承了它的默认优先级。当多个线程准备好执行时，JVM 选择并执行[具有最高优先级的`Runnable`线程](/web/20220531041936/https://www.baeldung.com/java-thread-lifecycle)。如果这个线程停止或变得不可运行，低优先级线程将执行。**如果两个线程具有相同的优先级，JVM 将按照 FIFO 顺序执行它们**。

有两种情况会导致不同的线程运行:

*   优先级比当前线程高的线程变得可运行
*   当前线程退出可运行状态或者[产生](/web/20220531041936/https://www.baeldung.com/java-thread-yield)(暂时暂停并允许其他线程)

通常，在任何时候，最高优先级的线程都在运行。但是有时，线程调度器可能会选择低优先级线程来执行，以避免饥饿。

## 4.了解和更改线程的优先级

Java 的`Thread`类提供了检查线程优先级和修改优先级的方法。 [`getPriority()`](https://web.archive.org/web/20220531041936/https://docs.oracle.com/en/java/javase/12/docs/api/java.base/java/lang/Thread.html#getPriority()) 实例方法返回代表其优先级的整数。 [`setPriority()`](https://web.archive.org/web/20220531041936/https://docs.oracle.com/en/java/javase/12/docs/api/java.base/java/lang/Thread.html#setPriority(int)) 实例方法采用 1 到 10 之间的整数来改变线程的优先级。如果我们传递一个 1-10 范围之外的值，该方法将抛出一个错误。

## 5.结论

在这篇短文中，我们研究了如何使用抢占式调度算法在 Java 中基于优先级执行多线程。我们进一步研究了优先级范围和默认线程优先级。此外，我们分析了检查线程优先级并在必要时操纵它的 Java 方法。