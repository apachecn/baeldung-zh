# Spring 集成 Java DSL

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-integration-java-dsl>

## 1。简介

在本教程中，我们将学习用于创建应用程序集成的 Spring Integration Java DSL。

我们将采用我们在[Spring Integration 简介](/web/20220525134642/https://www.baeldung.com/spring-integration)中构建的文件移动集成，而使用 DSL。

## 2。依赖性

Spring Integration Java DSL 是 [Spring Integration Core](https://web.archive.org/web/20220525134642/https://search.maven.org/classic/#search%7Cgav%7C1%7Cg%3A%22org.springframework.integration%22%20AND%20a%3A%22spring-integration-core%22) 的一部分。

因此，我们可以添加依赖关系:

```java
<dependency>
    <groupId>org.springframework.integration</groupId>
    <artifactId>spring-integration-core</artifactId>
    <version>5.0.6.RELEASE</version>
</dependency>
```

为了运行我们的文件移动应用程序，我们还需要 [Spring 集成文件](https://web.archive.org/web/20220525134642/https://search.maven.org/classic/#search%7Cgav%7C1%7Ca%3A%22spring-integration-file%22):

```java
<dependency>
    <groupId>org.springframework.integration</groupId>
    <artifactId>spring-integration-file</artifactId>
    <version>5.0.6.RELEASE</version>
</dependency>
```

## 3。Spring Integration Java DSL

在 Java DSL 出现之前，用户会用 XML 配置 Spring Integration 组件。

DSL 引入了一些流畅的构建器，从这些构建器中我们可以很容易地创建一个完全用 Java 编写的完整的 Spring 集成管道。

因此，假设我们想要创建一个通道来处理通过管道传输的任何数据。

在过去，我们可能会这样做:

```java
<int:channel id="input"/>

<int:transformer input-channel="input" expression="payload.toUpperCase()" />
```

现在我们可以做的是:

```java
@Bean
public IntegrationFlow upcaseFlow() {
    return IntegrationFlows.from("input")
      .transform(String::toUpperCase)
      .get();
}
```

## 4。文件移动应用

为了开始我们的文件移动集成，我们需要一些简单的构建块。

### 4.1。整合流程

我们需要的第一个构建块是集成流程，我们可以从`IntegrationFlows `构建器中获得:

```java
IntegrationFlows.from(...)
```

可以有几种类型，但在本教程中，我们只看三种:

*   `MessageSource`年代
*   `MessageChannel`年代，以及
*   `String`年代

我们很快就会谈到这三个问题。

在我们调用了`from`之后，现在我们可以使用一些定制方法:

```java
IntegrationFlow flow = IntegrationFlows.from(sourceDirectory())
  .filter(onlyJpgs())
  .handle(targetDirectory())
  // add more components
  .get();
```

最终，`IntegrationFlows`将总是产生一个`IntegrationFlow,` **的实例，这是任何 Spring 集成应用程序的最终产品。**

这种接受输入、执行适当的转换并输出结果的模式是所有 Spring 集成应用程序的基础。

### 4.2。描述输入源

首先，为了移动文件，我们需要向我们的集成流程指出应该在哪里寻找它们，为此，我们需要一个`MessageSource:`

```java
@Bean
public MessageSource<File> sourceDirectory() {
  // .. create a message source
}
```

简单地说，`MessageSource`是一个来自应用程序外部的[消息的地方。](https://web.archive.org/web/20220525134642/http://joshlong.com/jl/blogPost/spring_integration_adapters_gateways_and_channels.html)

更具体地说，我们需要能够将外部资源`adapt`到 Spring 消息表示中的东西。由于这个`adaptation` 集中在`input`，这些通常被称为`Input Channel Adapters.`

`spring-integration-file `依赖关系为我们提供了一个输入通道适配器，非常适合我们的用例:`FileReadingMessageSource:`

```java
@Bean
public MessageSource<File> sourceDirectory() {
    FileReadingMessageSource messageSource = new FileReadingMessageSource();
    messageSource.setDirectory(new File(INPUT_DIR));
    return messageSource;
}
```

在这里，我们的`FileReadingMessageSource `将读取一个由`INPUT_DIR`给出的目录，并从中创建一个`MessageSource`。

让我们在一个`IntegrationFlows.from `调用中将它指定为我们的源:

```java
IntegrationFlows.from(sourceDirectory());
```

### 4.3。配置输入源

现在，如果我们把它看作一个长期的应用程序，**我们可能希望能够在文件进入**时注意到它们，而不仅仅是在启动时移动已经存在的文件。

为了方便起见，`from`还可以将额外的`configurers`作为进一步定制的输入源:

```java
IntegrationFlows.from(sourceDirectory(), configurer -> configurer.poller(Pollers.fixedDelay(10000)));
```

在这种情况下，我们可以通过告诉 Spring Integration 每 10 秒轮询一次输入源——本例中是我们的文件系统——来提高输入源的弹性。

当然，这不仅仅适用于我们的文件输入源，我们可以将这个轮询器添加到任何`MessageSource`中。

### 4.4。过滤来自输入源的消息

接下来，假设我们希望文件移动应用程序只移动特定的文件，比如扩展名为`jpg` 的图像文件。

为此，我们可以使用`GenericSelector`:

```java
@Bean
public GenericSelector<File> onlyJpgs() {
    return new GenericSelector<File>() {

        @Override
        public boolean accept(File source) {
          return source.getName().endsWith(".jpg");
        }
    };
}
```

因此，让我们再次更新我们的集成流程:

```java
IntegrationFlows.from(sourceDirectory())
  .filter(onlyJpgs());
```

或者，**因为这个过滤器非常简单，我们可以用 lambda** 来定义它:

```java
IntegrationFlows.from(sourceDirectory())
  .filter(source -> ((File) source).getName().endsWith(".jpg"));
```

### 4.5。用服务激活器处理消息

现在我们有了一个经过筛选的文件列表，我们需要将它们写入一个新位置。

`Service Activator` `s `是我们在考虑 Spring Integration 中的输出时会用到的。

让我们使用来自`spring-integration-file`的`FileWritingMessageHandler` 服务激活器:

```java
@Bean
public MessageHandler targetDirectory() {
    FileWritingMessageHandler handler = new FileWritingMessageHandler(new File(OUTPUT_DIR));
    handler.setFileExistsMode(FileExistsMode.REPLACE);
    handler.setExpectReply(false);
    return handler;
}
```

在这里，我们的`FileWritingMessageHandler `将把它收到的每个`Message`有效载荷写到`OUTPUT_DIR`。

再说一遍，让我们更新一下:

```java
IntegrationFlows.from(sourceDirectory())
  .filter(onlyJpgs())
  .handle(targetDirectory());
```

并且顺便注意一下`setExpectReply`的用法。**因为集成流可以是** **双向的，**这个调用表明这个特定的管道是单向的。

### 4.6。激活我们的集成流程

当我们添加完所有组件后，我们需要**将我们的`IntegrationFlow `注册为 bean** 来激活它:

```java
@Bean
public IntegrationFlow fileMover() {
    return IntegrationFlows.from(sourceDirectory(), c -> c.poller(Pollers.fixedDelay(10000)))
      .filter(onlyJpgs())
      .handle(targetDirectory())
      .get();
}
```

` get `方法提取一个我们需要注册为 Spring Bean 的`IntegrationFlow` 实例。

一旦我们的应用程序上下文加载，包含在我们的`IntegrationFlow`中的所有组件都被激活。

现在，我们的应用程序将开始将文件从源目录移动到目标目录。

## 5。附加组件

在基于 DSL 的文件移动应用程序中，我们创建了一个入站通道适配器、一个消息过滤器和一个服务激活器。

让我们看看其他一些常见的 Spring 集成组件，看看我们如何使用它们。

### 5.1。消息通道

如前所述，`Message Channel`是初始化流的另一种方式:

```java
IntegrationFlows.from("anyChannel")
```

我们可以将此理解为“请找到或创建一个名为`anyChannel`的通道 bean。然后，读取从其他流输入到`anyChannel`的任何数据。

但是，实际上它比那更通用。

简单来说，渠道将生产者从消费者中抽象出来，我们可以把它看作一个 Java `Queue`。**可以在**流程中的任何一点插入一个通道。

例如，我们希望在文件从一个目录移动到下一个目录时对其进行优先级排序:

```java
@Bean
public PriorityChannel alphabetically() {
    return new PriorityChannel(1000, (left, right) -> 
      ((File)left.getPayload()).getName().compareTo(
        ((File)right.getPayload()).getName()));
}
```

然后，我们可以在流程之间插入对`channel`的调用:

```java
@Bean
public IntegrationFlow fileMover() {
    return IntegrationFlows.from(sourceDirectory())
      .filter(onlyJpgs())
      .channel("alphabetically")
      .handle(targetDirectory())
      .get();
}
```

有几十个通道可供选择，**一些更方便的通道用于并发、审计或中间持久性**(想想 Kafka 或 JMS buffers)。

此外，当与`Bridge` s 结合使用时，渠道可能会非常强大。

### 5.2。桥

当我们想要将两个通道合并时，我们使用一个`Bridge.`

让我们想象一下，不是直接写入输出目录，而是让我们的文件移动应用程序写入另一个通道:

```java
@Bean
public IntegrationFlow fileReader() {
    return IntegrationFlows.from(sourceDirectory())
      .filter(onlyJpgs())
      .channel("holdingTank")
      .get();
}
```

现在，因为我们已经简单地将它写到一个通道中，**我们可以从那里桥接到其他流**。

让我们创建一个桥来轮询我们的存储槽中的消息，并将它们写入目的地:

```java
@Bean
public IntegrationFlow fileWriter() {
    return IntegrationFlows.from("holdingTank")
      .bridge(e -> e.poller(Pollers.fixedRate(1, TimeUnit.SECONDS, 20)))
      .handle(targetDirectory())
      .get();
}
```

同样，因为我们写入了一个中间通道，所以现在我们可以添加另一个流**，它获取这些相同的文件，并以不同的速率写入它们**:

```java
@Bean
public IntegrationFlow anotherFileWriter() {
    return IntegrationFlows.from("holdingTank")
      .bridge(e -> e.poller(Pollers.fixedRate(2, TimeUnit.SECONDS, 10)))
      .handle(anotherTargetDirectory())
      .get();
}
```

正如我们所看到的，各个桥可以控制不同处理程序的轮询配置。

一旦加载了应用程序上下文，我们现在就有了一个更复杂的应用程序，它将开始把文件从源目录移动到两个目标目录。

## 6。结论

在本文中，我们看到了使用 Spring Integration Java DSL 构建不同集成管道的各种方法。

本质上，我们能够从以前的教程中重新创建文件移动应用程序，这一次使用纯 java。

此外，我们还看了一些其他组件，如通道和桥。

本教程中使用的完整源代码可以从 Github 上的[处获得。](https://web.archive.org/web/20220525134642/https://github.com/eugenp/tutorials/tree/master/spring-integration)