# 反应式应用中的弹簧 AMQP

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-amqp-reactive>

## 1.概观

本教程展示了如何创建一个简单的 Spring Boot 反应式应用程序，该应用程序与 RabbitMQ 消息服务器集成，rabbit MQ 消息服务器是 AMQP 消息标准的一个流行实现。

我们涵盖了这两种场景——点对点和发布-订阅场景——使用了一个分布式设置，突出了这两种模式之间的差异。

请注意，我们假设您对 AMQP、RabbitMQ 和 Spring Boot 有基本的了解，特别是交换、队列、主题等关键概念。有关这些概念的更多信息可以在下面的链接中找到:

*   [与春天 AMQP 的信息传递](/web/20221126222632/https://www.baeldung.com/spring-amqp)
*   [rabbit MQ 简介](/web/20221126222632/https://www.baeldung.com/rabbitmq)

## 2.RabbitMQ 服务器设置

尽管我们可以在本地设置一个本地 RabbitMQ，但在实践中，我们更可能使用一个具有额外特性(如高可用性、监控、安全性等)的专用安装。

为了在我们的开发机器中模拟这样的环境，我们将使用 Docker 来创建我们的应用程序将使用的服务器。

以下命令将启动独立的 RabbitMQ 服务器:

```java
$ docker run -d --name rabbitmq -p 5672:5672 rabbitmq:3 
```

我们没有声明任何持久卷，因此未读邮件将在重新启动之间丢失。该服务将在主机的端口 5672 上可用。

我们可以用`docker logs`命令检查服务器日志，它应该会产生如下输出:

```java
$ docker logs rabbitmq
2018-06-09 13:42:29.718 [info] <0.33.0>
  Application lager started on node [[email protected]](/web/20221126222632/https://www.baeldung.com/cdn-cgi/l/email-protection)
// ... some lines omitted
2018-06-09 13:42:33.491 [info] <0.226.0>
 Starting RabbitMQ 3.7.5 on Erlang 20.3.5
 Copyright (C) 2007-2018 Pivotal Software, Inc.
 Licensed under the MPL.  See http://www.rabbitmq.com/

  ##  ##
  ##  ##      RabbitMQ 3.7.5\. Copyright (C) 2007-2018 Pivotal Software, Inc.
  ##########  Licensed under the MPL.  See http://www.rabbitmq.com/
  ######  ##
  ##########  Logs: <stdout>

              Starting broker...
2018-06-09 13:42:33.494 [info] <0.226.0>
 node           : [[email protected]](/web/20221126222632/https://www.baeldung.com/cdn-cgi/l/email-protection)
 home dir       : /var/lib/rabbitmq
 config file(s) : /etc/rabbitmq/rabbitmq.conf
 cookie hash    : CY9rzUYh03PK3k6DJie09g==
 log(s)         : <stdout>
 database dir   : /var/lib/rabbitmq/mnesia/[[email protected]](/web/20221126222632/https://www.baeldung.com/cdn-cgi/l/email-protection)

// ... more log lines
```

由于映像包含了`rabbitmqctl`实用程序，我们可以使用它在运行映像的上下文中执行管理任务。

例如，我们可以使用以下命令获取服务器状态信息:

```java
$ docker exec rabbitmq rabbitmqctl status
Status of node [[email protected]](/web/20221126222632/https://www.baeldung.com/cdn-cgi/l/email-protection) ...
[{pid,299},
 {running_applications,
     [{rabbit,"RabbitMQ","3.7.5"},
      {rabbit_common,
          "Modules shared by rabbitmq-server and rabbitmq-erlang-client",
          "3.7.5"},
// ... other info omitted for brevity 
```

其他有用的命令包括:

*   `list_exchanges`:列出所有已申报的交易所
*   `list_queues`:列出所有声明的队列，包括未读消息的数量
*   `list_bindings`:列出所有定义的交换机和队列之间的绑定，也包括路由关键字

## 3.春季 AMQP 项目设置

一旦我们的 RabbitMQ 服务器启动并运行，我们就可以继续创建我们的 Spring 项目了。这个示例项目将允许任何 REST 客户机向消息传递服务器发送和/或接收消息，使用 Spring AMQP 模块和相应的 Spring Boot 启动器与它通信。

我们需要添加到`pom.xml`项目文件中的主要依赖项是:

```java
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-amqp</artifactId>
    <version>2.0.3.RELEASE</version>
</dependency>
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-webflux</artifactId>
    <version>2.0.2.RELEASE</version> 
</dependency>
```

`spring-boot-starter-amqp`带来了所有与 AMQP 相关的东西，而`spring-boot-starter-webflux`是用于实现我们的反应式 REST 服务器的核心依赖。

注:你可以在 Maven Central 上查看 Spring Boot 启动器 [AMQP](https://web.archive.org/web/20221126222632/https://search.maven.org/classic/#search%7Cgav%7C1%7Cg%3A%22org.springframework.amqp%22%20AND%20a%3A%22spring-amqp%22) 和 [Webflux](https://web.archive.org/web/20221126222632/https://search.maven.org/classic/#search%7Cgav%7C1%7Cg%3A%22org.springframework.boot%22%20AND%20a%3A%22spring-boot-starter-webflux%22) 模块的最新版本。

## 4.场景 1:点对点消息传递

在第一个场景中，我们将使用直接交换，它是从客户端接收消息的代理中的逻辑实体。

**直接交换会将所有传入消息路由到一个(且仅一个)队列**，供客户端使用。多个客户端可以订阅同一个队列，但是只有一个客户端会收到给定的消息。

### 4.1.Exchange 和队列设置

在我们的场景中，我们使用一个封装了交换名称和路由密钥的`DestinationInfo`对象。以目的地名称为关键字的地图将用于存储所有可用的目的地。

下面的 *@PostConstruct* 方法将负责该初始设置:

```java
@Autowired
private AmqpAdmin amqpAdmin;

@Autowired
private DestinationsConfig destinationsConfig;

@PostConstruct
public void setupQueueDestinations() {
    destinationsConfig.getQueues()
        .forEach((key, destination) -> {
            Exchange ex = ExchangeBuilder.directExchange(
              destination.getExchange())
              .durable(true)
              .build();
            amqpAdmin.declareExchange(ex);
            Queue q = QueueBuilder.durable(
              destination.getRoutingKey())
              .build();
            amqpAdmin.declareQueue(q);
            Binding b = BindingBuilder.bind(q)
              .to(ex)
              .with(destination.getRoutingKey())
              .noargs();
            amqpAdmin.declareBinding(b);
        });
}
```

该方法使用 Spring 创建的`adminAmqp ` bean 来声明交换、队列，并使用给定的路由键将它们绑定在一起。

所有目的地都来自一个`DestinationsConfig ` bean，这是我们示例中使用的一个`@ConfigurationProperties`类。

这个类有一个属性，它由从`application.yml`配置文件中读取的映射构建的`DestinationInfo`对象填充。

### 4.2.生产者终点

生产者将通过发送一个`HTTP POST`到`/queue/{name}`位置来发送消息。

这是一个反应端点，所以我们使用一个`Mono`来返回一个简单的确认:

```java
@SpringBootApplication
@EnableConfigurationProperties(DestinationsConfig.class)
@RestController
public class SpringWebfluxAmqpApplication {

    // ... other members omitted

    @Autowired
    private AmqpTemplate amqpTemplate;

    @PostMapping(value = "/queue/{name}")
    public Mono<ResponseEntity<?>> sendMessageToQueue(
      @PathVariable String name, @RequestBody String payload) {

        DestinationInfo d = destinationsConfig
          .getQueues().get(name);
        if (d == null) {
            return Mono.just(
              ResponseEntity.notFound().build());
        }

        return Mono.fromCallable(() -> {
            amqpTemplate.convertAndSend(
              d.getExchange(), 
              d.getRoutingKey(), 
              payload);  
            return ResponseEntity.accepted().build();
        });
    } 
```

我们首先检查 name 参数是否对应于一个有效的目的地，如果是，我们使用自动连接的`amqpTemplate`实例实际发送有效负载——一个简单的`String`消息——到 RabbitMQ。

### 4.3.`MessageListenerContainer`工厂

为了异步接收消息，Spring AMQP 使用了一个`MessageContainerListener`抽象类来协调来自 AMQP 队列和应用程序提供的监听器的信息流。

因为我们需要这个类的具体实现来附加我们的消息侦听器，所以我们定义了一个工厂，将控制器代码与其实际实现隔离开来。

在我们的例子中，每当我们调用工厂方法的`createMessageListenerContainer` 方法时，它都会返回一个新的`SimpleMessageContainerListener`:

```java
@Component
public class MessageListenerContainerFactory {

    @Autowired
    private ConnectionFactory connectionFactory;

    public MessageListenerContainerFactory() {}

    public MessageListenerContainer createMessageListenerContainer(String queueName) {
        SimpleMessageListenerContainer mlc = new SimpleMessageListenerContainer(connectionFactory);
        mlc.addQueueNames(queueName);
        return mlc;
    }
}
```

### 4.4.消费者端点

消费者将访问生产者使用的同一个端点地址(`/queue/{name}`)来获取消息。

该端点返回一个`Flux `事件，其中每个事件对应一个接收到的消息:

```java
@Autowired
private MessageListenerContainerFactory messageListenerContainerFactory;

@GetMapping(
  value = "/queue/{name}",
  produces = MediaType.TEXT_EVENT_STREAM_VALUE)
public Flux<?> receiveMessagesFromQueue(@PathVariable String name) {

    DestinationInfo d = destinationsConfig
      .getQueues()
      .get(name);
    if (d == null) {
        return Flux.just(ResponseEntity.notFound()
          .build());
    }

    MessageListenerContainer mlc = messageListenerContainerFactory
      .createMessageListenerContainer(d.getRoutingKey());

    Flux<String> f = Flux.<String> create(emitter -> {
        mlc.setupMessageListener((MessageListener) m -> {
            String payload = new String(m.getBody());
            emitter.next(payload);
        });
        emitter.onRequest(v -> {
            mlc.start();
        });
        emitter.onDispose(() -> {
            mlc.stop();
        });
      });

    return Flux.interval(Duration.ofSeconds(5))
      .map(v -> "No news is good news")
      .mergeWith(f);
}
```

在对目的地名称进行初始检查之后，消费者端点使用从我们的注册中心恢复的`MessageListenerContainerFactory`和队列名称创建 *MessageListenerContainer* 。

一旦我们有了`MessageListenerContainer`，我们就使用它的`create()`构建器方法之一创建消息`Flux`。

在我们的特例中，我们使用一个带有一个`FluxSink`参数的 lambda，然后我们用它将 Spring AMQP 的基于监听器的异步 API 连接到我们的反应式应用程序。

我们还将两个额外的 lambdas 附加到发射器的`onRequest() `和`onDispose()`回调，这样我们的`MessageListenerContainer `就可以在`Flux`的生命周期之后分配/释放它的内部资源。

最后，我们将得到的`Flux `与另一个用`interval(), `创建的事件合并，后者每五秒钟创建一个新事件。**这些虚拟消息在我们的案例中发挥了重要的作用**:如果没有它们，我们只能在收到消息但未能发送时检测到客户端断开连接，这可能需要很长时间，具体取决于您的特定用例。

### 4.5.测试

有了消费者和发布者端点设置，我们现在可以对我们的示例应用程序进行一些测试。

我们需要在我们的`application.yml`上定义 RabbitMQ 的服务器连接细节和至少一个目的地，应该是这样的:

```java
spring:
  rabbitmq:
    host: localhost
    port: 5672
    username: guest
    password: guest

destinations:
  queues:
    NYSE:
      exchange: nyse
      routing-key: NYSE 
```

`spring.rabbitmq.*`属性定义了连接到运行在本地 Docker 容器中的 RabbitMQ 服务器所需的基本属性。请注意，上面显示的 IP 只是一个示例，在特定设置中可能会有所不同。

使用`destinations.queues.<name>.*`定义队列，其中`<name>`用作目的地名称。在这里，我们声明了一个名为“nyse”的目的地，它将用“NYSE”路由关键字将消息发送到 RabbitMQ 上的“NYSE”交易所。

一旦我们通过命令行或 IDE 启动了服务器，我们就可以开始发送和接收消息了。我们将使用`curl`实用程序，这是一个适用于 Windows、Mac & Linux 操作系统的通用实用程序。

下面的清单显示了如何将消息发送到我们的目的地以及服务器的预期响应:

```java
$ curl -v -d "Test message" http://localhost:8080/queue/NYSE
* timeout on name lookup is not supported
*   Trying 127.0.0.1...
* Connected to localhost (127.0.0.1) port 8080 (#0)
> POST /queue/NYSE HTTP/1.1
> Host: localhost:8080
> User-Agent: curl/7.49.1
> Accept: */*
> Content-Length: 12
> Content-Type: application/x-www-form-urlencoded
>
* upload completely sent off: 12 out of 12 bytes
< HTTP/1.1 202 Accepted
< content-length: 0
<
* Connection #0 to host localhost left intact
```

执行此命令后，我们可以通过发出以下命令来验证 RabbitMQ 是否收到了消息，以及消息是否已准备好供使用:

```java
$ docker exec rabbitmq rabbitmqctl list_queues
Timeout: 60.0 seconds ...
Listing queues for vhost / ...
NYSE    1 
```

现在，我们可以使用 curl 通过以下命令读取消息:

```java
$ curl -v http://localhost:8080/queue/NYSE
* timeout on name lookup is not supported
*   Trying 127.0.0.1...
* Connected to localhost (127.0.0.1) port 8080 (#0)
> GET /queue/NYSE HTTP/1.1
> Host: localhost:8080
> User-Agent: curl/7.49.1
> Accept: */*
>
< HTTP/1.1 200 OK
< transfer-encoding: chunked
< Content-Type: text/event-stream;charset=UTF-8
<
data:Test message

data:No news is good news...

... same message repeating every 5 secs
```

正如我们所看到的，首先我们得到先前存储的消息，然后我们开始每 5 秒钟接收一次虚拟消息。

如果我们再次运行命令来列出队列，我们现在可以看到没有存储任何消息:

$ dock exec rabbitq rabbitqctl list _ queues

```java
Timeout: 60.0 seconds ...
Listing queues for vhost / ...
NYSE    0
```

## 5.场景 2:发布-订阅

消息传递应用程序的另一个常见场景是发布-订阅模式，其中一条消息必须发送给多个消费者。

RabbitMQ 提供了两种类型的交换来支持这些类型的应用程序:扇出和主题。

这两种类型的主要区别在于，后者允许我们根据注册时提供的路由关键字模式(例如“alarm.mailserver.*”)来过滤要接收的消息，而前者只是将传入的消息复制到所有绑定的队列中。

RabbitMQ 还支持头交换，这允许更复杂的消息过滤，但是它的使用超出了本文的范围。

### 5.1.目的地设置

我们在启动时用另一个 *@PostConstruct* 方法定义发布/订阅目的地，就像我们在点对点场景中所做的那样。

唯一的区别是我们只创建了`Exchanges`，而没有`Queues`——这些将按需创建并在以后绑定到`Exchange`，因为我们希望每个客户端都有一个专属的`Queue`:

```java
@PostConstruct
public void setupTopicDestinations(
    destinationsConfig.getTopics()
      .forEach((key, destination) -> {
          Exchange ex = ExchangeBuilder
            .topicExchange(destination.getExchange())
            .durable(true)
            .build();
            amqpAdmin.declareExchange(ex);
      });
}
```

### 5.2.发布者端点

客户端将使用在`/topic/{name}`位置可用的发布者端点来发布将被发送到所有连接的客户端的消息。

和前面的场景一样，我们使用一个`@PostMapping`在发送消息后返回一个带有状态的`Mono`:

```java
@PostMapping(value = "/topic/{name}")
public Mono<ResponseEntity<?>> sendMessageToTopic(
  @PathVariable String name, @RequestBody String payload) {

    DestinationInfo d = destinationsConfig
      .getTopics()
      .get(name);

    if (d == null) {
        return Mono.just(ResponseEntity.notFound().build());
    }      

   return Mono.fromCallable(() -> {
       amqpTemplate.convertAndSend(
         d.getExchange(), d.getRoutingKey(),payload);   
            return ResponseEntity.accepted().build();
        });
    }
```

### 5.3.订户端点

我们的订户端点将位于`/topic/{name}`，为连接的客户端生成`Flux`条消息。

这些消息包括收到的消息和每 5 秒钟生成的假消息:

```java
@GetMapping(
  value = "/topic/{name}",
  produces = MediaType.TEXT_EVENT_STREAM_VALUE)
public Flux<?> receiveMessagesFromTopic(@PathVariable String name) {
    DestinationInfo d = destinationsConfig.getTopics()
        .get(name);
    if (d == null) {
        return Flux.just(ResponseEntity.notFound()
            .build());
    }
    Queue topicQueue = createTopicQueue(d);
    String qname = topicQueue.getName();
    MessageListenerContainer mlc = messageListenerContainerFactory.createMessageListenerContainer(qname);
    Flux<String> f = Flux.<String> create(emitter -> {
        mlc.setupMessageListener((MessageListener) m -> {
            String payload = new String(m.getBody());
            emitter.next(payload);
        });
        emitter.onRequest(v -> {
            mlc.start();
        });
        emitter.onDispose(() -> {
            amqpAdmin.deleteQueue(qname);
            mlc.stop();
        });            
      });

    return Flux.interval(Duration.ofSeconds(5))
        .map(v -> "No news is good news")
        .mergeWith(f);
}
```

这段代码与我们在前面的案例中看到的基本相同，只有以下不同:首先，我们为每个新订户创建一个新的`Queue`。

我们通过调用`createTopicQueue()`方法来实现这一点，该方法使用来自`DestinationInfo`实例的信息来创建一个独占的、非持久的队列，然后我们使用配置的路由键将该队列绑定到`Exchange`:

```java
private Queue createTopicQueue(DestinationInfo destination) {

    Exchange ex = ExchangeBuilder
      .topicExchange(destination.getExchange())
      .durable(true)
      .build();
    amqpAdmin.declareExchange(ex);
    Queue q = QueueBuilder
      .nonDurable()
      .build();     
    amqpAdmin.declareQueue(q);
    Binding b = BindingBuilder.bind(q)
      .to(ex)
      .with(destination.getRoutingKey())
      .noargs();        
    amqpAdmin.declareBinding(b);
    return q;
}
```

注意，尽管我们再次声明了`Exchange` ，RabbitMQ 不会创建一个新的，因为我们已经在启动时声明了它。

第二个区别是我们传递给`onDispose()`方法的 lambda，这一次当订户断开连接时，它也将删除`Queue`。

### 5.3.测试

为了测试发布订阅场景，我们必须首先在 out `application.yml`中定义一个主题目的地，如下所示:

```java
destinations:
## ... queue destinations omitted      
  topics:
    weather:
      exchange: alerts
      routing-key: WEATHER
```

这里，我们定义了一个主题端点，它将在`/topic/weather`位置可用。这个端点将用于向 RabbitMQ 上的“alerts”交换发布带有“WEATHER”路由关键字的消息。

启动服务器后，我们可以使用`rabbitmqctl`命令验证是否已经创建了交换:

```java
$ docker exec docker_rabbitmq_1 rabbitmqctl list_exchanges
Listing exchanges for vhost / ...
amq.topic       topic
amq.fanout      fanout
amq.match       headers
amq.headers     headers
        direct
amq.rabbitmq.trace      topic
amq.direct      direct
alerts  topic
```

现在，如果我们发出`list_bindings`命令，我们可以看到没有与“警报”交换相关的队列:

```java
$ docker exec rabbitmq rabbitmqctl list_bindings
Listing bindings for vhost /...
        exchange        NYSE    queue   NYSE    []
nyse    exchange        NYSE    queue   NYSE    [] 
```

让我们启动几个订阅我们的目的地的订阅者，打开两个命令 shellss 并在每个 shell 上发出以下命令:

```java
$ curl -v http://localhost:8080/topic/weather
* timeout on name lookup is not supported
*   Trying 127.0.0.1...
* Connected to localhost (127.0.0.1) port 8080 (#0)
> GET /topic/weather HTTP/1.1
> Host: localhost:8080
> User-Agent: curl/7.49.1
> Accept: */*
>
< HTTP/1.1 200 OK
< transfer-encoding: chunked
< Content-Type: text/event-stream;charset=UTF-8
<
data:No news is good news...

# ... same message repeating indefinitely 
```

最后，我们再次使用 curl 向我们的订户发送一些警报:

```java
$ curl -v -d "Hurricane approaching!" http://localhost:8080/topic/weather
* timeout on name lookup is not supported
*   Trying 127.0.0.1...
* Connected to localhost (127.0.0.1) port 8080 (#0)
> POST /topic/weather HTTP/1.1
> Host: localhost:8080
> User-Agent: curl/7.49.1
> Accept: */*
> Content-Length: 22
> Content-Type: application/x-www-form-urlencoded
>
* upload completely sent off: 22 out of 22 bytes
< HTTP/1.1 202 Accepted
< content-length: 0
<
* Connection #0 to host localhost left intact
```

一旦我们发送信息，我们几乎可以立即看到“飓风即将来临！”在每个订户的 shell 上。

如果我们现在检查可用的绑定，我们可以看到每个订户都有一个队列:

```java
$ docker exec rabbitmq rabbitmqctl list_bindings
Listing bindings for vhost /...
        exchange        IBOV    queue   IBOV    []
        exchange        NYSE    queue   NYSE    []
        exchange        spring.gen-i0m0pbyKQMqpz2_KFZCd0g       
  queue   spring.gen-i0m0pbyKQMqpz2_KFZCd0g       []
        exchange        spring.gen-wCHALTsIS1q11PQbARJ7eQ       
  queue   spring.gen-wCHALTsIS1q11PQbARJ7eQ       []
alerts  exchange        spring.gen-i0m0pbyKQMqpz2_KFZCd0g     
  queue   WEATHER []
alerts  exchange        spring.gen-wCHALTsIS1q11PQbARJ7eQ     
  queue   WEATHER []
ibov    exchange        IBOV    queue   IBOV    []
nyse    exchange        NYSE    queue   NYSE    []
quotes  exchange        NYSE    queue   NYSE    []
```

一旦我们在订户的 shell 上点击 Ctrl-C，我们的网关将最终检测到客户端已经断开连接，并将移除这些绑定。

## 6.结论

在本文中，我们演示了如何创建一个简单的反应式应用程序，它使用 `spring-amqp`模块与 RabbitMQ 服务器进行交互。

只需几行代码，我们就能够创建一个功能性的 HTTP-to-AMQP 网关，它支持点对点和发布-订阅集成模式，我们可以很容易地扩展它，通过添加标准的 Spring 特性来添加额外的特性，比如安全性。

本文中显示的代码可以在 Github 上找到。