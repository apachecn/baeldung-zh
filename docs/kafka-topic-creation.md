# 使用 Java 创建 Kafka 主题

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/kafka-topic-creation>

## 1.概观

在本教程中，我们将简要介绍 [Apache Kafka](https://web.archive.org/web/20220628115935/https://kafka.apache.org/) ，然后看看如何在 Kafka 集群中以编程方式创建和配置主题。

## 2.卡夫卡简介

Apache Kafka 是一个强大的高性能分布式事件流平台。

通常，生产者应用程序将事件发布到 Kafka，而消费者订阅这些事件以便读取和处理它们。 **Kafka 使用** **主题来存储和分类这些事件，**例如，在一个电子商务应用程序中，可能有一个“订单”主题。

Kafka 主题是分区的，它将数据分布在多个代理上以实现可伸缩性。它们可以被复制，以便使数据具有容错性和高可用性。即使在消费之后，主题也可以根据需要保留事件。这都是通过 Kafka 命令行工具和键值配置在每个主题的基础上管理的**。**

然而，除了命令行工具， **Kafka 还提供了一个[管理 API](https://web.archive.org/web/20220628115935/https://kafka.apache.org/28/javadoc/org/apache/kafka/clients/admin/Admin.html) 来管理和检查主题、代理和其他 Kafka 对象**。在我们的例子中，我们将使用这个 API 来创建新的主题。

## 3.属国

要使用管理 API，让我们将[Kafka-clients dependency](https://web.archive.org/web/20220628115935/https://search.maven.org/artifact/org.apache.kafka/kafka-clients)y 添加到我们的`pom.xml`:

```java
<dependency>
    <groupId>org.apache.kafka</groupId>
    <artifactId>kafka-clients</artifactId>
    <version>2.8.0</version>
</dependency>
```

## 4.建立卡夫卡

在创建新主题之前，我们至少需要一个单节点 Kafka 集群。

在本教程中，我们将使用 [Testcontainers](https://web.archive.org/web/20220628115935/https://www.testcontainers.org/) 框架来实例化一个 Kafka 容器。然后，我们可以运行可靠且独立的集成测试，而不依赖于外部 Kafka 服务器的运行。为此，我们还需要两个专门用于测试的依赖项。

首先，让我们将 [Testcontainers Kafka 依赖项](https://web.archive.org/web/20220628115935/https://search.maven.org/artifact/org.testcontainers/kafka)添加到我们的`pom.xml`:

```java
<dependency>
    <groupId>org.testcontainers</groupId>
    <artifactId>kafka</artifactId>
    <version>1.15.3</version>
    <scope>test</scope>
</dependency>
```

接下来，我们将添加用于使用 JUnit 5 运行 Testcontainer 测试的 [junit-jupiter 工件](https://web.archive.org/web/20220628115935/https://search.maven.org/search?q=g:org.testcontainers%20AND%20a:junit-jupiter):

```java
<dependency>
    <groupId>org.testcontainers</groupId>
    <artifactId>junit-jupiter</artifactId>
    <version>1.15.3</version>
    <scope>test</scope>
</dependency>
```

现在我们已经配置了所有必需的依赖项，我们可以编写一个简单的应用程序来以编程方式创建新主题。

## 5\. Admin API

让我们首先为本地代理创建一个新的 [`Properties`](/web/20220628115935/https://www.baeldung.com/java-properties) 实例，并对其进行最少的配置:

```java
Properties properties = new Properties();
properties.put(
  AdminClientConfig.BOOTSTRAP_SERVERS_CONFIG, KAFKA_CONTAINER.getBootstrapServers()
);
```

现在我们可以获得一个`Admin`实例:

```java
Admin admin = Admin.create(properties)
```

`create`方法接受一个`Properties`对象(或一个带有 [`bootstrap.servers`](https://web.archive.org/web/20220628115935/https://kafka.apache.org/documentation/#adminclientconfigs_bootstrap.servers) 属性的`Map)`)并返回一个线程安全的实例。

管理客户端使用该属性来发现集群中的代理，并随后执行任何管理操作。因此，通常包含两三个代理地址就足够了，以应对某些实例不可用的可能性。

[`AdminClientConfig`](https://web.archive.org/web/20220628115935/https://kafka.apache.org/28/javadoc/org/apache/kafka/clients/admin/AdminClientConfig.html) 类包含所有[管理客户端配置](https://web.archive.org/web/20220628115935/https://kafka.apache.org/documentation/#adminclientconfigs)条目的常量。

## 6.话题创作

让我们首先用 Testcontainers 创建一个 [JUnit 5 测试来验证成功的主题创建。我们将利用](https://web.archive.org/web/20220628115935/https://www.testcontainers.org/quickstart/junit_5_quickstart/) [Kafka 模块](https://web.archive.org/web/20220628115935/https://www.testcontainers.org/modules/kafka/)，它使用官方 Kafka Docker 镜像用于[融合 OSS 平台](https://web.archive.org/web/20220628115935/https://hub.docker.com/r/confluentinc/cp-kafka/):

```java
@Test
void givenTopicName_whenCreateNewTopic_thenTopicIsCreated() throws Exception {
    kafkaTopicApplication.createTopic("test-topic");

    String topicCommand = "/usr/bin/kafka-topics --bootstrap-server=localhost:9092 --list";
    String stdout = KAFKA_CONTAINER.execInContainer("/bin/sh", "-c", topicCommand)
      .getStdout();

    assertThat(stdout).contains("test-topic");
}
```

在这里，Testcontainers 将在测试执行期间自动实例化和管理 Kafka 容器。我们只需调用我们的应用程序代码，并验证主题是否已在运行的容器中成功创建。

### 6.1.使用默认选项创建

[主题分区和复制因子](/web/20220628115935/https://www.baeldung.com/apache-kafka-data-modeling)是新主题的关键考虑因素。我们将保持简单，用 1 个分区和 1 的复制因子创建我们的示例主题:

```java
try (Admin admin = Admin.create(properties)) {
    int partitions = 1;
    short replicationFactor = 1;
    NewTopic newTopic = new NewTopic(topicName, partitions, replicationFactor);

    CreateTopicsResult result = admin.createTopics(
      Collections.singleton(newTopic)
    );

    KafkaFuture<Void> future = result.values().get(topicName);
    future.get();
}
```

这里，我们使用了**管理。`createTopics`用默认选项创建一批新主题的方法。**由于`Admin`接口扩展了`AutoCloseable`接口，我们使用了 [try-with-resources](/web/20220628115935/https://www.baeldung.com/java-try-with-resources) 来执行我们的操作。这确保了资源被适当地释放。

重要的是，该方法与控制器代理通信并异步执行。返回的`CreateTopicsResult`对象公开了一个`KafkaFuture`,用于访问请求批次中每个项目的结果。这遵循 Java 异步编程模式，并允许调用者使用 [`Future.get`](https://web.archive.org/web/20220628115935/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/concurrent/Future.html#get()) 方法获得操作结果。

对于同步行为，我们可以立即调用这个方法来检索操作的结果。这将一直阻止，直到操作完成或失败。在失败的情况下，它会导致一个包装了潜在原因的 ExecutionException。

### 6.2.用选项创建

除了默认选项，我们还可以使用 Admin 的**重载形式。`createTopics`方法并通过 [`CreateTopicsOptions`](https://web.archive.org/web/20220628115935/https://kafka.apache.org/28/javadoc/org/apache/kafka/clients/admin/CreateTopicsOptions.html) 对象**提供一些 **选项。我们可以使用这些来修改创建新主题时的管理客户端行为:**

```java
CreateTopicsOptions topicOptions = new CreateTopicsOptions()
  .validateOnly(true)
  .retryOnQuotaViolation(false);

CreateTopicsResult result = admin.createTopics(
  Collections.singleton(newTopic), topicOptions
);
```

这里，我们已经将`validateOnly`选项设置为 true，这意味着客户端将只进行验证，而不会实际创建主题。类似地，`retryOnQuotaViolation`选项被设置为 false，以便在违反配额的情况下不会重试该操作。

### 6.3.新主题配置

Kafka 有各种各样的[主题配置](https://web.archive.org/web/20220628115935/https://kafka.apache.org/documentation.html#topicconfigs)来控制主题行为，比如[数据保持](/web/20220628115935/https://www.baeldung.com/kafka-message-retention)和压缩等。这些既有服务器默认值，也有可选的每个主题的覆盖。

我们可以通过使用新主题的配置图来**提供主题配置:**

```java
// Create a compacted topic with 'lz4' compression codec
Map<String, String> newTopicConfig = new HashMap<>();
newTopicConfig.put(TopicConfig.CLEANUP_POLICY_CONFIG, TopicConfig.CLEANUP_POLICY_COMPACT);
newTopicConfig.put(TopicConfig.COMPRESSION_TYPE_CONFIG, "lz4");

NewTopic newTopic = new NewTopic(topicName, partitions, replicationFactor)
  .configs(newTopicConfig);
```

来自管理 API 的 [`TopicConfig`](https://web.archive.org/web/20220628115935/https://kafka.apache.org/28/javadoc/org/apache/kafka/common/config/TopicConfig.html) 类包含可用于在创建时配置主题的键。

## 7.其他主题操作

除了创建新主题的能力， **Admin API 还具有删除、[列出](https://web.archive.org/web/20220628115935/https://kafka.apache.org/28/javadoc/org/apache/kafka/clients/admin/Admin.html#listTopics())、[描述](https://web.archive.org/web/20220628115935/https://kafka.apache.org/28/javadoc/org/apache/kafka/clients/admin/Admin.html#describeTopics(java.util.Collection))主题**的操作。所有这些与主题相关的操作都遵循与我们看到的主题创建相同的模式。

这些操作方法中的每一个都有一个重载版本，它接受一个`xxxTopicOptions`对象作为输入。所有这些方法都返回相应的`xxxTopicsResult`对象。这反过来为访问异步操作的结果提供了`KafkaFuture`。

最后，值得一提的是，自从在 Kafka 版本 0.11.0.0 中引入以来，admin API 仍在不断发展，正如`InterfaceStability.Evolving`注释所示。这意味着 API 将来可能会改变，一个小版本可能会破坏兼容性。

## 8.结论

在本教程中，我们看到了如何使用 Java 管理客户端在 Kafka 中创建新主题。

最初，我们用默认选项创建了一个主题，然后用显式选项创建。在此基础上，我们看到了如何使用各种属性来配置新主题。最后，我们简要介绍了使用管理客户机的其他与主题相关的操作。

在这个过程中，我们还看到了如何使用 Testcontainers 从我们的测试中建立一个简单的单节点集群。

和往常一样，GitHub 上的[提供了这篇文章的完整源代码。](https://web.archive.org/web/20220628115935/https://github.com/eugenp/tutorials/tree/master/apache-kafka)