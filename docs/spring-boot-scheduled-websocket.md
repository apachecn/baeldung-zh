# Spring Boot 的预定 WebSocket 推送

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-boot-scheduled-websocket>

## 1.概观

在本教程中，我们将看到如何使用 [WebSockets](/web/20220630134626/https://www.baeldung.com/java-websockets) 从服务器向浏览器发送预定消息。另一种方法是使用[服务器发送事件](/web/20220630134626/https://www.baeldung.com/spring-server-sent-events) (SSE)，但我们不会在本文中涉及。

Spring 提供了多种调度选项。首先，我们将讨论`[@Scheduled](/web/20220630134626/https://www.baeldung.com/spring-scheduling-annotations#scheduled) `注释。然后，我们将看到一个由 Project Reactor 提供的使用`[Flux::interval](https://web.archive.org/web/20220630134626/https://projectreactor.io/docs/core/release/api/reactor/core/publisher/Flux.html#interval-java.time.Duration-)`方法的例子。这个库对于 [Webflux](/web/20220630134626/https://www.baeldung.com/spring-webflux) 应用程序来说是现成可用的，它可以作为一个独立的库在任何 Java 项目中使用。

此外，还存在更高级的机制，如 [Quartz 调度器](/web/20220630134626/https://www.baeldung.com/quartz)，但我们不会讨论它们。

## 2.一个简单的聊天应用

在[之前的文章](/web/20220630134626/https://www.baeldung.com/spring-websockets-sendtouser)中，我们使用 WebSockets 构建了一个聊天应用程序。让我们用一个新特性来扩展它:聊天机器人。这些机器人是服务器端组件，将预定消息推送到浏览器。

### 2.1.Maven 依赖性

让我们从在 Maven 中设置必要的依赖项开始。为了建立这个项目，我们的`pom.xml`应该有:

```
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-websocket</artifactId>
</dependency>
<dependency>
    <groupId>io.projectreactor</groupId>
    <artifactId>reactor-core</artifactId>
</dependency>
<dependency>
    <groupId>com.github.javafaker</groupId>
    <artifactId>javafaker</artifactId>
    <version>1.0.2</version>
</dependency>
<dependency>
    <groupId>com.google.code.gson</groupId>
    <artifactId>gson</artifactId>
</dependency>
```

### 2.2.JavaFaker 依赖项

我们将使用 [JavaFaker](/web/20220630134626/https://www.baeldung.com/java-faker) 库来生成我们的机器人消息。这个库通常用于生成测试数据。在这里，我们将添加一个名为“`Chuck Norris`”的客人到我们的聊天室。

让我们看看代码:

```
Faker faker = new Faker();
ChuckNorris chuckNorris = faker.chuckNorris();
String messageFromChuck = chuckNorris.fact();
```

Faker 将为各种数据生成器提供工厂方法。我们将使用 [`ChuckNorris`](https://web.archive.org/web/20220630134626/https://dius.github.io/java-faker/apidocs/com/github/javafaker/ChuckNorris.html) 发电机。调用`chuckNorris.fact()`将显示预定义消息列表中的一个随机句子。

### 2.3.数据模型

聊天应用程序使用一个简单的 POJO 作为消息包装器:

```
public class OutputMessage {

    private String from;
    private String text;
    private String time;

   // standard constructors, getters/setters, equals and hashcode
}
```

综上所述，以下是我们如何创建聊天信息的示例:

```
OutputMessage message = new OutputMessage(
  "Chatbot 1", "Hello there!", new SimpleDateFormat("HH:mm").format(new Date())));
```

### 2.4.客户端

我们的聊天客户端是一个简单的 HTML 页面。它使用一个 [SockJS 客户端](https://web.archive.org/web/20220630134626/https://github.com/sockjs/sockjs-client)和 [STOMP](https://web.archive.org/web/20220630134626/https://stomp.github.io/) 消息协议。

让我们看看客户是如何订阅主题的:

```
<html>
<head>
    <script src="./js/sockjs-0.3.4.js"></script>
    <script src="./js/stomp.js"></script>
    <script type="text/javascript">
        // ...
        stompClient = Stomp.over(socket);

        stompClient.connect({}, function(frame) {
            // ...
            stompClient.subscribe('/topic/pushmessages', function(messageOutput) {
                showMessageOutput(JSON.parse(messageOutput.body));
            });
        });
        // ...
    </script>
</head>
<!-- ... -->
</html>
```

首先，我们通过 SockJS 协议创建了一个 Stomp 客户端。然后，主题订阅充当服务器和连接的客户机之间的通信通道。

在我们的存储库中，这段代码在`webapp/bots.html`中。我们在本地运行时在[http://localhost:8080/bots . html](https://web.archive.org/web/20220630134626/http://localhost:8080/bots.html)访问它。当然，我们需要根据我们如何部署应用程序来调整主机和端口。

### 2.5.服务器端

在之前的文章中，我们已经看到了如何在 Spring 中配置 [WebSockets。让我们稍微修改一下配置:](/web/20220630134626/https://www.baeldung.com/websockets-spring)

```
@Configuration
@EnableWebSocketMessageBroker
public class WebSocketConfig implements WebSocketMessageBrokerConfigurer {

    @Override
    public void configureMessageBroker(MessageBrokerRegistry config) {
        config.enableSimpleBroker("/topic");
        config.setApplicationDestinationPrefixes("/app");
    }

    @Override
    public void registerStompEndpoints(StompEndpointRegistry registry) {
        // ...
        registry.addEndpoint("/chatwithbots");
        registry.addEndpoint("/chatwithbots").withSockJS();
    }
}
```

**为了推送我们的消息，我们使用了实用程序类 [`SimpMessagingTemplate`](https://web.archive.org/web/20220630134626/https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/messaging/simp/SimpMessagingTemplate.html)** 。默认情况下，它在 Spring 上下文中作为`@Bean`可用。当`[AbstractMessageBrokerConfiguration](https://web.archive.org/web/20220630134626/https://github.com/spring-projects/spring-framework/blob/5b910a87c38386e870eba1f3f8154db2de4df026/spring-messaging/src/main/java/org/springframework/messaging/simp/config/AbstractMessageBrokerConfiguration.java#L392)` 在类路径中时，我们可以看到它是如何通过自动配置来声明的。因此，我们可以将其注入到任何弹簧组件中。

接下来，我们使用它发布消息到主题`/topic/pushmessages`。我们假设我们的类在名为`simpMessagingTemplate`的变量中注入了 bean:

```
simpMessagingTemplate.convertAndSend("/topic/pushmessages", 
  new OutputMessage("Chuck Norris", faker.chuckNorris().fact(), time));
```

如前面的客户端示例所示，客户端订阅该主题，以便在消息到达时对其进行处理。

## 3.安排推送消息

在 Spring 生态系统中，我们可以选择多种调度方法。如果我们使用 Spring MVC，`@Scheduled`注释因其简单性而成为自然的选择。如果我们使用 Spring Webflux，我们也可以使用 Project Reactor 的`Flux::interval`方法。我们将看到每种方法的一个例子。

### 3.1.配置

我们的聊天机器人将使用 JavaFaker 的 Chuck Norris 发电机。我们将把它配置成一个 bean，这样我们就可以在需要的地方注入它。

```
@Configuration
class AppConfig {

    @Bean
    public ChuckNorris chuckNorris() {
        return (new Faker()).chuckNorris();
    }
}
```

### 3.2.使用`@Scheduled`

我们的示例机器人是预定的方法。当它们运行时，它们使用`SimpMessagingTemplate`通过 WebSocket 发送我们的`OutputMessage`POJO。

顾名思义，**[`@Scheduled`](/web/20220630134626/https://www.baeldung.com/spring-scheduling-annotations#scheduled)注释允许方法**的重复执行。有了它，我们可以使用简单的基于速率的调度或更复杂的“cron”表达式。

让我们编写我们的第一个聊天机器人:

```
@Service
public class ScheduledPushMessages {

    @Scheduled(fixedRate = 5000)
    public void sendMessage(SimpMessagingTemplate simpMessagingTemplate, ChuckNorris chuckNorris) {
        String time = new SimpleDateFormat("HH:mm").format(new Date());
        simpMessagingTemplate.convertAndSend("/topic/pushmessages", 
          new OutputMessage("Chuck Norris (@Scheduled)", chuckNorris().fact(), time));
    }

}
```

我们用`@Scheduled(fixedRate = 5000).` 来注释`sendMessage`方法，这使得`sendMessage`每五秒运行一次。然后，我们使用`simpMessagingTemplate`实例向主题发送一个`OutputMessage`。`simpMessagingTemplate `和`chuckNorris`实例作为方法参数从 Spring 上下文注入。

### 3.3.使用`Flux::interval()`

**如果我们使用 WebFlux，我们可以使用`[Flux::interval](https://web.archive.org/web/20220630134626/https://projectreactor.io/docs/core/release/api/reactor/core/publisher/Flux.html#interval-java.time.Duration-)`操作符。**它将发布由选定的 [`D` `uration`](/web/20220630134626/https://www.baeldung.com/java-period-duration#duration-class) 分隔的`Long`项的无限流。

现在，让我们用上一个例子中的 Flux。目标是每五秒钟从`Chuck Norris`发送一次报价。首先，我们需要实现`InitializingBean`接口，以便在[应用程序启动](/web/20220630134626/https://www.baeldung.com/running-setup-logic-on-startup-in-spring)时订阅`Flux`:

```
@Service
public class ReactiveScheduledPushMessages implements InitializingBean {

    private SimpMessagingTemplate simpMessagingTemplate;

    private ChuckNorris chuckNorris;

    @Autowired
    public ReactiveScheduledPushMessages(SimpMessagingTemplate simpMessagingTemplate, ChuckNorris chuckNorris) {
        this.simpMessagingTemplate = simpMessagingTemplate;
        this.chuckNorris = chuckNorris;
    }

    @Override
    public void afterPropertiesSet() throws Exception {
        Flux.interval(Duration.ofSeconds(5L))
            // discard the incoming Long, replace it by an OutputMessage
            .map((n) -> new OutputMessage("Chuck Norris (Flux::interval)", 
                              chuckNorris.fact(), 
                              new SimpleDateFormat("HH:mm").format(new Date()))) 
            .subscribe(message -> simpMessagingTemplate.convertAndSend("/topic/pushmessages", message));
    }
}
```

这里，我们使用构造函数注入来设置`simpMessagingTemplate `和`chuckNorris`实例。这一次，调度逻辑在`afterPropertiesSet(),` 中，我们在实现`InitializingBean`时会覆盖它。该方法将在服务启动后立即运行。

[`interval`](https://web.archive.org/web/20220630134626/https://projectreactor.io/docs/core/release/api/reactor/core/publisher/Flux.html#interval-java.time.Duration-) 操作器每五秒发出一次`Long`。然后， [`map`](https://web.archive.org/web/20220630134626/https://projectreactor.io/docs/core/release/api/reactor/core/publisher/Flux.html#map-java.util.function.Function-) 操作符丢弃该值并用我们的消息替换它。最后，我们用 [`subscribe`](https://web.archive.org/web/20220630134626/https://projectreactor.io/docs/core/release/api/reactor/core/publisher/Flux.html#subscribe-java.util.function.Consumer-) 到`Flux`来触发我们对每条消息的逻辑。

## 4.结论

在本教程中，我们已经看到实用程序类`SimpMessagingTemplate` 使得通过 WebSocket 推送服务器消息变得很容易。此外，我们已经看到了调度一段代码执行的两种方式。

与往常一样，GitHub 上的[提供了示例的源代码。](https://web.archive.org/web/20220630134626/https://github.com/eugenp/tutorials/tree/master/spring-websockets)