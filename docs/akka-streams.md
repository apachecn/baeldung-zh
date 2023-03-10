# 阿卡溪流指南

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/akka-streams>

## 1。概述

在这篇文章中，我们将看到建立在 Akka actor 框架之上的 [`akka-streams`](https://web.archive.org/web/20220628054908/http://doc.akka.io/docs/akka/current/scala/stream/) 库，它遵循[反应流宣言](https://web.archive.org/web/20220628054908/http://www.reactive-streams.org/)。**Akka Streams API 允许我们从独立的步骤中轻松地组合数据转换流。**

此外，所有处理都是以一种反应式、非阻塞和异步的方式完成的。

## 2。Maven 依赖关系

首先，我们需要将`[akka-stream](https://web.archive.org/web/20220628054908/https://search.maven.org/classic/#search%7Cgav%7C1%7Cg%3A%22com.typesafe.akka%22%20AND%20a%3A%22akka-stream_2.11%22)` 和 [`akka-stream-testkit`](https://web.archive.org/web/20220628054908/https://search.maven.org/classic/#search%7Cgav%7C1%7Cg%3A%22com.typesafe.akka%22%20AND%20a%3A%22akka-stream-testkit_2.11%22) 库添加到我们的`pom.xml:`中

```java
<dependency>
    <groupId>com.typesafe.akka</groupId>
    <artifactId>akka-stream_2.11</artifactId>
    <version>2.5.2</version>
</dependency>
<dependency>
    <groupId>com.typesafe.akka</groupId>
    <artifactId>akka-stream-testkit_2.11</artifactId>
    <version>2.5.2</version>
</dependency>
```

## 3。Akka 流 API

要使用 Akka 流，我们需要了解核心 API 概念:

*   **`[Source](https://web.archive.org/web/20220628054908/http://doc.akka.io/japi/akka/2.5.2/akka/stream/scaladsl/Source.html) –` 在`akka-stream`库**中处理的入口点——我们可以从多个源创建这个类的一个实例；例如，如果我们想从单个`String`创建一个`Source`，或者我们可以从一个`Iterable`元素创建一个`Source`，我们可以使用`single()` 方法
*   **[`Flow`](https://web.archive.org/web/20220628054908/http://doc.akka.io/japi/akka/2.5.2/akka/stream/scaladsl/Flow.html)–主处理模块**–每个`Flow` 实例都有一个输入和一个输出值
*   如果我们希望我们的`Flow`有一些副作用，比如记录日志或者保存结果，e 可以使用一个；最常见的是，我们将把`NotUsed` 别名作为`Materializer`传递，以表示我们的`Flow` 不应该有任何副作用
*   **`[Sink](https://web.archive.org/web/20220628054908/http://doc.akka.io/japi/akka/2.5.2/akka/stream/scaladsl/Sink.html)` 操作——当我们构建一个`Flow,` 时，它不会被执行，直到我们在它上面注册一个`[Sink](https://web.archive.org/web/20220628054908/http://doc.akka.io/japi/akka/2.5.2/akka/stream/scaladsl/Sink.html)` 操作**——它是一个终端操作，触发整个`Flow`中的所有计算

## 4。在阿卡流中创建`Flows`

让我们从构建一个简单的例子开始，在这里我们将展示如何**创建和组合多个`Flow`的**——处理一个整数流，并从该流中计算整数对的平均移动窗口。

我们将解析分号分隔的整数`String`作为输入，为示例创建我们的`akka-stream Source` 。

### 4.1。使用一个`Flow` 来解析输入

首先，让我们创建一个`DataImporter` 类，它将接受`ActorSystem` 的一个实例，稍后我们将使用它来创建我们的`Flow`:

```java
public class DataImporter {
    private ActorSystem actorSystem;

    // standard constructors, getters...
}
```

接下来，让我们创建一个`parseLine`方法，它将从我们的分隔输入`String.` 中生成一个`Integer`的`List`。请记住，我们在这里使用 Java Stream API 只是为了解析:

```java
private List<Integer> parseLine(String line) {
    String[] fields = line.split(";");
    return Arrays.stream(fields)
      .map(Integer::parseInt)
      .collect(Collectors.toList());
}
```

我们的初始`Flow` 将把`parseLine`应用到我们的输入中，创建一个输入类型为`String`输出类型为`Integer`的`Flow`:

```java
private Flow<String, Integer, NotUsed> parseContent() {
    return Flow.of(String.class)
      .mapConcat(this::parseLine);
}
```

当我们调用`parseLine()` 方法时，编译器知道 lambda 函数的参数将是一个`String`，与我们的`Flow`的输入类型相同。

注意，我们使用的是`mapConcat()` 方法——相当于 Java 8 `flatMap()` 方法——因为我们想要将`parseLine()`返回的`Integer`的`List`展平为`Integer`的`Flow`,这样我们处理中的后续步骤就不需要处理`List`。

### 4.2。使用`Flow`来执行计算

此时，我们已经有了解析过的整数的`Flow`。现在，我们需要**实现将所有输入元素分组成对的逻辑，并计算这些对的平均值**。

现在，我们将**创建一个`Integer`的`Flow` ，并使用`grouped()` 方法**将它们分组。

接下来，我们要计算一个平均值。

由于我们对这些平均值的处理顺序不感兴趣，我们可以通过使用`mapAsyncUnordered()` 方法，将线程数量作为参数传递给该方法，使用多线程来并行计算平均值。

将作为 lambda 传递给`Flow` 的动作需要返回一个`CompletableFuture` ，因为该动作将在单独的线程中异步计算:

```java
private Flow<Integer, Double, NotUsed> computeAverage() {
    return Flow.of(Integer.class)
      .grouped(2)
      .mapAsyncUnordered(8, integers ->
        CompletableFuture.supplyAsync(() -> integers.stream()
          .mapToDouble(v -> v)
          .average()
          .orElse(-1.0)));
}
```

我们正在计算八个并行线程的平均值。注意，我们使用 Java 8 Stream API 来计算平均值。

### 4.3。将多个`Flows`合成一个`Flow`

`Flow` API 是一个流畅的抽象，它允许我们**组合多个`Flow`实例来实现我们的最终处理目标**。我们可以有粒度流，例如，一个正在解析`JSON,`，另一个正在做一些转换，另一个正在收集一些统计数据。

这样的粒度将帮助我们创建更多可测试的代码，因为我们可以独立地测试每个处理步骤。

我们在上面创建了两个可以相互独立工作的流。现在，我们想把它们组合在一起。

首先，我们想要解析我们的输入`String`，接下来，我们想要计算元素流的平均值。

我们可以使用`via()` 方法来组合我们的流:

```java
Flow<String, Double, NotUsed> calculateAverage() {
    return Flow.of(String.class)
      .via(parseContent())
      .via(computeAverage());
}
```

我们创建了一个输入类型为`String`的`Flow` 和其后的两个流。`parseContent()`T3 接受一个`String`输入并返回一个`Integer`作为输出。`computeAverage() Flow` 取那个`Integer`并计算一个平均返回`Double`作为输出类型。

## 5。将`Sink`添加到`Flow`

正如我们提到的，到目前为止，整个`Flow`还没有被执行，因为它是懒惰的。**为了开始执行`Flow`，我们需要定义一个`Sink`T5。例如，`Sink`操作可以将数据保存到数据库中，或者将结果发送到一些外部 web 服务。**

假设我们有一个带有下面的`save()` 方法的`AverageRepository`类，它将结果写入我们的数据库:

```java
CompletionStage<Double> save(Double average) {
    return CompletableFuture.supplyAsync(() -> {
        // write to database
        return average;
    });
}
```

现在，我们想要创建一个`Sink`操作，它使用这个方法来保存我们的`Flow`处理的结果。为了创建我们的`Sink,`，我们首先需要**创建一个`Flow`，它将我们的处理结果作为输入类型**。接下来，我们希望将所有结果保存到数据库中。

同样，我们不关心元素的排序，所以我们可以使用`mapAsyncUnordered()` 方法**并行执行`save()`操作**。

为了从`Flow`创建一个`Sink`,我们需要调用`toMat()` ,将`Sink.ignore()` 作为第一个参数，将`Keep.right()`作为第二个参数，因为我们想要返回处理的状态:

```java
private Sink<Double, CompletionStage<Done>> storeAverages() {
    return Flow.of(Double.class)
      .mapAsyncUnordered(4, averageRepository::save)
      .toMat(Sink.ignore(), Keep.right());
}
```

## 6。`Flow`为定义一个源

我们需要做的最后一件事是**从输入** `**String**.` 创建一个`Source`我们可以使用`via()` 方法将一个`calculateAverage()` `Flow`应用到这个源。

然后，为了将`Sink` 添加到处理中，我们需要调用`runWith()`方法并传递我们刚刚创建的`storeAverages() Sink`:

```java
CompletionStage<Done> calculateAverageForContent(String content) {
    return Source.single(content)
      .via(calculateAverage())
      .runWith(storeAverages(), ActorMaterializer.create(actorSystem))
      .whenComplete((d, e) -> {
          if (d != null) {
              System.out.println("Import finished ");
          } else {
              e.printStackTrace();
          }
      });
}
```

注意，当处理完成时，我们添加了`whenComplete()`回调，其中我们可以根据处理的结果执行一些动作。

## 7。测试`Akka Streams`

我们可以使用`akka-stream-testkit.` 来测试我们的处理

**测试实际处理逻辑的最佳方式是测试所有的`Flow` 逻辑，并使用`TestSink` 来触发计算并断言结果。**

在我们的测试中，我们创建了我们想要测试的`Flow`，接下来，我们从测试输入内容创建了一个`Source` :

```java
@Test
public void givenStreamOfIntegers_whenCalculateAverageOfPairs_thenShouldReturnProperResults() {
    // given
    Flow<String, Double, NotUsed> tested = new DataImporter(actorSystem).calculateAverage();
    String input = "1;9;11;0";

    // when
    Source<Double, NotUsed> flow = Source.single(input).via(tested);

    // then
    flow
      .runWith(TestSink.probe(actorSystem), ActorMaterializer.create(actorSystem))
      .request(4)
      .expectNextUnordered(5d, 5.5);
}
```

我们正在检查是否期望四个输入参数，并且两个结果(平均值)可以以任何顺序到达，因为我们的处理是以异步和并行的方式完成的。

## 8。结论

在本文中，我们在看`akka-stream` 库。

我们定义了一个结合多个`Flows`来计算元素移动平均值的过程。然后，我们定义了一个作为流处理入口点的`Source`和一个触发实际处理的`Sink`。

最后，我们使用`akka-stream-testkit`为我们的处理编写了一个测试。

所有这些示例和代码片段的实现都可以在 [GitHub 项目](https://web.archive.org/web/20220628054908/https://github.com/eugenp/tutorials/tree/master/akka-modules/akka-streams)中找到——这是一个 Maven 项目，因此它应该很容易导入和运行。