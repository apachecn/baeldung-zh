# FastUtil 指南

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/fastutil>

## 1.介绍

在本教程中，我们将会看到`[FastUtil](https://web.archive.org/web/20221205144339/http://fastutil.di.unimi.it/) `库。

首先，我们将编写一些特定于类型的集合的例子。

然后，我们将分析赋予`FastUtil `名称的**性能。**

最后，让我们来看看 **`FastUtil`的`BigArray `实用程序。**

## 2.特征

Java 库试图扩展 Java 集合框架。它提供了**特定类型的映射、集合、列表和队列**，内存占用更小，访问和插入速度更快。`FastUtil `还提供了一套**工具，用于处理和操作大型(64 位)数组、集合和列表。**

该库还包括大量用于二进制和文本文件的实用输入/输出类。

它的最新版本`FastUtil 8,` 也发布了一系列[特定类型的功能](https://web.archive.org/web/20221205144339/http://fastutil.di.unimi.it/docs/it/unimi/dsi/fastutil/Function.html)，扩展了 JDK 的`[Functional Interfaces](https://web.archive.org/web/20221205144339/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/function/package-summary.html).`

### 2.1.速度

**在许多情况下，`FastUtil `实现是最快的。**作者甚至提供了他们自己的深度[基准测试报告](https://web.archive.org/web/20221205144339/http://java-performance.info/hashmap-overview-jdk-fastutil-goldman-sachs-hppc-koloboke-trove-january-2015/)，将其与类似的库进行比较，包括`HPPC `和`Trove.`

在本教程中，我们将使用 [Java 微基准测试工具(JMH)](/web/20221205144339/https://www.baeldung.com/java-microbenchmark-harness) 来定义我们自己的基准。

## 3.全尺寸依赖性

除了通常的`JUnit`依赖关系，我们将在本教程中使用 [`FastUtils`](https://web.archive.org/web/20221205144339/https://search.maven.org/search?q=g:it.unimi.dsi%20AND%20a:fastutil) 和`[JMH](https://web.archive.org/web/20221205144339/https://search.maven.org/search?q=a:jmh-core%20OR%20a:jmh-generator-annprocess)` [依赖关系](https://web.archive.org/web/20221205144339/https://search.maven.org/search?q=a:jmh-core%20OR%20a:jmh-generator-annprocess)。

在我们的`pom.xml `文件中，我们需要以下依赖项:

```java
<dependency>
    <groupId>it.unimi.dsi</groupId>
    <artifactId>fastutil</artifactId>
    <version>8.2.2</version>
</dependency>
<dependency>
    <groupId>org.openjdk.jmh</groupId>
    <artifactId>jmh-core</artifactId>
    <version>1.35</version>
    <scope>test</scope>
</dependency>
<dependency>
    <groupId>org.openjdk.jmh</groupId>
    <artifactId>jmh-generator-annprocess</artifactId>
    <version>1.35</version>
    <scope>test</scope>
</dependency>
```

或者对于 Gradle 用户:

```java
testCompile group: 'org.openjdk.jmh', name: 'jmh-core', version: '1.19'
testCompile group: 'org.openjdk.jmh', name: 'jmh-generator-annprocess', version: '1.19'
compile group: 'it.unimi.dsi', name: 'fastutil', version: '8.2.2'
```

### 3.1.定制的 Jar 文件

由于缺乏泛型，`FastUtils `生成了大量特定于类型的类。不幸的是，这导致了一个**巨大的 jar 文件。**

然而，对我们来说幸运的是，`FastUtils ` **包含了一个`find-deps.sh `脚本，它允许生成更小、更集中的 jar**，只包含我们希望在应用程序中使用的类。

## 4.特定于类型的集合

在开始之前，让我们快速浏览一下实例化特定于类型的集合的[简单过程。让我们选择一个使用`doubles. `存储键和值的`HashMap`](/web/20221205144339/https://www.baeldung.com/java-map-primitives)

为此，`FastUtils `提供了一个`[Double2DoubleMap](https://web.archive.org/web/20221205144339/http://fastutil.di.unimi.it/docs/it/unimi/dsi/fastutil/doubles/Double2DoubleMap.html) `接口和一个`[Double2DoubleOpenHashMap](https://web.archive.org/web/20221205144339/http://fastutil.di.unimi.it/docs/it/unimi/dsi/fastutil/doubles/Double2DoubleOpenHashMap.html) `实现:

```java
Double2DoubleMap d2dMap = new Double2DoubleOpenHashMap();
```

现在我们已经实例化了我们的类，我们可以简单地用 Java Collections API 中的任何`Map `填充数据:

```java
d2dMap.put(2.0, 5.5);
d2dMap.put(3.0, 6.6);
```

最后，我们可以检查数据是否已正确添加:

```java
assertEquals(5.5, d2dMap.get(2.0));
```

### 4.1.表演

**`FastUtils` 侧重于它的表演性实现。在本节中，我们将利用 JMH 来验证这一事实。**让我们对比一下 Java 集合`HashSet<Integer>` 实现与`FastUtil's ` [`IntOpenHashSet`](https://web.archive.org/web/20221205144339/http://fastutil.di.unimi.it/docs/it/unimi/dsi/fastutil/ints/IntOpenHashSet.html) 。

首先，让我们看看如何实现`IntOpenHashSet:`

```java
@Param({"100", "1000", "10000", "100000"})
public int setSize;

@Benchmark
public IntSet givenFastUtilsIntSetWithInitialSizeSet_whenPopulated_checkTimeTaken() {
    IntSet intSet = new IntOpenHashSet(setSize);
    for(int i = 0; i < setSize; i++) {
        intSet.add(i);
    }
    return intSet; 
}
```

上面，我们简单地声明了`IntSet `接口的`IntOpenHashSet` 实现。我们还用`@Param `注释声明了初始大小`setSize `。

简单地说，这些数字被输入 JMH，产生一系列不同规模的基准测试。

接下来，**让我们使用 Java 集合实现做同样的事情:**

```java
@Benchmark
public Set<Integer> givenCollectionsHashSetWithInitialSizeSet_whenPopulated_checkTimeTaken() {
    Set<Integer> intSet = new HashSet<>(setSize);
    for(int i = 0; i < setSize; i++) {
        intSet.add(i);
    }
    return intSet;
}
```

最后，让我们运行基准测试并比较这两种实现:

```java
Benchmark                                     (setSize)  Mode  Cnt     Score   Units
givenCollectionsHashSetWithInitialSizeSet...        100  avgt    2     1.460   us/op
givenCollectionsHashSetWithInitialSizeSet...       1000  avgt    2    12.740   us/op
givenCollectionsHashSetWithInitialSizeSet...      10000  avgt    2   109.803   us/op
givenCollectionsHashSetWithInitialSizeSet...     100000  avgt    2  1870.696   us/op
givenFastUtilsIntSetWithInitialSizeSet...           100  avgt    2     0.369   us/op
givenFastUtilsIntSetWithInitialSizeSet...          1000  avgt    2     2.351   us/op
givenFastUtilsIntSetWithInitialSizeSet...         10000  avgt    2    37.789   us/op
givenFastUtilsIntSetWithInitialSizeSet...        100000  avgt    2   896.467   us/op
```

这些结果清楚地表明 **`FastUtils `实现比 Java 集合替代方案的性能高得多。**

## 5.大型收藏

`**Fa**stUtils `的另一个重要的**特性是使用 64 位数组的能力。**默认情况下，Java 中的数组限制为 32 位。

首先，让我们看一下`Integer` 类型的`BigArrays`类。 **[`IntBigArrays`](https://web.archive.org/web/20221205144339/http://fastutil.di.unimi.it/docs/it/unimi/dsi/fastutil/ints/IntBigArrays.html) 提供了处理二维`Integer`数组的静态方法。通过使用这些提供的方法，我们可以将我们的数组包装成一个更加用户友好的一维数组。**

让我们来看看这是如何工作的。

首先，我们将从初始化一维数组开始，并使用`IntBigArray's wrap `方法将其转换为二维数组:

```java
int[] oneDArray = new int[] { 2, 1, 5, 2, 1, 7 };
int[][] twoDArray = IntBigArrays.wrap(oneDArray.clone());
```

我们应该**确保使用`clone` 方法来确保数组的深度拷贝。**

现在，就像我们处理`List `或`Map`一样，我们可以使用`get `方法来访问元素:

```java
int firstIndex = IntBigArrays.get(twoDArray, 0);
int lastIndex = IntBigArrays.get(twoDArray, IntBigArrays.length(twoDArray)-1);
```

最后，让我们添加一些检查来确保我们的`IntBigArray `返回正确的值:

```java
assertEquals(2, firstIndex);
assertEquals(7, lastIndex);
```

## 6.结论

在本文中，我们已经对`FastUtils `的核心特性进行了**探究。**

我们看了看`FastUtil`提供的一些**特定类型的系列，然后又玩了一些`BigCollections`。**

和往常一样，代码可以在 GitHub 上找到