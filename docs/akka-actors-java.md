# Java 中的 Akka Actors 简介

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/akka-actors-java>

## 1。简介

**[Akka](https://web.archive.org/web/20221023123703/https://akka.io/) 是一个开源库，通过利用 Actor 模型，帮助使用 Java 或 Scala 轻松开发并发和分布式应用**。

在本教程中，**我们将介绍一些基本特性，如定义演员、他们如何交流以及我们如何杀死他们**。在最后的笔记中，我们还将记录一些使用 Akka 时的最佳实践。

## 2。演员模型

演员模型对计算机科学界来说并不陌生。它最初是由 Carl Eddie Hewitt 在 1973 年提出的，作为处理并发计算的理论模型。

当软件行业开始意识到实现并发和分布式应用程序的缺陷时，它开始显示出它的实际适用性。

**一个 actor 代表一个独立的计算单元。**一些重要的特征是:

*   参与者封装了其状态和部分应用程序逻辑
*   参与者只通过异步消息交互，从不通过直接的方法调用
*   每个参与者都有一个唯一的地址和一个邮箱，其他参与者可以在其中传递消息
*   actor 将按顺序处理邮箱中的所有消息(邮箱的默认实现是 FIFO 队列)
*   actor 系统以树状层次结构组织
*   一个执行元可以创建其他执行元，可以向任何其他执行元发送消息并停止自己或任何已创建的执行元

### 2.1。优势

开发并发应用很困难，因为我们需要处理同步、锁和共享内存。通过使用 Akka actors，我们可以轻松地编写异步代码，而不需要锁和同步。

使用消息而不是方法调用的优点之一是**发送方线程在向另一个参与者**发送消息时不会阻塞来等待返回值。接收方参与者将通过向发送方发送回复消息来响应结果。

使用消息的另一个好处是，我们不必担心多线程环境中的同步问题。这是因为所有的消息都是按顺序处理的。

Akka actor 模型的另一个优点是错误处理。通过将参与者组织成一个层次结构，每个参与者都可以将失败通知给它的父参与者，这样它就可以相应地采取行动。父参与者可以决定停止或重新启动子参与者。

## 3。设置

为了利用 Akka actors，我们需要从 [Maven Central](https://web.archive.org/web/20221023123703/https://search.maven.org/classic/#search%7Cga%7C1%7Ca%3A%22akka-actor_2.12%22) 添加以下依赖项:

```java
<dependency>
    <groupId>com.typesafe.akka</groupId>
    <artifactId>akka-actor_2.12</artifactId>
    <version>2.5.11</version>
</dependency> 
```

## 4。创建演员

如前所述，参与者是在层级系统中定义的。共享一个公共配置的所有参与者将由一个`ActorSystem.`定义

现在，我们将简单地用默认配置和自定义名称定义一个`ActorSystem`:

```java
ActorSystem system = ActorSystem.create("test-system"); 
```

**尽管我们还没有创建任何参与者，但是系统已经包含了 3 个主要参与者:**

*   具有地址“/”的根监护人行动者，如名称所述，其代表行动者系统层级的根
*   具有地址“/user”的用户监护人演员。这将是我们定义的所有 actor 的父级
*   地址为“/system”的系统监护人 actor。这将是 Akka 系统内部定义的所有参与者的父级

任何 Akka 参与者都将扩展`AbstractActor`抽象类并实现`createReceive()`方法来处理来自其他参与者的传入消息:

```java
public class MyActor extends AbstractActor {
    public Receive createReceive() {
        return receiveBuilder().build();
    }
}
```

**这是我们能创造的最基本的演员。**它可以接收来自其他参与者的消息，并将丢弃它们，因为在`ReceiveBuilder.`中没有定义匹配的消息模式，我们将在本文稍后讨论消息模式匹配。

既然我们已经创建了第一个演员，我们应该将它包含在`ActorSystem`中:

```java
ActorRef readingActorRef 
  = system.actorOf(Props.create(MyActor.class), "my-actor");
```

### 4.1。演员配置

**`Props`类包含了 actor 配置。**我们可以配置调度程序、邮箱或部署配置。这个类是不可变的，因此是线程安全的，所以在创建新的 actors 时可以共享它。

强烈推荐在 actor 对象中定义工厂方法，并将其视为最佳实践，该方法将处理`Props`对象的创建。

举例来说，让我们定义一个执行一些文本处理的 actor。actor 将接收一个`String`对象，并对其进行处理:

```java
public class ReadingActor extends AbstractActor {
    private String text;

    public static Props props(String text) {
        return Props.create(ReadingActor.class, text);
    }
    // ...
}
```

现在，要创建这种类型的 actor 的实例，我们只需使用`props()`工厂方法将`String`参数传递给构造函数:

```java
ActorRef readingActorRef = system.actorOf(
  ReadingActor.props(TEXT), "readingActor");
```

现在我们知道了如何定义一个 actor，让我们看看他们在 actor 系统内部是如何交流的。

## 5。演员信息

为了相互交互，参与者可以发送和接收来自系统中任何其他参与者的消息。这些**消息可以是任何类型的对象，条件是它是不可变的**。

最佳实践是在 actor 类中定义消息。这有助于编写易于理解的代码，并知道参与者可以处理什么消息。

### 5.1。发送消息

在 Akka actor 系统内部，使用以下方法发送消息:

*   `tell()`
*   `ask()`
*   `forward()`

**当我们想发送一条消息，并且不期望得到响应时，我们可以使用`tell()`方法。**从性能角度来看，这是最有效的方法:

```java
readingActorRef.tell(new ReadingActor.ReadLines(), ActorRef.noSender()); 
```

第一个参数表示我们发送给角色地址`readingActorRef`的消息。

第二个参数指定发件人是谁。当接收消息的参与者需要向发送者以外的参与者发送响应时(例如，发送参与者的父参与者)，这是非常有用的。

通常，我们可以将第二个参数设置为`null`或`ActorRef.noSender()`，因为我们不期待回复。**当我们需要演员的回应时，我们可以使用`ask()`方法:**

```java
CompletableFuture<Object> future = ask(wordCounterActorRef, 
  new WordCounterActor.CountWords(line), 1000).toCompletableFuture();
```

当请求参与者的响应时，会返回一个`CompletionStage` 对象，因此处理保持非阻塞状态。

我们必须注意的一个非常重要的事实是错误处理内部将要响应的行为者。**要返回一个包含异常的`Future`对象，我们必须向发送方参与者发送一条`Status.Failure`消息。**

当参与者在处理消息时抛出异常，并且`ask()`调用将超时，并且在日志中将看不到对异常的引用时，这不会自动完成:

```java
@Override
public Receive createReceive() {
    return receiveBuilder()
      .match(CountWords.class, r -> {
          try {
              int numberOfWords = countWordsFromLine(r.line);
              getSender().tell(numberOfWords, getSelf());
          } catch (Exception ex) {
              getSender().tell(
               new akka.actor.Status.Failure(ex), getSelf());
               throw ex;
          }
    }).build();
}
```

我们还有类似于`tell()`的`forward()`方法。不同的是，发送消息时保留了消息的原始发送者，所以转发消息的行动者只充当中间行动者:

```java
printerActorRef.forward(
  new PrinterActor.PrintFinalResult(totalNumberOfWords), getContext());
```

### 5.2。接收消息

**每个参与者将实现`createReceive()`方法**，该方法处理所有传入的消息。`receiveBuilder()`的作用类似于 switch 语句，试图将接收到的消息与定义的消息类型相匹配:

```java
public Receive createReceive() {
    return receiveBuilder().matchEquals("printit", p -> {
        System.out.println("The address of this actor is: " + getSelf());
    }).build();
}
```

**当收到消息时，消息被放入 FIFO 队列，因此消息被顺序处理**。

## 6。杀死一名演员

当我们使用完一个演员**时，我们可以通过从`ActorRefFactory`接口调用`stop()`方法**来停止它:

```java
system.stop(myActorRef);
```

我们可以使用这个方法来终止任何子 actor 或 actor 本身。重要的是要注意停止是异步完成的，并且在 actor 终止之前，**当前消息处理将完成**。**演员邮箱**不再接受新的消息。

通过**停止父演员**，**，我们也将向它所产生的所有子演员**发送一个终止信号。

当我们不再需要 actor 系统时，我们可以终止它以释放所有资源并防止任何内存泄漏:

```java
Future<Terminated> terminateResponse = system.terminate();
```

这将停止系统监护人演员，因此在这个 Akka 系统中定义的所有演员。

**我们还可以向任何我们想杀死的演员发送`PoisonPill`消息**:

```java
myActorRef.tell(PoisonPill.getInstance(), ActorRef.noSender());
```

与任何其他消息一样,`PoisonPill`消息将被参与者接收并放入队列。**演员将处理所有的消息，直到它到达`PoisonPill`一个**。只有这样，演员将开始终止过程。

另一个用于杀死演员的特殊消息是`Kill`消息。与`PoisonPill,`不同，actor 在处理这个消息时会抛出一个`ActorKilledException` :

```java
myActorRef.tell(Kill.getInstance(), ActorRef.noSender());
```

## 7。结论

在本文中，我们介绍了 Akka 框架的基础知识。我们展示了如何定义参与者、它们如何相互通信以及如何终止它们。

我们将总结一些与 Akka 合作的最佳实践:

*   当关注性能时，使用`tell()`而不是`ask()`
*   当使用`ask()`时，我们应该总是通过发送`Failure`消息来处理异常
*   参与者不应该共享任何可变状态
*   一个 actor 不应该在另一个 actor 中声明
*   **演员不再被引用时不会自动停止**。当我们不再需要某个角色时，我们必须显式销毁它，以防止内存泄漏
*   参与者使用的消息**应该总是不可变的**

和往常一样，这篇文章的源代码可以在 GitHub 的[上找到。](https://web.archive.org/web/20221023123703/https://github.com/eugenp/tutorials/tree/master/libraries-5)