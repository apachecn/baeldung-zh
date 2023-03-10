# 探索 JVM 调优标志

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/jvm-tuning-flags>

## 1.概观

可以用各种调优标志来调优 HotSpot JVM。由于有数百个这样的标志，跟踪它们和它们的默认值可能有点令人生畏。

在本教程中，我们将介绍一些发现这种调优标志的方法，并学习如何使用它们。

## 2.Java 选项概述

`java `命令支持多种标志，分为以下几类:

*   保证所有 JVM 实现都支持的标准选项。通常，这些选项用于日常操作，如`–classpath, -cp, –version, `等
*   并非所有 JVM 实现都支持的额外选项，通常会发生变化。这些选项以`-X`开头

请注意，我们不应该随便使用这些额外的选项。此外，**一些附加选项更高级，以** `**-XX**. `开头

在整篇文章中，我们将关注更高级的`-XX `标志。

## 3.JVM 调优标志

**要列出全局 JVM 调优标志，我们可以启用`PrintFlagsFinal `标志，如下所示:**

```java
>> java -XX:+PrintFlagsFinal -version
[Global flags]
    uintx CodeCacheExpansionSize                   = 65536                                  {pd product} {default}
     bool CompactStrings                           = true                                   {pd product} {default}
     bool DoEscapeAnalysis                         = true                                   {C2 product} {default}
   double G1ConcMarkStepDurationMillis             = 10.000000                                 {product} {default}
   size_t G1HeapRegionSize                         = 1048576                                   {product} {ergonomic}
    uintx MaxHeapFreeRatio                         = 70                                     {manageable} {default}

// truncated
openjdk version "14" 2020-03-17
OpenJDK Runtime Environment (build 14+36-1461)
OpenJDK 64-Bit Server VM (build 14+36-1461, mixed mode, sharing)
```

如上所示，对于这个特定的 JVM 版本，一些标志有默认值。

某些标志的默认值在不同的平台上可能不同，这显示在最后一列中。例如，`product`意味着标志的默认设置在所有平台上都是一致的；`pd product`表示标志的默认设置是平台相关的。`manageable `值可以在运行时动态改变。

### 3.1.诊断标志

然而，`PrintFlagsFinal `标志并没有显示所有可能的调谐标志。例如，**要查看诊断调优标志，我们应该添加`UnlockDiagnosticVMOptions `标志:**

```java
>> java -XX:+PrintFlagsFinal -version | wc -l
557

>> java -XX:+PrintFlagsFinal -XX:+UnlockDiagnosticVMOptions -version | wc -l
728
```

显然，当我们包括诊断选项时，还有几百个标志。例如，打印本机内存跟踪统计数据只能作为诊断标志的一部分:

```java
bool PrintNMTStatistics                       = false                                  {diagnostic} {default}
```

### 3.2.实验旗帜

**要查看实验选项，我们应该添加`UnlockExperimentalVMOptions `标志:**

```java
>> java -XX:+UnlockDiagnosticVMOptions -XX:+UnlockExperimentalVMOptions -XX:+PrintFlagsFinal -version | wc -l
809
```

### 三点三。jvmci 标志

从 Java 9 开始，JVM 编译器接口或 JVMCI 使我们能够使用用 Java 编写的编译器，如 Graal，作为动态编译器。

要查看与 JVMCI 相关的选项，我们应该再添加几个标志，甚至还要启用 JVMCI:

```java
>> java -XX:+UnlockDiagnosticVMOptions -XX:+UnlockExperimentalVMOptions \
>> -XX:+JVMCIPrintProperties -XX:+EnableJVMCI -XX:+PrintFlagsFinal  -version | wc -l
1516
```

然而，大多数时候，使用全局、诊断和实验选项应该足够了，并且将帮助我们找到我们心目中的标志。

### 3.4.把所有的放在一起

这些选项的组合可以帮助我们找到一个调优标志，尤其是当我们不记得确切的名称时。例如，要在 Java 中找到与软引用相关的调优标志:

```java
>> alias jflags="java -XX:+UnlockDiagnosticVMOptions -XX:+UnlockExperimentalVMOptions -XX:+PrintFlagsFinal  -version"
>> jflags | grep Soft
size_t SoftMaxHeapSize                          = 4294967296                             {manageable} {ergonomic}
intx SoftRefLRUPolicyMSPerMB                    = 1000                                   {product} {default}
```

从结果中，我们可以很容易地猜到`SoftRefLRUPolicyMSPerMB`就是我们要寻找的旗帜。

## 4.不同类型的旗帜

在上一节中，我们忽略了一个重要的主题:标志类型。让我们再来看看`java -XX:+PrintFlagsFinal -version `的输出:

```java
[Global flags]
    uintx CodeCacheExpansionSize                   = 65536                                  {pd product} {default}
     bool CompactStrings                           = true                                   {pd product} {default}
     bool DoEscapeAnalysis                         = true                                   {C2 product} {default}
   double G1ConcMarkStepDurationMillis             = 10.000000                                 {product} {default}
   size_t G1HeapRegionSize                         = 1048576                                   {product} {ergonomic}
    uintx MaxHeapFreeRatio                         = 70                                     {manageable} {default}
// truncated
```

如上所示，每个标志都有特定的类型。

**布尔选项用于启用或禁用功能**。这样的选项不需要值。要启用它们，我们只需在选项名称前加上一个加号:

```java
-XX:+PrintFlagsFinal
```

相反，要禁用它们，我们必须在它们的名称前添加一个减号:

```java
-XX:-RestrictContended
```

其他标志类型需要一个参数值。可以用空格、冒号、等号将值与选项名分开，或者参数可以直接跟在选项名后面(每个选项的确切语法不同):

```java
-XX:ObjectAlignmentInBytes=16 -Xms5g -Xlog:gc
```

## 5.文档和源代码

找到正确的旗帜名称是一回事。找到那面特定的旗帜在引擎盖下做什么是另一回事。

找出这些细节的一个方法是查看文档。例如， [JDK 工具规范章节](https://web.archive.org/web/20221205164430/https://docs.oracle.com/en/java/javase/14/docs/specs/man/java.html)中的`java `命令的文档是一个很好的起点。

有时候，再多的文档也比不上源代码。因此，如果我们有一个特定标志的名称，那么我们就可以研究 JVM 源代码来找出发生了什么。

例如，我们可以从 [GitHub](https://web.archive.org/web/20221205164430/https://github.com/openjdk/jdk14u) 甚至他们的 [Mercurial 库](https://web.archive.org/web/20221205164430/http://hg.openjdk.java.net/jdk8)中检查 HotSpot JVM 的源代码，然后:

```java
>> git clone [[email protected]](/web/20221205164430/https://www.baeldung.com/cdn-cgi/l/email-protection):openjdk/jdk14u.git openjdk
>> cd openjdk/src/hotspot
>> grep -FR 'PrintFlagsFinal' .
./share/runtime/globals.hpp:  product(bool, PrintFlagsFinal, false,                                   
./share/runtime/init.cpp:  if (PrintFlagsFinal || PrintFlagsRanges) {
```

这里我们寻找所有包含`PrintFlagsFinal `字符串的文件。找到负责的文件后，我们可以查看一下，看看这个特定的标志是如何工作的。

## 6.结论

在本文中，我们看到了如何找到几乎所有可用的 JVM 调优标志，还学习了一些更有效地使用它们的技巧。