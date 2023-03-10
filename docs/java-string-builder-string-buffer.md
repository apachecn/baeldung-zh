# Java 中的 StringBuilder 与 StringBuffer

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-string-builder-string-buffer>

## 1。概述

在这篇短文中，我们将看看 Java 中 [`StringBuilder`](https://web.archive.org/web/20221001102523/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/lang/StringBuilder.html) 和 [`StringBuffer`](https://web.archive.org/web/20221001102523/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/lang/StringBuffer.html) 的异同。

简单地说，在 Java 1.5 中引入了 `StringBuilder`来代替`StringBuffer`。

## 2。相似之处

**`StringBuilder`和`StringBuffer`都创建持有可变字符序列的对象。**让我们看看这是如何工作的，以及它如何与不可变的`String`类进行比较:

```java
String immutable = "abc";
immutable = immutable + "def";
```

尽管看起来我们通过追加`“def”`来修改同一个对象，但我们是在创建一个新的对象，因为`String`实例不能被修改。

当使用`StringBuffer`或`StringBuilder,`时，我们可以使用`append()`方法:

```java
StringBuffer sb = new StringBuffer("abc");
sb.append("def");
```

在这种情况下，没有创建新的对象。我们已经在`sb`实例上调用了`append()`方法，并修改了它的内容。`StringBuffer`和`StringBuilder`是可变对象。

## 3。差异

**`StringBuffer`是同步的，因此是线程安全的。** `StringBuilder`与`StringBuffer` API 兼容，但不保证同步。

因为它不是线程安全的实现，所以速度更快，建议在不需要线程安全的地方使用它。

### 3.1。性能

在小的迭代中，性能差异是微不足道的。让我们用 [JMH](/web/20221001102523/https://www.baeldung.com/java-jvm-warmup) 做一个快速的微基准测试:

```java
@State(Scope.Benchmark)
public static class MyState {
    int iterations = 1000;
    String initial = "abc";
    String suffix = "def";
}

@Benchmark
public StringBuffer benchmarkStringBuffer(MyState state) {
    StringBuffer stringBuffer = new StringBuffer(state.initial);
    for (int i = 0; i < state.iterations; i++) {
        stringBuffer.append(state.suffix);
    }
    return stringBuffer;
}

@Benchmark
public StringBuilder benchmarkStringBuilder(MyState state) {
    StringBuilder stringBuilder = new StringBuilder(state.initial);
    for (int i = 0; i < state.iterations; i++) {
        stringBuilder.append(state.suffix);
    }
    return stringBuilder;
}
```

我们使用了默认的`Throughput`模式——即单位时间内的操作数(分数越高越好),它给出了:

```java
Benchmark                                          Mode  Cnt      Score      Error  Units
StringBufferStringBuilder.benchmarkStringBuffer   thrpt  200  86169.834 ±  972.477  ops/s
StringBufferStringBuilder.benchmarkStringBuilder  thrpt  200  91076.952 ± 2818.028  ops/s
```

如果我们将迭代次数从 1k 增加到 1m，那么我们得到:

```java
Benchmark                                          Mode  Cnt   Score   Error  Units
StringBufferStringBuilder.benchmarkStringBuffer   thrpt  200  77.178 ± 0.898  ops/s
StringBufferStringBuilder.benchmarkStringBuilder  thrpt  200  85.769 ± 1.966  ops/s
```

但是，我们要记住，这是一个微观基准，它可能会也可能不会对应用程序的实际性能产生真正的影响。

## 4。结论

**简单地说，`StringBuffer` 是一个线程安全的实现，因此比`StringBuilder`慢。**

在单线程程序中，我们可以采用`StringBuilder`。然而，**`StringBuilder`相对于`StringBuffer`的性能提升可能太小，不足以证明到处替换它的合理性。**在用一个实现替换另一个实现之前，对应用程序进行概要分析并理解其运行时性能特征总是一个好主意。

最后，和往常一样，讨论中使用的代码可以在 GitHub 上找到[。](https://web.archive.org/web/20221001102523/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-string-apis)