# JVM 中数组长度存储在哪里？

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-jvm-array-length>

## 1.概观

在这个快速教程中，我们将看到 HotSpot JVM 如何存储数组长度以及存储在哪里。

通常，运行时数据区的内存布局不是 JVM 规范的一部分，而是留给实现者来决定。因此，每个 JVM 实现可能有不同的策略来布局内存中的对象和数组。

在本教程中，我们将关注一个特定的 JVM 实现:HotSpot JVM。我们也可以互换使用 JVM 和 HotSpot JVM 这两个术语。

## 2.属国

为了检查 JVM 中数组的内存布局，我们将使用 Java 对象布局( [JOL](https://web.archive.org/web/20221208143956/https://openjdk.java.net/projects/code-tools/jol/) )工具。因此，我们需要添加 [`jol-core`](https://web.archive.org/web/20221208143956/https://search.maven.org/artifact/org.openjdk.jol/jol-core) 的依赖关系:

```java
<dependency> 
    <groupId>org.openjdk.jol</groupId> 
    <artifactId>jol-core</artifactId>    
    <version>0.10</version> 
</dependency>
```

## 3.数组长度

**HotSpot JVM 使用一种称为普通对象指针的数据结构([哎呀](https://web.archive.org/web/20221208143956/https://github.com/openjdk/jdk15/tree/master/src/hotspot/share/oops))来表示指向对象的指针。**更具体地说，HotSpot JVM 用一种叫做 [`arrayOop`](https://web.archive.org/web/20221208143956/https://github.com/openjdk/jdk15/blob/e208d9aa1f185c11734a07db399bab0be77ef15f/src/hotspot/share/oops/arrayOop.hpp#L35) 的特殊 OOP 来表示数组。每个`arrayOop`包括一个对象头，其详细信息如下:

*   一个标记字，用于存储身份散列码或 GC 信息
*   一个 klass 字存储通用类元数据
*   代表数组长度的 4 个字节

因此，**JVM 将数组长度存储在对象头**中。

让我们通过检查阵列的内存布局来验证这一点:

```java
int[] ints = new int[42];
System.out.println(ClassLayout.parseInstance(ints).toPrintable());
```

如上所示，我们正在从现有的数组实例解析内存布局。下面是 JVM 如何布置`int[]`:

```java
[I object internals:
 OFFSET  SIZE   TYPE DESCRIPTION               VALUE
      0     4        (object header)           01 00 00 00 (00000001 00000000 00000000 00000000) (1) # mark
      4     4        (object header)           00 00 00 00 (00000000 00000000 00000000 00000000) (0) # mark
      8     4        (object header)           6d 01 00 f8 (01101101 00000001 00000000 11111000) (-134217363) #klass
     12     4        (object header)           2a 00 00 00 (00101010 00000000 00000000 00000000) (42) # array length
     16   168    int [I.<elements>             N/A
Instance size: 184 bytes
```

如前所述，JVM 将数组长度存储在 mark 和 klass 字之后的对象头中。此外，数组长度将以 4 个字节存储，因此它不能大于 32 位整数的最大值。

在对象头之后，JVM 存储实际的数组元素。因为我们有一个 42 个整数的数组，所以数组的总大小是 168 字节——42 乘以 4。

## 4.结论

在这个简短的教程中，我们看到了 JVM 如何存储数组长度。

像往常一样，所有的例子都可以在 GitHub 上找到[。](https://web.archive.org/web/20221208143956/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-jvm-2)