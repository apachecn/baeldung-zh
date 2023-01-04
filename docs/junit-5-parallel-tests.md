# JUnit 5 的并行测试执行

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/junit-5-parallel-tests>

## 1.介绍

在本文中，我们将介绍如何使用 [JUnit 5](/web/20221223204047/https://www.baeldung.com/junit-5) 执行并行单元测试。首先，我们将介绍开始使用该特性的基本配置和最低要求。接下来，我们将展示不同情况下的代码示例，最后，我们将讨论共享资源的同步。

并行测试执行是一个实验性的特性，从版本 5.3 开始可以选择加入。

## 2.配置

首先，**我们需要在我们的`src/test/resources`文件夹中创建一个`junit-platform.properties` 文件来启用并行测试执行**。我们通过在提到的文件中添加以下行来启用并行化功能:

```java
junit.jupiter.execution.parallel.enabled = true
```

让我们通过运行一些测试来检查我们的配置。首先，我们将创建 `FirstParallelUnitTest` 类和其中的两个测试:

```java
public class FirstParallelUnitTest{

    @Test
    public void first() throws Exception{
        System.out.println("FirstParallelUnitTest first() start => " + Thread.currentThread().getName());
        Thread.sleep(500);
        System.out.println("FirstParallelUnitTest first() end => " + Thread.currentThread().getName());
    }

    @Test
    public void second() throws Exception{
        System.out.println("FirstParallelUnitTest second() start => " + Thread.currentThread().getName());
        Thread.sleep(500);
        System.out.println("FirstParallelUnitTest second() end => " + Thread.currentThread().getName());
    }
}
```

当我们运行测试时，我们在控制台中得到以下输出:

```java
FirstParallelUnitTest second() start => ForkJoinPool-1-worker-19
FirstParallelUnitTest second() end => ForkJoinPool-1-worker-19
FirstParallelUnitTest first() start => ForkJoinPool-1-worker-19
FirstParallelUnitTest first() end => ForkJoinPool-1-worker-19
```

在这个输出中，我们可以注意到两件事。首先，我们的测试按顺序运行。其次，我们使用 [ForkJoin](/web/20221223204047/https://www.baeldung.com/java-fork-join) 线程池。**通过启用并行执行，JUnit 引擎开始使用 ForkJoin 线程池。**

接下来，我们需要添加一个配置来利用这个线程池。我们需要选择并行化策略。 **JUnit 提供了两个实现(`dynamic`和`fixed`)和`a custom`选项来创建我们的实现。**

动态策略根据处理器/内核数量乘以因子参数(默认为 1)来确定线程数量，该因子参数由以下公式指定:

```java
junit.jupiter.execution.parallel.config.dynamic.factor
```

另一方面，固定策略依赖于预定义数量的线程，这些线程由以下各项指定:

```java
junit.jupiter.execution.parallel.config.fixed.parallelism
```

要使用定制策略，我们需要首先通过实现`ParallelExecutionConfigurationStrategy` 接口来创建它。

## 3.测试类内的并行化

我们已经启用了并行执行并选择了一个策略。现在是时候在同一个类中并行执行测试了。有两种方法可以进行配置。一个是使用`@Execution(ExecutionMode.CONCURRENT)` 注释，第二个是使用属性文件和行:

```java
junit.jupiter.execution.parallel.mode.default = concurrent
```

在我们选择如何配置并运行我们的`FirstParallelUnitTest` 类之后，我们可以看到下面的输出:

```java
FirstParallelUnitTest second() start => ForkJoinPool-1-worker-5
FirstParallelUnitTest first() start => ForkJoinPool-1-worker-19
FirstParallelUnitTest second() end => ForkJoinPool-1-worker-5
FirstParallelUnitTest first() end => ForkJoinPool-1-worker-19
```

从输出中，我们可以看到两个测试同时开始，并且在两个不同的线程中。请注意，从一次运行到另一次运行，输出可能会发生变化。使用 ForkJoin 线程池时，这是正常的。

还有一个在同一个线程中运行`FirstParallelUnitTest` 类中所有测试的选项。在当前的范围内，使用并行性和相同的线程选项是不可行的，所以让我们扩大我们的范围，在下一节中再添加一个测试类。

## 4.测试模块内的并行化

在引入新属性之前，我们将创建一个`SecondParallelUnitTest` 类，它有两个类似于`FirstParallelUnitTest:`的方法

```java
public class SecondParallelUnitTest{

    @Test
    public void first() throws Exception{
        System.out.println("SecondParallelUnitTest first() start => " + Thread.currentThread().getName());
        Thread.sleep(500);
        System.out.println("SecondParallelUnitTest first() end => " + Thread.currentThread().getName());
    }

    @Test
    public void second() throws Exception{
        System.out.println("SecondParallelUnitTest second() start => " + Thread.currentThread().getName());
        Thread.sleep(500);
        System.out.println("SecondParallelUnitTest second() end => " + Thread.currentThread().getName());
    }
}
```

在我们在同一个批处理中运行我们的测试之前，我们需要设置属性:

```java
junit.jupiter.execution.parallel.mode.classes.default = concurrent
```

当我们运行这两个测试类时，我们得到以下输出:

```java
SecondParallelUnitTest second() start => ForkJoinPool-1-worker-23
FirstParallelUnitTest first() start => ForkJoinPool-1-worker-19
FirstParallelUnitTest second() start => ForkJoinPool-1-worker-9
SecondParallelUnitTest first() start => ForkJoinPool-1-worker-5
FirstParallelUnitTest first() end => ForkJoinPool-1-worker-19
SecondParallelUnitTest first() end => ForkJoinPool-1-worker-5
FirstParallelUnitTest second() end => ForkJoinPool-1-worker-9
SecondParallelUnitTest second() end => ForkJoinPool-1-worker-23
```

从输出中，我们可以看到所有四个测试在不同的线程中并行运行。

结合我们在本节和上一节中提到的两个属性及其值(`same_thread and concurrent`)，我们得到四种不同的执行模式:

1.  (`same_thread, same_thread`)–所有测试按顺序运行
2.  (`same_thread, concurrent`)–来自一个类的测试按顺序运行，但是多个类并行运行
3.  (`concurrent, same_thread`)–来自一个类的测试并行运行，但是每个类单独运行
4.  (`concurrent, concurrent`)–测试并行运行

## 5.同步

在理想情况下，我们所有的单元测试都是独立和隔离的。然而，有时这很难实现，因为它们依赖于共享资源。然后，当并行运行测试时，我们需要同步测试中的公共资源。JUnit5 以`@ResourceLock`注释的形式为我们提供了这样的机制。

类似地，和以前一样，让我们创建`ParallelResourceLockUnitTest` 类:

```java
public class ParallelResourceLockUnitTest{
    private List<String> resources;
    @BeforeEach
    void before() {
        resources = new ArrayList<>();
        resources.add("test");
    }
    @AfterEach
    void after() {
        resources.clear();
    }
    @Test
    @ResourceLock(value = "resources")
    public void first() throws Exception {
        System.out.println("ParallelResourceLockUnitTest first() start => " + Thread.currentThread().getName());
        resources.add("first");
        System.out.println(resources);
        Thread.sleep(500);
        System.out.println("ParallelResourceLockUnitTest first() end => " + Thread.currentThread().getName());
    }
    @Test
    @ResourceLock(value = "resources")
    public void second() throws Exception {
        System.out.println("ParallelResourceLockUnitTest second() start => " + Thread.currentThread().getName());
        resources.add("second");
        System.out.println(resources);
        Thread.sleep(500);
        System.out.println("ParallelResourceLockUnitTest second() end => " + Thread.currentThread().getName());
    }
}
```

**`@ResourceLock`允许我们指定哪个资源是共享的，以及我们想要使用的锁的类型(默认为`ResourceAccessMode.READ_WRITE` )** 。使用当前的设置，JUnit 引擎将检测到我们的测试使用了共享资源，并将按顺序执行它们:

```java
ParallelResourceLockUnitTest second() start => ForkJoinPool-1-worker-5
[test, second]
ParallelResourceLockUnitTest second() end => ForkJoinPool-1-worker-5
ParallelResourceLockUnitTest first() start => ForkJoinPool-1-worker-19
[test, first]
ParallelResourceLockUnitTest first() end => ForkJoinPool-1-worker-19
```

## 6.结论

在本文中，首先，我们介绍了如何配置并行执行。接下来，什么是可用的并行策略，以及如何配置大量线程？之后，我们讨论了不同的配置如何影响测试执行。最后，我们讨论了共享资源的同步。

和往常一样，这篇文章的代码可以在 GitHub 上找到[。](https://web.archive.org/web/20221223204047/https://github.com/eugenp/tutorials/tree/master/testing-modules/junit-5-advanced)