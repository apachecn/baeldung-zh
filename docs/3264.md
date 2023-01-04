# Java 中对象的内存布局

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-memory-layout>

## 1.概观

在本教程中，我们将看到 JVM 如何在堆中布置对象和数组。

首先，我们将从一点理论开始。然后，我们将探索不同环境中不同的对象和数组内存布局。

通常，运行时数据区的内存布局不是 JVM 规范的一部分，而是留给实现者来决定。因此，每个 JVM 实现可能有不同的策略来布局内存中的对象和数组。在本教程中，我们将关注一个特定的 JVM 实现:HotSpot JVM。

我们也可以互换使用 JVM 和 HotSpot JVM 这两个术语。

## 2.普通对象指针(哎呀)

**HotSpot JVM 使用一种称为普通对象指针的数据结构([哎呀](https://web.archive.org/web/20220628094821/https://github.com/openjdk/jdk15/tree/master/src/hotspot/share/oops))来表示指向对象的指针。**JVM 中的所有指针(对象和数组)都是基于一种叫做`[oopDesc](https://web.archive.org/web/20220628094821/https://github.com/openjdk/jdk15/blob/e208d9aa1f185c11734a07db399bab0be77ef15f/src/hotspot/share/oops/oop.hpp#L52). `的特殊数据结构，每个`oopDesc `用以下信息描述指针:

*   一个[标记字](https://web.archive.org/web/20220628094821/https://github.com/openjdk/jdk15/blob/e208d9aa1f185c11734a07db399bab0be77ef15f/src/hotspot/share/oops/oop.hpp#L56)
*   一个，可能是压缩的， [klass 单词](https://web.archive.org/web/20220628094821/https://github.com/openjdk/jdk15/blob/e208d9aa1f185c11734a07db399bab0be77ef15f/src/hotspot/share/oops/oop.hpp#L57)

[标记字](https://web.archive.org/web/20220628094821/https://github.com/openjdk/jdk15/blob/e208d9aa1f185c11734a07db399bab0be77ef15f/src/hotspot/share/oops/markWord.hpp#L33)描述对象头。**HotSpot JVM 使用这个词来存储身份 hashcode、偏向锁定模式、锁定信息和 GC 元数据。**

此外，标记字状态仅包含一个`[uintptr_t](https://web.archive.org/web/20220628094821/https://github.com/openjdk/jdk15/blob/e208d9aa1f185c11734a07db399bab0be77ef15f/src/hotspot/share/oops/markWord.hpp#L96), `因此，**其大小在 32 位和 64 位架构中分别在 4 和 8 字节之间变化。**同样，标志词对于有偏见的和正常的物体是不同的。然而，我们将只考虑普通对象，因为 Java 15 将[反对有偏见的锁定](https://web.archive.org/web/20220628094821/https://openjdk.java.net/jeps/374)。

另外， [klass 单词](https://web.archive.org/web/20220628094821/https://github.com/openjdk/jdk15/blob/bf1e6903a2499d0c2ab2f8703a1dc29046e8375d/src/hotspot/share/oops/klass.hpp#L54)封装了语言级别的类信息，比如类名、它的修饰符、超类信息等等。

对于 Java 中的普通对象，表示为`[instanceOop](https://web.archive.org/web/20220628094821/https://github.com/openjdk/jdk15/blob/master/src/hotspot/share/oops/instanceOop.hpp)`，**，对象头由 mark 和 klass 字加上可能的对齐填充**组成。在对象头之后，可能有零个或多个对实例字段的引用。因此，在 64 位架构中至少有 16 个字节，因为 8 个字节是标记，4 个字节是 klass，另外 4 个字节是填充。

对于数组，表示为 [`arrayOop`](https://web.archive.org/web/20220628094821/https://github.com/openjdk/jdk15/blob/e208d9aa1f185c11734a07db399bab0be77ef15f/src/hotspot/share/oops/arrayOop.hpp#L35) `, ` **对象头除了 mark、klass、paddings 外，还包含一个 4 字节的数组长度。**同样，由于 8 个字节的标记、4 个字节的 klass 和另外 4 个字节的数组长度，这至少需要 16 个字节。

既然我们对理论有了足够的了解，让我们看看内存布局在实践中是如何工作的。

## 3.设置 JOL

为了检查 JVM 中对象的内存布局，我们将广泛使用 Java 对象布局( [JOL](https://web.archive.org/web/20220628094821/https://openjdk.java.net/projects/code-tools/jol/) )。因此，我们需要添加 [`jol-core`](https://web.archive.org/web/20220628094821/https://search.maven.org/artifact/org.openjdk.jol/jol-core) 的依赖关系:

```
<dependency>
    <groupId>org.openjdk.jol</groupId>
    <artifactId>jol-core</artifactId>
    <version>0.10</version>
</dependency>
```

## 4.存储器布局示例

让我们先来看看虚拟机的一般详细信息:

```
System.out.println(VM.current().details());
```

这将打印:

```
# Running 64-bit HotSpot VM.
# Objects are 8 bytes aligned.
# Field sizes by type: 4, 1, 1, 2, 2, 4, 4, 8, 8 [bytes]
# Array element sizes: 4, 1, 1, 2, 2, 4, 4, 8, 8 [bytes]
```

这意味着引用占用 4 个字节，`boolean` s 和`byte` s 占用 1 个字节，`short` s 和`char` s 占用 2 个字节，`int` s 和`float` s 占用 4 个字节，最后，`long` s 和`double` s 占用 8 个字节。有趣的是，如果我们将它们用作数组元素，它们消耗的内存量是一样的。

同样，如果我们通过`-XX:-UseCompressedOops, `禁用[压缩引用](/web/20220628094821/https://www.baeldung.com/jvm-compressed-oops)，只有引用大小变为 8 字节:

```
# Field sizes by type: 8, 1, 1, 2, 2, 4, 4, 8, 8 [bytes]
# Array element sizes: 8, 1, 1, 2, 2, 4, 4, 8, 8 [bytes]
```

### 4.1.基础

让我们考虑一个`SimpleInt`类:

```
public class SimpleInt {
    private int state;
}
```

如果我们打印它的类布局:

```
System.out.println(ClassLayout.parseClass(SimpleInt.class).toPrintable());
```

我们会看到类似这样的内容:

```
SimpleInt object internals:
 OFFSET  SIZE   TYPE DESCRIPTION                               VALUE
      0    12        (object header)                           N/A
     12     4    int SimpleInt.state                           N/A
Instance size: 16 bytes
Space losses: 0 bytes internal + 0 bytes external = 0 bytes total
```

如上所示，对象头是 12 字节，包括 8 字节的 mark 和 4 字节的 klass。之后，我们有 4 个字节用于`int state`。总之，这个类中的任何对象都将消耗 16 个字节。

此外，对象头和状态没有值，因为我们解析的是类布局，而不是实例布局。

### 4.2.身份散列码

`[hashCode()](/web/20220628094821/https://www.baeldung.com/java-hashcode) `是所有 Java 对象的通用方法之一。**当我们没有为一个类声明一个`hashCode() `** **方法时，Java 会为它使用 identity hash 代码。**

对象的标识哈希代码在其生命周期内不会改变。因此，**HotSpot JVM 在计算后将这个值存储在标记字中。**

让我们看看一个对象实例的内存布局:

```
SimpleInt instance = new SimpleInt();
System.out.println(ClassLayout.parseInstance(instance).toPrintable());
```

HotSpot JVM 缓慢地计算身份哈希代码:

```
SimpleInt object internals:
 OFFSET  SIZE   TYPE DESCRIPTION               VALUE
      0     4        (object header)           01 00 00 00 (00000001 00000000 00000000 00000000) (1) # mark
      4     4        (object header)           00 00 00 00 (00000000 00000000 00000000 00000000) (0) # mark
      8     4        (object header)           9b 1b 01 f8 (10011011 00011011 00000001 11111000) (-134145125) # klass
     12     4    int SimpleInt.state           0
Instance size: 16 bytes
Space losses: 0 bytes internal + 0 bytes external = 0 bytes total
```

如上所示，标记词目前似乎还没有存储任何重要的内容。

然而，如果我们在对象实例上调用`[System.identityHashCode()](https://web.archive.org/web/20220628094821/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/lang/System.html#identityHashCode(java.lang.Object)) `甚至`Object.hashCode() `，这将会改变:

```
System.out.println("The identity hash code is " + System.identityHashCode(instance));
System.out.println(ClassLayout.parseInstance(instance).toPrintable());
```

现在，我们可以将身份散列码视为标记字的一部分:

```
The identity hash code is 1702146597
SimpleInt object internals:
 OFFSET  SIZE   TYPE DESCRIPTION               VALUE
      0     4        (object header)           01 25 b2 74 (00000001 00100101 10110010 01110100) (1957831937)
      4     4        (object header)           65 00 00 00 (01100101 00000000 00000000 00000000) (101)
      8     4        (object header)           9b 1b 01 f8 (10011011 00011011 00000001 11111000) (-134145125)
     12     4    int SimpleInt.state           0
```

HotSpot JVM 将标识 hashcode 存储为标记字中的“25 b2 74 65”。最高有效字节是 65，因为 JVM 以 little-endian 格式存储该值。因此，要恢复十进制的哈希码值(1702146597)，我们必须以相反的顺序读取“25 b2 74 65”字节序列:

```
65 74 b2 25 = 01100101 01110100 10110010 00100101 = 1702146597
```

### 4.3.对齐

默认情况下，JVM 向对象添加足够的填充，使其大小是 8 的倍数。

例如，考虑一下`SimpleLong`类:

```
public class SimpleLong {
    private long state;
}
```

如果我们解析类布局:

```
System.out.println(ClassLayout.parseClass(SimpleLong.class).toPrintable());
```

然后 JOL 将打印内存布局:

```
SimpleLong object internals:
 OFFSET  SIZE   TYPE DESCRIPTION                               VALUE
      0    12        (object header)                           N/A
     12     4        (alignment/padding gap)                  
     16     8   long SimpleLong.state                          N/A
Instance size: 24 bytes
Space losses: 4 bytes internal + 0 bytes external = 4 bytes total
```

如上图所示，对象头和`long state`总共消耗 20 个字节。为了使这个大小是 8 字节的倍数，JVM 增加了 4 字节的填充。

**我们也可以通过`-XX:ObjectAlignmentInBytes `调谐标志改变默认的校准尺寸。**例如，对于同一个类，`-XX:ObjectAlignmentInBytes=16 `的内存布局是:

```
SimpleLong object internals:
 OFFSET  SIZE   TYPE DESCRIPTION                               VALUE
      0    12        (object header)                           N/A
     12     4        (alignment/padding gap)                  
     16     8   long SimpleLong.state                          N/A
     24     8        (loss due to the next object alignment)
Instance size: 32 bytes
Space losses: 4 bytes internal + 8 bytes external = 12 bytes total
```

对象头和`long `变量仍然总共消耗 20 个字节。所以，我们应该再增加 12 个字节，使它成为 16 的倍数。

如上所示，它添加了 4 个内部填充字节，以从偏移量 16 开始`long `变量(支持更对齐的访问)。然后，它在`long `变量后添加剩余的 8 个字节。

### 4.4.现场包装

**当一个类有多个字段时，JVM 可能会以最小化填充浪费的方式来分配这些字段。**例如，考虑一下`FieldsArrangement`类:

```
public class FieldsArrangement {
    private boolean first;
    private char second;
    private double third;
    private int fourth;
    private boolean fifth;
}
```

字段声明顺序及其在内存布局中的顺序是不同的:

```
OFFSET  SIZE      TYPE DESCRIPTION                               VALUE
      0    12           (object header)                           N/A
     12     4       int FieldsArrangement.fourth                  N/A
     16     8    double FieldsArrangement.third                   N/A
     24     2      char FieldsArrangement.second                  N/A
     26     1   boolean FieldsArrangement.first                   N/A
     27     1   boolean FieldsArrangement.fifth                   N/A
     28     4           (loss due to the next object alignment)
```

这背后的主要动机是最小化填充浪费。

### 4.5.锁

JVM 还维护标记字中的锁信息。让我们来看看实际情况:

```
public class Lock {}
```

如果我们创建这个类`,`的一个实例，它的内存布局将是:

```
Lock object internals:
 OFFSET  SIZE   TYPE DESCRIPTION                               VALUE
      0     4        (object header)                           01 00 00 00 
      4     4        (object header)                           00 00 00 00
      8     4        (object header)                           85 23 02 f8
     12     4        (loss due to the next object alignment)
Instance size: 16 bytes
```

但是，如果我们在此实例上同步:

```
synchronized (lock) {
    System.out.println(ClassLayout.parseInstance(lock).toPrintable());
}
```

内存布局更改为:

```
Lock object internals:
 OFFSET  SIZE   TYPE DESCRIPTION                               VALUE
      0     4        (object header)                           f0 78 12 03
      4     4        (object header)                           00 70 00 00
      8     4        (object header)                           85 23 02 f8
     12     4        (loss due to the next object alignment)
```

如上所示，**当我们锁定监视器时，标记字的位模式会改变。**

### 4.6.年龄和任期

为了将一个对象提升到老一代(当然是在世代[GCs](/web/20220628094821/https://www.baeldung.com/jvm-garbage-collectors)中)，JVM 需要跟踪每个对象的存活数。如前所述，JVM 也在标记字中维护这些信息。

为了模拟较小的 GC，我们将通过将一个对象赋给一个`volatile `变量来创建大量的垃圾。这样我们可以防止 JIT 编译器可能的[死代码消除](/web/20220628094821/https://www.baeldung.com/java-microbenchmark-harness#dead-code-elimination):

```
volatile Object consumer;
Object instance = new Object();
long lastAddr = VM.current().addressOf(instance);
ClassLayout layout = ClassLayout.parseInstance(instance);

for (int i = 0; i < 10_000; i++) {
    long currentAddr = VM.current().addressOf(instance);
    if (currentAddr != lastAddr) {
        System.out.println(layout.toPrintable());
    }

    for (int j = 0; j < 10_000; j++) {
        consumer = new Object();
    }

    lastAddr = currentAddr;
}
```

每次一个活动对象的地址改变，那可能是因为小的 GC 和幸存者空间之间的移动。对于每个变更，我们还打印新的对象布局以查看老化对象。

标记字的前 4 个字节随时间的变化如下:

```
09 00 00 00 (00001001 00000000 00000000 00000000)
              ^^^^
11 00 00 00 (00010001 00000000 00000000 00000000)
              ^^^^
19 00 00 00 (00011001 00000000 00000000 00000000)
              ^^^^
21 00 00 00 (00100001 00000000 00000000 00000000)
              ^^^^
29 00 00 00 (00101001 00000000 00000000 00000000)
              ^^^^
31 00 00 00 (00110001 00000000 00000000 00000000)
              ^^^^
31 00 00 00 (00110001 00000000 00000000 00000000)
              ^^^^
```

### 4.7.虚假分享和`@Contended`

**`jdk.internal.vm.annotation.Contended`注释(或者 Java 8 上的`sun.misc.Contended`)是对 JVM 的一个提示，用于隔离被注释的字段，以避免[错误共享](https://web.archive.org/web/20220628094821/https://alidg.me/blog/2020/4/24/thread-local-random#false-sharing)。**

简而言之，`Contended `注释在每个带注释的字段周围添加了一些填充，以将每个字段隔离在其自己的缓存行上。因此，这将影响内存布局。

为了更好地理解这一点，让我们考虑一个例子:

```
public class Isolated {

    @Contended
    private int v1;

    @Contended
    private long v2;
}
```

如果我们检查这个类的内存布局，我们会看到类似这样的内容:

```
Isolated object internals:
 OFFSET  SIZE   TYPE DESCRIPTION                               VALUE
      0    12        (object header)                           N/A
     12   128        (alignment/padding gap)                  
    140     4    int Isolated.i                                N/A
    144   128        (alignment/padding gap)                  
    272     8   long Isolated.l                                N/A
Instance size: 280 bytes
Space losses: 256 bytes internal + 0 bytes external = 256 bytes total
```

如上所示，JVM 在每个带注释的字段周围添加了 128 字节的填充。**在大多数现代机器中，高速缓存行的大小大约是 64/128 字节，因此有 128 字节的填充。**当然，我们可以用`-XX:ContendedPaddingWidth `调优标志来控制`Contended `填充大小。

请注意，`Contended `注释是 JDK 内部的，因此我们应该避免使用它。

此外，我们应该用`-XX:-RestrictContended `调优标志运行我们的代码；否则，注释不会生效。基本上，默认情况下，该注释仅供内部使用，禁用`RestrictContended `将为公共 API 解锁该特性。

### 4.8.数组

**正如我们之前提到的，数组长度也是数组 oop 的一部分。**例如，对于一个包含 3 个元素的`boolean`数组:

```
boolean[] booleans = new boolean[3];
System.out.println(ClassLayout.parseInstance(booleans).toPrintable());
```

内存布局如下所示:

```
[Z object internals:
 OFFSET  SIZE      TYPE DESCRIPTION                               VALUE
      0     4           (object header)                           01 00 00 00 # mark
      4     4           (object header)                           00 00 00 00 # mark
      8     4           (object header)                           05 00 00 f8 # klass
     12     4           (object header)                           03 00 00 00 # array length
     16     3   boolean [Z.<elements>                             N/A
     19     5           (loss due to the next object alignment)
Instance size: 24 bytes
Space losses: 0 bytes internal + 5 bytes external = 5 bytes total
```

这里，我们有 16 个字节的对象头，包含 8 个字节的标记字、4 个字节的 klass 字和 4 个字节的长度。紧接在对象头之后，我们有 3 个字节用于一个包含 3 个元素的`boolean `数组。

### 4.9.压缩引用

到目前为止，我们的示例是在启用了压缩引用的 64 位架构中执行的。

通过 8 字节对齐，我们可以使用多达 32 GB 的堆来压缩引用。**如果我们超越这个限制，甚至手动禁用压缩引用，那么 klass 字将消耗 8 个字节而不是 4 个字节。**

让我们看看当使用`-XX:-UseCompressedOops `调整标志禁用压缩 oops 时，同一阵列示例的内存布局:

```
[Z object internals:
 OFFSET  SIZE      TYPE DESCRIPTION                               VALUE
      0     4           (object header)                           01 00 00 00 # mark
      4     4           (object header)                           00 00 00 00 # mark
      8     4           (object header)                           28 60 d2 11 # klass
     12     4           (object header)                           01 00 00 00 # klass
     16     4           (object header)                           03 00 00 00 # length
     20     4           (alignment/padding gap)                  
     24     3   boolean [Z.<elements>                             N/A
     27     5           (loss due to the next object alignment)
```

正如承诺的那样，现在 klass 字又多了 4 个字节。

## 5.结论

在本教程中，我们看到了 JVM 如何在堆中布置对象和数组。

要进行更详细的探索，强烈建议查看 JVM 源代码的 [oops 部分。另外，](https://web.archive.org/web/20220628094821/http://hg.openjdk.java.net/jdk8/jdk8/hotspot/file/87ee5ee27509/src/share/vm/oops/)[阿列克谢·希皮洛夫](https://web.archive.org/web/20220628094821/https://shipilev.net/)在这方面有一篇[更深入的文章](https://web.archive.org/web/20220628094821/https://shipilev.net/jvm/objects-inside-out/)。

此外，更多 JOL 的[示例作为项目源代码的一部分可用。](https://web.archive.org/web/20220628094821/https://hg.openjdk.java.net/code-tools/jol/file/tip/jol-samples/src/main/java/org/openjdk/jol/samples/)

像往常一样，所有的例子都可以在 GitHub 上找到[。](https://web.archive.org/web/20220628094821/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-jvm-2)