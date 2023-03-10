# ksqlDB 简介

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/ksqldb>

## 1.介绍

[ksqlDB](https://web.archive.org/web/20220628055254/https://ksqldb.io/) 可以被描述为建立在[阿帕奇卡夫卡](/web/20220628055254/https://www.baeldung.com/apache-kafka-data-modeling)和[卡夫卡流](/web/20220628055254/https://www.baeldung.com/java-kafka-streams)之上的实时事件流数据库。它结合了强大的流处理和使用 SQL 语法的关系数据库模型。

在本教程中，我们将介绍 ksqlDB 的基本概念，并构建一个示例应用程序来演示一个实际用例。

## 2.概观

由于 ksqlDB 是一个事件流数据库，流和表是它的核心抽象。本质上，这些是可以实时转换和处理的数据集合。

流处理支持在这些无限的事件流上进行连续计算。我们可以使用 SQL 对集合进行转换、过滤、聚合和连接，从而派生出新的集合或物化视图。此外，新事件不断更新这些集合和视图，以提供实时数据。

最后，查询发布各种流处理操作的结果。 **ksqlDB 查询支持异步实时应用程序流和同步请求/响应流，类似于传统数据库**。

## 3.设置

为了查看 ksqlDB 的运行情况，我们将构建一个事件驱动的 Java 应用程序。这将聚集和查询来自各种传感器源的无限读数流。

主要用例是检测特定时间段内读数平均值超过指定阈值的情况。此外，一个关键要求是应用程序必须提供实时信息，例如，在构建仪表板或警告系统时可以使用这些信息。

我们将使用 ksqlDB Java 客户端与服务器进行交互，以便创建表、聚合查询和执行各种查询。

### 3.1.码头工人

由于 ksqlDB 运行在 Kafka 之上，我们将使用 Docker Compose 来运行 Kafka 组件、ksqlDB 服务器和 ksqlDB CLI 客户端:

```java
services:
  zookeeper:
    image: confluentinc/cp-zookeeper:6.2.0
    hostname: zookeeper
    ...

  broker:
    image: confluentinc/cp-kafka:6.2.0
    hostname: broker
    ...

  ksqldb-server:
    image: confluentinc/ksqldb-server:0.19.0
    hostname: ksqldb-server
    depends_on:
      - broker
    ports:
      - "8088:8088"
    healthcheck:
      test: curl -f http://ksqldb-server:8088/ || exit 1
    environment:
      KSQL_LISTENERS: http://0.0.0.0:8088
      KSQL_BOOTSTRAP_SERVERS: broker:9092
      KSQL_KSQL_LOGGING_PROCESSING_STREAM_AUTO_CREATE: "true"
      KSQL_KSQL_LOGGING_PROCESSING_TOPIC_AUTO_CREATE: "true"

  ksqldb-cli:
    image: confluentinc/ksqldb-cli:0.19.0
    container_name: ksqldb-cli
    depends_on:
      - broker
      - ksqldb-server
    entrypoint: /bin/sh
    tty: true
```

此外，我们还将在 Java 应用程序中使用这个`docker-compose.yml`文件，为我们使用 [Testcontainers](/web/20220628055254/https://www.baeldung.com/docker-test-containers) 框架的集成测试构建一个环境。

首先，让我们运行以下命令来打开堆栈:

```java
docker-compose up
```

接下来，在所有服务启动后，让我们连接到交互式 CLI。这对测试和与服务器交互很有用:

```java
docker exec -it ksqldb-cli ksql http://ksqldb-server:8088
```

我们还将告诉 ksqlDB 从每个主题中最早的点开始所有查询:

```java
ksql> SET 'auto.offset.reset' = 'earliest';
```

### 3.2.属国

在这个项目中，我们将主要使用 Java 客户端与 ksqlDB 进行交互。更具体地说，我们将把 ksqlDB 用于汇合平台(CP ),所以我们需要将 [CP Maven 库](https://web.archive.org/web/20220628055254/http://packages.confluent.io/maven/)添加到我们的 POM 文件中:

```java
<repository>
    <id>confluent</id>
    <name>confluent-repo</name>
    <url>http://packages.confluent.io/maven/</url>
</repository>
```

现在，让我们为客户端添加[依赖关系](https://web.archive.org/web/20220628055254/http://packages.confluent.io/maven/io/confluent/ksql/ksqldb-api-client/6.2.0/):

```java
<dependency>
    <groupId>io.confluent.ksql</groupId>
    <artifactId>ksqldb-api-client</artifactId>
    <version>6.2.0</version>
</dependency>
```

## 4.实时数据聚合

在这一节中，我们将看到如何创建一个物化视图来表示我们的应用程序所需的实时聚合。

### 4.1.创建流

在卡夫卡那里，一个主题存储了事件的集合。类似地，**在 ksqkDB 中，流代表事件，由 Kafka 主题**支持。

让我们从创建存储传入传感器数据的流开始:

```java
CREATE STREAM readings (sensor_id VARCHAR KEY, timestamp VARCHAR, reading INT)
  WITH (KAFKA_TOPIC = 'readings',
        VALUE_FORMAT = 'JSON',
        TIMESTAMP = 'timestamp',
        TIMESTAMP_FORMAT = 'yyyy-MM-dd HH:mm:ss',
        PARTITIONS = 1);
```

这里，ksqlDB 创建了`readings`主题，以 JSON 格式存储流数据。因为事件代表时间数据，所以每个读数包含一个指示事件时间的时间戳是很重要的。`timestamp`字段以指定的格式存储这些数据。这确保了 ksqlDB 对与时间相关的操作和无序事件应用事件时间语义。

接下来，我们将使用 ksqlDB 服务器连接详细信息创建一个 [`Client`](https://web.archive.org/web/20220628055254/https://docs.ksqldb.io/en/latest/developer-guide/ksqldb-clients/java-client/) 实例，并使用它来执行我们的 SQL 语句:

```java
ClientOptions options = ClientOptions.create()
  .setHost(KSQLDB_SERVER_HOST)
  .setPort(KSQLDB_SERVER_PORT);

Client client = Client.create(options);

Map<String, Object> properties = Collections.singletonMap(
  "auto.offset.reset", "earliest"
);

CompletableFuture<ExecuteStatementResult> result = 
  client.executeStatement(CREATE_READINGS_STREAM, properties); 
```

与前面的 CLI 一样，我们将`auto.offset.reset`属性的值设置为“`earliest`”。这确保了在没有 Kafka 偏移量的情况下，查询从最早的偏移量开始读取相关主题。

`executeStatement`方法是客户端提供的异步 API 的一部分。在向服务器发送任何请求之前，它会立即返回一个`CompletableFuture`。然后，调用代码可能决定阻塞并等待完成(通过调用`get`或`join`方法)或执行其他非阻塞操作。

### 4.2.创建实体化视图

现在我们有了底层的事件流，我们可以从`readings`流中派生出一个新的`alerts`表。这个**持久查询(或物化视图)无限期地在服务器上运行，并处理来自源流或表**的事件。

在我们的例子中，当每个传感器的平均读数在 30 分钟内超过 25 时，它应该发出警报:

```java
CREATE TABLE alerts AS
  SELECT
    sensor_id,
    TIMESTAMPTOSTRING(WINDOWSTART, 'yyyy-MM-dd HH:mm:ss', 'UTC') 
      AS start_period,
    TIMESTAMPTOSTRING(WINDOWEND, 'yyyy-MM-dd HH:mm:ss', 'UTC') 
      AS end_period,
    AVG(reading) AS average_reading
  FROM readings
  WINDOW TUMBLING (SIZE 30 MINUTES)
  GROUP BY id 
  HAVING AVG(reading) > 25
  EMIT CHANGES;
```

在这个查询中，我们对每个传感器在 30 分钟的[滚动窗口](https://web.archive.org/web/20220628055254/https://docs.ksqldb.io/en/latest/concepts/time-and-windows-in-ksqldb-queries/#tumbling-window)中聚合新的传入事件。我们还使用了`TIMESTAMPTOSTRING`函数将 UNIX 时间戳转换成可读性更强的内容。

重要的是，**只有当新事件成功地与聚合函数**集成时，物化视图才会更新数据。

和前面一样，让我们使用客户机异步执行这条语句，并创建我们的物化视图:

```java
CompletableFuture<ExecuteStatementResult> result = 
  client.executeStatement(CREATE_ALERTS_TABLE, properties)
```

一旦创建，这样的视图以递增的方式更新。这是高效和高性能实时更新查询的关键。

### 4.3.插入样本数据

在运行查询之前，让我们生成一些以 10 分钟为间隔表示各种读数的示例事件。

让我们使用`KsqlObject`为流列提供键/值映射:

```java
List<KsqlObject> rows = Arrays.asList(
  new KsqlObject().put("sensor_id", "sensor-1")
    .put("timestamp", "2021-08-01 09:00:00").put("reading", 22),
  new KsqlObject().put("sensor_id", "sensor-1")
    .put("timestamp", "2021-08-01 09:10:00").put("reading", 20),
  new KsqlObject().put("sensor_id", "sensor-2")
    .put("timestamp", "2021-08-01 10:00:00").put("reading", 26),

  // additional rows
);

CompletableFuture<Void> result = CompletableFuture.allOf(
  rows.stream()
    .map(row -> client.insertInto(READINGS_TABLE, row))
    .toArray(CompletableFuture[]::new)
);
```

这里，为了方便起见，我们将所有单独的插入操作合并到一个`Future`中。这在所有底层`CompletableFuture`实例成功完成时完成。

## 5.查询数据

查询允许调用者将物化视图数据带入应用程序。这些可以分为两种类型。

### 5.1.推送查询

这种类型的查询向客户端推送连续的更新流。这些查询**特别适合异步应用程序流**，因为它们使客户端能够实时对新信息做出反应。

但是，与持久查询不同，服务器不会将这种查询的结果存储在 Kafka 主题中。因此，我们应该**保持这些查询尽可能简单，同时将所有繁重的工作转移到持久查询**中。

让我们创建一个简单的推送查询来订阅前面创建的`alerts`物化视图的结果:

```java
SELECT * FROM alerts EMIT CHANGES;
```

在这里，重要的是要注意`EMIT`子句，它向客户端发出所有的更改。由于查询不包含任何限制，它将继续传输所有结果，直到终止。

接下来，我们订阅查询结果以接收流数据:

```java
public CompletableFuture<Void> subscribeOnAlerts(Subscriber<Row> subscriber) {
    return client.streamQuery(ALERTS_QUERY, PROPERTIES)
      .thenAccept(streamedQueryResult -> streamedQueryResult.subscribe(subscriber))
      .whenComplete((result, ex) -> {
          if (ex != null) {
              log.error("Alerts push query failed", ex);
          }
      });
}
```

这里，我们调用了`streamQuery`方法，该方法返回一个用于获取流数据的`StreamedQueryResult`。这从[反应流](https://web.archive.org/web/20220628055254/https://www.reactive-streams.org/)扩展了`Publisher`接口。因此，我们能够通过使用反应式 [`Subscriber`](https://web.archive.org/web/20220628055254/https://www.reactive-streams.org/reactive-streams-1.0.3-javadoc/org/reactivestreams/Subscriber.html) 来**异步地消费结果。事实上，订阅者是一个简单的 [Reactive Streams 实现](/web/20220628055254/https://www.baeldung.com/java-9-reactive-streams)，它接收 ksqlDB 行作为 JSON，并将它们转换为`Alert` POJO。**

我们现在可以使用我们的组合文件和来自 [Testcontainers](https://web.archive.org/web/20220628055254/https://www.testcontainers.org/) 的 [`DockerComposeContainer`](https://web.archive.org/web/20220628055254/https://www.testcontainers.org/modules/docker_compose/) 来测试这个:

```java
@Testcontainers
class KsqlDBApplicationLiveTest {

    @Container
    public static DockerComposeContainer dockerComposeContainer =
      new DockerComposeContainer<>(KSQLDB_COMPOSE_FILE)
        .withServices("zookeeper", "broker", "ksqldb-server")
        .withExposedService("ksqldb-server", 8088,
          Wait.forHealthcheck().withStartupTimeout(Duration.ofMinutes(5)))
        .withLocalCompose(true);

    // setup and teardown

    @Test
    void givenSensorReadings_whenSubscribedToAlerts_thenAlertsAreConsumed() {
        createAlertsMaterializedView();

        // Reactive Streams Subscriber impl for receiving streaming data
        RowSubscriber<Alert> alertSubscriber = new RowSubscriber<>(Alert.class);

        ksqlDBApplication.subscribeOnAlerts(alertSubscriber);
        insertSampleData();

        await().atMost(Duration.ofMinutes(3)).untilAsserted(() ->
          assertThat(alertSubscriber.consumedItems)
            .containsOnly(
              expectedAlert("sensor-1", "2021-08-01 09:30:00", "2021-08-01 10:00:00", 28.0),
              expectedAlert("sensor-2", "2021-08-01 10:00:00", "2021-08-01 10:30:00", 26.0)
            )
        );
    }
}
```

这里，我们为集成测试构建了一个完整的 ksqlDB 环境。该测试将样本行插入到流中，ksqlDB 执行窗口聚合。最后，我们断言我们的订户使用了最新的警报，这是意料之中的。

### 5.2.拉查询

与推查询相比，拉查询检索不动态更新的数据，这与传统的 RDBMS 非常相似。这种查询会立即返回一个有限的结果集。因此，**拉查询非常适合同步请求/响应应用程序流**。

举个简单的例子，让我们创建一个查询来检索特定传感器 id 触发的所有警报:

```java
String pullQuery = "SELECT * FROM alerts WHERE sensor_id = 'sensor-2';";

List<Row> rows = client.executeQuery(pullQuery, PROPERTIES).get() 
```

与推送查询相比，该查询在执行时从实体化视图返回所有可用的数据。这对于查询实体化视图的当前状态非常有用。

### 5.3.杂项操作

客户端的 [API 文档](https://web.archive.org/web/20220628055254/https://docs.ksqldb.io/en/latest/developer-guide/ksqldb-clients/java-client/api/io/confluent/ksql/api/client/package-summary.html?_ga=2.213461271.240953982.1628434775-911830345.1628201123)提供了其他操作的进一步信息，如描述来源；列出流、表、主题；终止查询等等。

## 6.结论

在本文中，我们讨论了流、表和查询的核心概念，这些概念支持 ksqlDB 作为一个高效的事件流数据库。

在此过程中，我们使用简洁且可组合的 SQL 结构构建了一个简单的反应式应用程序。我们还了解了如何使用 Java 客户机创建流和表，对物化视图发出查询，以及检索实时数据。

和往常一样，完整的源代码可以在 GitHub 上找到[。](https://web.archive.org/web/20220628055254/https://github.com/eugenp/tutorials/tree/master/ksqldb)