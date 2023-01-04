# Java 偷工减料指南

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-work-stealing>

## 1.概观

在本教程中，我们将看看 Java 中工作窃取的**概念。**

## 2.什么是偷工减料？

工作窃取是在 Java 中引入的，目的是**减少多线程应用中的争用**。这是使用 [fork/join 框架](/web/20221205163736/https://www.baeldung.com/java-fork-join)完成的。

### 2.1.分而治之

在 fork/join 框架中，**问题或任务被递归地分解成子任务**。然后单独解决子任务，将子结果组合起来形成结果:

```java
Result solve(Problem problem) {
    if (problem is small)
        directly solve problem
    else {
        split problem into independent parts
        fork new subtasks to solve each part
        join all subtasks
        compose result from subresults
    }
}
```

### 2.2.工作线程

**分解的任务在[线程池](/web/20221205163736/https://www.baeldung.com/thread-pool-java-and-guava)提供的工作线程的帮助下解决。每个工作线程都有它负责的子任务。这些存储在双端队列中([队列](/web/20221205163736/https://www.baeldung.com/java-queue#3-deques))。**

每个工作线程通过不断从队列顶部弹出子任务来从队列中获取子任务。当一个工作线程的队列为空时，意味着所有的子任务都已经被弹出并完成。

此时，工作线程随机选择一个它可以“窃取”工作的对等线程池线程。然后，它使用先入先出的方法(FIFO)从受害者队列的尾端获取子任务。

## 3.Fork/Join 框架实现

**我们可以使用 [`ForkJoinPool`类](/web/20221205163736/https://www.baeldung.com/java-fork-join)或者 [`Executors`类](/web/20221205163736/https://www.baeldung.com/java-executor-service-tutorial) :** 创建一个窃取工作的线程池

```java
ForkJoinPool commonPool = ForkJoinPool.commonPool();
ExecutorService workStealingPool = Executors.newWorkStealingPool();
```

`Executors`类有一个重载的`newWorkStealingPool`方法，它采用一个整数参数来表示并行度的**级别**。

`Executors.newWorkStealingPool`是`ForkJoinPool.commonPool`的抽象。唯一的区别是`Executors.newWorkStealingPool `以异步模式创建一个池，而`ForkJoinPool.commonPool` 没有。

## 4.同步与异步线程池

**`ForkJoinPool.commonPool` 使用后进先出(LIFO)队列配置，而`Executors.newWorkStealingPool `使用先进先出(FIFO)方式。**

根据 Doug Lea 的说法，先进先出法比后进先出法有以下优势:

*   它通过让偷窃者作为所有者在队列的另一边操作来减少争用
*   它利用递归分治算法的特性，在早期生成“大型”任务

上面的第二点意味着有可能通过一个窃取它的线程进一步分解一个旧的被窃取的任务。

根据 [Java 文档](https://web.archive.org/web/20221205163736/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/concurrent/ForkJoinPool.html)，将`asyncMode`设置为`true`可能适用于从未加入的事件风格任务。

## 5.工作示例-寻找质数

我们将使用从一组数字中寻找素数的例子来展示偷功框架的**计算时间优势。我们还将展示使用同步线程池和异步线程池之间的区别。**

### 5.1.素数问题

从一组数字中寻找素数可能是一个计算量很大的过程。这主要是由于数字集合的大小。

`PrimeNumbers` 类帮助我们找到质数:

```java
public class PrimeNumbers extends RecursiveAction {

    private int lowerBound;
    private int upperBound;
    private int granularity;
    static final List<Integer> GRANULARITIES
      = Arrays.asList(1, 10, 100, 1000, 10000);
    private AtomicInteger noOfPrimeNumbers;

    PrimeNumbers(int lowerBound, int upperBound, int granularity, AtomicInteger noOfPrimeNumbers) {
        this.lowerBound = lowerBound;
        this.upperBound = upperBound;
        this.granularity = granularity;
        this.noOfPrimeNumbers = noOfPrimeNumbers;
    }

    // other constructors and methods

    private List<PrimeNumbers> subTasks() {
        List<PrimeNumbers> subTasks = new ArrayList<>();

        for (int i = 1; i <= this.upperBound / granularity; i++) {
            int upper = i * granularity;
            int lower = (upper - granularity) + 1;
            subTasks.add(new PrimeNumbers(lower, upper, noOfPrimeNumbers));
        }
        return subTasks;
    }

    @Override
    protected void compute() {
        if (((upperBound + 1) - lowerBound) > granularity) {
            ForkJoinTask.invokeAll(subTasks());
        } else {
            findPrimeNumbers();
        }
    }

    void findPrimeNumbers() {
        for (int num = lowerBound; num <= upperBound; num++) {
            if (isPrime(num)) {
                noOfPrimeNumbers.getAndIncrement();
            }
        }
    }

    public int noOfPrimeNumbers() {
        return noOfPrimeNumbers.intValue();
    }
}
```

关于这个类，需要注意一些重要的事情:

*   它扩展了`RecursiveAction`，允许我们使用线程池实现计算任务中使用的`compute`方法
*   它基于`granularity` 值递归地将任务分解成子任务
*   构造函数取`lower`和`upper`边界值，它们控制我们想要确定素数的数字范围
*   它使我们能够使用窃取工作的线程池或单个线程来确定素数

### 5.2.使用线程池更快地解决问题

让我们以单线程的方式确定质数，并使用窃取工作的线程池。

首先，让我们看看**单线程方法**:

```java
PrimeNumbers primes = new PrimeNumbers(10000);
primes.findPrimeNumbers();
```

现在， **`ForkJoinPool.commonPool`接近**:

```java
PrimeNumbers primes = new PrimeNumbers(10000);
ForkJoinPool pool = ForkJoinPool.commonPool();
pool.invoke(primes);
pool.shutdown();
```

最后，我们来看看 **`Executors.newWorkStealingPool` 接近**:

```java
PrimeNumbers primes = new PrimeNumbers(10000);
int parallelism = ForkJoinPool.getCommonPoolParallelism();
ForkJoinPool stealer = (ForkJoinPool) Executors.newWorkStealingPool(parallelism);
stealer.invoke(primes);
stealer.shutdown();
```

我们使用`ForkJoinPool`类的`invoke`方法将任务传递给线程池。这个方法接受`RecursiveAction`子类的实例。使用 [Java 微基准管理](/web/20221205163736/https://www.baeldung.com/java-microbenchmark-harness)，我们根据每次操作的平均时间对这些不同的方法进行了基准测试:

```java
# Run complete. Total time: 00:04:50

Benchmark                                                      Mode  Cnt    Score   Error  Units
PrimeNumbersUnitTest.Benchmarker.commonPoolBenchmark           avgt   20  119.885 ± 9.917  ms/op
PrimeNumbersUnitTest.Benchmarker.newWorkStealingPoolBenchmark  avgt   20  119.791 ± 7.811  ms/op
PrimeNumbersUnitTest.Benchmarker.singleThread                  avgt   20  475.964 ± 7.929  ms/op
```

很明显， **`ForkJoinPool.commonPool`和`Executors.newWorkStealingPool`都允许我们比单线程方法更快地确定素数。**

fork/join pool 框架允许我们将任务分解成子任务。我们将 10，000 个整数的集合分成 1-100、101-200、201-300 等批次。然后我们确定每批的质数，并用我们的`noOfPrimeNumbers`方法得到质数的总数。

### 5.3.窃取工作进行计算

对于同步线程池， **`ForkJoinPool.commonPool`只要任务仍在进行中，就将线程放入池中。** **因此，偷工减料的程度并不依赖于任务粒度的高低。**

异步 **`Executors.newWorkStealingPool` 更受管理，允许工作窃取的级别依赖于任务粒度的级别。**

我们使用`ForkJoinPool`类的`getStealCount`来获得偷工减料的级别:

```java
long steals = forkJoinPool.getStealCount();
```

确定`Executors.newWorkStealingPool` 和`ForkJoinPool.commonPool`的偷工减料计数给了我们不同的行为:

```java
Executors.newWorkStealingPool ->
Granularity: [1], Steals: [6564]
Granularity: [10], Steals: [572]
Granularity: [100], Steals: [56]
Granularity: [1000], Steals: [60]
Granularity: [10000], Steals: [1]

ForkJoinPool.commonPool ->
Granularity: [1], Steals: [6923]
Granularity: [10], Steals: [7540]
Granularity: [100], Steals: [7605]
Granularity: [1000], Steals: [7681]
Granularity: [10000], Steals: [7681]
```

**当`Executors.newWorkStealingPool`粒度由细变粗** (1 比 1 万)**时，偷工减料等级降低**。因此，当任务没有被分解时(粒度为 10，000)，窃取计数为 1。

**`ForkJoinPool.commonPool`有不同的行为。**偷工减料的水平总是很高，并且不受任务粒度变化的太大影响。

从技术上讲，我们的质数例子是一个支持事件风格任务的异步处理的例子。这是因为我们的实现没有强制结果的连接。

可以证明`Executors.newWorkStealingPool `在解决问题时提供了资源的最佳利用。

## 6.结论

在本文中，我们研究了工作窃取以及如何使用 fork/join 框架来应用它。我们还看了工作窃取的例子，以及它如何改进处理时间和资源的使用。

和往常一样，这个例子的完整源代码可以从 GitHub 上的[处获得。](https://web.archive.org/web/20221205163736/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-concurrency-advanced-3)