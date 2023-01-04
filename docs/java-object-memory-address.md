# Java 中对象的内存地址

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-object-memory-address>

## 1.概观

在这个快速教程中，我们将看到如何在 Java 中找到对象的内存地址。

在进一步讨论之前，值得一提的是，运行时数据区的内存布局不是 JVM 规范的一部分，而是留给实现者来决定。因此，每个 JVM 实现可能有不同的策略来布局内存中的对象和数组。这反过来会影响内存地址。

在本教程中，我们将关注一个特定的 JVM 实现:HotSpot JVM。在整个教程中，我们也可能会互换使用 JVM 和 HotSpot JVM 这两个术语。

## 2.属国

为了找到 JVM 中对象的内存地址，我们将使用 Java 对象布局( [JOL](https://web.archive.org/web/20220628052827/https://openjdk.java.net/projects/code-tools/jol/) )工具。因此，我们需要添加 [`jol-core`](https://web.archive.org/web/20220628052827/https://search.maven.org/artifact/org.openjdk.jol/jol-core) 的依赖关系:

```
<dependency> 
    <groupId>org.openjdk.jol</groupId> 
    <artifactId>jol-core</artifactId>    
    <version>0.10</version> 
</dependency>
```

## 3.存储地址

为了找到 JVM 中特定对象的内存地址，我们可以使用`[addressOf()](https://web.archive.org/web/20220628052827/https://www.javadoc.io/doc/org.openjdk.jol/jol-core/latest/org/openjdk/jol/vm/VirtualMachine.html#sizeOf-java.lang.Object-) `方法:

```
String answer = "42";

System.out.println("The memory address is " + VM.current().addressOf(answer));
```

这将打印:

```
The memory address is 31864981224
```

HotSpot JVM 中有不同的[压缩参考模式](https://web.archive.org/web/20220628052827/https://shipilev.net/jvm/anatomy-quarks/23-compressed-references/#_compressed_references)。由于这些模式，该值可能不完全准确。因此，我们不应该基于这个地址执行一些本机内存操作，因为这可能会导致奇怪的内存损坏。

此外，大多数 JVM 实现中的内存地址会随着 GC 不时地移动对象而发生变化。

## 4.身份散列码

有一种常见的误解，认为 JVM 中对象的内存地址被表示为它们默认的`toString `实现的一部分，比如`[[email protected]](/web/20220628052827/https://www.baeldung.com/cdn-cgi/l/email-protection)`。也就是说，许多人认为`“60addb54” `是那个特定对象的内存地址。

让我们检查一下这个假设:

```
Object obj = new Object();

System.out.println("Memory address: " + VM.current().addressOf(obj));
System.out.println("toString: " + obj);
System.out.println("hashCode: " + obj.hashCode());
System.out.println("hashCode: " + System.identityHashCode(obj));
```

这将打印以下内容:

```
Memory address: 31879960584
toString: [[email protected]](/web/20220628052827/https://www.baeldung.com/cdn-cgi/l/email-protection)
hashCode: 1622006612
hashCode: 1622006612
```

非常有趣的是，`“60addb54” `是哈希码的十六进制版本，即 1622006612。`[hashCode()](/web/20220628052827/https://www.baeldung.com/java-hashcode)`方法是所有 Java 对象的通用方法之一。**当我们没有为一个类声明一个`hashCode() `** **方法时，Java 会为它使用 identity hash 代码。**

如上图，**身份哈希码**(在`toString``@`之后的那部分)**和内存地址不一样**。

## 5.结论

在这个简短的教程中，我们看到了如何在 Java 中找到对象的内存地址。

像往常一样，所有的例子都可以在 GitHub 上找到[。](https://web.archive.org/web/20220628052827/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-jvm-2)