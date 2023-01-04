# DVM 和 JVM 有什么区别？

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-jvm-vs-dvm>

## 1.介绍

在本文中，我们将探讨 [**Java 虚拟机(JVM)**](https://web.archive.org/web/20221126230202/https://docs.oracle.com/javase/specs/index.html) 和 [**Dalvik 虚拟机(DVM)**](https://web.archive.org/web/20221126230202/https://source.android.com/devices/tech/dalvik) 之间的区别。我们将首先快速浏览一下它们，然后进行比较。

注意，从 Android 5.0 开始，Dalvik 虚拟机已经被 Android Runtime (ART)取代。

## 2.什么是运行时？

运行时系统提供了一个环境来**将 Java 之类的高级语言编写的代码翻译成中央处理器(CPU)可以理解的机器码**。

我们可以区分这些类型的翻译者:

*   汇编程序:他们直接把汇编代码翻译成机器代码，所以速度很快
*   编译器:他们把代码翻译成汇编代码，然后使用汇编器把结果代码翻译成二进制。使用这种技术很慢，但是执行起来很快。此外，生成的机器码是平台相关的
*   解释者:他们在执行代码时翻译代码。由于翻译发生在运行时，执行可能会很慢

## 3.Java 虚拟机

JVM 是运行 Java 桌面、服务器和 web 应用程序的虚拟机。关于 Java 的另一件重要的事情是，它是在考虑到可移植性的情况下开发的。因此，**JVM 也被改造成支持多主机架构，并且可以在任何地方运行**。但是，它对于嵌入式设备来说太重了。

Java 有一个活跃的社区，并且在未来会继续被广泛使用。而且，HotSpot 是 JVM 参考实现。此外，开源社区还维护了五个以上的其他实现。

有了基于 cadence 的新版本，Java 和 JVM 每六个月就会收到新的更新。例如，我们可以为下一个版本列出一些建议，比如[外部内存访问](https://web.archive.org/web/20221126230202/https://openjdk.java.net/jeps/383)和[打包工具](https://web.archive.org/web/20221126230202/https://openjdk.java.net/jeps/343)。

## 4.达尔维克虚拟机

DVM 是运行 Android 应用程序的虚拟机。DVM 执行 Dalvik 字节码，它是用 Java 语言编写的程序编译的。请注意，DVM 不是 JVM。

DVM 的一个关键设计原则是**它应该运行在低内存的移动设备上**并且加载速度比任何 JVM 都要快。此外，当虚拟机在同一台设备上运行多个实例时，效率会更高。

2014 年，谷歌为 Android 5 发布了 [Android Runtime (ART)](https://web.archive.org/web/20221126230202/https://source.android.com/devices/tech/dalvik#features) ，它取代了 Dalvik，提高了应用性能和电池使用。上一个版本是安卓 4.4 上的 1.6.0。

## 5.JVM 和 DVM 的区别

### 5.1.体系结构

JVM 是一个基于堆栈的 VM，其中所有的算术和逻辑操作都是通过 push 和 pop 操作数执行的，结果存储在堆栈中。堆栈也是存储方法的数据结构。

**相反，DVM 是基于寄存器的虚拟机**。这些位于 CPU 中的寄存器执行所有的算术和逻辑运算。寄存器是存储操作数的数据结构。

### 5.2.汇编

Java 代码在 JVM 内部被编译成一种叫做 [Java 字节码](/web/20221126230202/https://www.baeldung.com/java-class-view-bytecode)的中间格式。类文件)。然后，JVM **解析生成的 Java 字节码，并将其翻译成机器码**。

在 Android 设备上，DVM 将 Java 代码编译成一种称为 Java 字节码的中间格式。类文件)类似于 JVM。然后，在一个叫做 **Dalvik eXchange 或 dx 的工具的帮助下，它将 Java 字节码转换成 Dalvik 字节码**。最后，**DVM 将 Dalvik 字节码翻译成二进制机器码**。

**两个虚拟机都使用[实时](/web/20221126230202/https://www.baeldung.com/graal-java-jit-compiler) [(](/web/20221126230202/https://www.baeldung.com/graal-java-jit-compiler) [JIT](/web/20221126230202/https://www.baeldung.com/graal-java-jit-compiler) [编译器](/web/20221126230202/https://www.baeldung.com/graal-java-jit-compiler)** )。JIT 编译器是一种在运行时执行编译的编译器。

### 5.3.表演

如前所述，JVM 是基于堆栈的 VM，而 d VM 是基于寄存器的 VM。基于堆栈的 VM 字节码非常紧凑，因为操作数的位置隐含在操作数堆栈中。基于寄存器的 VM 字节码要求所有隐式操作数都是指令的一部分。这表明**基于寄存器的代码通常比基于堆栈的字节码大得多。**

另一方面，基于寄存器的 VM 可以使用比相应的基于堆栈的 VM 更少的 VM 指令来表达计算。分派一条 VM 指令是昂贵的，**因此减少执行的** **VM 指令可能会显著提高基于寄存器的 VM** 的速度。

当然，只有在解释模式下运行 VM 时，这种区别才有意义。

### 5.4.执行

尽管可以为每个正在运行的应用程序设置一个 JVM 实例，但通常我们只会为一个 JVM 实例配置共享进程和内存空间，以运行我们已经部署的所有应用程序。

然而，Android 被设计为运行多个 DVM 实例。因此，为了运行应用程序或服务， **Android 操作系统在共享内存空间中创建一个新的 DVM 实例，并部署代码来运行应用程序。**

## 6.结论

在本教程中，我们介绍了 JVM 和 DVM 之间的主要区别。两个虚拟机都运行用 Java 编写的应用程序，但是它们使用不同的技术和过程来编译和运行代码。