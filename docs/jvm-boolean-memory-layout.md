# JVM 中的布尔和布尔[]内存布局

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/jvm-boolean-memory-layout>

## 1.概观

在这篇简短的文章中，我们将看到在不同的情况下，JVM 中的一个`boolean `值的足迹是什么。

首先，我们将检查 JVM 以查看对象大小。然后，我们将理解这些尺寸背后的基本原理。

## 2.设置

为了检查 JVM 中对象的内存布局，我们将广泛使用 Java 对象布局( [JOL](https://web.archive.org/web/20220523144001/https://openjdk.java.net/projects/code-tools/jol/) )。因此，我们需要添加 [`jol-core`](https://web.archive.org/web/20220523144001/https://search.maven.org/artifact/org.openjdk.jol/jol-core) 的依赖关系:

```java
<dependency>
    <groupId>org.openjdk.jol</groupId>
    <artifactId>jol-core</artifactId>
    <version>0.10</version>
</dependency>
```

## 3.对象大小

如果我们要求 JOL 打印关于对象大小的 VM 细节:

```java
System.out.println(VM.current().details());
```

当[压缩引用](/web/20220523144001/https://www.baeldung.com/jvm-compressed-oops)被启用时(默认行为)，我们将看到输出:

```java
# Running 64-bit HotSpot VM.
# Using compressed oop with 3-bit shift.
# Using compressed klass with 3-bit shift.
# Objects are 8 bytes aligned.
# Field sizes by type: 4, 1, 1, 2, 2, 4, 4, 8, 8 [bytes]
# Array element sizes: 4, 1, 1, 2, 2, 4, 4, 8, 8 [bytes]
```

在前几行中，我们可以看到一些关于虚拟机的一般信息。之后，我们学习物体的大小:

*   Java 引用消耗 4 字节，`boolean` s/ `byte` s 为 1 字节，`char` s/ `short` s 为 2 字节，`int` s/ `float` s 为 4 字节，最后，`long` s/ `double` s 为 8 字节
*   即使我们将它们用作数组元素，这些类型也消耗相同数量的内存

**因此，在存在压缩引用的情况下，每个`boolean `值占用 1 个字节。类似地，`boolean[] `中的每个`boolean `消耗 1 个字节。**然而，对齐填充和对象头会增加`boolean `和`boolean[] `消耗的空间，我们稍后会看到。

### 3.1.没有压缩的引用

即使我们通过`-XX:-UseCompressedOops`、**禁用压缩引用，布尔大小也不会改变**:

```java
# Field sizes by type: 8, 1, 1, 2, 2, 4, 4, 8, 8 [bytes]
# Array element sizes: 8, 1, 1, 2, 2, 4, 4, 8, 8 [bytes]
```

另一方面，Java 引用占用了两倍的内存。

**所以，不管我们一开始可能会怎么想，*布尔值*消耗的是 1 个字节，而不是 1 位。**

### 3.2.文字撕裂

在大多数体系结构中，没有办法原子地访问单个位。即使我们想这样做，我们可能会在更新另一个位的同时写入相邻的位。

**JVM 的设计目标之一就是防止这种现象，被称为[字撕裂](https://web.archive.org/web/20220523144001/https://docs.oracle.com/javase/specs/jls/se8/html/jls-17.html#jls-17.6)** 。也就是说，在 JVM 中，每个字段和数组元素都应该是不同的；对一个字段或元素的更新不能与任何其他字段或元素的读取或更新交互。

**总的来说，可寻址性问题和字撕裂是*布尔*不仅仅是一位的主要原因。**

## 4.普通对象指针(哎呀)

现在我们知道`boolean`是 1 个字节，让我们考虑这个简单的类:

```java
class BooleanWrapper {
    private boolean value;
}
```

如果我们使用 JOL 检查这个类的内存布局:

```java
System.out.println(ClassLayout.parseClass(BooleanWrapper.class).toPrintable());
```

然后 JOL 将打印内存布局:

```java
 OFFSET  SIZE      TYPE DESCRIPTION                               VALUE
      0    12           (object header)                           N/A
     12     1   boolean BooleanWrapper.value                      N/A
     13     3           (loss due to the next object alignment)
Instance size: 16 bytes
Space losses: 0 bytes internal + 3 bytes external = 3 bytes total
```

`BooleanWrapper `布局包括:

*   12 字节为报头，包括两个 [`mark`](https://web.archive.org/web/20220523144001/http://hg.openjdk.java.net/jdk8/jdk8/hotspot/file/87ee5ee27509/src/share/vm/oops/markOop.hpp) 字和一个 [`klass`](https://web.archive.org/web/20220523144001/http://hg.openjdk.java.net/jdk8/jdk8/hotspot/file/87ee5ee27509/src/share/vm/oops/klass.hpp) 字。HotSpot JVM 使用`mark`字来存储 GC 元数据、身份 hashcode 和锁定信息。此外，它使用`klass`字来存储类元数据，比如运行时类型检查
*   实际`boolean `值的 1 个字节
*   **用于对齐目的的 3 字节填充符**

**默认情况下，对象引用应该按 8 字节对齐。**因此，JVM 将 3 个字节添加到 13 个字节的头中，并将`boolean `添加到 16 个字节中。

因此，`boolean `字段可能会因其字段对齐方式而消耗更多内存。

### 4.1.自定义对齐

如果我们通过`-XX:ObjectAlignmentInBytes=32, `将对齐值更改为 32，则相同的类布局更改为:

```java
OFFSET  SIZE      TYPE DESCRIPTION                               VALUE
      0    12           (object header)                           N/A
     12     1   boolean BooleanWrapper.value                      N/A
     13    19           (loss due to the next object alignment)
Instance size: 32 bytes
Space losses: 0 bytes internal + 19 bytes external = 19 bytes total
```

如上所示，JVM 添加了 19 字节的填充，使对象大小是 32 的倍数。

## 5.数组糟糕

让我们看看 JVM 如何在内存中布置一个`boolean `数组:

```java
boolean[] value = new boolean[3];
System.out.println(ClassLayout.parseInstance(value).toPrintable());
```

这将打印实例布局，如下所示:

```java
OFFSET  SIZE      TYPE DESCRIPTION                              
      0     4           (object header)  # mark word
      4     4           (object header)  # mark word
      8     4           (object header)  # klass word
     12     4           (object header)  # array length
     16     3   boolean [Z.<elements>    # [Z means boolean array                        
     19     5           (loss due to the next object alignment)
```

除了两个`mark`字和一个`klass`字， **[数组指针](https://web.archive.org/web/20220523144001/http://hg.openjdk.java.net/jdk8/jdk8/hotspot/file/87ee5ee27509/src/share/vm/oops/arrayOop.hpp)包含额外的 4 个字节来存储它们的长度。**

因为我们的数组有三个元素，所以数组元素的大小是 3 个字节。然而，**这 3 个字节将由 5 个字段对齐字节填充，以确保正确对齐。**

虽然数组中的每个`boolean `元素只有 1 个字节，但是整个数组消耗了更多的内存。换句话说，**在计算数组大小时，我们应该考虑头和填充开销。**

## 6.结论

在这个快速教程中，我们看到`boolean `字段消耗了 1 个字节。此外，我们了解到我们应该考虑对象大小中的头和填充开销。

关于更详细的讨论，强烈建议查看 JVM 源代码的 [oops 部分。另外，](https://web.archive.org/web/20220523144001/http://hg.openjdk.java.net/jdk8/jdk8/hotspot/file/87ee5ee27509/src/share/vm/oops/)[阿列克谢·希皮洛夫](https://web.archive.org/web/20220523144001/https://shipilev.net/)在这方面有一篇[更深入的文章](https://web.archive.org/web/20220523144001/https://shipilev.net/jvm/objects-inside-out/)。

像往常一样，所有的例子都可以在 GitHub 上找到[。](https://web.archive.org/web/20220523144001/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-jvm-2)