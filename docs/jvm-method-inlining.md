# JVM 中的方法内联

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/jvm-method-inlining>

## 1。简介

在本教程中，我们将看看 Java 虚拟机中的方法内联是什么，以及它是如何工作的。

我们还将看到如何从 JVM 中获取和读取与内联相关的信息，以及我们可以利用这些信息做些什么来优化我们的代码。

## 2.什么是方法内联？

基本上，**内联是一种在运行时优化编译的源代码的方法，它用代码体替换最常执行的方法的调用。**

尽管涉及到编译，但它不是由传统的`javac`编译器执行的，而是由 JVM 本身执行的。更准确地说，**是实时(JIT)编译器**的责任，它是 JVM 的一部分；`javac`只生成一个字节码，让 JIT 变魔术，优化源代码。

这种方法最重要的结果之一是，如果我们使用旧的 Java 编译代码，同样的。在新的 JVM 上文件会更快。这样我们不需要重新编译源代码，只需要更新 Java。

## 3.JIT 是怎么做到的？

本质上， **JIT 编译器试图内联我们经常调用的方法，这样我们可以避免方法调用**的开销。在决定是否内联一个方法时，需要考虑两件事情。

首先，它使用计数器来记录我们调用该方法的次数。当该方法被调用超过特定次数时，它就变成“热的”。这个阈值默认设置为 10，000，但是我们可以在 Java 启动时通过 JVM 标志来配置它。我们绝对不希望内联所有的东西，因为这很耗时，而且会产生巨大的字节码。

我们应该记住，只有当我们达到稳定状态时，内联才会发生。这意味着我们需要多次重复执行，以便为 JIT 编译器提供足够的分析信息。

此外,“热”并不保证该方法将被内联。如果它太大，JIT 不会内联它。可接受的大小受到`-XX:FreqInlineSize=`标志的限制，该标志指定了一个方法内联的字节码指令的最大数量。

尽管如此，强烈建议不要更改这个标志的默认值，除非我们绝对确定知道它会产生什么影响。默认值取决于平台——对于 64 位 Linux，它是 325。

**JIT 一般内联`static`、`private`、**、**或`final`方法**。虽然 **`public`方法也是内联的候选方法，但并不是每个公共方法都必须被内联。JVM 需要确定这种方法只有一个实现**。任何额外的子类都会阻止内联，性能将不可避免地下降。

## 4.寻找热门方法

我们当然不想猜测 JIT 在做什么。因此，我们需要某种方法来查看哪些方法是内联的，哪些是未内联的。我们可以很容易地实现这一点，并通过在启动时设置一些额外的 JVM 标志将所有这些信息记录到标准输出中:

```java
-XX:+PrintCompilation -XX:+UnlockDiagnosticVMOptions -XX:+PrintInlining
```

当 JIT 编译发生时，第一个标志将被记录。第二个标志启用了包括`-XX:+PrintInlining`在内的附加标志，这些标志将打印哪些方法被内联以及在哪里被内联。

这将以树的形式向我们展示内联的方法。树叶被标注并标记有以下选项之一:

*   `inline (hot)`–该方法被标记为热方法，并被内联
*   这个方法不热，但是它生成的字节码太大了，所以没有内联
*   这是一个热门的方法，但是因为字节码太大，所以没有内联

**我们应该关注第三个值，尝试优化带有“热方法太大”标签的方法。**

一般来说，如果我们发现一个带有非常复杂的条件语句的热方法，我们应该尝试分离出`if-`语句的内容，并增加粒度，以便 JIT 可以优化代码。这同样适用于`switch`和`for-`循环语句。

我们可以得出结论，为了优化代码，我们不需要手动方法内联。JVM 做起来更有效率，我们可能会使代码很长，很难理解。

### 4.1.例子

现在让我们看看如何在实践中检验这一点。我们将首先创建一个简单的类来计算第一个`N`连续正整数的和:

```java
public class ConsecutiveNumbersSum {

    private long totalSum;
    private int totalNumbers;

    public ConsecutiveNumbersSum(int totalNumbers) {
        this.totalNumbers = totalNumbers;
    }

    public long getTotalSum() {
        totalSum = 0;
        for (int i = 0; i < totalNumbers; i++) {
            totalSum += i;
        }
        return totalSum;
    }
}
```

接下来，一个简单的方法将利用类来执行计算:

```java
private static long calculateSum(int n) {
    return new ConsecutiveNumbersSum(n).getTotalSum();
}
```

最后，我们将多次调用该方法，看看会发生什么:

```java
for (int i = 1; i < NUMBERS_OF_ITERATIONS; i++) {
    calculateSum(i);
}
```

在第一次运行中，我们将运行它 1000 次(小于上面提到的阈值 10，000)。如果我们在输出中搜索`calculateSum()`方法，我们不会找到它。这是意料之中的，因为我们调用它的次数不够多。

如果我们现在将迭代次数更改为 15，000 次，并再次搜索输出，我们将看到:

```java
664 262 % com.baeldung.inlining.InliningExample::main @ 2 (21 bytes)
  @ 10   com.baeldung.inlining.InliningExample::calculateSum (12 bytes)   inline (hot)
```

我们可以看到，这次该方法满足了内联的条件，JVM 对它进行了内联。

值得一提的是，如果方法太大，无论迭代多少次，JIT 都不会内联它。我们可以通过在运行应用程序时添加另一个标志来检查这一点:

```java
-XX:FreqInlineSize=10
```

正如我们在前面的输出中看到的，我们的方法的大小是 12 字节。`-XX:` `FreqInlineSize`标志将符合内联条件的方法大小限制为 10 个字节。因此，这次不应该发生内联。事实上，我们可以通过查看输出来确认这一点:

```java
330 266 % com.baeldung.inlining.InliningExample::main @ 2 (21 bytes)
  @ 10   com.baeldung.inlining.InliningExample::calculateSum (12 bytes)   hot method too big
```

尽管为了便于说明，我们在这里更改了标志值，但我们必须强调，除非绝对必要，否则不要更改`-XX:FreqInlineSize`标志的默认值。

## 5.结论

在本文中，我们看到了 JVM 中的方法内联以及 JIT 是如何做的。我们描述了如何检查我们的方法是否有资格进行内联，并建议如何通过尝试减少频繁调用的长方法的大小来利用这些信息，这些长方法太大而无法进行内联。

最后，我们举例说明了如何在实践中识别一个热方法。

文章中提到的所有代码片段都可以在我们的 GitHub 资源库中找到。