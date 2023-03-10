# Java 查找指南。类别版本

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-find-class-version>

## 1。概述

在本教程中，我们将看看如何找到一个`.class`文件的 Java 发布版本。此外，我们将了解如何检查 jar 文件中的 Java 版本。

## 2。`.class`Java 版本

编译 Java 文件时，会生成一个`.class`文件。在某些情况下，我们需要找到编译后的类文件的 Java 发布版本。**每个 Java 主发行版都会为它生成的`.class`文件分配一个主版本。**

在此表中，我们将主版本号`.class`映射到引入该类版本的 JDK 版本，并显示了该版本号的十六进制表示:

| Java 版本 | 类主要版本 | 十六进制 |
| Java SE 18 | Sixty-two | 003e |
| Java SE 17 | Sixty-one | 003d |
| Java SE 16 | Sixty | 003c |
| Java SE 15 | Fifty-nine | 003b |
| Java SE 14 | Fifty-eight | 003a |
| Java SE 13 | Fifty-seven | 0039 |
| Java SE 12 | fifty-six | 0038 |
| Java SE 11 | Fifty-five | 0037 |
| Java SE 10 | Fifty-four | 0036 |
| Java SE 9 | Fifty-three | 0035 |
| Java SE 8 | fifty-two | 0034 |
| Java SE 7 | Fifty-one | 0033 |
| Java SE 6 | Fifty | 0032 |
| Java SE 5 | forty-nine | 0031 |
| JDK 1.4 | Forty-eight | 0030 |
| jdk1.3 | Forty-seven | 002f |
| JDK 1.2 | Forty-six | 002e |
| JDK 1.1 | Forty-five | 002d |

## 3。`.class`版本的`javap`命令

让我们创建一个简单的类，并用 JDK 8:

```java
public class Sample {
    public static void main(String[] args) {
        System.out.println("Baeldung tutorials");
    }
}
```

**为了识别类文件的版本，我们可以使用 Java 类文件反汇编器 [`javap`](/web/20221104163944/https://www.baeldung.com/java-class-view-bytecode#javaCommandLine) 。**

下面是`javap`命令的语法:

```java
javap [option] [classname]
```

让我们以检查`Sample.class`的版本为例:

```java
javap -verbose Sample

//stripped output ..
..
..
Compiled from "Sample.java"
public class test.Sample
  minor version: 0
  major version: 52
..
.. 
```

正如我们在`javap`命令的输出中看到的，主要版本是 52，表明它是用于 JDK8 的。

虽然`javap`给出了许多细节，但我们只关心主要版本。

对于任何基于 Linux 的系统，我们可以使用下面的命令只获取主要版本:

```java
javap -verbose Sample | grep major
```

同样，对于 Windows 系统，我们可以使用以下命令:

```java
javap -verbose Sample | findstr major
```

在我们的例子中，这给出了主要版本 52。

需要注意的是，这个版本值并不表示应用程序是用相应的 JDK 构建的。类文件版本可以不同于用于编译的 JDK。

**例如，如果我们用 JDK11 构建我们的代码，它应该产生一个版本为 55 的`.class`文件。但是，如果我们在编译期间通过`[-target](/web/20221104163944/https://www.baeldung.com/java-source-target-options) 8`** **，那么`.class`文件将会有版本 52。**

## 4。`hexdump`为`.class`版本

也可以使用任何十六进制编辑器来检查版本。Java 类文件遵循一个[规范](https://web.archive.org/web/20221104163944/https://en.wikipedia.org/wiki/Java_class_file)。让我们看看它的结构:

```java
ClassFile {
    u4             magic;
    u2             minor_version;
    u2             major_version;
    // other details
}
```

这里，`u1`、`u2`和`u4`类型分别表示一个无符号的一字节、两字节和四字节整数。
`u4`是标识类文件格式的幻数。它的值是 0xCAFEBABE，`u2`是主要版本。

**对于基于 Linux 的系统，我们可以使用 [`hexdump`](/web/20221104163944/https://www.baeldung.com/linux/create-hex-dump#using-hexdump) 实用程序来解析任何`.class`文件**:

```java
> hexdump -v Sample.class
0000000 ca fe ba be 00 00 00 34 00 22 07 00 02 01 00 0b
0000010 74 65 73 74 2f 53 61 6d 70 6c 65 07 00 04 01 00
...truncated 
```

在这个例子中，我们使用 JDK8 进行编译。第一行中的 7 和 8 索引提供了类文件的主要版本。所以 0034 是十六进制表示，JDK8 是对应的发布号(来自我们之前看到的映射表)。

作为替代，**我们可以用`hexdump`** 直接得到十进制的主发布版本:

```java
> hexdump -s 7 -n 1 -e '"%d"' Sample.class
52
```

这里，输出 52 是对应于 JDK8 的类版本。

## 5。罐子的版本

Java 生态系统中的 jar 文件由捆绑在一起的类文件集合组成。为了找出构建或编译 jar 的 Java 版本，**我们可以提取 jar 文件并使用 [`javap`](/web/20221104163944/https://www.baeldung.com/java-class-view-bytecode#javaCommandLine) 或 [`hexdump`](/web/20221104163944/https://www.baeldung.com/linux/create-hex-dump#using-hexdump) 来检查`.class`文件的版本。**

jar 文件中还有一个`[MANIFEST.MF](/web/20221104163944/https://www.baeldung.com/java-jar-manifest)`文件，它包含一些关于所用 JDK 的头信息。

例如，`Build-Jdk`或`Created-By`头根据 jar 的构建方式存储 JDK 值:

```java
Build-Jdk: 17.0.4
```

或者

```java
Created-By: 17.0.4
```

## 5。结论

在本文中，我们学习了如何找到一个.`class`文件的 Java 版本。我们看到了`javap`和`hexdump`命令以及它们查找版本的用法。此外，我们还了解了如何检查 jar 文件中的 Java 版本。