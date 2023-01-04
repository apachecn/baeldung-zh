# System.arraycopy()与 Arrays.copyOf()的性能

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-system-arraycopy-arrays-copyof-performance>

## 1.介绍

在本教程中，我们将看看两个 Java 方法的性能:`System.arraycopy()`和`Arrays.copyOf()`。首先，我们将分析它们的实现。其次，我们将运行一些基准来比较它们的平均执行时间。

## 2.`System.arraycopy()`的性能

`System.arraycopy()`从源数组的指定位置开始，将数组内容复制到目标数组的指定位置。此外，在复制之前，JVM 检查源和目标类型是否相同。

**在估算`System.arraycopy()`的性能时，我们需要记住它是一个原生方法。**本地方法在平台相关的代码(通常是 C)中实现，并通过 JNI 调用来访问。

因为本地方法已经针对特定的架构进行了编译，所以我们无法精确地估计运行时的复杂性。此外，它们的复杂性可能因平台而异。我们可以确定最坏的情况是`O(N)`。然而，处理器可以一次一个块地复制连续的内存块(C 中的`memcpy()`)，所以实际结果可能会更好。

我们只能查看`System.arraycopy()`的签名:

```java
public static native void arraycopy(Object src, int srcPos, Object dest, int destPos, int length);
```

## 3.`Arrays.copyOf()`的性能

`Arrays.copyOf()`在`System.arraycopy()`实现的基础上提供额外的功能。当`System.arraycopy()`简单地将值从源数组复制到目的地时， **`Arrays.copyOf()`也会创建新的数组**。如有必要，它将截断或填充内容。

第二个区别是新数组可以是与源数组不同的类型。如果是这样的话， **JVM 将使用反射，这会增加性能开销**。

当用`Object`数组调用时，`copyOf()`将调用反射的`Array.newInstance()`方法:

```java
public static <T,U> T[] copyOf(U[] original, int newLength, Class<? extends T[]> newType) {
    @SuppressWarnings("unchecked")
    T[] copy = ((Object)newType == (Object)Object[].class) 
      ? (T[]) new Object[newLength]
      : (T[]) Array.newInstance(newType.getComponentType(), newLength);
    System.arraycopy(original, 0, copy, 0, Math.min(original.length, newLength));
    return copy;
}
```

然而，当使用原语作为参数调用时，它不需要反射来创建目标数组:

```java
public static int[] copyOf(int[] original, int newLength) {
    int[] copy = new int[newLength];
    System.arraycopy(original, 0, copy, 0, Math.min(original.length, newLength));
    return copy;
}
```

我们可以清楚地看到，**当前，`Arrays.copyOf()`的实现调用`System.arraycopy()`** 。因此，运行时执行应该是相似的。为了证实我们的怀疑，我们将用原语和对象作为参数对上述方法进行基准测试。

## 4.代码基准

让我们用真题测试一下哪种抄法更快。为此，我们将使用 [JMH](/web/20220628115219/https://www.baeldung.com/java-microbenchmark-harness "JMH") (Java 微基准测试工具)。我们将创建一个简单的测试，使用`System.arraycopy()`和`Arrays.copyOf()`将值从一个数组复制到另一个数组。

我们将创建两个测试类。在一个测试类中，我们将测试原语，在第二个测试类中，我们将测试对象。在这两种情况下，基准配置是相同的。

### 4.1.基准配置

首先，让我们定义我们的基准参数:

```java
@BenchmarkMode(Mode.AverageTime)
@State(Scope.Thread)
@OutputTimeUnit(TimeUnit.NANOSECONDS)
@Warmup(iterations = 10)
@Fork(1)
@Measurement(iterations = 100)
```

这里，我们指定只运行一次基准测试，10 次预热迭代和 100 次测量迭代。此外，我们希望计算平均执行时间，并以纳秒为单位收集结果。为了获得精确的结果，至少执行五次预热迭代是很重要的。

### 4.2.参数设置

我们需要确保我们只测量花费在方法执行上的时间，而不是数组创建上的时间。为此，我们将在基准设置阶段初始化源数组。用大数字和小数字来运行基准测试是一个好主意。

在设置方法中，我们简单地用随机参数初始化一个数组。首先，我们定义了原语的基准设置:

```java
public class PrimitivesCopyBenchmark {

    @Param({ "10", "1000000" })
    public int SIZE;

    int[] src;

    @Setup
    public void setup() {
        Random r = new Random();
        src = new int[SIZE];

        for (int i = 0; i < SIZE; i++) {
            src[i] = r.nextInt();
        }
    }
} 
```

对象基准测试遵循相同的设置:

```java
public class ObjectsCopyBenchmark {

    @Param({ "10", "1000000" })
    public int SIZE;
    Integer[] src;

    @Setup
    public void setup() {
        Random r = new Random();
        src = new Integer[SIZE];

        for (int i = 0; i < SIZE; i++) {
            src[i] = r.nextInt();
        }
    }
}
```

### 4.3.试验

我们定义了两个将执行复制操作的基准。首先，我们将调用`System.arraycopy()`:

```java
@Benchmark
public Integer[] systemArrayCopyBenchmark() {
    Integer[] target = new Integer[SIZE];
    System.arraycopy(src, 0, target, 0, SIZE);
    return target;
}
```

为了使这两个测试等价，我们在基准测试中包括了目标数组的创建。

其次，我们将测量`Arrays.copyOf()`的性能:

```java
@Benchmark
public Integer[] arraysCopyOfBenchmark() {
    return Arrays.copyOf(src, SIZE);
}
```

### 4.4.结果

运行测试后，让我们看看结果:

```java
Benchmark                                          (SIZE)  Mode  Cnt        Score       Error  Units
ObjectsCopyBenchmark.arraysCopyOfBenchmark             10  avgt  100        8.535 ±     0.006  ns/op
ObjectsCopyBenchmark.arraysCopyOfBenchmark        1000000  avgt  100  2831316.981 ± 15956.082  ns/op
ObjectsCopyBenchmark.systemArrayCopyBenchmark          10  avgt  100        9.278 ±     0.005  ns/op
ObjectsCopyBenchmark.systemArrayCopyBenchmark     1000000  avgt  100  2826917.513 ± 15585.400  ns/op
PrimitivesCopyBenchmark.arraysCopyOfBenchmark          10  avgt  100        9.172 ±     0.008  ns/op
PrimitivesCopyBenchmark.arraysCopyOfBenchmark     1000000  avgt  100   476395.127 ±   310.189  ns/op
PrimitivesCopyBenchmark.systemArrayCopyBenchmark       10  avgt  100        8.952 ±     0.004  ns/op
PrimitivesCopyBenchmark.systemArrayCopyBenchmark  1000000  avgt  100   475088.291 ±   726.416  ns/op
```

正如我们所见，`System.arraycopy()`和`Arrays.copyOf()`的性能在图元和`Integer`对象的测量误差范围上有所不同。考虑到`Arrays.copyOf()`在引擎盖下使用`System.arraycopy()`，这并不奇怪。因为我们使用了两个原始的`int`数组，所以没有进行反射调用。

我们需要记住，JMH 给**的只是对执行时间**的粗略估计，机器和 JVM 之间的结果可能不同。

## 5.内在候选人

值得注意的是，在 HotSpot JVM 16 中，`Arrays.copyOf()`和`System.arraycopy()`都被标记为`@IntrinsicCandidate`。这种注释意味着 HotSpot VM 可以用更快的底层代码替换注释的方法。

JIT 编译器可以(对于某些或所有架构)用依赖于机器的、经过极大优化的指令来替代内部方法。由于本机方法对编译器来说是一个黑盒，有很大的调用开销，所以这两种方法的性能都可以更好。同样，这样的性能提升也不能保证。

## 6.结论

在这个例子中，我们研究了`System.arraycopy(`和`Arrays.copyOf(`的性能。首先，我们分析了这两种方法的源代码。其次，我们建立了一个示例基准来测量它们的平均执行时间。

结果，我们证实了我们的理论，因为`Arrays.copyOf()`使用`System.arraycopy()`，两种方法的性能非常相似。

和往常一样，本文中使用的例子可以在 GitHub 的[上找到。](https://web.archive.org/web/20220628115219/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-arrays-operations-advanced)