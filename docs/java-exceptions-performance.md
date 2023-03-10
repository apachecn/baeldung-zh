# Java 中异常的性能影响

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-exceptions-performance>

## 1.概观

在 Java 中，异常通常被认为是昂贵的，不应该用于流控制。本教程**将证明这种看法是正确的，并指出导致性能问题的原因。**

## 2.设置环境

在编写评估性能成本的代码之前，我们需要建立一个基准测试环境。

### 2.1.Java 微基准测试工具

测量异常开销不像在一个简单的循环中执行一个方法并记录下总时间那么简单。

原因是实时编译器可能会妨碍并优化代码。这种优化可能会使代码的性能比它在生产环境中的实际性能更好。换句话说，它可能会产生假阳性结果。

为了创建一个可以减轻 JVM 优化的受控环境，我们将使用 [Java 微基准测试工具](https://web.archive.org/web/20221109210314/https://openjdk.java.net/projects/code-tools/jmh/)，简称 JMH。

接下来的小节将介绍如何建立一个基准测试环境，而不涉及 JMH 的细节。有关该工具的更多信息，请查看我们的【Java 微基准测试教程。

### 2.2.获得 JMH 文物

要获得 JMH 工件，将这两个依赖项添加到 POM 中:

```java
<dependency>
    <groupId>org.openjdk.jmh</groupId>
    <artifactId>jmh-core</artifactId>
    <version>1.35</version>
</dependency>
<dependency>
    <groupId>org.openjdk.jmh</groupId>
    <artifactId>jmh-generator-annprocess</artifactId>
    <version>1.35</version>
</dependency>
```

关于 [JMH 核](https://web.archive.org/web/20221109210314/https://search.maven.org/search?q=g:org.openjdk.jmh%20a:jmh-core)和 [JMH 注释处理器](https://web.archive.org/web/20221109210314/https://search.maven.org/search?q=g:org.openjdk.jmh%20a:jmh-generator-annprocess)的最新版本，请参考 Maven Central。

### 2.3.基准类

我们需要一个类来保存基准:

```java
@Fork(1)
@Warmup(iterations = 2)
@Measurement(iterations = 10)
@BenchmarkMode(Mode.AverageTime)
@OutputTimeUnit(TimeUnit.MILLISECONDS)
public class ExceptionBenchmark {
    private static final int LIMIT = 10_000;
    // benchmarks go here
}
```

让我们浏览一下上面显示的 JMH 注释:

*   指定 JMH 必须产生一个新进程来运行基准测试的次数。我们将它的值设置为 1，以便只生成一个流程，避免等待太长时间才能看到结果
*   `@Warmup`:携带预热参数。`iterations`元素为 2 意味着在计算结果时忽略前两次运行
*   `@Measurement`:携带测量参数。值为 10 的`iterations`表示 JMH 将执行每个方法 10 次
*   JHM 应该这样收集执行结果。值`AverageTime`要求 JMH 计算一个方法完成其操作所需的平均时间
*   `@OutputTimeUnit`:表示输出时间单位，本例中为毫秒

另外，在类体内有一个静态字段，即`LIMIT`。这是每个方法体中的迭代次数。

### 2.4.执行基准

为了执行基准测试，我们需要一个`main`方法:

```java
public class MappingFrameworksPerformance {
    public static void main(String[] args) throws Exception {
        org.openjdk.jmh.Main.main(args);
    }
}
```

我们可以将项目打包到一个 JAR 文件中，并在命令行中运行它。当然，现在这样做会产生一个空输出，因为我们没有添加任何基准测试方法。

为了方便起见，我们可以将`maven-jar-plugin`添加到 POM 中。这个插件允许我们在 ide 中执行`main`方法:

```java
<groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-jar-plugin</artifactId>
    <version>3.2.0</version>
    <configuration>
        <archive>
            <manifest>
                <mainClass>com.baeldung.performancetests.MappingFrameworksPerformance</mainClass>
            </manifest>
        </archive>
    </configuration>
</plugin>
```

最新版本的`maven-jar-plugin`可以在[这里](https://web.archive.org/web/20221109210314/https://search.maven.org/search?q=g:org.apache.maven.plugins%20a:maven-jar-plugin)找到。

## 3.性能测定

是时候有一些标杆方法来衡量绩效了。这些方法中的每一个都必须带有`@Benchmark`注释。

### 3.1.方法正常返回

让我们从一个正常返回的方法开始；也就是说，一个不抛出异常的方法:

```java
@Benchmark
public void doNotThrowException(Blackhole blackhole) {
    for (int i = 0; i < LIMIT; i++) {
        blackhole.consume(new Object());
    }
}
```

`blackhole`参数引用了`Blackhole`的一个实例。这是一个 JMH 类，有助于防止死代码消除，这是一种实时编译器可能执行的优化。

在这种情况下，基准测试不会抛出任何异常。事实上，**我们将用它作为参考来评估那些抛出异常的程序的性能。**

执行`main`方法会给我们一个报告:

```java
Benchmark                               Mode  Cnt  Score   Error  Units
ExceptionBenchmark.doNotThrowException  avgt   10  0.049 ± 0.006  ms/op
```

这个结果没什么特别的。基准测试的平均执行时间是 0.049 毫秒，这本身是没有意义的。

### 3.2.创建并引发异常

下面是另一个抛出和捕捉异常的基准:

```java
@Benchmark
public void throwAndCatchException(Blackhole blackhole) {
    for (int i = 0; i < LIMIT; i++) {
        try {
            throw new Exception();
        } catch (Exception e) {
            blackhole.consume(e);
        }
    }
}
```

让我们来看看输出:

```java
Benchmark                                  Mode  Cnt   Score   Error  Units
ExceptionBenchmark.doNotThrowException     avgt   10   0.048 ± 0.003  ms/op
ExceptionBenchmark.throwAndCatchException  avgt   10  17.942 ± 0.846  ms/op
```

方法`doNotThrowException`执行时间的微小变化并不重要。只是底层 OS 和 JVM 状态的波动。关键的一点是**抛出异常会使方法运行速度慢几百倍。**

接下来的几个小节将找出究竟是什么导致了如此巨大的差异。

### 3.3.创建异常而不抛出它

我们不创建、抛出和捕获异常，而是创建它:

```java
@Benchmark
public void createExceptionWithoutThrowingIt(Blackhole blackhole) {
    for (int i = 0; i < LIMIT; i++) {
        blackhole.consume(new Exception());
    }
}
```

现在，让我们执行我们已经宣布的三个基准:

```java
Benchmark                                            Mode  Cnt   Score   Error  Units
ExceptionBenchmark.createExceptionWithoutThrowingIt  avgt   10  17.601 ± 3.152  ms/op
ExceptionBenchmark.doNotThrowException               avgt   10   0.054 ± 0.014  ms/op
ExceptionBenchmark.throwAndCatchException            avgt   10  17.174 ± 0.474  ms/op
```

结果可能会令人惊讶:第一种和第三种方法的执行时间几乎相同，而第二种方法的执行时间要短得多。

在这一点上，很明显**`throw`和`catch`语句本身相当便宜。另一方面，创建异常会产生很高的开销。**

### 3.4.不添加堆栈跟踪就引发异常

让我们弄清楚为什么构造一个异常比处理一个普通对象要昂贵得多:

```java
@Benchmark
@Fork(value = 1, jvmArgs = "-XX:-StackTraceInThrowable")
public void throwExceptionWithoutAddingStackTrace(Blackhole blackhole) {
    for (int i = 0; i < LIMIT; i++) {
        try {
            throw new Exception();
        } catch (Exception e) {
            blackhole.consume(e);
        }
    }
}
```

该方法与第 3.2 小节中的方法的唯一区别是`jvmArgs`元素。它的值`-XX:-StackTraceInThrowable`是一个 JVM 选项，防止堆栈跟踪被添加到异常中。

让我们再次运行基准测试:

```java
Benchmark                                                 Mode  Cnt   Score   Error  Units
ExceptionBenchmark.createExceptionWithoutThrowingIt       avgt   10  17.874 ± 3.199  ms/op
ExceptionBenchmark.doNotThrowException                    avgt   10   0.046 ± 0.003  ms/op
ExceptionBenchmark.throwAndCatchException                 avgt   10  16.268 ± 0.239  ms/op
ExceptionBenchmark.throwExceptionWithoutAddingStackTrace  avgt   10   1.174 ± 0.014  ms/op
```

通过不使用堆栈跟踪填充异常，我们将执行持续时间减少了 100 多倍。显然，遍历堆栈并将它的帧添加到异常中会导致我们已经看到的缓慢。

### 3.5.引发异常并展开其堆栈跟踪

最后，让我们看看如果我们抛出一个异常并在捕获它时展开堆栈跟踪会发生什么:

```java
@Benchmark
public void throwExceptionAndUnwindStackTrace(Blackhole blackhole) {
    for (int i = 0; i < LIMIT; i++) {
        try {
            throw new Exception();
        } catch (Exception e) {
            blackhole.consume(e.getStackTrace());
        }
    }
}
```

结果是这样的:

```java
Benchmark                                                 Mode  Cnt    Score   Error  Units
ExceptionBenchmark.createExceptionWithoutThrowingIt       avgt   10   16.605 ± 0.988  ms/op
ExceptionBenchmark.doNotThrowException                    avgt   10    0.047 ± 0.006  ms/op
ExceptionBenchmark.throwAndCatchException                 avgt   10   16.449 ± 0.304  ms/op
ExceptionBenchmark.throwExceptionAndUnwindStackTrace      avgt   10  326.560 ± 4.991  ms/op
ExceptionBenchmark.throwExceptionWithoutAddingStackTrace  avgt   10    1.185 ± 0.015  ms/op
```

仅仅通过展开堆栈跟踪，我们就可以看到执行时间增加了大约 20 倍。换句话说，如果我们除了抛出异常之外还从异常中提取堆栈跟踪，性能会差得多。

## 4.结论

在本教程中，我们分析了异常对性能的影响。具体来说，它发现性能成本主要是在异常中添加堆栈跟踪。如果这个堆栈跟踪后来被展开，开销会变得更大。

由于抛出和处理异常的代价很高，我们不应该在正常的程序流中使用它。相反，顾名思义，异常应该只用于异常情况。

完整的源代码可以在 GitHub 上找到。