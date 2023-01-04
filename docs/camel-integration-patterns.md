# 与 Apache Camel 的集成模式

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/camel-integration-patterns>

## 1。概述

本文将介绍 Apache Camel 支持的一些基本的企业集成模式(EIP)。集成模式通过为集成系统的标准化方式提供解决方案来提供帮助。

如果你需要先复习一下 Apache Camel 的基础知识，一定要访问[这篇文章](/web/20220124003034/https://www.baeldung.com/apache-camel-intro)来温习基础知识。

## 2。关于 EIPs

企业集成模式是旨在为集成挑战提供解决方案的设计模式。Camel 为许多这些模式提供了实现。要查看支持模式的完整列表，请访问[此链接](https://web.archive.org/web/20220124003034/https://camel.apache.org/enterprise-integration-patterns.html)。

在本文中，我们将讨论基于内容的路由器、消息转换器、多播、分离器和死信通道集成模式。

## 2。基于内容的路由器

基于内容的路由器是一种消息路由器，它根据消息头、部分有效负载或消息交换中我们认为是内容的任何内容将消息路由到其目的地。

它以`choice()` DSL 语句开始，后跟一个或多个`when()` DSL 语句。每个`when()`包含一个谓词表达式，如果满足该表达式，将导致所包含的处理步骤的执行。

让我们通过定义一个路由来说明这种 EIP，该路由消耗一个文件夹中的文件，并根据文件扩展名将它们移动到两个不同的文件夹中。我们的路线在 Spring XML 文件中引用，使用 Camel 的自定义 XML 语法:

```java
<bean id="contentBasedFileRouter" 
  class="com.baeldung.camel.file.ContentBasedFileRouter" />

<camelContext >
    <routeBuilder ref="contentBasedFileRouter" />
</camelContext>
```

路由定义包含在`ContentBasedFileRouter`类中，其中文件根据其扩展名从源文件夹路由到两个不同的目标文件夹。

或者，我们可以在这里使用 Spring Java config 方法，而不是使用 Spring XML 文件。为此，我们需要向我们的项目添加一个额外的依赖项:

```java
<dependency>
    <groupId>org.apache.camel</groupId>
    <artifactId>camel-spring-javaconfig</artifactId>
    <version>2.18.1</version>
</dependency>
```

最新版本的神器可以在[这里](https://web.archive.org/web/20220124003034/https://search.maven.org/classic/#search%7Cga%7C1%7Ca%3A%22camel-spring-javaconfig%22)找到。

之后，我们需要扩展`CamelConfiguration`类并覆盖引用`ContentBasedFileRouter`的`routes()`方法:

```java
@Configuration
public class ContentBasedFileRouterConfig extends CamelConfiguration {

    @Bean
    ContentBasedFileRouter getContentBasedFileRouter() {
        return new ContentBasedFileRouter();
    }

    @Override
    public List<RouteBuilder> routes() {
        return Arrays.asList(getContentBasedFileRouter());
    }
}
```

使用[简单表达式语言](https://web.archive.org/web/20220124003034/https://camel.apache.org/simple.html)通过`simple` () DSL 语句对扩展进行评估，DSL 语句旨在用于评估表达式和谓词:

```java
public class ContentBasedFileRouter extends RouteBuilder {

    private static final String SOURCE_FOLDER 
      = "src/test/source-folder";
    private static final String DESTINATION_FOLDER_TXT 
      = "src/test/destination-folder-txt";
    private static final String DESTINATION_FOLDER_OTHER 
      = "src/test/destination-folder-other";

    @Override
    public void configure() throws Exception {
        from("file://" + SOURCE_FOLDER + "?delete=true").choice()
          .when(simple("${file:ext} == 'txt'"))
          .to("file://" + DESTINATION_FOLDER_TXT).otherwise()
          .to("file://" + DESTINATION_FOLDER_OTHER);
    }
}
```

这里我们额外使用了`otherwise()` DSL 语句，以便路由所有不满足用`when()`语句给出的谓词的消息。

## 3。消息翻译器

由于每个系统都使用自己的数据格式，因此经常需要将来自另一个系统的消息转换成目标系统支持的数据格式。

Camel 支持`MessageTranslator`路由器，它允许我们使用路由逻辑中的定制处理器、使用特定的 bean 来执行转换或者使用`transform()` DSL 语句来转换消息。

使用定制处理器的例子可以在[上一篇文章](/web/20220124003034/https://www.baeldung.com/apache-camel-intro)中找到，其中我们定义了一个处理器，它为每个输入文件的文件名添加一个时间戳。

现在让我们通过使用`transform()`语句来演示如何使用消息转换器:

```java
public class MessageTranslatorFileRouter extends RouteBuilder {
    private static final String SOURCE_FOLDER 
      = "src/test/source-folder";
    private static final String DESTINATION_FOLDER 
      = "src/test/destination-folder";

    @Override
    public void configure() throws Exception {
        from("file://" + SOURCE_FOLDER + "?delete=true")
          .transform(body().append(header(Exchange.FILE_NAME)))
          .to("file://" + DESTINATION_FOLDER);
    }
}
```

在本例中，我们通过`transform()`语句为源文件夹中的每个文件将文件名附加到文件内容，并将转换后的文件移动到目标文件夹。

## 4。多播

多播允许我们**将相同的消息路由到一组不同的端点，并以不同的方式处理它们**。

这可以通过使用`multicast()` DSL 语句，然后列出端点和其中的处理步骤来实现。

默认情况下，不同端点上的处理不是并行完成的，但是这可以通过使用`parallelProcessing()` DSL 语句来改变。

默认情况下，多点传送后，Camel 将使用最后一次回复作为传出消息。然而，可以定义不同的聚合策略用于组合来自多播的回复。

让我们通过一个例子来看看多播 EIP 是什么样子的。我们将把文件从源文件夹多播到两条不同的路径上，在那里我们将转换它们的内容，并把它们发送到不同的目标文件夹。这里我们使用 [`direct:` 组件](https://web.archive.org/web/20220124003034/https://camel.apache.org/direct.html)，它允许我们将两条路由链接在一起:

```java
public class MulticastFileRouter extends RouteBuilder {
    private static final String SOURCE_FOLDER 
      = "src/test/source-folder";
    private static final String DESTINATION_FOLDER_WORLD 
      = "src/test/destination-folder-world";
    private static final String DESTINATION_FOLDER_HELLO 
      = "src/test/destination-folder-hello";

    @Override
    public void configure() throws Exception {
        from("file://" + SOURCE_FOLDER + "?delete=true")
          .multicast()
          .to("direct:append", "direct:prepend").end();

        from("direct:append")
          .transform(body().append("World"))
          .to("file://" + DESTINATION_FOLDER_WORLD);

        from("direct:prepend")
           .transform(body().prepend("Hello"))
           .to("file://" + DESTINATION_FOLDER_HELLO);
    }
}
```

## 5。分割器

分割器允许我们**将收到的消息分割成许多片段，并单独处理每一个片段。**这可以通过使用`split()` DSL 语句来实现。

与多播相反，拆分器将更改传入的消息，而多播将保持不变。

为了在一个例子中演示这一点，我们将定义一个路径，在该路径中，文件中的每一行被分割并转换成一个单独的文件，然后该文件被移动到不同的目标文件夹中。创建每个新文件时，文件名将与文件内容相同:

```java
public class SplitterFileRouter extends RouteBuilder {
    private static final String SOURCE_FOLDER 
      = "src/test/source-folder";
    private static final String DESTINATION_FOLDER  
      = "src/test/destination-folder";

    @Override
    public void configure() throws Exception {
        from("file://" + SOURCE_FOLDER + "?delete=true")
          .split(body().convertToString().tokenize("\n"))
          .setHeader(Exchange.FILE_NAME, body())
          .to("file://" + DESTINATION_FOLDER);
    }
}
```

## 6。死信通道

这是很常见的，应该预料到有时会发生问题，例如，数据库死锁会导致消息无法按预期传递。但是，在某些情况下，在一定的延迟后重试会有所帮助，并且会处理一条消息。

死信通道允许我们控制一旦消息未能送达后会发生什么。使用死信通道，我们可以指定是否将抛出的异常传播给调用者，以及将失败的交换路由到哪里。

当消息传递失败时，死信通道(如果使用)会将消息移动到死信端点。

让我们通过在路线上抛出异常的例子来演示这一点:

```java
public class DeadLetterChannelFileRouter extends RouteBuilder {
    private static final String SOURCE_FOLDER 
      = "src/test/source-folder";

    @Override
    public void configure() throws Exception {
        errorHandler(deadLetterChannel("log:dead?level=ERROR")
          .maximumRedeliveries(3).redeliveryDelay(1000)
          .retryAttemptedLogLevel(LoggingLevel.ERROR));

        from("file://" + SOURCE_FOLDER + "?delete=true")
          .process(exchange -> {
            throw new IllegalArgumentException("Exception thrown!");
        });
    }
}
```

这里我们定义了一个`errorHandler` ,它记录了失败的交付并定义了重新交付策略。通过设置`retryAttemptedLogLevel()`，每一次重新传递尝试都将以指定的日志级别记录。

为了让它完全发挥作用，我们还需要配置一个日志记录器。

运行此测试后，在控制台中可以看到以下日志语句:

```java
ERROR DeadLetterChannel:156 - Failed delivery for 
(MessageId: ID-ZAG0025-50922-1481340325657-0-1 on 
ExchangeId: ID-ZAG0025-50922-1481340325657-0-2). 
On delivery attempt: 0 caught: java.lang.IllegalArgumentException: 
Exception thrown!
ERROR DeadLetterChannel:156 - Failed delivery for 
(MessageId: ID-ZAG0025-50922-1481340325657-0-1 on 
ExchangeId: ID-ZAG0025-50922-1481340325657-0-2). 
On delivery attempt: 1 caught: java.lang.IllegalArgumentException: 
Exception thrown!
ERROR DeadLetterChannel:156 - Failed delivery for 
(MessageId: ID-ZAG0025-50922-1481340325657-0-1 on 
ExchangeId: ID-ZAG0025-50922-1481340325657-0-2). 
On delivery attempt: 2 caught: java.lang.IllegalArgumentException: 
Exception thrown!
ERROR DeadLetterChannel:156 - Failed delivery for 
(MessageId: ID-ZAG0025-50922-1481340325657-0-1 on 
ExchangeId: ID-ZAG0025-50922-1481340325657-0-2). 
On delivery attempt: 3 caught: java.lang.IllegalArgumentException: 
Exception thrown!
ERROR dead:156 - Exchange[ExchangePattern: InOnly, 
BodyType: org.apache.camel.component.file.GenericFile, 
Body: [Body is file based: GenericFile[File.txt]]]
```

正如您所注意到的，每个重新传递尝试都被记录下来，显示传递不成功的 Exchange。

## 7。结论

在本文中，我们介绍了使用 Apache Camel 的集成模式，并用几个例子演示了它们。

我们展示了如何使用这些集成模式，以及为什么它们有利于解决集成挑战。

这篇文章的代码可以在 GitHub 上找到[。](https://web.archive.org/web/20220124003034/https://github.com/eugenp/tutorials/tree/master/spring-apache-camel)