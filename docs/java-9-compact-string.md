# Java 9 中的紧凑字符串

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-9-compact-string>

## 1。概述

Java 中的`Strings`在内部由包含`String`字符的`char[]`表示。并且，每个`char`由 2 个字节组成，因为 **Java 内部使用 UTF-16。**

例如，如果一个`String`包含一个英语单词，那么每个`char`的前 8 位都是 0，因为一个 ASCII 字符可以用一个字节来表示。

许多字符需要 16 位来表示，但统计上大多数只需要 8 位，即拉丁 1 字符表示。因此，内存消耗和性能还有改进的余地。

同样重要的是, `String`通常占据 JVM 堆空间的很大一部分。而且，由于 JVM 存储它们的方式，在大多数情况下，**一个`String`实例可以占用两倍的**空间**它实际需要的是**。

在本文中，我们将讨论 JDK6 中引入的压缩字符串选项和 JDK9 中最近引入的新的压缩字符串。这两种方法都旨在优化 JMV 上字符串的内存消耗。

## 2。压缩的`String`–Java 6

JDK 6 更新 21 性能版引入了一个新的虚拟机选项:

```java
-XX:+UseCompressedStrings
```

启用此选项后，`Strings`将存储为`byte[]`，而不是 `char[] –` ，从而节省大量内存。然而，这个选项最终在 JDK 7 中被删除了，主要是因为它有一些意想不到的性能后果。

## 3。精简版`String`–Java 9

Java 9 带来了紧凑的概念

这意味着**每当我们创建一个`String`时，如果`String`的所有字符都可以用一个 byte-LATIN-1 表示法来表示，那么一个字节数组将在**内部使用，这样一个字节用于一个字符。

在其他情况下，如果任何字符需要超过 8 位来表示，则所有字符都使用两个字节来存储，每个字节都用 UTF-16 表示。

所以基本上，只要有可能，每个字符只用一个字节。

现在，问题是——所有的`String`操作将如何工作？它将如何区分 LATIN-1 和 UTF-16 表示？

**为了解决这个问题，对`String`的内部实现做了另一个改变。我们有一个最后的字段`coder`，保存这个信息。**

### 3.1。`String`用 Java 9 实现

到目前为止，`String`被存储为`char[]`:

```java
private final char[] value;
```

从现在开始，这将是一个`byte[]:`

```java
private final byte[] value;
```

变量`coder`:

```java
private final byte coder;
```

其中`coder`可以是:

```java
static final byte LATIN1 = 0;
static final byte UTF16 = 1;
```

大多数`String`操作现在检查编码器并分派给特定的实现:

```java
public int indexOf(int ch, int fromIndex) {
    return isLatin1() 
      ? StringLatin1.indexOf(value, ch, fromIndex) 
      : StringUTF16.indexOf(value, ch, fromIndex);
}  

private boolean isLatin1() {
    return COMPACT_STRINGS && coder == LATIN1;
} 
```

有了 JVM 需要的所有信息，默认情况下,`CompactString` VM 选项是启用的。要禁用它，我们可以使用:

```java
+XX:-CompactStrings
```

### 3.2。如何运作

在 Java 9 `String`类实现中，长度计算如下:

```java
public int length() {
    return value.length >> coder;
}
```

如果`String`只包含 LATIN-1，那么`coder`的值将为 0，因此`String`的长度将与字节数组的长度相同。

在其他情况下，如果`String`是 UTF-16 表示，那么`coder`的值将是 1，因此长度将是实际字节数组大小的一半。

**注意，对 Compact `String,`所做的所有更改都在`String`类的内部实现中，并且对使用`String`的开发人员来说是完全透明的。**

## 4。压缩`Strings`与压缩`Strings`

在 JDK 6 Compressed `Strings,`的情况下，面临的一个主要问题是`String`构造器只接受`char[]`作为参数。除此之外，许多`String`操作依赖于`char[]`表示，而不是字节数组。因此，必须进行大量的解包工作，这会影响性能。

而在紧凑`String,`的情况下，维护额外的字段“编码器”也会增加开销。为了降低`coder`和`byte`到`char`的解包成本(在 UTF-16 表示的情况下)，一些方法被[具体化](https://web.archive.org/web/20220630012154/https://en.wikipedia.org/wiki/Intrinsic_function)，JIT 编译器生成的 ASM 代码也得到了改进。

这一变化导致了一些违背直觉的结果。LATIN-1 `indexOf(String)`调用内部方法，而`indexOf(char)`不调用。对于 UTF-16，这两种方法都调用一个内部方法。此问题仅影响拉丁语-1 `String`，将在未来版本中修复。

因此，就性能而言，压缩的`Strings`比压缩的`Strings`更好。

为了找出使用 Compact `Strings,`节省了多少内存，对各种 Java 应用程序堆转储进行了分析。此外，虽然结果在很大程度上取决于具体的应用，但总体改进几乎总是相当可观的。

### 4.1。性能差异

让我们看一个非常简单的例子来说明启用和禁用压缩的性能差异`Strings:`

```java
long startTime = System.currentTimeMillis();

List strings = IntStream.rangeClosed(1, 10_000_000)
  .mapToObj(Integer::toString) 
  .collect(toList());

long totalTime = System.currentTimeMillis() - startTime;
System.out.println(
  "Generated " + strings.size() + " strings in " + totalTime + " ms.");

startTime = System.currentTimeMillis();

String appended = (String) strings.stream()
  .limit(100_000)
  .reduce("", (l, r) -> l.toString() + r.toString());

totalTime = System.currentTimeMillis() - startTime;
System.out.println("Created string of length " + appended.length() 
  + " in " + totalTime + " ms.");
```

在这里，我们创建了 1000 万个`String`然后以一种简单的方式添加它们。当我们运行这段代码(默认情况下启用压缩字符串)时，我们得到输出:

```java
Generated 10000000 strings in 854 ms.
Created string of length 488895 in 5130 ms.
```

类似地，如果我们通过使用:`-XX:-CompactStrings` 选项禁用压缩字符串来运行它，输出是:

```java
Generated 10000000 strings in 936 ms.
Created string of length 488895 in 9727 ms.
```

显然，这是一个表面水平的测试，它不可能具有很高的代表性-它只是新选项在这种特定情况下可以提高性能的快照。

## 5。结论

在本教程中，我们看到了在 JVM 上优化性能和内存消耗的尝试——通过以一种内存高效的方式存储`String`。

和往常一样，Github 上的[提供了完整的代码。](https://web.archive.org/web/20220630012154/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-strings)