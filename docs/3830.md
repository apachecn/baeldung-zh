# Java 10 的性能改进

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-10-performance-improvements>

[This article is part of a series:](javascript:void(0);)[• Java 10 LocalVariable Type-Inference](/web/20221206234834/https://www.baeldung.com/java-10-local-variable-type-inference)
• Java 10 Performance Improvements (current article)[• New Features in Java 10](/web/20221206234834/https://www.baeldung.com/java-10-overview)

## 1。概述

在这个快速教程中，我们将讨论最新的 Java 10 版本带来的性能改进。

这些改进适用于在 JDK 10 下运行的所有应用程序，不需要任何代码更改来利用它们。

## 2。G1 的平行全气相色谱

从 JDK 9 开始，G1 垃圾收集器就是默认的。但是，G1 的完整 GC 使用了单线程标记-扫描-压缩算法。

在 Java 10 中，这已经被**更改为并行标记-清除-压缩算法**,有效地减少了完全 GC 期间的停止时间。

## 3。应用程序类-数据共享

JDK 5 中引入的类数据共享允许将一组类预处理到一个共享的归档文件中，然后可以在运行时进行内存映射，以减少启动时间，这也可以减少多个 JVM 共享同一个归档文件时的动态内存占用。

CDS 只允许引导类装入器，限制了系统类的特性。应用程序 CDS (AppCDS)扩展 CDS 以允许内置系统类加载器(也称为“应用程序类加载器”)、内置平台类加载器和定制类加载器加载存档的类。**这使得将该功能用于应用程序类别成为可能。**

我们可以通过以下步骤来利用这一功能:

**1。获取要存档的课程列表**

以下命令将把由`HelloWorld `应用程序加载的类转储到`hello.lst`:

```
$ java -Xshare:off -XX:+UseAppCDS -XX:DumpLoadedClassList=hello.lst \ 
    -cp hello.jar HelloWorld
```

**2。创建 AppCDS 归档文件**

以下命令使用`hello.lst `作为输入创建`hello.js a `:

```
$ java -Xshare:dump -XX:+UseAppCDS -XX:SharedClassListFile=hello.lst \
    -XX:SharedArchiveFile=hello.jsa -cp hello.jar
```

**3。使用 AppCDS 档案**

以下命令以`hello.jsa `作为输入启动`HelloWorld `应用程序:

```
$ java -Xshare:on -XX:+UseAppCDS -XX:SharedArchiveFile=hello.jsa \
    -cp hello.jar HelloWorld
```

AppCDS 是甲骨文 JDK 公司为 JDK 8 版和 JDK 9 版推出的一项商业功能。现在它是开源的，可以公开获得。

## 4。实验性的基于 Java 的 JIT 编译器

[Graal](https://web.archive.org/web/20221206234834/https://github.com/oracle/graal/blob/master/compiler/README.md) 是用 Java 编写的动态编译器，集成了 HotSpot JVM 它专注于高性能和可扩展性。它也是 JDK 9 中引入的实验性超前(AOT)编译器的基础。

JDK 10 使 Graal 编译器能够在 Linux/x64 平台上作为实验性的 JIT 编译器使用。

要启用 Graal 作为 JIT 编译器，请在 java 命令行上使用以下选项:

```
-XX:+UnlockExperimentalVMOptions -XX:+UseJVMCICompiler
```

请注意，这是一个实验性的特性，我们不一定能获得比现有的 JIT 编译器更好的性能。

## 5。结论

在这篇简短的文章中，我们关注并探索了 Java 10 中的性能改进特性。

Next **»**[New Features in Java 10](/web/20221206234834/https://www.baeldung.com/java-10-overview)**«** Previous[Java 10 LocalVariable Type-Inference](/web/20221206234834/https://www.baeldung.com/java-10-local-variable-type-inference)