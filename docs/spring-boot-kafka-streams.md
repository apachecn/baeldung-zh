# 卡夫卡与 Spring Boot 同流合污

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-boot-kafka-streams>

## 1。简介

在这篇文章中，我们将看到如何使用 Spring Boot 设置[卡夫卡流](/web/20220707145357/https://www.baeldung.com/java-kafka-streams)。Kafka Streams 是一个基于 Apache Kafka 构建的客户端库。它支持以声明的方式处理无界的事件流。

流数据的一些真实例子可以是传感器数据、股票市场事件流和系统日志。对于本教程，我们将构建一个简单的字数统计流应用程序。让我们从 Kafka Streams 的概述开始，然后设置这个例子及其在 Spring Boot 的测试。

## 2.概观

Kafka Streams 提供了 Kafka 主题和关系数据库表之间的二元性。它使我们能够对一个或多个流事件进行连接、分组、聚合和过滤等操作。

Kafka 流的一个重要概念是处理器拓扑。**处理器拓扑是 Kafka 流在一个或多个事件流**上操作的蓝图。本质上，处理器拓扑可以被认为是一个[有向无环图](/web/20220707145357/https://www.baeldung.com/cs/dag-topological-sort)。在该图中，节点被分类为源、处理器和接收器节点，而边表示流事件的流程。

拓扑顶部的源从 Kafka 接收流数据，将其向下传递到执行自定义操作的处理器节点，并通过接收器节点流出到新的 Kafka 主题。除了核心处理之外，还使用检查点定期保存流的状态，以实现容错和弹性。

## 3.属国

我们首先将 [`spring-kafka`](https://web.archive.org/web/20220707145357/https://search.maven.org/search?q=g:org.springframework.kafka%20AND%20a:spring-kafka) 和 [`kafka-streams`](https://web.archive.org/web/20220707145357/https://search.maven.org/search?q=g:org.apache.kafka%20AND%20a:kafka-streams) 依赖项添加到 POM 中:

```java
<dependency>
    <groupId>org.springframework.kafka</groupId>
    <artifactId>spring-kafka</artifactId>
    <version>2.7.8</version>
</dependency>
<dependency>
    <groupId>org.apache.kafka</groupId
    <artifactId>kafka-streams</artifactId>
    <version>2.7.1</version>
</dependency> 
```

## 4.例子

我们的示例应用程序从输入 Kafka 主题中读取流事件。一旦记录被读取，它就对它们进行处理，以分割文本并对单个单词进行计数。随后，它将更新的字数发送到 Kafka 输出。除了输出主题，我们还将创建一个简单的 REST 服务，通过 HTTP 端点公开这个计数。

总的来说，输出主题将随着从输入事件中提取的单词及其更新的计数而不断更新。

### 4.1.配置

首先，让我们在 Java config 类中定义 Kafka 流配置:

```java
@Configuration
@EnableKafka
@EnableKafkaStreams
public class KafkaConfig {

    @Value(value = "${spring.kafka.bootstrap-servers}")
    private String bootstrapAddress;

    @Bean(name = KafkaStreamsDefaultConfiguration.DEFAULT_STREAMS_CONFIG_BEAN_NAME)
    KafkaStreamsConfiguration kStreamsConfig() {
        Map<String, Object> props = new HashMap<>();
        props.put(APPLICATION_ID_CONFIG, "streams-app");
        props.put(BOOTSTRAP_SERVERS_CONFIG, bootstrapAddress);
        props.put(DEFAULT_KEY_SERDE_CLASS_CONFIG, Serdes.String().getClass().getName());
        props.put(DEFAULT_VALUE_SERDE_CLASS_CONFIG, Serdes.String().getClass().getName());

        return new KafkaStreamsConfiguration(props);
    }

    // other config
}
```

这里，我们使用了`@EnableKafkaStreams`注释来自动配置所需的组件。这个自动配置需要一个名称由`DEFAULT_STREAMS_CONFIG_BEAN_NAME`指定的`KafkaStreamsConfiguration` bean。因此， **Spring Boot 使用这种配置并创建了一个`KafkaStreams`客户端来管理我们的应用生命周期**。

在我们的例子中，我们已经为我们的配置提供了应用程序 id、引导服务器连接细节和 SerDes(串行器/解串器)。

### 4.2.拓扑学

现在我们已经设置了配置，让我们为我们的应用程序构建拓扑，以记录来自输入消息的单词数:

```java
@Component
public class WordCountProcessor {

    private static final Serde<String> STRING_SERDE = Serdes.String();

    @Autowired
    void buildPipeline(StreamsBuilder streamsBuilder) {
        KStream<String, String> messageStream = streamsBuilder
          .stream("input-topic", Consumed.with(STRING_SERDE, STRING_SERDE));

        KTable<String, Long> wordCounts = messageStream
          .mapValues((ValueMapper<String, String>) String::toLowerCase)
          .flatMapValues(value -> Arrays.asList(value.split("\\W+")))
          .groupBy((key, word) -> word, Grouped.with(STRING_SERDE, STRING_SERDE))
          .count();

        wordCounts.toStream().to("output-topic");
    }
}
```

这里，我们定义了一个配置方法，并用`@Autowired`对其进行了注释。Spring 处理这个注释，并将容器中的匹配 bean 连接到`StreamsBuilder`参数中。或者，我们也可以在配置类中创建一个 bean 来生成拓扑。

让我们可以访问所有的 Kafka Streams APIs，它变得像一个普通的 Kafka Streams 应用程序。在我们的例子中，我们使用这个**高级 DSL 来定义应用程序的转换**:

*   使用指定的键和值 SerDes 从输入主题创建一个`KStream`。
*   通过转换、拆分、分组，然后对数据进行计数来创建一个`KTable`。
*   将结果具体化为输出流。

本质上， **Spring Boot 在管理我们的`KStream`实例**的生命周期时，提供了一个非常薄的 Streams API 包装器。它创建和配置拓扑所需的组件，并执行我们的 Streams 应用程序。重要的是，这让我们在 Spring 管理生命周期的同时，专注于我们的核心业务逻辑。

### 4.3.休息服务

在用声明性步骤定义了管道之后，让我们创建 REST 控制器。这提供了端点，以便将消息发布到输入主题，并获得指定单词的计数。但是重要的是，**应用程序从 Kafka Streams 状态存储中检索数据，而不是输出主题**。

首先，让我们修改前面的`KTable`,并将聚合计数具体化为本地状态存储。这可以从 REST 控制器中查询:

```java
KTable<String, Long> wordCounts = textStream
  .mapValues((ValueMapper<String, String>) String::toLowerCase)
  .flatMapValues(value -> Arrays.asList(value.split("\\W+")))
  .groupBy((key, value) -> value, Grouped.with(STRING_SERDE, STRING_SERDE))
  .count(Materialized.as("counts"));
```

在此之后，我们可以更新控制器，从这个`counts`状态存储中检索计数值:

```java
@GetMapping("/count/{word}")
public Long getWordCount(@PathVariable String word) {
    KafkaStreams kafkaStreams = factoryBean.getKafkaStreams();
    ReadOnlyKeyValueStore<String, Long> counts = kafkaStreams.store(
      StoreQueryParameters.fromNameAndType("counts", QueryableStoreTypes.keyValueStore())
    );
    return counts.get(word);
}
```

这里，`factoryBean`是连接到控制器中的`StreamsBuilderFactoryBean`的一个实例。这提供了由该工厂 bean 管理的`KafkaStreams`实例。因此，我们可以获得之前创建的`counts`键/值状态存储，由`KTable`表示。此时，我们可以使用它从本地状态存储中获取所请求的字的当前计数。

## 5.测试

测试是开发和验证应用程序拓扑的关键部分。Spring Kafka test library 和 [Testcontainers](/web/20220707145357/https://www.baeldung.com/docker-test-containers) 都为在不同级别测试我们的应用程序提供了出色的支持。

### 5.1.单元测试

首先，让我们使用`TopologyTestDriver`为我们的拓扑设置一个单元测试。这是测试 Kafka Streams 应用程序的主要测试工具:

```java
@Test
void givenInputMessages_whenProcessed_thenWordCountIsProduced() {
    StreamsBuilder streamsBuilder = new StreamsBuilder();
    wordCountProcessor.buildPipeline(streamsBuilder);
    Topology topology = streamsBuilder.build();

    try (TopologyTestDriver topologyTestDriver = new TopologyTestDriver(topology, new Properties())) {
        TestInputTopic<String, String> inputTopic = topologyTestDriver
          .createInputTopic("input-topic", new StringSerializer(), new StringSerializer());

        TestOutputTopic<String, Long> outputTopic = topologyTestDriver
          .createOutputTopic("output-topic", new StringDeserializer(), new LongDeserializer());

        inputTopic.pipeInput("key", "hello world");
        inputTopic.pipeInput("key2", "hello");

        assertThat(outputTopic.readKeyValuesToList())
          .containsExactly(
            KeyValue.pair("hello", 1L),
            KeyValue.pair("world", 1L),
            KeyValue.pair("hello", 2L)
          );
    }
}
```

在这里，我们需要的第一件事是封装我们在测试中的业务逻辑的`Topology`。现在，我们可以使用`TopologyTestDriver`来为我们的测试创建输入和输出主题。至关重要的是，这个**消除了运行代理并验证管道行为**的需要。换句话说，它使得在不使用真正的 Kafka 代理的情况下验证我们的管道行为变得快速而容易。

### 5.2.集成测试

最后，让我们使用 Testcontainers 框架来端到端地测试我们的应用程序。这使用了一个正在运行的 Kafka 代理，并启动我们的应用程序进行一个完整的测试:

```java
@Testcontainers
@SpringBootTest(classes = KafkaStreamsApplication.class, webEnvironment = WebEnvironment.RANDOM_PORT)
class KafkaStreamsApplicationLiveTest {

    @Container
    private static final KafkaContainer KAFKA = new KafkaContainer(
      DockerImageName.parse("confluentinc/cp-kafka:5.4.3"));

    private final BlockingQueue<String> output = new LinkedBlockingQueue<>();

    // other test setup

    @Test
    void givenInputMessages_whenPostToEndpoint_thenWordCountsReceivedOnOutput() throws Exception {
        postMessage("test message");

        startOutputTopicConsumer();

        // assert correct counts on output topic
        assertThat(output.poll(2, MINUTES)).isEqualTo("test:1");
        assertThat(output.poll(2, MINUTES)).isEqualTo("message:1");

        // assert correct count from REST service
        assertThat(getCountFromRestServiceFor("test")).isEqualTo(1);
        assertThat(getCountFromRestServiceFor("message")).isEqualTo(1);
    }
}
```

在这里，我们已经向 REST 控制器发送了一个 POST，它又将消息发送到 Kafka 输入主题。作为设置的一部分，我们还启动了一个 Kafka 消费者。它异步监听输出 Kafka 主题，并用接收到的字数更新`BlockingQueue`。

在测试执行期间，应用程序应该处理输入消息。接下来，我们可以使用 REST 服务验证来自主题和状态存储的预期输出。

## 6.结论

在本教程中，我们看到了如何创建一个简单的事件驱动的应用程序来处理 Kafka 流和 Spring Boot 的消息。

在简要概述了核心流概念之后，我们看了流拓扑的配置和创建。然后，我们看到了如何将它与 Spring Boot 提供的 REST 功能集成在一起。最后，我们讨论了一些有效测试和验证我们的拓扑和应用程序行为的方法。

和往常一样，完整的源代码可以在 GitHub 上找到[。](https://web.archive.org/web/20220707145357/https://github.com/eugenp/tutorials/tree/master/spring-kafka)