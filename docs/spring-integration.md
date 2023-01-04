# Spring 集成简介

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-integration>

## 1。简介

本文将主要通过小的实际例子来介绍 Spring Integration 的核心概念。

Spring Integration 提供了许多强大的组件，可以极大地增强企业架构中系统和流程的互连性。

它包含了一些最好的和最流行的设计模式，帮助开发人员避免使用他们自己的模式。

我们将看看这个库在企业应用程序中满足的特定需求，以及为什么它比它的一些替代方案更可取。我们还将研究一些可用的工具，以进一步简化基于 Spring Integration 的应用程序的开发。

## 2。设置

```
<dependency>
    <groupId>org.springframework.integration</groupId>
    <artifactId>spring-integration-core</artifactId>
    <version>4.3.5.RELEASE</version>
</dependency>
<dependency>
    <groupId>org.springframework.integration</groupId>
    <artifactId>spring-integration-file</artifactId>
    <version>4.3.5.RELEASE</version>
</dependency> 
```

你可以从 Maven Central 下载最新版本的 [Spring Integration Core](https://web.archive.org/web/20220122142401/https://search.maven.org/classic/#search%7Cga%7C1%7Ca%3A%22spring-integration-core%22) 和[Spring Integration File Support](https://web.archive.org/web/20220122142401/https://search.maven.org/classic/#search%7Cga%7C1%7Cspring-integration-file)。

## 3。消息模式

这个库中的一个基本模式是消息传递。该模式以消息为中心——通过预定义的通道从原始系统或流程转移到一个或多个系统或流程的离散数据有效负载。

从历史上看，这种模式是集成多个不同系统的最灵活的方式，它可以:

*   几乎完全解耦集成中涉及的系统
*   允许集成中的参与系统完全不知道彼此的底层协议、格式或其他实现细节
*   鼓励集成中涉及的组件的开发和重用

## 4。消息传递整合在行动****

让我们考虑一个基本的例子,它将一个 MPEG 视频文件从一个指定的文件夹复制到另一个配置好的文件夹:

```
@Configuration
@EnableIntegration
public class BasicIntegrationConfig{
    public String INPUT_DIR = "the_source_dir";
    public String OUTPUT_DIR = "the_dest_dir";
    public String FILE_PATTERN = "*.mpeg";

    @Bean
    public MessageChannel fileChannel() {
        return new DirectChannel();
    }

    @Bean
    @InboundChannelAdapter(value = "fileChannel", poller = @Poller(fixedDelay = "1000"))
    public MessageSource<File> fileReadingMessageSource() {
        FileReadingMessageSource sourceReader= new FileReadingMessageSource();
        sourceReader.setDirectory(new File(INPUT_DIR));
        sourceReader.setFilter(new SimplePatternFileListFilter(FILE_PATTERN));
        return sourceReader;
    }

    @Bean
    @ServiceActivator(inputChannel= "fileChannel")
    public MessageHandler fileWritingMessageHandler() {
        FileWritingMessageHandler handler = new FileWritingMessageHandler(new File(OUTPUT_DIR));
        handler.setFileExistsMode(FileExistsMode.REPLACE);
        handler.setExpectReply(false);
        return handler;
    }
}
```

上面的代码配置了一个服务激活器、一个集成通道和一个入站通道适配器。

我们将很快更详细地检查这些组件类型。`@EnableIntegration`注释将这个类指定为 Spring 集成配置。

让我们开始我们的 Spring 集成应用程序上下文:

```
public static void main(String... args) {
    AbstractApplicationContext context 
      = new AnnotationConfigApplicationContext(BasicIntegrationConfig.class);
    context.registerShutdownHook();

    Scanner scanner = new Scanner(System.in);
    System.out.print("Please enter q and press <enter> to exit the program: ");

    while (true) {
       String input = scanner.nextLine();
       if("q".equals(input.trim())) {
          break;
      }
    }
    System.exit(0);
}
```

上面的主要方法启动集成上下文；它还接受从命令行输入的“`q`”字符来退出程序。让我们更详细地检查组件。

## 5。弹簧集成组件****

### 5.1。消息

`org.springframework.integration.Message` 接口定义了 spring 消息:Spring 集成上下文中的数据传输单元。

```
public interface Message<T> {
    T getPayload();
    MessageHeaders getHeaders();
}
```

它定义了两个关键元素的访问器:

*   消息头，本质上是一个键值容器，可用于传输元数据，如在`org.springframework.integration.MessageHeaders` 类中定义的
*   消息有效负载，即有价值的实际传输数据——在我们的用例中，视频文件就是有效负载

### 5.2。频道

Spring Integration(实际上是 EAI)中的通道是集成架构中的基本管道。它是信息从一个系统传递到另一个系统的管道。

您可以将它视为一个文字管道，集成的系统或流程可以通过它向其他系统发送消息(或从其他系统接收消息)。

Spring Integration 中的通道有多种风格，这取决于您的需要。它们很大程度上是可配置的和现成可用的，不需要任何定制代码，但是如果您有定制需求，有一个健壮的框架可用。

**点对点(P2P)** 通道用于在系统或组件之间建立一对一的通信线路。一个组件将消息发布到通道，以便另一个组件可以获取它。通道的每一端只能有一个组件。

正如我们已经看到的，配置通道就像返回一个`DirectChannel`的实例一样简单:

```
@Bean
public MessageChannel fileChannel1() {
    return new DirectChannel();
}

@Bean
public MessageChannel fileChannel2() {
    return new DirectChannel();
}

@Bean
public MessageChannel fileChannel3() {
    return new DirectChannel();
}
```

这里，我们定义了三个独立的通道，它们都由各自的 getter 方法的名称来标识。

**发布-订阅(Pub-Sub)** 通道用于在系统或组件之间建立一对多的通信线路。这将允许我们发布到我们之前创建的所有 3 个直接渠道。

因此，按照我们的示例，我们可以用发布-订阅通道来替换 P2P 通道:

```
@Bean
public MessageChannel pubSubFileChannel() {
    return new PublishSubscribeChannel();
}

@Bean
@InboundChannelAdapter(value = "pubSubFileChannel", poller = @Poller(fixedDelay = "1000"))
public MessageSource<File> fileReadingMessageSource() {
    FileReadingMessageSource sourceReader = new FileReadingMessageSource();
    sourceReader.setDirectory(new File(INPUT_DIR));
    sourceReader.setFilter(new SimplePatternFileListFilter(FILE_PATTERN));
    return sourceReader;
} 
```

我们现在已经将入站通道适配器转换为发布到发布订阅通道。这将允许我们将从源文件夹中读取的文件发送到多个目的地。

### 5.3。桥

Spring Integration 中的桥用于连接两个消息通道或适配器，如果由于任何原因它们不能直接连接的话。

在我们的例子中，我们可以使用一个桥将我们的发布-订阅通道连接到三个不同的 P2P 通道(因为 P2P 和发布-订阅通道不能直接连接):

```
@Bean
@BridgeFrom(value = "pubSubFileChannel")
public MessageChannel fileChannel1() {
    return new DirectChannel();
}

@Bean
@BridgeFrom(value = "pubSubFileChannel")
public MessageChannel fileChannel2() {
    return new DirectChannel();
}

@Bean
@BridgeFrom(value = "pubSubFileChannel")
public MessageChannel fileChannel3() {
    return new DirectChannel();
}
```

上面的 bean 配置现在将`pubSubFileChannel`连接到三个 P2P 通道。`@BridgeFrom`注释定义了一个桥，可以应用于任何数量的需要订阅发布-订阅通道的通道。

我们可以将上面的代码理解为“创建一个从`pubSubFileChannel`到`fileChannel1, fileChannel2, and` `fileChannel3` 的桥梁，这样来自`pubSubFileChannel`的消息就可以同时传送到所有三个通道。”

### 5.4。服务激活器

服务激活器是定义给定方法上的`@ServiceActivator` 注释的任何 POJO。这允许我们在收到来自入站通道的消息时在 POJO 上执行任何方法，并允许我们将消息写入出站通道。

在我们的例子中，我们的服务激活器从已配置的`input channel` 接收一个文件，并将其写入已配置的文件夹。

### 5.5。适配器

适配器是一个基于企业集成模式的组件，它允许用户“插入”系统或数据源。我们知道，它几乎就是一个插在墙上插座或电子设备上的适配器。

它允许可重用的连接到其他“黑盒”系统，如数据库、FTP 服务器和消息系统，如 JMS、AMQP 和 Twitter 等社交网络。连接到这些系统的需求无处不在，这意味着适配器是非常便携和可重用的(事实上，有一个很小的适配器目录，免费提供，任何人都可以使用)。

**适配器分为两大类——入站和出站。**

让我们在示例场景中使用的适配器的上下文中检查这些类别:

如我们所见，入站适配器用于从外部系统(在本例中是文件系统目录)引入消息。

我们的入站适配器配置包括:

*   一个将 bean 配置标记为适配器的`@InboundChannelAdapter`注释——我们配置适配器将向其提供消息的通道(在我们的例子中，是一个 MPEG 文件)和一个`poller`,一个帮助适配器以指定的时间间隔轮询配置的文件夹的组件
*   一个标准的 Spring java 配置类，它返回一个处理文件系统轮询的 Spring Integration 类实现

**出站适配器**用于向外发送消息。Spring Integration 支持各种常见用例的各种现成适配器。

## 6。结论

我们研究了一个 Spring 集成的基本用例，它展示了库的基于 java 的配置和可用组件的可重用性。

Spring Integration 代码可以作为 JavaSE 中的一个独立项目进行部署，也可以作为 Jakarta EE 环境中更大项目的一部分。虽然它不直接与其他以 EAI 为中心的产品和模式(如企业服务总线(ESB ))竞争，但它是一个可行的轻量级替代方案，可以解决 ESB 要解决的许多问题。

你可以在 Github 项目中找到这篇文章[的源代码。](https://web.archive.org/web/20220122142401/https://github.com/eugenp/tutorials/tree/master/spring-integration)