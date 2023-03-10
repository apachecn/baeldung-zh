# 用 RxJava 处理背压

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/rxjava-backpressure>

## 1。概述

在本文中，我们将看看 [RxJava 库](https://web.archive.org/web/20220812062509/https://github.com/ReactiveX/RxJava)帮助我们处理背压的方式。

简而言之——rx Java 通过引入一个或多个`Observers`可以订阅的`Observables,`,利用了反应流的概念。**处理可能无限的流是非常具有挑战性的，因为我们需要面对一个背压的问题。**

很容易出现这样一种情况，即`Observable`发送物品的速度比订阅者消耗物品的速度快。我们将研究未消耗项目缓冲区增长问题的不同解决方案。

## 2。热`Observables`对冷`Observables`对 

首先，让我们创建一个简单的消费者函数，它将被用作来自`Observables`的元素的消费者，我们将在后面定义:

```java
public class ComputeFunction {
    public static void compute(Integer v) {
        try {
            System.out.println("compute integer v: " + v);
            Thread.sleep(1000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }
}
```

我们的`compute()` 函数只是打印参数。这里需要注意的重要一点是对一个`Thread.sleep(1000)`方法的调用——我们这样做是为了模拟一些长时间运行的任务，这些任务将导致`Observable`更快地填满项目，以便`Observer`能够消耗它们。

我们有两种类型的`Observables – Hot`和`Cold` ——它们在背压处理方面完全不同。

### 2.1。`Observables`寒

一个冷的`Observable`发出一个特定的物品序列，但是当它的`Observer`觉得方便的时候可以开始发出这个序列，并且以`Observer`想要的任何速率，而不会破坏序列的完整性。**冷`Observable`在懒懒的提供物品。**

只有当准备好处理项目时，`Observer`才获取元素，项目不需要在`Observable`中缓冲，因为它们是以拉的方式被请求的。

例如，如果您基于从 1 到 100 万的静态元素范围创建一个`Observable`,那么无论这些项目被观察的频率如何，该`Observable`都将发出相同的项目序列:

```java
Observable.range(1, 1_000_000)
  .observeOn(Schedulers.computation())
  .subscribe(ComputeFunction::compute);
```

当我们开始程序时，项目将由`Observer` 懒惰地计算，并以拉的方式被请求。`Schedulers.computation()`方法意味着我们希望在`RxJava.` 的计算线程池中运行我们的`Observer`

程序的输出将由一个`compute()`方法的结果组成，该方法从一个`Observable`中为一个项目调用:

```java
compute integer v: 1
compute integer v: 2
compute integer v: 3
compute integer v: 4
...
```

Cold `Observables`不需要任何形式的背压，因为它们以拉的方式工作。cold `Observable`发出的项目示例可能包括数据库查询、文件检索或 web 请求的结果。

### 2.2。热点`Observables`

hot `Observable`开始生成项目，并在它们被创建时立即发出。这与冷`Observables` 拉动模式的处理相反。**热`Observable`以自己的速度发射物品，要靠它的观察者才能跟上。**

当`Observer` 无法像`Observable`生产物品那样快速消费物品时，它们需要被缓冲或以其他方式处理，因为它们会填满内存，最终导致`OutOfMemoryException.`

让我们考虑一个热点`Observable,` 的例子，即生产 100 万件商品给正在处理这些商品的最终消费者。当`Observer`中的`compute()`方法花费一些时间来处理每一个项目时，`Observable` 开始用项目填满内存，导致程序失败:

```java
PublishSubject<Integer> source = PublishSubject.<Integer>create();

source.observeOn(Schedulers.computation())
  .subscribe(ComputeFunction::compute, Throwable::printStackTrace);

IntStream.range(1, 1_000_000).forEach(source::onNext); 
```

运行该程序将会失败，并显示一个`MissingBackpressureException` ，因为我们没有定义一种处理过量生产`Observable`的方法。

hot `Observable`发出的项目示例可能包括鼠标&键盘事件、系统事件或股票价格。

## 3。`Observable`缓冲过剩

处理过量生产`Observable` 的第一种方法是为不能被`Observer.` 处理的元素定义某种缓冲区

我们可以通过调用一个`buffer()` 方法来实现:

```java
PublishSubject<Integer> source = PublishSubject.<Integer>create();

source.buffer(1024)
  .observeOn(Schedulers.computation())
  .subscribe(ComputeFunction::compute, Throwable::printStackTrace); 
```

定义一个大小为 1024 的缓冲区将会给`Observer` 一些时间来赶上一个过量生产的源。缓冲区将存储尚未处理的项目。

我们可以增加缓冲区的大小，为生成的值提供足够的空间。

然而，请注意，一般来说，**这可能只是一个临时的解决方案**,因为如果源过度产生预测的缓冲区大小，溢出仍然会发生。

## 4。批量发送项目

我们可以在 N 个元素的窗口中批量生产过剩的项目。

当`Observable`生成元素的速度快于`Observer`处理元素的速度时，我们可以通过将生成的元素组合在一起，并将一批元素发送给能够处理一组元素而不是逐个处理元素的`Observer` 来缓解这种情况:

```java
PublishSubject<Integer> source = PublishSubject.<Integer>create();

source.window(500)
  .observeOn(Schedulers.computation())
  .subscribe(ComputeFunction::compute, Throwable::printStackTrace); 
```

使用带有参数`500,` 的`window()` 方法将告诉`Observable` 将元素分组到 500 大小的批次中。与逐个处理元件相比，当`Observer` 能够更快地处理一批元件时，这种技术可以减少过量生产`Observable` 的问题。

## 5。跳过元素

如果`Observable` 产生的一些值可以安全地忽略，我们可以使用特定时间内的采样和节流操作符。

方法`sample()`和 `throttleFirst()` 以持续时间为参数:

*   s `ample()` 方法定期查看元素序列，并发出在参数指定的持续时间内产生的最后一个项目
*   `throttleFirst()` 方法发出在作为参数指定的持续时间之后产生的第一个项目

持续时间是从产生的元素序列中选取一个特定元素之后的时间。我们可以通过跳过元素来指定处理背压的策略:

```java
PublishSubject<Integer> source = PublishSubject.<Integer>create();

source.sample(100, TimeUnit.MILLISECONDS)
  .observeOn(Schedulers.computation())
  .subscribe(ComputeFunction::compute, Throwable::printStackTrace);
```

我们指定跳过元素的策略将是一个`sample()` 方法。我们想要一个持续时间为 100 毫秒的序列样本。该元素将被发射到`Observer.`

但是，请记住，这些运算符只会降低下游`Observer`接收值的速率，因此它们仍然可能导致`MissingBackpressureException`。

## 6。处理填充`Observable`缓冲器

如果我们的采样或批处理元素的策略无助于填满一个缓冲区`,` ,我们需要实现一个在缓冲区填满时处理案例的策略。

我们需要使用一种`onBackpressureBuffer()` 方法来防止`BufferOverflowException.`

`onBackpressureBuffer()` 方法有三个参数:`Observable` 缓冲区的容量，当缓冲区填满时调用的方法，以及处理需要从缓冲区丢弃的元素的策略。溢出策略在一个`BackpressureOverflow`类中。

当缓冲区填满时，有 4 种类型的操作可以执行:

*   `ON_OVERFLOW_ERROR –` 这是当缓冲区已满时发出`BufferOverflowException`信号的默认行为
*   `ON_OVERFLOW_DEFAULT –` 目前与`ON_OVERFLOW_ERROR`相同
*   `ON_OVERFLOW_DROP_LATEST`–如果发生溢出，当前值将被忽略，一旦下游`Observer`请求，将只传送旧值
*   `ON_OVERFLOW_DROP_OLDEST`–删除缓冲区中最旧的元素，并添加当前值

让我们看看如何指定该策略:

```java
Observable.range(1, 1_000_000)
  .onBackpressureBuffer(16, () -> {}, BackpressureOverflow.ON_OVERFLOW_DROP_OLDEST)
  .observeOn(Schedulers.computation())
  .subscribe(e -> {}, Throwable::printStackTrace); 
```

这里，我们处理溢出缓冲区的策略是丢弃缓冲区中最旧的元素，并添加由`Observable`产生的最新项目。

请注意，最后两种策略会导致流的不连续性，因为它们会删除元素。另外，他们不会发信号`BufferOverflowException`。

## 7。放弃所有过度生产的元素

每当下游的`Observer`没有准备好接收一个元素时，我们可以使用一个`onBackpressureDrop()` 方法从序列中删除这个元素。

我们可以把这个方法看作是一个策略为`ON_OVERFLOW_DROP_LATEST.` 的缓冲区容量设置为零的`onBackpressureBuffer()` 方法

当我们可以安全地忽略来自源`Observable`的值(如鼠标移动或当前 GPS 位置信号)时，该运算符很有用，因为稍后会有更多的最新值:

```java
Observable.range(1, 1_000_000)
  .onBackpressureDrop()
  .observeOn(Schedulers.computation())
  .doOnNext(ComputeFunction::compute)
  .subscribe(v -> {}, Throwable::printStackTrace);
```

方法`onBackpressureDrop()` 正在消除过量生产`Observable`的问题，但需要谨慎使用。

## 8。结论

在本文中，我们研究了过量生产的问题`Observable` 以及处理背压的方法。我们研究了当`Observer` 不能像`Observable.`产生元素那样快地消耗元素时，缓冲、批处理和跳过元素的策略

所有这些示例和代码片段的实现都可以在 [GitHub 项目](https://web.archive.org/web/20220812062509/https://github.com/eugenp/tutorials/tree/master/rxjava-modules/rxjava-core)中找到——这是一个 Maven 项目，因此它应该很容易导入和运行。