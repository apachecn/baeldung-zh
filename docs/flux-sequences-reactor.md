# 使用 Project Reactor 以编程方式创建序列

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/flux-sequences-reactor>

## 1。概述

在本教程中，我们将使用[项目反应器基础知识](/web/20220628145912/https://www.baeldung.com/reactor-core)来学习一些创建 [`Flux` es](https://web.archive.org/web/20220628145912/https://projectreactor.io/docs/core/release/api/reactor/core/publisher/Flux.html) 的技术。

## 2。Maven 依赖关系

让我们从几个依赖项开始。我们需要`[reactor-core](https://web.archive.org/web/20220628145912/https://search.maven.org/search?q=g:io.projectreactor%20AND%20a:reactor-core&core=gav) `和 [`reactor-test`](https://web.archive.org/web/20220628145912/https://search.maven.org/search?q=g:io.projectreactor%20AND%20a:reactor-test) :

```java
<dependency>
    <groupId>io.projectreactor</groupId>
    <artifactId>reactor-core</artifactId>
    <version>3.2.6.RELEASE</version>
</dependency>
<dependency>
    <groupId>io.projectreactor</groupId>
    <artifactId>reactor-test</artifactId>
    <version>3.2.6.RELEASE</version>
    <scope>test</scope>
</dependency>
```

## 3。同步发射

创建一个`Flux`最简单的方法就是`Flux#` `generate`。**该方法依靠生成器函数来生成一系列项目。**

但是首先，让我们定义一个类来保存我们的方法，说明`generate`方法:

```java
public class SequenceGenerator {
    // methods that will follow
}
```

### 3.1。具有新状态的发电机

让我们看看如何用 Reactor 生成[斐波那契数列](https://web.archive.org/web/20220628145912/https://en.wikipedia.org/wiki/Fibonacci_number):

```java
public Flux<Integer> generateFibonacciWithTuples() {
    return Flux.generate(
            () -> Tuples.of(0, 1),
            (state, sink) -> {
                sink.next(state.getT1());
                return Tuples.of(state.getT2(), state.getT1() + state.getT2());
            }
    );
}
```

不难看出这个 [`generate`](https://web.archive.org/web/20220628145912/https://projectreactor.io/docs/core/release/api/reactor/core/publisher/Flux.html#generate-java.util.concurrent.Callable-java.util.function.BiFunction-) 方法以两个函数作为它的参数——一个`Callable`和一个`BiFunction`:

*   `Callable`函数设置发生器的初始状态——在本例中，它是一个带有元素`0`和`1`的 [`Tuples`](https://web.archive.org/web/20220628145912/https://projectreactor.io/docs/core/release/api/reactor/util/function/Tuples.html)
*   `BiFuntion`函数是一个生成器，消耗一个 [`SynchronousSink`](https://web.archive.org/web/20220628145912/https://projectreactor.io/docs/core/release/api/reactor/core/publisher/SynchronousSink.html) ，然后用水槽的`next`方法和当前状态每轮发射一个物品

顾名思义，`SynchronousSink`对象是同步工作的。然而，**注意，我们不能在每个生成器调用中多次调用这个对象的`next`方法。**

让我们用 [`StepVerifier`](/web/20220628145912/https://www.baeldung.com/reactive-streams-step-verifier-test-publisher) 来验证生成的序列:

```java
@Test
public void whenGeneratingNumbersWithTuplesState_thenFibonacciSequenceIsProduced() {
    SequenceGenerator sequenceGenerator = new SequenceGenerator();
    Flux<Integer> fibonacciFlux = sequenceGenerator.generateFibonacciWithTuples().take(5);

    StepVerifier.create(fibonacciFlux)
      .expectNext(0, 1, 1, 2, 3)
      .expectComplete()
      .verify();
}
```

在这个例子中，订户只请求了五个项目，因此生成的序列以数字`3`结束。

正如我们所见，**生成器返回一个新的状态**对象，将在下一次传递中使用。**虽然没有必要这样做。**我们可以为生成器的所有调用重用一个状态实例。

### 3.2。具有可变状态的发电机

假设我们想要生成具有循环状态的斐波那契数列。为了演示这个用例，让我们首先定义一个类:

```java
public class FibonacciState {
    private int former;
    private int latter;

    // constructor, getters and setters
}
```

我们将使用这个类的一个实例来保存生成器的状态。这个实例的两个属性`former`和`latter`在序列中存储两个连续的数字。

如果我们修改最初的例子，我们现在将使用带有`generate`的可变状态:

```java
public Flux<Integer> generateFibonacciWithCustomClass(int limit) {
    return Flux.generate(
      () -> new FibonacciState(0, 1),
      (state, sink) -> {
        sink.next(state.getFormer());
        if (state.getLatter() > limit) {
            sink.complete();
        }
        int temp = state.getFormer();
        state.setFormer(state.getLatter());
        state.setLatter(temp + state.getLatter());
        return state;
    });
}
```

类似于前面的例子，这个 [`generate`变量](https://web.archive.org/web/20220628145912/https://projectreactor.io/docs/core/release/api/reactor/core/publisher/Flux.html#generate-java.util.concurrent.Callable-java.util.function.BiFunction-java.util.function.Consumer-)具有状态供应商和发电机参数。

类型为`Callable`的状态提供者简单地创建了一个具有初始属性`0`和`1`的`FibonacciState`对象。这个状态对象将在生成器的整个生命周期中被重用。

就像 Fibonacci-with- `Tuples`示例中的`SynchronousSink`一样，这里的接收器一个接一个地产生项目。然而，与那个例子不同，**生成器每次被调用时都返回相同的状态对象。**

还要注意这一次，**为了避免一个无限序列，**我们指示接收在产生的值达到一个极限时完成。

让我们再次做一个快速测试来确认它的工作情况:

```java
@Test
public void whenGeneratingNumbersWithCustomClass_thenFibonacciSequenceIsProduced() {
    SequenceGenerator sequenceGenerator = new SequenceGenerator();

    StepVerifier.create(sequenceGenerator.generateFibonacciWithCustomClass(10))
      .expectNext(0, 1, 1, 2, 3, 5, 8)
      .expectComplete()
      .verify();
}
```

### 3.3。无状态变体

`generate`方法有[的另一个变体](https://web.archive.org/web/20220628145912/https://projectreactor.io/docs/core/release/api/reactor/core/publisher/Flux.html#generate-java.util.function.Consumer-)，它只有一个类型为`Consumer<SynchronousSink>`的参数。这种变体只适合于产生预定的序列，因此没有那么强大。那我们就不详细介绍了。

## 4。异步发射

同步发射不是程序化创建`Flux`的唯一解决方案。

相反，**我们可以使用`create`和`push`操作符以异步方式在一轮发射中产生多个项目。**

### 4.1。`create`法

**使用 [`create`](https://web.archive.org/web/20220628145912/https://projectreactor.io/docs/core/release/api/reactor/core/publisher/Flux.html#create-java.util.function.Consumer-) 方法，我们可以从多个线程中产生项目。**在这个例子中，我们将从两个不同的来源收集元素到一个序列中。

首先，我们来看看`create`和`generate`有什么不同:

```java
public class SequenceCreator {
    public Consumer<List<Integer>> consumer;

    public Flux<Integer> createNumberSequence() {
        return Flux.create(sink -> SequenceCreator.this.consumer = items -> items.forEach(sink::next));
    }
}
```

与`generate`操作符不同，**`create`方法不维护状态。**传递给这个方法的发射器从外部源接收元素，而不是自己生成项目，**。**

同样，我们可以看到`create`操作符要求我们输入一个 [`FluxSink`](https://web.archive.org/web/20220628145912/https://projectreactor.io/docs/core/release/api/reactor/core/publisher/FluxSink.html) 而不是一个`SynchronousSink`。有了`FluxSink`、**，我们可以根据需要多次调用`next() `。**

在我们的例子中，我们将为列表`items`中的每个项目调用`next()`，一个接一个地发出每个项目。我们马上就会看到如何填充`items `。

在这种情况下，我们的外部源是一个虚构的`consumer`字段，尽管这也可能是一些可观察到的 API。

让我们使用`create`操作符，从两个数字序列开始:

```java
@Test
public void whenCreatingNumbers_thenSequenceIsProducedAsynchronously() throws InterruptedException {
    SequenceGenerator sequenceGenerator = new SequenceGenerator();
    List<Integer> sequence1 = sequenceGenerator.generateFibonacciWithTuples().take(3).collectList().block();
    List<Integer> sequence2 = sequenceGenerator.generateFibonacciWithTuples().take(4).collectList().block();

    // other statements described below
}
```

这些序列`sequence1`和`sequence2`将作为生成序列的项目源。

接下来是两个`Thread`对象，它们将元素注入发布者:

```java
SequenceCreator sequenceCreator = new SequenceCreator();
Thread producingThread1 = new Thread(
  () -> sequenceCreator.consumer.accept(sequence1)
);
Thread producingThread2 = new Thread(
  () -> sequenceCreator.consumer.accept(sequence2)
);
```

当调用`accept`操作符时，元素开始流入序列源。

然后，我们可以听，或`subscribe`，我们的新的，巩固的序列:

```java
List<Integer> consolidated = new ArrayList<>();
sequenceCreator.createNumberSequence().subscribe(consolidated::add);
```

通过订阅我们的序列，我们指出序列发出的每个项目应该发生什么。在这里，它将来自不同来源的每个项目添加到一个合并的列表中。

现在，我们触发看到项目在两个不同线程上移动的整个过程:

```java
producingThread1.start();
producingThread2.start();
producingThread1.join();
producingThread2.join();
```

通常，最后一步是验证操作的结果:

```java
assertThat(consolidated).containsExactlyInAnyOrder(0, 1, 1, 0, 1, 1, 2);
```

接收序列中的前三个数字来自`sequence1`，而后四个来自`sequence2`。由于异步操作的性质，这些序列中元素的顺序是不确定的。

`create`方法有[的另一个变体](https://web.archive.org/web/20220628145912/https://projectreactor.io/docs/core/release/api/reactor/core/publisher/Flux.html#create-java.util.function.Consumer-reactor.core.publisher.FluxSink.OverflowStrategy-)，接受类型`[OverflowStrategy](https://web.archive.org/web/20220628145912/https://projectreactor.io/docs/core/release/api/reactor/core/publisher/FluxSink.OverflowStrategy.html)`的参数。顾名思义，当下游跟不上发布者时，这个论点管理反压力。**默认情况下，发布者会缓冲这种情况下的所有元素。**

### 4.2。`push`法

除了`create`操作符之外，`Flux`类还有另一个静态方法来异步发出序列，即 [`push`](https://web.archive.org/web/20220628145912/https://projectreactor.io/docs/core/release/api/reactor/core/publisher/Flux.html#push-java.util.function.Consumer-) 。除了一次只允许一个生产线程发出信号之外，这个方法就像`create`、**一样工作。**

我们可以用`push`替换刚才例子中的`create`方法，代码仍然可以编译

然而，有时我们会看到断言错误，因为`push`操作符阻止了`FluxSink#next`在不同的线程上被并发调用。因此，**只有当我们不打算使用多线程时，才应该使用`push`。**

## 5。处理顺序

到目前为止，我们看到的所有方法都是静态的，允许从给定的源创建序列。**`Flux`API 还提供了一个名为`[handle](https://web.archive.org/web/20220628145912/https://projectreactor.io/docs/core/release/api/reactor/core/publisher/Flux.html#handle-java.util.function.BiConsumer-)`的实例方法，用于处理发行商制作的序列。**

这个`handle`操作符处理一个序列，进行一些处理，并可能删除一些元素。在这方面，**我们可以说`handle`操作符就像`map`和`filter`一样工作。**

让我们看一下`handle`方法的简单说明:

```java
public class SequenceHandler {
    public Flux<Integer> handleIntegerSequence(Flux<Integer> sequence) {
        return sequence.handle((number, sink) -> {
            if (number % 2 == 0) {
                sink.next(number / 2);
            }
        });
    }
}
```

在这个例子中，`handle`操作符接受一个数字序列，如果是偶数，则用值除以`2`。如果值是奇数，操作符不做任何事情，这意味着这样的数字被忽略。

另一件需要注意的事情是，与`generate`方法一样， **`handle`采用了`SynchronousSink`，并且只允许一个接一个的排放。**

最后，我们需要测试一些东西。让我们最后一次使用`StepVerifier`来确认我们的处理程序工作正常:

```java
@Test
public void whenHandlingNumbers_thenSequenceIsMappedAndFiltered() {
    SequenceHandler sequenceHandler = new SequenceHandler();
    SequenceGenerator sequenceGenerator = new SequenceGenerator();
    Flux<Integer> sequence = sequenceGenerator.generateFibonacciWithTuples().take(10);

    StepVerifier.create(sequenceHandler.handleIntegerSequence(sequence))
      .expectNext(0, 1, 4, 17)
      .expectComplete()
      .verify();
}
```

在斐波那契数列的前 10 项中有四个偶数:`0`、`2`、`8`和`34`，因此我们将参数传递给了`expectNext`方法。

## 6。结论

在本文中，我们介绍了可以用来以编程方式生成序列的`Flux` API 的各种方法，特别是`generate`和`create`操作符。

本教程的源代码可以在 GitHub 上的[处获得。这是一个 Maven 项目，应该能够按原样运行。](https://web.archive.org/web/20220628145912/https://github.com/eugenp/tutorials/tree/master/reactor-core)