# 在 Java 中测试多线程代码

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-testing-multithreaded>

## 1.介绍

在本教程中，我们将介绍测试并发程序的一些基础知识。我们将主要关注基于线程的并发性及其在测试中出现的问题。

我们还将了解如何解决这些问题，并在 Java 中有效地测试多线程代码。

## 2.并发编程

并发编程指的是我们**将一大块计算分解成较小的、相对独立的计算**的编程。

本练习的目的是同时运行这些较小的计算，甚至可能是并行的。虽然有几种方法可以实现这一点，但目标都是为了更快地运行程序。

### 2.1.线程和并发编程

随着处理器封装了比以往更多的内核，并发编程正处于有效利用它们的前沿。然而，事实是**并发程序更难设计、编写、测试和维护**。因此，如果我们能为并发程序编写有效的自动化测试用例，我们就能解决这些问题中的一大块。

那么，是什么使得为并发代码编写测试如此困难呢？为了理解这一点，我们必须理解我们如何在程序中实现并发性。最流行的并发编程技术之一是使用线程。

现在，线程可以是本机的，在这种情况下，它们由底层操作系统调度。我们也可以使用所谓的绿色线程，它们由运行时直接调度。

### 2.2.测试并发程序的困难

不管我们使用什么类型的线程，使它们难以使用的是线程通信。如果我们确实设法编写了一个包含线程但没有线程通信的程序，那就再好不过了！更实际的情况是，线程通常必须进行通信。有两种方法可以实现这一点——共享内存和消息传递。

与并发编程相关的大部分**问题产生于使用带有共享内存**的本地线程。出于同样的原因，测试这样的程序是困难的。访问共享内存的多个线程通常需要互斥。我们通常通过使用锁的某种保护机制来实现这一点。

但是这仍然会导致许多问题，比如竞争条件、[活锁、死锁和线程饥饿](/web/20221129020343/https://www.baeldung.com/cs/deadlock-livelock-starvation)，等等。此外，这些问题是间歇性的，因为本地线程的线程调度是完全不确定的。

因此，为并发程序编写能够以确定的方式检测这些问题的有效测试确实是一个挑战！

### 2.3.线程交错的剖析

我们知道本机线程可能会被操作系统不可预测地调度。如果这些**线程访问和修改共享数据，就会产生有趣的线程交错**。虽然这些交错中的一些可能是完全可接受的，但是其他的可能会使最终数据处于不期望的状态。

我们举个例子。假设我们有一个全局计数器，它由每个线程递增。在处理结束时，我们希望该计数器的状态与已经执行的线程数完全相同:

```java
private int counter;
public void increment() {
    counter++;
}
```

现在，在 Java 中增加一个原始整数不是一个原子操作。它包括读取值，增加值，最后保存值。当多个线程执行相同的操作时，可能会产生许多可能的交错:

[![Screenshot-2020-03-27-at-06.53.27](img/6968500ec031052892713d9b018134e2.png)](/web/20221129020343/https://www.baeldung.com/wp-content/uploads/2020/03/Screenshot-2020-03-27-at-06.53.27.png)

虽然这种特殊的交错产生了完全可以接受的结果，但这个呢:

[![Screenshot-2020-03-27-at-06.54.15](img/687abbd4b6828f109a1b26b87abe98a0.png)](/web/20221129020343/https://www.baeldung.com/wp-content/uploads/2020/03/Screenshot-2020-03-27-at-06.54.15.png)

这不是我们所期望的。现在，想象数百个线程运行比这复杂得多的代码。这将产生难以想象的线程交叉方式。

有几种方法可以编写代码来避免这个问题，但这不是本教程的主题。使用锁的同步是常见的方法之一，但是它有与竞争条件相关的问题。

## 3.测试多线程代码

现在我们已经了解了测试多线程代码的基本挑战，我们将看看如何克服它们。我们将构建一个简单的用例，并尝试模拟尽可能多的与并发性相关的问题。

让我们从定义一个简单的类开始，这个类记录可能发生的任何事情:

```java
public class MyCounter {
    private int count;
    public void increment() {
        int temp = count;
        count = temp + 1;
    }
    // Getter for count
}
```

这是一段看似无害的代码，但是**不难理解它不是线程安全的**。如果我们碰巧用这个类写了一个并发程序，它肯定是有缺陷的。这里测试的目的是识别这样的缺陷。

### 3.1.测试非并发零件

作为一条经验法则，**将代码与任何并发行为隔离开来进行测试总是明智的。这是为了合理地确定代码中没有与并发性无关的其他缺陷。让我们看看如何做到这一点:**

```java
@Test
public void testCounter() {
    MyCounter counter = new MyCounter();
    for (int i = 0; i < 500; i++) {
        counter.increment();
    }
    assertEquals(500, counter.getCount());
}
```

虽然这里没有什么，但是这个测试给了我们信心，它至少在没有并发的情况下是有效的。

### 3.2.首次尝试并发测试

让我们继续测试相同的代码，这次是在并发设置中。我们将尝试用多个线程访问该类的同一个实例，并观察它的行为:

```java
@Test
public void testCounterWithConcurrency() throws InterruptedException {
    int numberOfThreads = 10;
    ExecutorService service = Executors.newFixedThreadPool(10);
    CountDownLatch latch = new CountDownLatch(numberOfThreads);
    MyCounter counter = new MyCounter();
    for (int i = 0; i < numberOfThreads; i++) {
        service.execute(() -> {
            counter.increment();
            latch.countDown();
        });
    }
    latch.await();
    assertEquals(numberOfThreads, counter.getCount());
}
```

这个测试是合理的，因为我们试图用几个线程来操作共享数据。当我们保持低线程数时，比如 10，我们会注意到它几乎一直在通过。有趣的是，**如果我们开始增加线程的数量，比方说增加到 100，我们将会看到测试在大多数情况下都会失败**。

### 3.3.并发测试的更好尝试

虽然之前的测试确实揭示了我们的代码不是线程安全的，但是这个测试有一个问题。这个测试是不确定的，因为底层线程以不确定的方式交错。我们的程序真的不能依赖这个测试。

我们需要的是**一种控制线程交错的方法，这样我们就可以用更少的线程以确定的方式揭示并发问题**。我们将从稍微调整一下正在测试的代码开始:

```java
public synchronized void increment() throws InterruptedException {
    int temp = count;
    wait(100);
    count = temp + 1;
}
```

这里，我们创建了方法`synchronized`，并在方法的两个步骤之间引入了一个等待。`synchronized`关键字确保一次只有一个线程可以修改`count`变量，等待会在每个线程执行之间引入一个延迟。

请注意，我们不一定要修改我们想要测试的代码。然而，由于我们没有太多的方法可以影响线程调度，所以我们求助于此。

在后面的部分中，我们将看到如何在不修改代码的情况下做到这一点。

现在，让我们像前面一样测试这段代码:

```java
@Test
public void testSummationWithConcurrency() throws InterruptedException {
    int numberOfThreads = 2;
    ExecutorService service = Executors.newFixedThreadPool(10);
    CountDownLatch latch = new CountDownLatch(numberOfThreads);
    MyCounter counter = new MyCounter();
    for (int i = 0; i < numberOfThreads; i++) {
        service.submit(() -> {
            try {
                counter.increment();
            } catch (InterruptedException e) {
                // Handle exception
            }
            latch.countDown();
        });
    }
    latch.await();
    assertEquals(numberOfThreads, counter.getCount());
}
```

在这里，我们只用两个线程来运行这个程序，有可能我们会发现我们一直没有发现的缺陷。我们在这里所做的是尝试实现一个特定的线程交错，我们知道它会影响我们。虽然对于演示来说很好，**但我们可能会发现这在实际应用中并不有用**。

## 4.可用的测试工具

随着线程数量的增长，它们可能交错的路数也呈指数增长。要找出所有这样的交错并对它们进行测试是不可能的。我们必须依靠工具来为我们承担相同或相似的工作。幸运的是，有一些工具可以让我们的生活变得更轻松。

我们可以使用两大类工具来测试并发代码。第一种方法使我们能够在有许多线程的并发代码上产生相当高的压力。压力增加了罕见交错的可能性，从而增加了我们发现缺陷的机会。

第二个使我们能够模拟特定的线程交错，从而帮助我们更确定地发现缺陷。

### 4.1.坦普斯-富吉特

[tempus-fugit](https://web.archive.org/web/20221129020343/http://tempusfugitlibrary.org/) Java 库**帮助我们轻松编写和测试并发代码**。我们在这里只关注这个库的测试部分。我们之前看到，对多线程代码施加压力会增加发现与并发性相关的缺陷的机会。

虽然我们可以自己编写实用程序来产生压力，但 tempus-fugit 提供了实现这一点的便捷方法。

让我们重温一下之前我们试图产生压力的代码，并理解如何使用 tempus-fugit 实现同样的效果:

```java
public class MyCounterTests {
    @Rule
    public ConcurrentRule concurrently = new ConcurrentRule();
    @Rule
    public RepeatingRule rule = new RepeatingRule();
    private static MyCounter counter = new MyCounter();

    @Test
    @Concurrent(count = 10)
    @Repeating(repetition = 10)
    public void runsMultipleTimes() {
        counter.increment();
    }

    @AfterClass
    public static void annotatedTestRunsMultipleTimes() throws InterruptedException {
        assertEquals(counter.getCount(), 100);
    }
}
```

这里，我们使用 tempus-fugit 提供的两个`Rule`。这些规则截取测试并帮助我们应用期望的行为，比如重复和并发。因此，实际上，我们从十个不同的线程中重复测试操作十次。

随着我们增加重复和并发性，我们检测到与并发性相关的缺陷的机会将会增加。

### 4.2.线织工

[Thread Weaver](https://web.archive.org/web/20221129020343/https://github.com/google/thread-weaver) 本质上是**一个测试多线程代码的 Java 框架**。我们之前已经看到线程交错是非常不可预测的，因此，我们可能永远无法通过常规测试发现某些缺陷。我们实际上需要的是一种控制交织并测试所有可能交织的方法。在我们之前的尝试中，这已经被证明是一项相当复杂的任务。

让我们看看 Thread Weaver 如何帮助我们。Thread Weaver 允许我们以多种方式交叉执行两个独立的线程，而不必担心如何执行。它还让我们有可能对我们希望线程如何交错进行细粒度控制。

让我们看看如何改进我们之前天真的尝试:

```java
public class MyCounterTests {
    private MyCounter counter;

    @ThreadedBefore
    public void before() {
        counter = new MyCounter();
    }
    @ThreadedMain
    public void mainThread() {
        counter.increment();
    }
    @ThreadedSecondary
    public void secondThread() {
        counter.increment();
    }
    @ThreadedAfter
    public void after() {
        assertEquals(2, counter.getCount());
    }

    @Test
    public void testCounter() {
        new AnnotatedTestRunner().runTests(this.getClass(), MyCounter.class);
    }
}
```

这里，我们定义了两个试图增加计数器的线程。Thread Weaver 将尝试在所有可能的交叉场景中使用这些线程运行该测试。可能在某个交错中，我们会发现缺陷，这在我们的代码中非常明显。

### 4.3.多线程 dTC

[多线程](https://web.archive.org/web/20221129020343/http://www.cs.umd.edu/projects/PL/multithreadedtc/overview.html)是**另一个测试并发应用**的框架。它具有一个节拍器，用于对多线程中的活动序列进行精细控制。它支持执行特定线程交错的测试用例。因此，我们应该能够在一个单独的线程中确定性地测试每一个重要的交错。

现在，对这个功能丰富的库的完整介绍已经超出了本教程的范围。但是，我们当然可以看到如何快速设置测试，为我们提供执行线程之间可能的交错。

让我们看看如何用多线程更确定地测试我们的代码 dTC:

```java
public class MyTests extends MultithreadedTestCase {
    private MyCounter counter;
    @Override
    public void initialize() {
        counter = new MyCounter();
    }
    public void thread1() throws InterruptedException {
        counter.increment();
    }
    public void thread2() throws InterruptedException {
        counter.increment();
    }
    @Override
    public void finish() {
        assertEquals(2, counter.getCount());
    }

    @Test
    public void testCounter() throws Throwable {
        TestFramework.runManyTimes(new MyTests(), 1000);
    }
}
```

这里，我们设置了两个线程来操作共享计数器并递增它。我们将 MultithreadedTC 配置为使用这些线程对多达一千个不同的交错执行该测试，直到它检测到一个失败为止。

### 4.4.Java `jcstress`

OpenJDK 维护代码工具项目，为开发人员提供处理 OpenJDK 项目的工具。这个项目下有几个有用的工具，包括 [Java 并发压力测试(jcstress)](https://web.archive.org/web/20221129020343/https://openjdk.java.net/projects/code-tools/jcstress/) 。这是作为实验工具和测试套件开发的，用于调查 Java 中并发支持的正确性。

尽管这是一个实验性的工具，我们仍然可以利用它来分析并发代码并编写测试来解决相关的缺陷。让我们看看如何测试我们在本教程中一直使用的代码。从使用的角度来看，这个概念非常相似:

```java
@JCStressTest
@Outcome(id = "1", expect = ACCEPTABLE_INTERESTING, desc = "One update lost.")
@Outcome(id = "2", expect = ACCEPTABLE, desc = "Both updates.")
@State
public class MyCounterTests {

    private MyCounter counter;

    @Actor
    public void actor1() {
        counter.increment();
    }

    @Actor
    public void actor2() {
        counter.increment();
    }

    @Arbiter
    public void arbiter(I_Result r) {
        r.r1 = counter.getCount();
    }
}
```

这里，我们用注释`State`标记了这个类，这表明它保存了被多个线程变异的数据。此外，我们使用了一个注释`Actor`，它标记了保存不同线程所做动作的方法。

最后，我们有一个用注释`Arbiter`标记的方法，它实际上只在所有的`Actor`都访问了它之后才访问状态。我们还使用注释`Outcome`来定义我们的期望。

总的来说，这种设置非常简单直观。我们可以使用框架给出的一个测试工具来运行它，这个测试工具会找到所有用`JCStressTest`注释的类，并在几次迭代中执行它们以获得所有可能的交错。

## 5.检测并发问题的其他方法

为并发代码编写测试是困难的，但却是可能的。我们已经看到了挑战和一些克服挑战的流行方法。然而，**我们可能无法单独通过测试识别所有可能的并发问题**——尤其是当编写更多测试的增量成本开始超过它们的好处时。

因此，结合合理数量的自动化测试，我们可以使用其他技术来识别并发性问题。这将增加我们发现并发性问题的机会，而不会太深入自动化测试的复杂性。我们将在本节中讨论其中的一些内容。

### 5.1.静态分析

静态分析**是指在不实际执行程序的情况下对程序进行分析**。现在，这样的分析能有什么好处呢？我们将会谈到这一点，但是让我们首先理解它与动态分析的对比。到目前为止，我们编写的单元测试需要实际执行它们测试的程序。这就是为什么它们是我们通常所说的动态分析的一部分。

请注意，静态分析绝不是动态分析的替代品。然而，它提供了一个宝贵的工具，可以在我们执行代码之前很久就检查代码结构并识别可能的缺陷。静态分析利用了大量的模板，这些模板是根据经验和理解精心策划的。

虽然很有可能只是浏览代码，并与我们策划的最佳实践和规则进行比较，但我们必须承认，这对于更大的程序来说是不合理的。然而，有几个工具可以为我们执行这种分析。它们相当成熟，拥有大多数流行编程语言的大量规则。

一个流行的 Java 静态分析工具是 FindBugs。FindBugs 寻找“错误模式”的实例。bug 模式是一个经常出错的代码习语。这可能是由于几个原因造成的，比如困难的语言特性、被误解的方法和被误解的不变量。

FindBugs 检查 Java 字节码中 bug 模式的出现，而不实际执行字节码。这是非常方便的使用和快速运行。FindBugs 报告属于许多类别的错误，如条件、设计和重复代码。

它还包括与并发性相关的缺陷。但是，必须注意，FindBugs 可能会报告误报。这些在实践中较少，但必须与手动分析相关联。

### 5.2.模型检查

模型检查**是一种检查系统的有限状态模型是否满足给定规范**的方法。现在，这个定义可能听起来太学术了，但是暂时忍一忍吧！

我们通常可以将计算问题表示为有限状态机。虽然这本身是一个很大的领域，但它给了我们一个模型，其中有一组有限的状态，以及它们之间的转换规则，有明确定义的开始和结束状态。

现在，**规范定义了一个模型应该如何表现才能被认为是正确的**。本质上，这个规范包含了模型所代表的系统的所有需求。获取规格说明的方法之一是使用阿米尔·伯努利开发的时序逻辑公式。

虽然手动执行模型检查在逻辑上是可能的，但这是非常不切实际的。幸运的是，这里有许多工具可以帮助我们。Java 可用的一个这样的工具是 [Java PathFinder](https://web.archive.org/web/20221129020343/http://javapathfinder.sourceforge.net/) (JPF)。JPF 是由美国宇航局多年的经验和研究开发出来的。

具体来说， **JPF 是 Java 字节码**的模型检查器。它以所有可能的方式运行程序，从而沿着所有可能的执行路径检查是否有违反属性的情况，如死锁和未处理的异常。因此，在任何程序中发现与并发性相关的缺陷时，它都是非常有用的。

## 6.事后思考

到目前为止，我们不应该惊讶于**尽可能避免与多线程代码**相关的复杂性。开发设计简单、易于测试和维护的程序应该是我们的首要目标。我们不得不承认并发编程对于现代应用程序来说经常是必要的。

然而，**我们可以在开发并发程序时采用一些最佳实践和原则**,让我们的生活更加轻松。在本节中，我们将介绍一些最佳实践，但是我们应该记住，这个列表还远远不够完整！

### 6.1.降低复杂性

复杂性是一个因素，即使没有任何并发元素，它也会使测试程序变得困难。这只是在并发面前的复合。不难理解为什么**更简单和更小的程序更容易推理，从而更有效地测试**。这里有几个最好的模式可以帮助我们，比如 SRP(单一责任模式)和 KISS(简单明了)，这里仅举几个例子。

现在，虽然这些没有直接解决为并发代码编写测试的问题，但是它们使得这项工作更容易尝试。

### 6.2.考虑原子操作

原子操作**是彼此完全独立运行的操作**。因此，可以简单地避免预测和测试交织的困难。比较和交换就是这样一种广泛使用的原子指令。简单地说，它将内存位置的内容与给定值进行比较，只有当它们相同时，才修改该内存位置的内容。

大多数现代微处理器都提供这种指令的某种变体。Java 提供了一系列原子类，如`AtomicInteger`和`AtomicBoolean`，在底层提供了比较和交换指令的好处。

### 6.3.拥抱永恒

在多线程编程中，可以更改的共享数据总是会留下出错的空间。不变性**是指数据结构在实例化**后不能被修改的情况。这是并发程序的天作之合。如果一个对象的状态在创建后不能被改变，那么竞争的线程就不必在它们上面申请互斥。这大大简化了并发程序的编写和测试。

然而，请注意，我们可能并不总是有选择不变性的自由，但我们必须在可能的时候选择它。

### 6.4.避免共享内存

大多数与多线程编程相关的问题都可以归因于我们在竞争线程之间共享内存这一事实。如果我们能摆脱他们呢！我们仍然需要一些机制来让线程进行通信。

并发应用程序的替代设计模式为我们提供了这种可能性。其中一个流行的是 actor 模型，它规定 Actor 是并发的基本单元。在这个模型中，参与者通过发送消息来进行交互。

Akka 是用 Scala 编写的框架，它利用 [Actor 模型](https://web.archive.org/web/20221129020343/https://doc.akka.io//docs/akka/2.5/actors.html#introduction)来提供更好的并发原语。

## 7.结论

在本教程中，我们讲述了与并发编程相关的一些基础知识。我们特别详细地讨论了 Java 中的多线程并发。在测试这些代码时，我们经历了它给我们带来的挑战，特别是在共享数据的情况下。此外，我们研究了一些可用于测试并发代码的工具和技术。

我们还讨论了避免并发问题的其他方法，包括自动化测试之外的工具和技术。最后，我们回顾了一些与并发编程相关的编程最佳实践。

这篇文章的源代码可以在 GitHub 上找到[。](https://web.archive.org/web/20221129020343/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-concurrency-2)