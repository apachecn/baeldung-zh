# 测试卡夫卡和 Spring Boot

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-boot-kafka-testing>

## 1.概观

Apache Kafka 是一个强大的分布式容错流处理系统。在之前的教程中，我们学习了[如何使用 Spring 和 Kafka](/web/20220626083646/https://www.baeldung.com/spring-kafka) 。

在本教程中，**我们将建立在前一个的基础上，学习如何编写可靠的、自包含的集成测试，这些测试不依赖于运行的外部 Kafka 服务器。**

首先，我们将从如何使用和配置 Kafka 的嵌入式实例开始。

然后我们将看看如何从我们的测试中利用流行的框架[test containers](https://web.archive.org/web/20220626083646/https://www.testcontainers.org/)。

## 2.属国

当然，我们需要将标准的 [`spring-kafka`依赖关系](https://web.archive.org/web/20220626083646/https://search.maven.org/classic/#search%7Cga%7C1%7Cg%3A%22org.springframework.kafka%22%20AND%20a%3A%22spring-kafka%22)添加到我们的`pom.xml`中:

```java
<dependency>
    <groupId>org.springframework.kafka</groupId>
    <artifactId>spring-kafka</artifactId>
    <version>2.7.2</version>
</dependency>
```

那么我们将需要另外两个专门用于测试的依赖项。

首先，我们将添加 [`spring-kafka-test`神器](https://web.archive.org/web/20220626083646/https://search.maven.org/classic/#search%7Cga%7C1%7Cg%3A%22org.springframework.kafka%22%20AND%20a%3A%22spring-kafka-test%22):

```java
<dependency>
    <groupId>org.springframework.kafka</groupId>
    <artifactId>spring-kafka-test</artifactId>
    <version>2.6.3.RELEASE</version>
    <scope>test</scope>
</dependency>
```

最后，我们将添加 Testcontainers Kafka 依赖项，它也可以在 [Maven Central](https://web.archive.org/web/20220626083646/https://search.maven.org/classic/#search%7Cga%7C1%7Cg%3A%22org.testcontainers%22%20AND%20a%3A%22kafka%22) 上获得:

```java
<dependency>
    <groupId>org.testcontainers</groupId>
    <artifactId>kafka</artifactId>
    <version>1.15.3</version>
    <scope>test</scope>
</dependency>
```

现在我们已经配置了所有必要的依赖项，我们可以使用 Kafka 编写一个简单的 Spring Boot 应用程序。

## 3.一个简单的 Kafka 生产者-消费者应用程序

在本教程中，我们测试的重点将是一个简单的生产者-消费者 Spring Boot·卡夫卡应用程序。

让我们从定义应用程序入口点开始:

```java
@SpringBootApplication
public class KafkaProducerConsumerApplication {

    public static void main(String[] args) {
        SpringApplication.run(KafkaProducerConsumerApplication.class, args);
    }
}
```

正如我们所看到的，这是一个标准的 Spring Boot 应用程序。

### 3.1.生产者设置

接下来，让我们考虑一个生产者 bean，我们将使用它向给定的 Kafka 主题发送消息:

```java
@Component
public class KafkaProducer {

    private static final Logger LOGGER = LoggerFactory.getLogger(KafkaProducer.class);

    @Autowired
    private KafkaTemplate<String, String> kafkaTemplate;

    public void send(String topic, String payload) {
        LOGGER.info("sending payload='{}' to topic='{}'", payload, topic);
        kafkaTemplate.send(topic, payload);
    }
}
```

我们上面定义的`KafkaProducer` bean 仅仅是一个`KafkaTemplate`类的包装器。**这个类提供了高级别的线程安全操作，比如向提供的主题发送数据，这正是我们在`send`方法中所做的。**

### 3.2.消费者设置

同样，我们现在将定义一个简单的消费者 bean，它将侦听 Kafka 主题并接收消息:

```java
@Component
public class KafkaConsumer {

    private static final Logger LOGGER = LoggerFactory.getLogger(KafkaConsumer.class);

    private CountDownLatch latch = new CountDownLatch(1);
    private String payload;

    @KafkaListener(topics = "${test.topic}")
    public void receive(ConsumerRecord<?, ?> consumerRecord) {
        LOGGER.info("received payload='{}'", consumerRecord.toString());
        payload = consumerRecord.toString();
        latch.countDown();
    }

    public void resetLatch() {
        latch = new CountDownLatch(1);
    }

    // other getters
}
```

我们的简单消费者使用`receive`方法上的`@KafkaListener`注释来监听给定主题的消息。我们将在稍后的测试中看到如何配置`test.topic`。

此外，receive 方法将消息内容存储在我们的 bean 中，并减少变量`latch`的计数。**这个变量是一个简单的[线程安全计数器](/web/20220626083646/https://www.baeldung.com/cs/async-vs-multi-threading)字段，我们将在稍后的测试中使用它来确保我们成功接收到一条消息。**

现在我们已经实现了使用 Spring Boot 的简单 Kafka 应用程序，让我们看看如何编写集成测试。

## 4.关于测试的一句话

一般来说，当编写干净的集成测试时，我们不应该依赖我们可能无法控制或者可能突然停止工作的外部服务。这可能会对我们的测试结果产生不利影响。

类似地，如果我们依赖外部服务，在这种情况下，一个运行的 Kafka 代理，我们可能无法在测试中以我们想要的方式设置、控制和拆除它。

### 4.1.应用程序属性

我们将在测试中使用一组非常简单的应用程序配置属性。

我们将在我们的`src/test/resources/application.yml`文件中定义这些属性:

```java
spring:
  kafka:
    consumer:
      auto-offset-reset: earliest
      group-id: baeldung
test:
  topic: embedded-test-topic
```

这是我们在使用 Kafka 的嵌入式实例或本地代理时需要的最小属性集。

其中大多数都是不言自明的，**，但我们应该强调的是消费者属性`auto-offset-reset: earliest`。**该属性确保我们的消费者组能够收到我们发送的消息，因为容器可能会在发送完成后启动。

此外，我们用值`embedded-test-topic`配置一个主题属性，这是我们将在测试中使用的主题。

## 5.使用嵌入式 Kafka 进行测试

在这一节中，我们将看看如何使用内存中的 Kafka 实例来运行我们的测试。这也被称为嵌入式卡夫卡。

我们之前添加的依赖项`spring-kafka-test` 包含了一些有用的工具来帮助测试我们的应用程序。**最值得注意的是，它包含了`EmbeddedKafkaBroker` 类。**

记住这一点，让我们继续编写我们的第一个集成测试:

```java
@SpringBootTest
@DirtiesContext
@EmbeddedKafka(partitions = 1, brokerProperties = { "listeners=PLAINTEXT://localhost:9092", "port=9092" })
class EmbeddedKafkaIntegrationTest {

    @Autowired
    private KafkaConsumer consumer;

    @Autowired
    private KafkaProducer producer;

    @Value("${test.topic}")
    private String topic;

    @Test
    public void givenEmbeddedKafkaBroker_whenSendingWithSimpleProducer_thenMessageReceived() 
      throws Exception {
        String data = "Sending with our own simple KafkaProducer";

        producer.send(topic, data);

        boolean messageConsumed = consumer.getLatch().await(10, TimeUnit.SECONDS);
        assertTrue(messageConsumed);
        assertThat(consumer.getPayload(), containsString(data));
    }
}
```

让我们看一下测试的关键部分。

首先，我们开始用两个非常标准的 Spring 注释来装饰我们的测试类:

*   [`@SpringBootTest`](/web/20220626083646/https://www.baeldung.com/spring-boot-testing) 注释将确保我们的测试启动 Spring 应用程序上下文。
*   我们还使用了`[@DirtiesContext](/web/20220626083646/https://www.baeldung.com/spring-dirtiescontext)`注释，这将确保这个上下文在不同的测试之间被清理和重置。

**关键的部分来了——我们使用`@EmbeddedKafka` 注释将一个`EmbeddedKafkaBroker`的实例注入到我们的测试中。**

此外，我们可以使用几个属性来配置嵌入式 Kafka 节点:

*   `partitions`–这是每个主题使用的分区数量。为了让事情简单明了，我们只希望在测试中使用一个。
*   `brokerProperties`–Kafka 经纪人的额外财产。同样，我们保持简单，指定一个纯文本侦听器和一个端口号。

接下来，我们自动连接我们的`consumer`和`producer`类，并配置一个主题来使用来自我们的`application.properties`的值。

对于拼图的最后一块，**我们简单地向我们的测试主题发送一条消息，并验证该消息已经被接收并且包含了我们的测试主题的名称。**

当我们运行测试时，我们将在冗长的 Spring 输出中看到以下内容:

```java
...
12:45:35.099 [main] INFO  c.b.kafka.embedded.KafkaProducer -
  sending payload='Sending with our own simple KafkaProducer' to topic='embedded-test-topic'
...
12:45:35.103 [org.springframework.kafka.KafkaListenerEndpointContainer#0-0-C-1]
  INFO  c.b.kafka.embedded.KafkaConsumer - received payload=
  'ConsumerRecord(topic = embedded-test-topic, partition = 0, leaderEpoch = 0, offset = 1,
  CreateTime = 1605267935099, serialized key size = -1, 
  serialized value size = 41, headers = RecordHeaders(headers = [], isReadOnly = false),
  key = null, value = Sending with our own simple KafkaProducer key)'
```

这证实了我们的测试工作正常。厉害！我们现在有了一种使用内存中的 Kafka 代理编写自包含、独立的集成测试的方法。

## 6.用测试容器测试 Kafka

有时，我们可能会看到真实的外部服务与专门为测试目的而提供的服务的嵌入式内存实例之间的微小差异。虽然不太可能，但也有可能是我们测试中使用的端口被占用，导致了故障。

记住这一点，在本节中，我们将看到我们之前使用 [Testcontainers 框架](/web/20220626083646/https://www.baeldung.com/docker-test-containers)进行测试的方法的变化。从我们的集成测试中，我们将看到如何实例化和管理托管在 [Docker 容器](/web/20220626083646/https://www.baeldung.com/tag/docker/)中的外部 Apache Kafka 代理。

让我们定义另一个集成测试，它与我们在上一节中看到的非常相似:

```java
@RunWith(SpringRunner.class)
@Import(com.baeldung.kafka.testcontainers.KafkaTestContainersLiveTest.KafkaTestContainersConfiguration.class)
@SpringBootTest(classes = KafkaProducerConsumerApplication.class)
@DirtiesContext
public class KafkaTestContainersLiveTest {

    @ClassRule
    public static KafkaContainer kafka = 
      new KafkaContainer(DockerImageName.parse("confluentinc/cp-kafka:5.4.3"));

    @Autowired
    private KafkaConsumer consumer;

    @Autowired
    private KafkaProducer producer;

    @Value("${test.topic}")
    private String topic;

    @Test
    public void givenKafkaDockerContainer_whenSendingWithSimpleProducer_thenMessageReceived() 
      throws Exception {
        String data = "Sending with our own simple KafkaProducer";

        producer.send(topic, data);

        boolean messageConsumed = consumer.getLatch().await(10, TimeUnit.SECONDS);

        assertTrue(messageConsumed);
        assertThat(consumer.getPayload(), containsString(data));
    }
}
```

让我们来看看不同之处。我们声明了`kafka`字段，这是一个标准的 [JUnit `@ClassRule`](/web/20220626083646/https://www.baeldung.com/junit-4-rules) 。**这个字段是`KafkaContainer`类的一个实例，它将准备和管理我们运行 Kafka 的容器的生命周期。**

为了避免端口冲突，Testcontainers 会在 docker 容器启动时动态分配一个端口号。

出于这个原因，我们使用类`KafkaTestContainersConfiguration`提供了一个定制的消费者和生产者工厂配置:

```java
@Bean
public Map<String, Object> consumerConfigs() {
    Map<String, Object> props = new HashMap<>();
    props.put(ConsumerConfig.BOOTSTRAP_SERVERS_CONFIG, kafka.getBootstrapServers());
    props.put(ConsumerConfig.AUTO_OFFSET_RESET_CONFIG, "earliest");
    props.put(ConsumerConfig.GROUP_ID_CONFIG, "baeldung");
    // more standard configuration
    return props;
}

@Bean
public ProducerFactory<String, String> producerFactory() {
    Map<String, Object> configProps = new HashMap<>();
    configProps.put(ProducerConfig.BOOTSTRAP_SERVERS_CONFIG, kafka.getBootstrapServers());
    // more standard configuration
    return new DefaultKafkaProducerFactory<>(configProps);
}
```

然后，我们在测试开始时通过`@Import`注释引用这个配置。

原因是我们需要一种方法将服务器地址注入到我们的应用程序中，如前所述，这是动态生成的。

**我们通过调用`getBootstrapServers()`方法来实现这一点，该方法将返回引导服务器位置**:

```java
bootstrap.servers = [PLAINTEXT://localhost:32789]
```

现在，当我们运行测试时，我们应该看到 Testcontainers 做了几件事:

*   检查我们的本地 Docker 设置
*   如有必要，拉动`confluentinc/cp-kafka:5.4.3` docker 图像
*   启动一个新容器，并等待它准备就绪
*   最后，在我们的测试完成后，关闭并删除容器

同样，通过检查测试输出可以确认这一点:

```java
13:33:10.396 [main] INFO  ? [confluentinc/cp-kafka:5.4.3]
  - Creating container for image: confluentinc/cp-kafka:5.4.3
13:33:10.454 [main] INFO  ? [confluentinc/cp-kafka:5.4.3]
  - Starting container with ID: b22b752cee2e9e9e6ade38e46d0c6d881ad941d17223bda073afe4d2fe0559c3
13:33:10.785 [main] INFO  ? [confluentinc/cp-kafka:5.4.3]
  - Container confluentinc/cp-kafka:5.4.3 is starting: b22b752cee2e9e9e6ade38e46d0c6d881ad941d17223bda073afe4d2fe0559c3
```

转眼间。使用 Kafka docker 容器的工作集成测试。

## 7.结论

在本文中，我们了解了用 Spring Boot 测试 Kafka 应用程序的几种方法。

在第一种方法中，我们看到了如何配置和使用本地内存中的 Kafka 代理。

然后，我们从测试中看到了如何使用 Testcontainers 来设置一个运行在 docker 容器内部的外部 Kafka 代理。

和往常一样，GitHub 上的[提供了这篇文章的完整源代码。](https://web.archive.org/web/20220626083646/https://github.com/eugenp/tutorials/tree/master/spring-kafka)