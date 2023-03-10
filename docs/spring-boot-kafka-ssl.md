# 使用 Spring Boot 配置 Kafka SSL

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-boot-kafka-ssl>

## 1.介绍

在本教程中，我们将介绍使用 SSL 身份验证将 Spring Boot 客户端连接到 [Apache Kafka](/web/20220625080310/https://www.baeldung.com/spring-kafka) 代理的基本设置。

自 2015 年以来，安全套接字层(SSL)实际上已被弃用，并被传输层安全性(TLS)所取代。然而，由于历史原因，Kafka(和 Java)仍然引用“SSL ”,我们在本文中也将遵循这一约定。

## 2.SSL 概述

默认情况下，Apache Kafka 以明文形式发送所有数据，不进行任何身份验证。

首先，我们可以为代理和客户端之间的加密配置 SSL。默认情况下，这需要使用公钥加密的**单向认证，其中客户端认证服务器证书**。

此外，服务器还可以使用单独的机制(如 SSL 或 SASL)对客户端进行身份验证，从而实现双向身份验证或双向 TLS (mTLS)。基本上，**双向 SSL 认证确保客户端和服务器都使用 SSL 证书来验证彼此的身份，并在两个方向上相互信任**。

在本文中，**代理将使用 SSL 对客户端**进行身份验证，而[密钥库](/web/20220625080310/https://www.baeldung.com/java-keystore-truststore-difference#java-keystore)和[信任库](/web/20220625080310/https://www.baeldung.com/java-keystore-truststore-difference#java-keystore)将用于保存证书和密钥。

每个代理都需要自己的密钥库，其中包含私有密钥和公共证书。客户端使用其信任库来验证该证书并信任服务器。类似地，每个客户端也需要自己的密钥库，其中包含自己的私钥和公共证书。服务器使用其信任库来验证和信任客户端的证书，并建立安全连接。

信任库可以包含一个认证中心(CA ),它可以对证书进行签名。在这种情况下，**代理或客户信任信任库中存在的由 CA 签署的任何证书**。这简化了证书认证，因为添加新的客户端或代理不需要更改信任库。

## 3.依赖性和设置

我们的示例应用程序将是一个简单的 Spring Boot 应用程序。

为了连接到卡夫卡，让我们在 POM 文件中添加 [`spring-kafka`](https://web.archive.org/web/20220625080310/https://search.maven.org/classic/#search%7Cga%7C1%7Cg%3A%22org.springframework.kafka%22%20AND%20a%3A%22spring-kafka%22) 依赖:

```java
<dependency>
    <groupId>org.springframework.kafka</groupId>
    <artifactId>spring-kafka</artifactId>
    <version>2.7.2</version>
</dependency>
```

我们还将使用一个 [Docker Compose](/web/20220625080310/https://www.baeldung.com/ops/docker-compose) 文件来配置和测试 Kafka 服务器设置。最初，让我们在没有任何 SSL 配置的情况下这样做:

```java
---
version: '2'
services:
  zookeeper:
    image: confluentinc/cp-zookeeper:6.2.0
    environment:
      ZOOKEEPER_CLIENT_PORT: 2181
      ZOOKEEPER_TICK_TIME: 2000

  kafka:
    image: confluentinc/cp-kafka:6.2.0
    depends_on:
      - zookeeper
    ports:
      - 9092:9092
    environment:
      KAFKA_BROKER_ID: 1
      KAFKA_ZOOKEEPER_CONNECT: zookeeper:2181
      KAFKA_ADVERTISED_LISTENERS: PLAINTEXT://localhost:9092
      KAFKA_AUTO_CREATE_TOPICS_ENABLE: 'true'
      KAFKA_OFFSETS_TOPIC_REPLICATION_FACTOR: 1 
```

现在，让我们启动容器:

```java
docker-compose up
```

这应该会显示带有默认配置的代理。

## 4.代理配置

让我们首先来看看为了建立安全连接，代理所需的最低配置。

### 4.1.独立经纪人

虽然我们在这个例子中没有使用代理的独立实例，但是了解启用 SSL 身份验证所需的配置更改是很有用的。

首先，我们需要**配置代理在端口 9093 上监听 SSL 连接**，在`server.properties`:

```java
listeners=PLAINTEXT://kafka1:9092,SSL://kafka1:9093
advertised.listeners=PLAINTEXT://localhost:9092,SSL://localhost:9093
```

接下来，**需要使用证书位置和凭证配置与密钥库和信任库相关的属性**:

```java
ssl.keystore.location=/certs/kafka.server.keystore.jks
ssl.keystore.password=password
ssl.truststore.location=/certs/kafka.server.truststore.jks
ssl.truststore.password=password
ssl.key.password=password
```

最后，**代理必须配置为对客户端**进行身份验证，以实现双向身份验证:

```java
ssl.client.auth=required
```

### 4.2 .复合坞站

因为我们使用 Compose 来管理我们的代理环境，所以让我们将上述所有属性添加到我们的`docker-compose.yml`文件中:

```java
kafka:
  image: confluentinc/cp-kafka:6.2.0
  depends_on:
    - zookeeper
  ports:
    - 9092:9092
    - 9093:9093
  environment:
    ...
    KAFKA_ADVERTISED_LISTENERS: PLAINTEXT://localhost:9092,SSL://localhost:9093
    KAFKA_SSL_CLIENT_AUTH: 'required'
    KAFKA_SSL_KEYSTORE_FILENAME: '/certs/kafka.server.keystore.jks'
    KAFKA_SSL_KEYSTORE_CREDENTIALS: '/certs/kafka_keystore_credentials'
    KAFKA_SSL_KEY_CREDENTIALS: '/certs/kafka_sslkey_credentials'
    KAFKA_SSL_TRUSTSTORE_FILENAME: '/certs/kafka.server.truststore.jks'
    KAFKA_SSL_TRUSTSTORE_CREDENTIALS: '/certs/kafka_truststore_credentials'
  volumes:
    - ./certs/:/etc/kafka/secrets/certs
```

这里，我们已经在配置的`ports`部分公开了 SSL 端口(9093)。此外，我们已经在配置文件的`volumes`部分安装了`certs`项目文件夹。这包含所需的证书和相关凭据。

现在，使用 Compose 重新启动堆栈会在代理日志中显示相关的 SSL 细节:

```java
...
kafka_1      | uid=1000(appuser) gid=1000(appuser) groups=1000(appuser)
kafka_1      | ===> Configuring ...
<strong>kafka_1      | SSL is enabled.</strong>
....
kafka_1      | [2021-08-20 22:45:10,772] INFO KafkaConfig values:
<strong>kafka_1      |  advertised.listeners = PLAINTEXT://localhost:9092,SSL://localhost:9093
kafka_1      |  ssl.client.auth = required</strong>
<strong>kafka_1      |  ssl.enabled.protocols = [TLSv1.2, TLSv1.3]</strong>
kafka_1      |  ssl.endpoint.identification.algorithm = https
kafka_1      |  ssl.key.password = [hidden]
kafka_1      |  ssl.keymanager.algorithm = SunX509
<strong>kafka_1      |  ssl.keystore.location = /etc/kafka/secrets/certs/kafka.server.keystore.jks</strong>
kafka_1      |  ssl.keystore.password = [hidden]
kafka_1      |  ssl.keystore.type = JKS
kafka_1      |  ssl.principal.mapping.rules = DEFAULT
<strong>kafka_1      |  ssl.protocol = TLSv1.3</strong>
kafka_1      |  ssl.trustmanager.algorithm = PKIX
kafka_1      |  ssl.truststore.certificates = null
<strong>kafka_1      |  ssl.truststore.location = /etc/kafka/secrets/certs/kafka.server.truststore.jks</strong>
kafka_1      |  ssl.truststore.password = [hidden]
kafka_1      |  ssl.truststore.type = JKS
....
```

## 5.Spring Boot 客户

现在服务器设置已经完成，我们将创建所需的 Spring Boot 组件。这些将与我们的代理交互，代理现在需要 SSL 进行双向身份验证。

### 5.1.生产者

首先，让我们使用 [`KafkaTemplate`](https://web.archive.org/web/20220625080310/https://docs.spring.io/spring-kafka/api/org/springframework/kafka/core/KafkaTemplate.html) 向指定主题发送消息:

```java
public class KafkaProducer {

    private final KafkaTemplate<String, String> kafkaTemplate;

    public void sendMessage(String message, String topic) {
        log.info("Producing message: {}", message);
        kafkaTemplate.send(topic, "key", message)
          .addCallback(
            result -> log.info("Message sent to topic: {}", message),
            ex -> log.error("Failed to send message", ex)
          );
    }
}
```

`send`方法是异步操作。因此，我们附加了一个简单的回调，它只在代理收到消息时记录一些信息。

### 5.2.消费者

接下来，让我们使用 [@KafkaListener](https://web.archive.org/web/20220625080310/https://docs.spring.io/spring-kafka/api/org/springframework/kafka/annotation/KafkaListener.html) 创建一个简单的消费者。这将连接到代理，并使用来自生产者使用的相同主题的消息:

```java
public class KafkaConsumer {

    public static final String TOPIC = "test-topic";

    public final List<String> messages = new ArrayList<>();

    @KafkaListener(topics = TOPIC)
    public void receive(ConsumerRecord<String, String> consumerRecord) {
        log.info("Received payload: '{}'", consumerRecord.toString());
        messages.add(consumerRecord.value());
    }
}
```

在我们的演示应用程序中，我们保持事情简单，消费者简单地将消息存储在一个`List`中。在实际的系统中，消费者接收消息并根据应用程序的业务逻辑处理它们。

### 5.3.配置

最后，让我们为我们的`application.yml`添加必要的配置:

```java
spring:
  kafka:
    security:
      protocol: "SSL"
    bootstrap-servers: localhost:9093
    ssl:
      trust-store-location: classpath:/client-certs/kafka.client.truststore.jks
      trust-store-password: <password>
      key-store-location:  classpath:/client-certs/kafka.client.keystore.jks
      key-store-password: <password>

    # additional config for producer/consumer 
```

这里，我们已经设置了由 Spring Boot 提供的必需属性来配置生产者和消费者。由于这两个组件都连接到同一个代理，我们可以在`spring.kafka`部分中声明所有的基本属性。然而，如果生产者和消费者连接到不同的代理，我们将分别在`spring.kafka.producer`和`spring.kafka.consumer`部分指定这些。

在配置的`ssl`部分，我们**指向 JKS 信任商店，以便认证 Kafka 经纪人**。这包含 CA 的证书，该 CA 也签署了代理证书。此外，我们还**提供了 Spring 客户端密钥库的路径，该密钥库包含由 CA** 签署的证书，该证书应该存在于代理端的信任库中。

### 5.4.测试

因为我们正在使用一个组合文件，所以让我们使用 [Testcontainers](/web/20220625080310/https://www.baeldung.com/spring-boot-kafka-testing#testing-kafka-with-testcontainers) 框架，用我们的`Producer`和`Consumer`创建一个端到端的测试:

```java
@ActiveProfiles("ssl")
@Testcontainers
@SpringBootTest(classes = KafkaSslApplication.class)
class KafkaSslApplicationLiveTest {

    private static final String KAFKA_SERVICE = "kafka";
    private static final int SSL_PORT = 9093;  

    @Container
    public DockerComposeContainer<?> container =
      new DockerComposeContainer<>(KAFKA_COMPOSE_FILE)
        .withExposedService(KAFKA_SERVICE, SSL_PORT, Wait.forListeningPort());

    @Autowired
    private KafkaProducer kafkaProducer;

    @Autowired
    private KafkaConsumer kafkaConsumer;

    @Test
    void givenSslIsConfigured_whenProducerSendsMessageOverSsl_thenConsumerReceivesOverSsl() {
        String message = generateSampleMessage();
        kafkaProducer.sendMessage(message, TOPIC);

        await().atMost(Duration.ofMinutes(2))
          .untilAsserted(() -> assertThat(kafkaConsumer.messages).containsExactly(message));
    }

    private static String generateSampleMessage() {
        return UUID.randomUUID().toString();
    }
}
```

当我们运行测试时，Testcontainers 使用我们的 Compose 文件启动 Kafka 代理，包括 SSL 配置。应用程序也从它的 SSL 配置开始，并通过加密和认证的连接连接到代理。由于这是一个异步的事件序列，我们使用了 [Awaitlity](/web/20220625080310/https://www.baeldung.com/awaitlity-testing) 来轮询消费者消息存储库中的预期消息。这将验证所有配置以及代理和客户端之间的成功双向身份验证。

## 6.结论

在本文中，我们介绍了 Kafka 代理和 Spring Boot 客户端之间所需的 SSL 身份验证设置的基础知识。

最初，我们查看了启用双向身份验证所需的代理设置。然后，我们查看了客户端所需的配置，以便通过加密和认证的连接连接到代理。最后，我们使用集成测试来验证代理和客户机之间的安全连接。

和往常一样，完整的源代码可以在 GitHub 上找到[。](https://web.archive.org/web/20220625080310/https://github.com/eugenp/tutorials/tree/master/spring-kafka)