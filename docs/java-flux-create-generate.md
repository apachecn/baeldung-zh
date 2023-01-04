# Flux.create 和 Flux.generate 的区别

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-flux-create-generate>

## 1.介绍

Project Reactor 为 JVM 提供了完全非阻塞的编程基础。它提供了反应流规范的实现，并提供了可组合的异步 API，如 Flux。通量是带有几个反应操作符的反应流发布器。它发出 0 到 N 个元素，然后成功完成或出错。[根据我们的需求，它可以用几种不同的方式制作](/web/20220802122224/https://www.baeldung.com/flux-sequences-reactor)。

## 2.理解通量

**一个通量是一个反应流发布器，可以发出 0 到 N 个元素**。它有几个操作符，用于生成、编排和转换通量序列。通量可以成功完成，也可以有错误地完成。

Flux API 在 Flux 上提供了几个静态工厂方法来创建源代码或从几个回调类型中生成。它还提供实例方法和运算符来构建异步处理管道。这个管道产生一个异步序列。

在接下来的部分中，让我们看看 Flux `generate()`和`create()`方法的一些用法。

## 3. **Maven 依赖关系**

我们需要`[reactor-core](https://web.archive.org/web/20220802122224/https://search.maven.org/search?q=g:io.projectreactor%20AND%20a:reactor-core&core=gav) `[`reactor-test`](https://web.archive.org/web/20220802122224/https://search.maven.org/search?q=g:io.projectreactor%20AND%20a:reactor-test)美凤的依赖:

```java
<dependency>
    <groupId>io.projectreactor</groupId>
    <artifactId>reactor-core</artifactId>
    <version>3.4.17</version>
</dependency>
<dependency>
    <groupId>io.projectreactor</groupId>
    <artifactId>reactor-test</artifactId>
    <version>3.4.17</version>
    <scope>test</scope>
</dependency>
```

## 4.通量生成

Flux API 的`generate()`方法提供了一个简单直接的编程方法来创建一个 Flux。`generate()`方法使用一个生成器函数来生成一系列项目。

生成方法有三种变体:

*   `generate(Consumer<SynchronousSink<T>> generator)`
*   `generate(Callable<S> stateSupplier, BiFunction<S, SynchronousSink<T>, S> generator)`
*   `generate(Callable<S> stateSupplier, BiFunction<S, SynchronousSink<T>, S> generator, Consumer<? super S> stateConsumer)`

**生成方法按需计算并发出值**。在计算可能无法在下游使用的元素太昂贵的情况下，最好使用。如果发出的事件受应用程序状态的影响，也可以使用它。

### 4.1.例子

在这个例子中，让我们使用`generate(Callable<S> stateSupplier, BiFunction<S, SynchronousSink<T>, S> generator)`来生成一个`Flux`:

```java
public class CharacterGenerator {

    public Flux<Character> generateCharacters() {

        return Flux.generate(() -> 97, (state, sink) -> {
            char value = (char) state.intValue();
            sink.next(value);
            if (value == 'z') {
                sink.complete();
            }
            return state + 1;
        });
    }
}
```

在`generate()`方法中，我们提供了两个函数作为参数:

*   第一个是一个`Callable`函数。该函数用值 97 定义发电机的初始状态
*   第二个是一个`BiFunction.` 这是一个生成器函数，它消耗一个 [`SynchronousSink.`](https://web.archive.org/web/20220802122224/https://projectreactor.io/docs/core/release/api/reactor/core/publisher/SynchronousSink.html) 每当调用接收器的`next`方法时，这个 SynchronousSink 返回一个项目

根据其名称，`SynchronousSink`实例同步工作。然而，**我们不能在每个生成器调用中多次调用`SynchronousSink`对象的`next`方法。**

让我们用 [`StepVerifier`](/web/20220802122224/https://www.baeldung.com/reactive-streams-step-verifier-test-publisher) 来验证生成的序列:

```java
@Test
public void whenGeneratingCharacters_thenCharactersAreProduced() {
    CharacterGenerator characterGenerator = new CharacterGenerator();
    Flux<Character> characterFlux = characterGenerator.generateCharacters().take(3);

    StepVerifier.create(characterFlux)
      .expectNext('a', 'b', 'c')
      .expectComplete()
      .verify();
}
```

在这个例子中，订户只请求了三个项目。因此，生成的序列以发出三个字符结束——a、b 和 c。`expectNext()`期望我们期望从通量中得到的元素。`expectComplete`()表示从焊剂中发射元素的完成。

## 5.通量创建

**当我们想要计算不受应用程序状态**影响的多个 **值时，使用 Flux 中的`create()`方法。这是因为 Flux `create()`方法的底层方法一直在计算元素。**

此外，下游系统确定它需要多少元素。因此，如果下游系统跟不上，已经发出的元素要么被缓冲，要么被移除。

默认情况下，发出的元素被缓冲，直到下游系统请求更多的元素。

### 5.1.例子

现在让我们演示一下`create()`方法的例子:

```java
public class CharacterCreator {
    public Consumer<List<Character>> consumer;

    public Flux<Character> createCharacterSequence() {
        return Flux.create(sink -> CharacterCreator.this.consumer = items -> items.forEach(sink::next));
    }
}
```

我们可以注意到，`create`操作符要求我们输入一个 [`FluxSink`](https://web.archive.org/web/20220802122224/https://projectreactor.io/docs/core/release/api/reactor/core/publisher/FluxSink.html) ，而不是在`generate`()中使用的`SynchronousSink` 。在这种情况下，我们将为列表`items`中的每一项调用`next()`，一个接一个地发出。

现在让我们使用带有两个字符序列的`CharacterCreator`:

```java
@Test
public void whenCreatingCharactersWithMultipleThreads_thenSequenceIsProducedAsynchronously() throws InterruptedException {
    CharacterGenerator characterGenerator = new CharacterGenerator();
    List<Character> sequence1 = characterGenerator.generateCharacters().take(3).collectList().block();
    List<Character> sequence2 = characterGenerator.generateCharacters().take(2).collectList().block();
} 
```

我们在上面的代码片段中创建了两个序列—`sequence1`和`sequence2`。这些序列作为角色项目的来源。注意，我们使用`CharacterGenerator`实例来获取字符序列。

现在让我们定义一个`characterCreator` 实例和两个线程实例:

```java
CharacterCreator characterCreator = new CharacterCreator();
Thread producerThread1 = new Thread(() -> characterCreator.consumer.accept(sequence1));
Thread producerThread2 = new Thread(() -> characterCreator.consumer.accept(sequence2));
```

我们现在正在创建两个线程实例，它们将向发布者提供元素。当调用 accept 操作符时，字符元素开始流入序列源。接下来，我们`subscribe`到新的合并序列:

```java
List<Character> consolidated = new ArrayList<>();
characterCreator.createCharacterSequence().subscribe(consolidated::add);
```

请注意，`createCharacterSequence`返回了我们订阅的流量，并使用了`consolidated`列表中的元素。接下来，让我们触发看到项目在两个不同线程上移动的整个过程:

```java
producerThread1.start();
producerThread2.start();
producerThread1.join();
producerThread2.join(); 
```

最后，让我们验证操作的结果:

```java
assertThat(consolidated).containsExactlyInAnyOrder('a', 'b', 'c', 'a', 'b');
```

接收序列中的前三个字符来自`sequence1\.` ，后两个字符来自`sequence2`。因为这是一个异步操作，所以不能保证这些序列中元素的顺序。

## 6.产生通量与产生通量

以下是创建和生成操作之间的一些差异:

| 通量创建 | 通量生成 |
| 这个方法接受一个`Consumer<FluxSink>`的实例 | 这个方法接受一个`Consumer<SynchronousSink>`的实例 |
| Create 方法只调用使用者一次 | 生成方法根据下游应用程序的需要多次调用使用者方法 |
| 消费者可以排放 0..立即 n 个元素 | 只能发射一种元素 |
| 发布者不知道下游状态。因此，create 接受溢出策略作为流量控制的附加参数 | 发布者根据下游应用程序的需求生成元素 |
| 如果需要的话,`FluxSink`允许我们使用多线程来发出元素 | 对于多线程没有用，因为它一次只发出一个元素 |

## 7.结论

在本文中，我们讨论了 Flux API 的创建和生成方法之间的区别。

首先，我们介绍了反应式编程的概念，并讨论了 Flux API。然后，我们讨论了 Flux API 的创建和生成方法。最后，我们提供了 Flux API 的 create 和 generate 方法之间的差异列表。

本教程的源代码可以在 GitHub 上的[处获得。](https://web.archive.org/web/20220802122224/https://github.com/eugenp/tutorials/tree/master/reactor-core)