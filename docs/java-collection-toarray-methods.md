# Collection.toArray(new T[0])或。toArray(新 T[大小])

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-collection-toarray-methods>

## 1.概观

Java 编程语言提供了[数组](/web/20220831131001/https://www.baeldung.com/java-arrays-guide)和[集合](https://web.archive.org/web/20220831131001/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/Collection.html)来将对象分组在一起。大多数情况下，一个集合由一个数组支持，并用一组方法建模，以处理它包含的元素。

在开发软件时，使用这两种数据结构是很常见的。因此，程序员需要一种桥接机制来将这些元素从一种形式转换成另一种形式。来自 [`Arrays`](/web/20220831131001/https://www.baeldung.com/java-util-arrays) 类的`[asList](/web/20220831131001/https://www.baeldung.com/java-arrays-aslist-vs-new-arraylist)`方法和 [`Collection`](https://web.archive.org/web/20220831131001/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/Collection.html) 接口的 [`toArray`](/web/20220831131001/https://www.baeldung.com/convert-array-to-list-and-list-to-array) 方法形成了这个桥梁。

在本教程中，我们将深入分析一个有趣的论点:**使用哪个`toArray`方法，为什么？**我们还将使用 [JMH](/web/20220831131001/https://www.baeldung.com/java-microbenchmark-harness) 协助的基准测试来支持这些论点。

## 2.`toArray`兔子洞

在漫无目的地调用`toArray `方法之前，让我们先了解一下盒子里面是什么。`Collection `接口提供了两种将集合转换成数组的方法:

```java
Object[] toArray()

<T> T[] toArray(T[] a)
```

这两种方法都返回包含集合中所有元素的数组。为了演示这一点，让我们创建一个自然数列表:

```java
List<Integer> naturalNumbers = IntStream
    .range(1, 10000)
    .boxed()
    .collect(Collectors.toList());
```

### 2.1.`Collection.toArray()`

`toArray()`方法分配一个新的内存数组，长度等于集合的大小。**内部，** **调用底层数组上的`[Arrays.copyOf](/web/20220831131001/https://www.baeldung.com/java-array-copy)` 支持集合**。因此，返回的数组没有对它的引用，可以安全使用:

```java
Object[] naturalNumbersArray = naturalNumbers.toArray();
```

然而，我们不能仅仅将结果转换成一个`Integer[]. `，这样做将导致一个 [`ClassCastException`](/web/20220831131001/https://www.baeldung.com/java-classcastexception) 。

### 2.2.`<T> T[] Collection.toArray(T[] a)`

与非参数化方法不同，这种方法接受预先分配的数组作为参数。此外，在方法定义中使用[泛型](/web/20220831131001/https://www.baeldung.com/java-generics)要求输入和返回数组具有相同的类型。这也解决了之前观察到的迭代一个`Object[]`的问题。

这种变体基于输入数组的大小特别有效:

*   如果预分配数组的长度小于集合的大小，则分配所需长度和相同类型的新数组:

```java
Integer[] naturalNumbersArray = naturalNumbers.toArray(new Integer[0]);
```

*   如果输入数组足够大，可以包含集合的元素，则返回时会包含这些元素:

```java
Integer[] naturalNumbersArray = naturalNumbers.toArray(new Integer[naturalNumbers.size]);
```

现在，让我们回到最初的问题，选择速度更快、表现更好的候选人。

## 3.性能测试

让我们从一个简单的实验开始，该实验比较了**零大小(`toArray(new T[0]` )** **和预大小(`toArray(new T[size]`)变体**。我们将使用流行的`ArrayList` 和`AbstractCollection`支持的`TreeSet`进行测试。此外，我们将包括不同大小(小、中和大)的集合，以获得广泛的样本数据。

### 3.1.JMH 基准

接下来，让我们为我们的试验建立一个 JMH (Java 微基准测试工具)基准。我们将为基准配置集合的大小和类型参数:

```java
@Param({ "10", "10000", "10000000" })
private int size;

@Param({ "array-list", "tree-set" })
private String type;
```

此外，我们将为零大小和预设大小的`toArray`变体定义基准方法:

```java
@Benchmark
public String[] zero_sized() {
    return collection.toArray(new String[0]);
}

@Benchmark
public String[] pre_sized() {
    return collection.toArray(new String[collection.size()]);
}
```

### 3.2.基准测试结果

在一台 8 vCPU、32 GB RAM、Linux x86_64 虚拟机上运行上述基准测试，运行 JMH(1.28 版)和 JDK (1.8.0_292 版)的结果如下所示。该分数揭示了每个基准测试方法的平均执行时间，单位为每操作纳秒数。

值越低，性能越好:

```java
Benchmark                   (size)      (type)  Mode  Cnt          Score          Error  Units

<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>

TestBenchmark.zero_sized        10  array-list  avgt   15         24.939 ±        1.202  ns/op
TestBenchmark.pre_sized         10  array-list  avgt   15         38.196 ±        3.767  ns/op
----------------------------------------------------------------------------------------------
TestBenchmark.zero_sized     10000  array-list  avgt   15      15244.367 ±      238.676  ns/op
TestBenchmark.pre_sized      10000  array-list  avgt   15      21263.225 ±      802.684  ns/op
----------------------------------------------------------------------------------------------
TestBenchmark.zero_sized  10000000  array-list  avgt   15   82710389.163 ±  6616266.065  ns/op
TestBenchmark.pre_sized   10000000  array-list  avgt   15  100426920.878 ± 10381964.911  ns/op

<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>

TestBenchmark.zero_sized        10    tree-set  avgt   15         66.802 ±        5.667  ns/op
TestBenchmark.pre_sized         10    tree-set  avgt   15         66.009 ±        4.504  ns/op
----------------------------------------------------------------------------------------------
TestBenchmark.zero_sized     10000    tree-set  avgt   15      85141.622 ±     2323.420  ns/op
TestBenchmark.pre_sized      10000    tree-set  avgt   15      89090.155 ±     4895.966  ns/op
----------------------------------------------------------------------------------------------
TestBenchmark.zero_sized  10000000    tree-set  avgt   15  211896860.317 ± 21019102.769  ns/op
TestBenchmark.pre_sized   10000000    tree-set  avgt   15  212882486.630 ± 20921740.965  ns/op
```

仔细观察上述结果后，很明显，在这个试验中，对于所有大小和集合类型，零大小方法调用都赢了。

目前，这些数字只是数据。要有详细的了解，我们来深入挖掘分析一下。

### 3.3.分配率

假设，**由于优化了每个操作的内存分配**，可以假设零大小的`toArray `方法调用比预先确定大小的方法调用执行得更好。让我们通过执行另一个基准测试并量化基准测试方法的**平均分配率——每次操作分配的内存字节数——来澄清这一点。**

JMH 提供了一个 [GC 分析器](https://web.archive.org/web/20220831131001/http://mail.openjdk.java.net/pipermail/jmh-dev/2015-April/001828.html) ( `-prof gc`)，它在内部使用`ThreadMXBean#getThreadAllocatedBytes` 来计算每个@ `Benchmark`的分配率:

```java
Benchmark                                                    (size)      (type)  Mode  Cnt          Score           Error   Units

<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>

TestBenchmark.zero_sized:·gc.alloc.rate.norm                     10  array-list  avgt   15         72.000 ±         0.001    B/op
TestBenchmark.pre_sized:·gc.alloc.rate.norm                      10  array-list  avgt   15         56.000 ±         0.001    B/op
---------------------------------------------------------------------------------------------------------------------------------
TestBenchmark.zero_sized:·gc.alloc.rate.norm                  10000  array-list  avgt   15      40032.007 ±         0.001    B/op
TestBenchmark.pre_sized:·gc.alloc.rate.norm                   10000  array-list  avgt   15      40016.010 ±         0.001    B/op
---------------------------------------------------------------------------------------------------------------------------------
TestBenchmark.zero_sized:·gc.alloc.rate.norm               10000000  array-list  avgt   15   40000075.796 ±         8.882    B/op
TestBenchmark.pre_sized:·gc.alloc.rate.norm                10000000  array-list  avgt   15   40000062.213 ±         4.739    B/op

<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>

TestBenchmark.zero_sized:·gc.alloc.rate.norm                     10    tree-set  avgt   15         56.000 ±         0.001    B/op
TestBenchmark.pre_sized:·gc.alloc.rate.norm                      10    tree-set  avgt   15         56.000 ±         0.001    B/op
---------------------------------------------------------------------------------------------------------------------------------
TestBenchmark.zero_sized:·gc.alloc.rate.norm                  10000    tree-set  avgt   15      40055.818 ±        16.723    B/op
TestBenchmark.pre_sized:·gc.alloc.rate.norm                   10000    tree-set  avgt   15      41069.423 ±      1644.717    B/op
---------------------------------------------------------------------------------------------------------------------------------
TestBenchmark.zero_sized:·gc.alloc.rate.norm               10000000    tree-set  avgt   15   40000155.947 ±         9.416    B/op
TestBenchmark.pre_sized:·gc.alloc.rate.norm                10000000    tree-set  avgt   15   40000138.987 ±         7.987    B/op
```

显然，上述数字证明，无论集合类型或`toArray`变量如何，相同规模的分配率大致相同。因此，**否定了任何推测性假设，即预先调整大小的`toArray`变体由于其内存分配率**的不规则性而表现不同。

### 3.4.`toArray(T[] a)`内部构件

为了进一步找出问题的原因，让我们深入研究一下`ArrayList`的内部原因:

```java
if (a.length < size)
    return (T[]) Arrays.copyOf(elementData, size, a.getClass());
System.arraycopy(elementData, 0, a, 0, size);
if (a.length > size)
    a[size] = null;
return a;
```

基本上，根据预分配数组的长度，要么是一个`Arrays.copyOf`要么是一个[本机](/web/20220831131001/https://www.baeldung.com/java-native) `System.arraycopy` 方法调用将集合的底层元素复制到一个数组中。

此外，注意一下`copyOf `方法，很明显，首先创建一个长度等于集合大小的复制数组，然后是`System.arraycopy `调用:

```java
T[] copy = ((Object)newType == (Object)Object[].class)
    ? (T[]) new Object[newLength]
    : (T[]) Array.newInstance(newType.getComponentType(), newLength);
System.arraycopy(original, 0, copy, 0,
    Math.min(original.length, newLength));
```

当零大小和预大小方法最终都调用本机`System.arraycopy`方法时，零大小方法调用如何更快？

**奥秘在于为外部预分配的数组执行零初始化所花费的 CPU 时间的直接成本，这使得`toArray(new T[size])` 方法慢得多。**

## 4.零初始化

Java 语言规范指示**新实例化的数组和对象应该具有默认的字段值**，而不是内存中不规则的剩余值。因此，运行时必须将预先分配的存储清零。[基准测试实验](https://web.archive.org/web/20220831131001/https://bugs.openjdk.java.net/browse/JDK-8146828)已经证明零大小数组方法调用设法避免了归零，但是预先确定大小的情况却不能。

让我们考虑几个基准:

```java
@Benchmark
public Foo[] arraycopy_srcLength() {
    Object[] src = this.src;
    Foo[] dst = new Foo[size];
    System.arraycopy(src, 0, dst, 0, src.length);
    return dst;
}

@Benchmark
public Foo[] arraycopy_dstLength() {
    Object[] src = this.src;
    Foo[] dst = new Foo[size];
    System.arraycopy(src, 0, dst, 0, dst.length);
    return dst;
}
```

[实验观察](https://web.archive.org/web/20220831131001/https://bugs.openjdk.java.net/browse/JDK-8146828)显示在 **`arraycopy_srcLength`基准中紧随数组分配的`System.arraycopy`能够避免`dst `数组**的预归零。然而， **`arraycopy_dstLength` 执行无法避免预调零**。

巧合的是，后一种`**arraycopy_dstLength** `情况类似于预先确定大小的阵列方法`**collection.toArray(new String[collection.size()])** `，其中无法消除调零，因此其速度较慢。

## 5.较新 JDK 的基准测试

最后，让我们在最近发布的 JDK 上执行最初的基准测试，并配置 JVM 使用更新的、改进很多的 [G1 垃圾收集器](https://web.archive.org/web/20220831131001/https://docs.oracle.com/javase/9/gctuning/garbage-first-garbage-collector.htm#JSGCT-GUID-0394E76A-1A8F-425E-A0D0-B48A3DC82B42):

```java
# VM version: JDK 11.0.2, OpenJDK 64-Bit Server VM, 11.0.2+9
-----------------------------------------------------------------------------------
Benchmark                    (size)      (type)  Mode  Cnt    Score    Error  Units
-----------------------------------------------------------------------------------
ToArrayBenchmark.zero_sized     100  array-list  avgt   15  199.920 ± 11.309  ns/op
ToArrayBenchmark.pre_sized      100  array-list  avgt   15  237.342 ± 14.166  ns/op
-----------------------------------------------------------------------------------
ToArrayBenchmark.zero_sized     100    tree-set  avgt   15  819.306 ± 85.916  ns/op
ToArrayBenchmark.pre_sized      100    tree-set  avgt   15  972.771 ± 69.743  ns/op
```

```java
###################################################################################

# VM version: JDK 14.0.2, OpenJDK 64-Bit Server VM, 14.0.2+12-46
------------------------------------------------------------------------------------
Benchmark                    (size)      (type)  Mode  Cnt    Score    Error   Units
------------------------------------------------------------------------------------
ToArrayBenchmark.zero_sized     100  array-list  avgt   15  158.344 ±   3.862  ns/op
ToArrayBenchmark.pre_sized      100  array-list  avgt   15  214.340 ±   5.877  ns/op
------------------------------------------------------------------------------------
ToArrayBenchmark.zero_sized     100    tree-set  avgt   15  877.289 ± 132.673  ns/op
ToArrayBenchmark.pre_sized      100    tree-set  avgt   15  934.550 ± 148.660  ns/op

####################################################################################

# VM version: JDK 15.0.2, OpenJDK 64-Bit Server VM, 15.0.2+7-27
------------------------------------------------------------------------------------
Benchmark                    (size)      (type)  Mode  Cnt    Score     Error  Units
------------------------------------------------------------------------------------
ToArrayBenchmark.zero_sized     100  array-list  avgt   15  147.925 ±   3.968  ns/op
ToArrayBenchmark.pre_sized      100  array-list  avgt   15  213.525 ±   6.378  ns/op
------------------------------------------------------------------------------------
ToArrayBenchmark.zero_sized     100    tree-set  avgt   15  820.853 ± 105.491  ns/op
ToArrayBenchmark.pre_sized      100    tree-set  avgt   15  947.433 ± 123.782  ns/op

####################################################################################

# VM version: JDK 16, OpenJDK 64-Bit Server VM, 16+36-2231
------------------------------------------------------------------------------------
Benchmark                    (size)      (type)  Mode  Cnt    Score     Error  Units
------------------------------------------------------------------------------------
ToArrayBenchmark.zero_sized     100  array-list  avgt   15  146.431 ±   2.639  ns/op
ToArrayBenchmark.pre_sized      100  array-list  avgt   15  214.117 ±   3.679  ns/op
------------------------------------------------------------------------------------
ToArrayBenchmark.zero_sized     100    tree-set  avgt   15  818.370 ± 104.643  ns/op
ToArrayBenchmark.pre_sized      100    tree-set  avgt   15  964.072 ± 142.008  ns/op

####################################################################################
```

有趣的是， **`toArray(new T[0]) `方法一直比** `**toArray(new T[size])**`快。此外，它的性能随着 JDK 的每一个新版本而不断提高。

### 5.1.Java 11 `Collection.toArray(IntFunction<T[]>) `

在 Java 11 中，`Collection`接口引入了一个新的[默认](/web/20220831131001/https://www.baeldung.com/java-static-default-methods#default-interface-methods-in-action) `toArray`方法，该方法接受一个`[IntFunction](/web/20220831131001/https://www.baeldung.com/java-8-functional-interfaces#Primitive)<T[]>`生成器作为参数(该生成器将生成一个所需类型和提供长度的新数组)。

**该方法通过调用值为零**的生成器函数来保证`new T[0]` 数组的初始化，从而确保总是执行更快更好的零大小`toArray(T[])`方法。

## 6.结论

在本文中，我们探讨了`Collection `接口的不同的`toArray` 重载方法。我们还在不同的 JDK 上利用 JMH 微基准测试工具进行了性能测试。

我们了解了零位调整的必要性和影响，并观察了内部分配的阵列如何消除零位调整，从而赢得了性能竞赛。最后，我们可以肯定地得出结论，`toArray(new T[0])` 变体比`toArray(new T[size])` 更快，因此，当我们必须将集合转换为数组时，它应该总是首选。

和往常一样，本文中使用的代码可以在 GitHub 上找到[。](https://web.archive.org/web/20220831131001/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-collections-4)