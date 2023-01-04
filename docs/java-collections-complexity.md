# Java 集合的时间复杂度

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-collections-complexity>

## 1。概述

在本教程中，**我们将讨论来自 Java 集合 API** 的不同集合的性能。当我们谈到集合时，我们通常会想到`List, Map,` 和 `Set`数据结构，以及它们的通用实现。

首先，我们将着眼于常见操作的 Big-O 复杂性洞察。然后，我们将显示一些收集操作运行时间的真实数字。

## 2。时间复杂度

通常，**当我们谈论时间复杂性时，我们指的是 Big-O 符号**。简而言之，符号描述了执行算法的时间如何随着输入大小而增长。

有用的文章可以用来学习更多关于 Big-O 符号的理论和实用的 Java 例子。

## 3。`List`

让我们从一个简单的列表开始，它是一个有序的集合。

这里我们来看一下`ArrayList, LinkedList,` 和 `CopyOnWriteArrayList`实现的性能概述。

### 3.1。`ArrayList`

**Java 中的 `ArrayList` 由数组**做后盾。这有助于理解其实现的内部逻辑。本文中的[为`ArrayList`提供了更全面的指南。](/web/20220930004241/https://www.baeldung.com/java-arraylist)

因此，让我们首先从较高的层面来关注常见操作的时间复杂性:

*   **`add()`**–花费`O(1)`时间；然而，在最坏的情况下，当一个新的数组必须被创建并且所有的元素都被复制到其中，这就是`O(n)`
*   **`add(index, element)`**——平均在`O(n)`时间内运行
*   **`get()`**–总是恒定时间`O(1)`运行
*   **`remove()`**–以线性`O(n)`时间运行。我们必须迭代整个数组来找到符合移除条件的元素。
*   `**indexOf()**` –也以线性时间运行。它遍历内部数组，逐个检查每个元素，所以这个操作的时间复杂度总是需要`O(n)`时间。
*   **`contains()`**–实现基于`indexOf(),` ，所以它也将在`O(n)`时间内运行。

### 3.2。`CopyOnWriteArrayList`

当处理多线程应用程序时，`List`接口的实现**是有益的。它是线程安全的，在的本指南[中有很好的解释。](/web/20220930004241/https://www.baeldung.com/java-copy-on-write-arraylist)**

下面是`CopyOnWriteArrayList`的 Big-O 符号性能概述:

*   `**add()**`–取决于我们添加值的位置，所以复杂度为`O(n)`
*   `**get()**`–是`O(1)` 恒定时间运行
*   `**remove()**`–花费`O(n)` 时间
*   `**contains()**`–同样，复杂度为`O(n)`

正如我们所看到的，由于`add()`方法的性能特征，使用这个集合是非常昂贵的。

### 3.3。`LinkedList`

**`LinkedList`是一个线性数据结构，由保存数据字段的节点和对另一个节点**的引用组成。更多的`LinkedList`特性和功能，请看[的这篇文章](/web/20220930004241/https://www.baeldung.com/java-linkedlist)。

让我们给出执行一些基本操作所需的平均估计时间:

*   **`add()`**–在列表末尾追加一个元素。它只更新一个尾部，因此，它是`O(1)`常数时间复杂度。
*   `**add(index, element)**`–平均在`O(n)`时间内运行
*   **`get()`**–搜索一个元素需要`O(n)` 时间。
*   **`remove(element)`**——要移除一个元素，我们首先需要找到它。这个操作是`O(n).`
*   `**remove(index)**`–要通过索引删除一个元素，我们首先需要从头开始跟踪链接；因此，整体复杂度为`O(n).`
*   **`contains()`**——也有`O(n)`的时间复杂度

### 3.4。预热 JVM

现在，为了证明理论，让我们用实际数据来玩。**更准确地说，我们将展示最常见的收集操作的 JMH (Java 微基准测试工具)测试结果**。

如果我们不熟悉 JMH 工具，我们可以看看这个[有用的指南](/web/20220930004241/https://www.baeldung.com/java-microbenchmark-harness)。

首先，我们将介绍基准测试的主要参数:

```java
@BenchmarkMode(Mode.AverageTime)
@OutputTimeUnit(TimeUnit.MICROSECONDS)
@Warmup(iterations = 10)
public class ArrayListBenchmark {
}
```

然后我们将预热迭代次数设置为`10`。请注意，我们还希望看到以微秒为单位显示的结果的平均运行时间。

### 3.5。基准测试

现在是时候运行我们的性能测试了。首先，我们从`ArrayList`开始:

```java
@State(Scope.Thread)
public static class MyState {

    List<Employee> employeeList = new ArrayList<>();

    long iterations = 100000;

    Employee employee = new Employee(100L, "Harry");

    int employeeIndex = -1;

    @Setup(Level.Trial)
    public void setUp() {
        for (long i = 0; i < iterations; i++) {
            employeeList.add(new Employee(i, "John"));
        }

        employeeList.add(employee);
        employeeIndex = employeeList.indexOf(employee);
    }
}
```

在我们的`ArrayListBenchmark`中，我们添加了`State`类来保存初始数据。

在这里，我们创建了一个由`Employee`个对象组成的`ArrayList`。然后**我们用`setUp()`方法中的`100.000`项初始化它。`@State`表示`@Benchmark`测试可以在同一个线程中完全访问其中声明的变量。**

最后，是时候为`add(), contains(), indexOf(), remove(),` 和`get()`方法添加基准测试了:

```java
@Benchmark
public void testAdd(ArrayListBenchmark.MyState state) {
    state.employeeList.add(new Employee(state.iterations + 1, "John"));
}

@Benchmark
public void testAddAt(ArrayListBenchmark.MyState state) {
    state.employeeList.add((int) (state.iterations), new Employee(state.iterations, "John"));
}

@Benchmark
public boolean testContains(ArrayListBenchmark.MyState state) {
    return state.employeeList.contains(state.employee);
}

@Benchmark
public int testIndexOf(ArrayListBenchmark.MyState state) {
    return state.employeeList.indexOf(state.employee);
}

@Benchmark
public Employee testGet(ArrayListBenchmark.MyState state) {
    return state.employeeList.get(state.employeeIndex);
}

@Benchmark
public boolean testRemove(ArrayListBenchmark.MyState state) {
    return state.employeeList.remove(state.employee);
}
```

### 3.6。测试结果

所有结果都以微秒表示:

```java
Benchmark                        Mode  Cnt     Score     Error
ArrayListBenchmark.testAdd       avgt   20     2.296 ±   0.007
ArrayListBenchmark.testAddAt     avgt   20   101.092 ±  14.145
ArrayListBenchmark.testContains  avgt   20   709.404 ±  64.331
ArrayListBenchmark.testGet       avgt   20     0.007 ±   0.001
ArrayListBenchmark.testIndexOf   avgt   20   717.158 ±  58.782
ArrayListBenchmark.testRemove    avgt   20   624.856 ±  51.101
```

**从结果中，我们了解到`testContains()` 和`testIndexOf()`方法几乎同时运行**。我们还可以从其余的结果中清楚地看到`testAdd()` 和 `testGet()`方法得分之间的巨大差异。添加一个元素需要 2 `.296`微秒，获得一个元素是一个 0.007 微秒的操作。

此外，搜索或删除一个元素大约需要`700`微秒。这些数字是理论部分的证明，在这里我们了解到`add(),` 和`get()`有`O(1)`的时间复杂度，其他方法是`O(n)`。`n=10.000`我们例子中的元素。

类似地，我们可以为`CopyOnWriteArrayList`集合编写相同的测试。我们需要做的就是用`CopyOnWriteArrayList`实例替换 employeeList 中的`ArrayList`。

以下是基准测试的结果:

```java
Benchmark                          Mode  Cnt    Score     Error
CopyOnWriteBenchmark.testAdd       avgt   20  652.189 ±  36.641
CopyOnWriteBenchmark.testAddAt     avgt   20  897.258 ±  35.363
CopyOnWriteBenchmark.testContains  avgt   20  537.098 ±  54.235
CopyOnWriteBenchmark.testGet       avgt   20    0.006 ±   0.001
CopyOnWriteBenchmark.testIndexOf   avgt   20  547.207 ±  48.904
CopyOnWriteBenchmark.testRemove    avgt   20  648.162 ± 138.379
```

在这里，数字再次证实了这个理论。我们可以看到，`testGet()`平均运行时间为 0.006 ms，我们可以认为是`O(1)`。**与`ArrayList`相比，我们还注意到`testAdd()`方法结果之间的显著差异，因为这里我们有`add()`方法与`ArrayList's O(1).`** 方法的`O(n)`复杂性

**我们可以清楚地看到时间的线性增长，因为性能数字是`878.166`与`0.051`** 的对比。

现在是`LinkedList`时间:

```java
Benchmark        Cnt     Score       Error
testAdd          20     2.580        ± 0.003
testContains     20     1808.102     ± 68.155
testGet          20     1561.831     ± 70.876 
testRemove       20     0.006        ± 0.001
```

从分数中我们可以看出`LinkedList`中添加和删除元素的速度相当快。

此外，添加/删除和获取/包含操作之间存在显著的性能差距。

## 4。`Map`

在最新的 JDK 版本中，我们见证了`Map`实现的显著性能提升，比如在`HashMap,` 和 `LinkedHashMap` 内部实现中用平衡的树节点结构替换了`LinkedList`。**这在`HashMap`冲突**期间将元素查找最坏情况从`O(n)`缩短到`O(log(n))` 时间。

然而，如果我们实现适当的`.equals()`和`.hashcode()`方法，冲突是不太可能的。

要了解更多关于`HashMap`碰撞的信息，请看[这篇文章](/web/20220930004241/https://www.baeldung.com/java-hashmap)。**从这篇文章中，我们还将了解到从`HashMap`中存储和检索元素需要恒定的`O(1)`时间**。

### 4.1。测试`O(1)`操作

现在让我们来看一些实际数字。第一，`HashMap`:

```java
Benchmark                         Mode  Cnt  Score   Error
HashMapBenchmark.testContainsKey  avgt   20  0.009 ± 0.002
HashMapBenchmark.testGet          avgt   20  0.011 ± 0.001
HashMapBenchmark.testPut          avgt   20  0.019 ± 0.002
HashMapBenchmark.testRemove       avgt   20  0.010 ± 0.001
```

**正如我们所见，这些数字证明了运行上述方法的`O(1)`恒定时间。**现在让我们比较一下`HashMap`测试分数和其他`Map`实例分数。

对于所有列出的方法，**有`O(1)`对应`HashMap, LinkedHashMap, IdentityHashMap, WeakHashMap, EnumMap`和`ConcurrentHashMap.`**

让我们以表格的形式呈现剩余测试分数的结果:

```java
Benchmark      LinkedHashMap  IdentityHashMap  WeakHashMap  ConcurrentHashMap
testContainsKey    0.008         0.009          0.014          0.011
testGet            0.011         0.109          0.019          0.012
testPut            0.020         0.013          0.020          0.031
testRemove         0.011         0.115          0.021          0.019
```

从输出的数字中，我们可以证实`O(1)`时间复杂度的说法。

### 4.2。测试`O(log(n))`操作

**为树形结构 `TreeMap`和`ConcurrentSkipListMap,``put(), get(), remove(),` 和 `containsKey()` 操作时间为`O(log(n)).`**

这里**我们希望确保我们的性能测试大约在对数时间内运行**。出于这个原因，我们将连续用 n 个`=1000, 10,000, 100,000, 1,000,000`项初始化地图。

在这种情况下，我们感兴趣的是总的执行时间:

```java
items count (n)         1000      10,000     100,000   1,000,000
all tests total score   00:03:17  00:03:17   00:03:30  00:05:27 
```

当`n=1000,`我们总共有`00:03:17`毫秒的执行时间。在`n=10,000,`时间几乎不变，`00:03:18 ms. n=100,000`在`00:03:30`略有增加。最后，当`n=1,000,000,`时，运行在`00:05:27 ms`中完成。

**将运行时数与每个`n`的`log(n)`函数进行比较后，我们可以确认两个函数的相关性匹配。**

## 5。`Set`

一般来说，`Set`是独特元素的集合。这里我们将检查`Set`接口的`HashSet`、`LinkedHashSet`、`EnumSet, TreeSet, CopyOnWriteArraySet,`和 `ConcurrentSkipListSet`实现。

为了更好地理解`HashSet`、[的内部原理，本指南](/web/20220930004241/https://www.baeldung.com/java-hashset)将为您提供帮助。

现在让我们跳到前面来展示时间复杂度数字。**对于`HashSet`、`LinkedHashSet,` 和`EnumSet,` ，`add(), remove()` 和`contains()`操作成本恒定`O(1)`时间得益于内部`HashMap`实现。**

**同样，** **`TreeSet`对于前一组中列出的操作，具有`O(log(n))`时间复杂度**。这是因为`TreeMap`的实现。`ConcurrentSkipListSet`的时间复杂度也是`O(log(n))`时间，因为它基于跳表数据结构。

对于`CopyOnWriteArraySet,`，`add(), remove()` 和`contains()`方法的平均时间复杂度为 O(n)。

### 5.1。测试方法

现在让我们跳到我们的基准测试:

```java
@Benchmark
public boolean testAdd(SetBenchMark.MyState state) {
    return state.employeeSet.add(state.employee);
}

@Benchmark
public Boolean testContains(SetBenchMark.MyState state) {
    return state.employeeSet.contains(state.employee);
}

@Benchmark
public boolean testRemove(SetBenchMark.MyState state) {
    return state.employeeSet.remove(state.employee);
}
```

我们将保持其余的基准配置不变。

### 5.2。比较数字

让我们看看拥有`n = 1000; 10,000; 100,000`项的`HashSet` 和`LinkedHashSet` 的运行时执行分数的行为。

对于`HashSet,` ，数字是:

```java
Benchmark      1000    10,000    100,000
.add()         0.026   0.023     0.024
.remove()      0.009   0.009     0.009
.contains()    0.009   0.009     0.010
```

类似地，`LinkedHashSet`的结果是:

```java
Benchmark      1000    10,000    100,000
.add()         0.022   0.026     0.027
.remove()      0.008   0.012     0.009
.contains()    0.008   0.013     0.009
```

正如我们所看到的，每次操作的得分几乎相同。当我们将它们与`HashMap`测试输出进行比较时，它们看起来也是一样的。

**因此，我们确认所有测试的方法都在恒定的`O(1)`时间内运行。**

## 6。结论

本文展示了 Java 数据结构最常见实现的时间复杂性。

通过 JVM 基准测试，我们看到了每种集合的实际运行时性能。我们还比较了不同集合中相同操作的性能。因此，我们学会了如何选择合适的系列来满足我们的需求。

和往常一样，这篇文章的完整代码可以在 GitHub 的[上找到。](https://web.archive.org/web/20220930004241/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-collections)