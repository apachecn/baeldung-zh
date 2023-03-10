# Java 是编译语言还是解释语言？

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-compiled-interpreted>

## 1.概观

编程语言是根据它们的抽象层次来分类的。我们区分高级语言(Java，Python，JavaScript，C++，Go)，低级语言(汇编语言)，最后是机器码。

每一个高级语言代码，像 Java，**都需要翻译成机器原生代码才能执行。**这个翻译过程可以是编译，也可以是解释。然而，还有第三种选择。寻求两种方法优势的组合。

在本教程中，我们将探索 Java 代码如何在多个平台上编译和执行。我们将研究一些 Java 和 JVM 设计细节。这些将帮助我们确定 Java 是编译的，解释的，还是两者的混合。

## 2.**编译与解释**

让我们先来看看[编译和解释编程语言](/web/20220524022550/https://www.baeldung.com/cs/compiled-vs-interpreted-languages)之间的一些基本区别。

### 2.1.**编译语言**

编译语言(C++，Go)被编译器程序直接转换成机器本机代码。

它们在执行之前需要一个显式的构建步骤。这就是为什么我们每次修改代码时都需要重新构建程序。

编译语言往往比解释语言更快更有效**。**然而，他们生成的机器码是特定于平台的。

### 2.2.解释语言

另一方面，在解释语言(Python，JavaScript)中，没有构建步骤。相反，解释器在执行程序时对程序的源代码进行操作。

解释语言曾经被认为比编译语言慢得多。然而，随着实时(JIT)编译的发展，性能差距正在缩小。然而，我们应该注意，JIT 编译器在程序运行时将代码从解释语言转换成机器本机代码。

此外，我们可以在多种平台上执行解释语言代码，比如 Windows、Linux 或 Mac。解释的代码与特定类型的 CPU 架构没有密切关系。

## 3.一次编写，随处运行

Java 和 JVM 在设计时就考虑到了可移植性。因此，今天大多数流行的平台都可以运行 Java 代码。

这听起来像是暗示 Java 是一种纯解释语言。但是，在执行之前， **Java 源代码需要编译成[字节码](/web/20220524022550/https://www.baeldung.com/java-class-view-bytecode)。字节码是 JVM `.`** 本地的一种特殊的机器语言，JVM 在运行时解释并执行这些代码。

它是为每个支持 Java 的平台构建和定制的 JVM，而不是我们的程序或库。

现代 JVM 也有 JIT 编译器。这意味着 JVM 在运行时优化我们的代码，以获得与编译语言相似的性能优势。

## 4.Java 编译器

**[javac](/web/20220524022550/https://www.baeldung.com/javac) 命令行工具将 Java 源代码编译成包含平台无关字节码的 Java 类文件**:

`$ javac HelloWorld.java`

源代码文件有`.java`后缀，而包含字节码的类文件是用。`class`后缀。

[![](img/c968030bfa65f09863eb265485fde975.png)](/web/20220524022550/https://www.baeldung.com/wp-content/uploads/2021/01/java_compilation-3.png)

## 5.Java 虚拟机

编译后的类文件(字节码)可以由 [Java 虚拟机(JVM)](/web/20220524022550/https://www.baeldung.com/jvm-vs-jre-vs-jdk) 执行[:](/web/20220524022550/https://www.baeldung.com/java-single-file-source-code)

`$ java HelloWorld
Hello Java!`

现在让我们更深入地了解一下 JVM 架构。我们的目标是确定如何在运行时将字节码转换成机器本机代码。

### 5.1.架构概述

JVM 由五个子系统组成:

*   类加载器
*   JVM 内存
*   执行引擎
*   本机方法接口和
*   本机方法库

[![](img/f0832db89fea742bba11a6ad3d069a68.png)](/web/20220524022550/https://www.baeldung.com/wp-content/uploads/2021/01/jvm_arhitecture.png)

### 5.2.类加载器

JVM 利用[类加载器](/web/20220524022550/https://www.baeldung.com/java-classloaders)子系统**将编译好的类文件带入** [**JVM 内存**。](/web/20220524022550/https://www.baeldung.com/java-stack-heap)

除了加载之外，类加载器还执行链接和初始化。这包括:

*   验证字节码是否有任何安全漏洞
*   为静态变量分配内存
*   用原始引用替换符号内存引用
*   将初始值赋给静态变量
*   执行所有静态代码块

### 5.3.执行引擎

执行引擎子系统负责**读取字节码，将其转换成机器本机代码，并执行它。**

三个主要组件负责执行，包括解释器和编译器:

*   因为 JVM 是平台中立的，所以它使用解释器来执行字节码
*   JIT 编译器通过为重复的方法调用将字节码编译成本机代码来提高性能
*   [垃圾收集器](/web/20220524022550/https://www.baeldung.com/jvm-garbage-collectors)收集并移除所有未引用的对象

执行引擎利用[本地方法接口(JNI)](/web/20220524022550/https://www.baeldung.com/jni) 来调用本地库和应用。

### 5.4.即时编译器

解释器的主要缺点是每次调用方法都需要解释，这可能比编译后的本机代码要慢。Java 利用 JIT 编译器来克服这个问题。

JIT 编译器并没有完全取代解释器。执行引擎仍然使用它。然而，JVM 根据方法被调用的频率来使用 JIT 编译器。

JIT 编译器将整个方法的字节码编译成机器本地代码，因此它可以被直接重用**。**与标准编译器一样，需要生成中间代码、优化，然后生成机器本机代码。

分析器是 JIT 编译器的一个特殊组件，负责寻找热点。JVM 根据运行时收集的分析信息决定 JIT 编译哪些代码。

[![](img/496ada4cdc8e45dac9b54551b131752f.png)](/web/20220524022550/https://www.baeldung.com/wp-content/uploads/2021/01/jit_compiler1.png)

这样做的一个效果是，在几个执行周期之后，Java 程序可以更快地执行它的工作。一旦 JVM 了解了热点，它就能够创建本机代码，让事情运行得更快。

## 6.性能比较

让我们看看 JIT 编译如何提高 Java 的运行时性能。

### 6.1.斐波那契性能测试

我们将使用一个简单的递归方法来计算第 n 个斐波那契数:

```java
private static int fibonacci(int index) {
    if (index <= 1) {
        return index;
    }
    return fibonacci(index-1) + fibonacci(index-2);
}
```

为了测量重复方法调用的性能优势，我们将运行斐波纳契方法 100 次:

```java
for (int i = 0; i < 100; i++) {
    long startTime = System.nanoTime();
    int result = fibonacci(12);
    long totalTime = System.nanoTime() - startTime;
    System.out.println(totalTime);
}
```

首先，我们将正常编译和执行 Java 代码:

`$ java Fibonacci.java`

然后，我们将执行禁用 JIT 编译器的相同代码:

`$ java -Djava.compiler=NONE Fibonacci.java`

最后，我们将在 C++和 JavaScript 中实现并运行相同的算法进行比较。

### 6.2.性能测试结果

让我们来看看在运行 Fibonacci 递归测试后，以纳秒为单位测得的平均性能:

*   使用 JIT 编译器的 Java–2726 ns–最快
*   没有 JIT 编译器的 Java–17965 ns–慢 559%
*   没有 O2 优化的 c++ 9435 ns，慢了 246%
*   带 O2 优化的 c++ 3639 ns，速度降低 33%
*   JavaScript–22998 ns–慢了 743%

在这个例子中，**使用 JIT 编译器**，Java 的性能提高了 500%以上。然而，JIT 编译器需要运行几次才能生效。

有趣的是，Java 的性能比 C++代码高 33%,即使 C++编译时启用了 O2 优化标志。不出所料， **C++在最初的几次运行**中表现得好得多，那时 Java 还在被解释。

Java 也优于 Node 上运行的同等 JavaScript 代码，Node 也使用 JIT 编译器。结果显示性能提高了 700%以上。主要原因是 **Java 的 JIT 编译器运行得更快**。

## 7.需要考虑的事项

从技术上讲，可以将任何静态编程语言代码直接编译成机器代码。也有可能一步一步地解释任何编程代码。

与许多其他现代编程语言类似，Java 使用编译器和解释器的组合。目标是利用两个世界的优势，**实现高性能和平台无关的执行**。

在本文中，我们重点解释了 HotSpot 的工作原理。HotSpot 是 Oracle 默认的开源 JVM 实现。 [Graal VM](/web/20220524022550/https://www.baeldung.com/graal-java-jit-compiler) 也是基于 HotSpot 的，所以同样的原理也适用。

如今大多数流行的 JVM 实现使用解释器和 JIT 编译器的组合。然而，他们中的一些人可能使用了不同的方法。

## 8.结论

在本文中，我们研究了 Java 和 JVM 的内部机制。我们的目标是确定 Java 是编译语言还是解释语言。我们探索了 Java 编译器和 JVM 执行引擎的内部机制。

基于此，我们得出结论 **Java 使用了两种方法的组合。**

在构建过程中，我们用 Java 编写的源代码首先被编译成字节码。然后 JVM 解释生成的字节码以便执行。然而，JVM 也在运行时使用 JIT 编译器来提高性能。

和往常一样，源代码可以在 GitHub 上的[处获得。](https://web.archive.org/web/20220524022550/https://github.com/eugenp/tutorials/tree/master/performance-tests)