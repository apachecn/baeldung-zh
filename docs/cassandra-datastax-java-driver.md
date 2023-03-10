# Apache Cassandra 的 DataStax Java 驱动程序简介

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/cassandra-datastax-java-driver>

## 1.概观

Apache Cassandra 的 DataStax 发行版是一个生产就绪的分布式数据库，与开源的 Cassandra 兼容。它增加了一些开源发行版中没有的特性，包括监控、改进的批处理和流数据处理。

DataStax 还为其 Apache Cassandra 发行版提供了一个 Java 客户端。这个驱动程序是高度可调的，可以利用 DataStax 发行版中的所有额外功能，而且它也完全兼容开源版本。

在本教程中，我们将看到**如何使用用于 Apache Cassandra** 的 [DataStax Java 驱动程序连接到 Cassandra 数据库并执行基本的数据操作。](https://web.archive.org/web/20221129021453/https://github.com/datastax/java-driver)

 [## 延伸阅读:](https://web.archive.org/web/20221129021453/https://github.com/datastax/java-driver) [](https://web.archive.org/web/20221129021453/https://github.com/datastax/java-driver)[](https://web.archive.org/web/20221129021453/https://github.com/datastax/java-driver)

## [](https://web.archive.org/web/20221129021453/https://github.com/datastax/java-driver)[使用 Cassandra、Astra 和 Stargate 构建仪表板](/web/20221129021453/https://www.baeldung.com/cassandra-astra-stargate-dashboard)

了解如何使用 DataStax Astra 构建仪表板，这是一种由 Apache Cassandra 和 Stargate APIs 支持的数据库即服务。[阅读更多](/web/20221129021453/https://www.baeldung.com/cassandra-astra-stargate-dashboard) →

## [用 Cassandra、Astra、REST&graph QL——记录状态更新](/web/20221129021453/https://www.baeldung.com/cassandra-astra-rest-dashboard-updates)

用 Cassandra 存储时序数据的例子。[阅读更多信息](/web/20221129021453/https://www.baeldung.com/cassandra-astra-rest-dashboard-updates) →

## [使用 Cassandra、Astra 和 CQL 构建一个仪表盘——绘制事件数据](/web/20221129021453/https://www.baeldung.com/cassandra-astra-rest-dashboard-map)

了解如何根据 Astra 数据库中存储的数据在交互式地图上显示事件。[阅读更多](/web/20221129021453/https://www.baeldung.com/cassandra-astra-rest-dashboard-map) →

## 2.Maven 依赖性

为了使用 Apache Cassandra 的 DataStax Java 驱动程序，我们需要将它包含在我们的类路径中。

使用 Maven，我们只需将 [`java-driver-core`依赖项](https://web.archive.org/web/20221129021453/https://search.maven.org/search?q=g:com.datastax.oss%20a:java-driver-core)添加到我们的`pom.xml`中:

```java
<dependency>
    <groupId>com.datastax.oss</groupId>
    <artifactId>java-driver-core</artifactId>
    <version>4.1.0</version>
</dependency>

<dependency>
    <groupId>com.datastax.oss</groupId>
    <artifactId>java-driver-query-builder</artifactId>
    <version>4.1.0</version>
</dependency>
```

## 3.使用数据税驱动程序

现在我们已经有了驱动程序，让我们看看我们能做些什么。

### 3.1.连接到数据库

为了连接到数据库，我们将创建一个`CqlSession`:

```java
CqlSession session = CqlSession.builder().build();
```

如果我们没有明确定义任何接触点，构建器将默认为`127.0.0.1:9042`。

让我们创建一个连接器类，用一些可配置的参数来构建`CqlSession`:

```java
public class CassandraConnector {

    private CqlSession session;

    public void connect(String node, Integer port, String dataCenter) {
        CqlSessionBuilder builder = CqlSession.builder();
        builder.addContactPoint(new InetSocketAddress(node, port));
        builder.withLocalDatacenter(dataCenter);

        session = builder.build();
    }

    public CqlSession getSession() {
        return this.session;
    }

    public void close() {
        session.close();
    }
}
```

### 3.2.创建密钥空间

现在我们已经连接到数据库，我们需要创建我们的键空间。让我们从编写一个简单的知识库类来使用我们的密钥空间开始。

对于本教程，**我们将使用`SimpleStrategy`复制策略，其中副本数量设置为 1** :

```java
public class KeyspaceRepository {

    public void createKeyspace(String keyspaceName, int numberOfReplicas) {
        CreateKeyspace createKeyspace = SchemaBuilder.createKeyspace(keyspaceName)
          .ifNotExists()
          .withSimpleStrategy(numberOfReplicas);

        session.execute(createKeyspace.build());
    }

    // ...
}
```

同样，我们可以**在当前会话**中开始使用密钥空间:

```java
public class KeyspaceRepository {

    //...

    public void useKeyspace(String keyspace) {
        session.execute("USE " + CqlIdentifier.fromCql(keyspace));
    }
}
```

### 3.3.创建表格

驱动程序提供语句来配置和执行数据库中的查询。例如，**我们可以设置在每个语句**中单独使用的密钥空间。

我们将定义`Video`模型并创建一个表来表示它:

```java
public class Video {
    private UUID id;
    private String title;
    private Instant creationDate;

    // standard getters and setters
}
```

让我们创建我们的表，有可能定义我们想要在其中执行查询的键空间。我们将编写一个简单的`VideoRepository`类来处理我们的视频数据:

```java
public class VideoRepository {
    private static final String TABLE_NAME = "videos";

    public void createTable() {
        createTable(null);
    }

    public void createTable(String keyspace) {
        CreateTable createTable = SchemaBuilder.createTable(TABLE_NAME)
          .withPartitionKey("video_id", DataTypes.UUID)
          .withColumn("title", DataTypes.TEXT)
          .withColumn("creation_date", DataTypes.TIMESTAMP);

        executeStatement(createTable.build(), keyspace);
    }

    private ResultSet executeStatement(SimpleStatement statement, String keyspace) {
        if (keyspace != null) {
            statement.setKeyspace(CqlIdentifier.fromCql(keyspace));
        }

        return session.execute(statement);
    }

    // ...
}
```

注意，我们重载了方法`createTable`。

重载该方法的想法是为表的创建提供两种选择:

*   在特定的键空间中创建表，将键空间名称作为参数发送，与会话当前使用的键空间无关
*   开始在会话中使用一个键空间，并使用不带任何参数的方法创建表，在这种情况下，将在会话当前使用的键空间中创建表

### 3.4.插入数据

此外，驱动程序还提供了准备好的有界语句。

**`PreparedStatement `通常用于经常执行的查询，只改变值。**

我们可以用我们需要的值填充`PreparedStatement`。之后，我们将创建一个`BoundStatement`并执行它。

让我们编写一个将一些数据插入数据库的方法:

```java
public class VideoRepository {

    //...

    public UUID insertVideo(Video video, String keyspace) {
        UUID videoId = UUID.randomUUID();

        video.setId(videoId);

        RegularInsert insertInto = QueryBuilder.insertInto(TABLE_NAME)
          .value("video_id", QueryBuilder.bindMarker())
          .value("title", QueryBuilder.bindMarker())
          .value("creation_date", QueryBuilder.bindMarker());

        SimpleStatement insertStatement = insertInto.build();

        if (keyspace != null) {
            insertStatement = insertStatement.setKeyspace(keyspace);
        }

        PreparedStatement preparedStatement = session.prepare(insertStatement);

        BoundStatement statement = preparedStatement.bind()
          .setUuid(0, video.getId())
          .setString(1, video.getTitle())
          .setInstant(2, video.getCreationDate());

        session.execute(statement);

        return videoId;
    }

    // ...
}
```

### 3.5.查询数据

现在，让我们添加一个方法，该方法创建一个简单的查询来获取我们存储在数据库中的数据:

```java
public class VideoRepository {

    // ...

    public List<Video> selectAll(String keyspace) {
        Select select = QueryBuilder.selectFrom(TABLE_NAME).all();

        ResultSet resultSet = executeStatement(select.build(), keyspace);

        List<Video> result = new ArrayList<>();

        resultSet.forEach(x -> result.add(
            new Video(x.getUuid("video_id"), x.getString("title"), x.getInstant("creation_date"))
        ));

        return result;
    }

    // ...
}
```

### 3.6.把所有的放在一起

最后，让我们来看一个例子，它使用了我们在本教程中讨论过的每个部分:

```java
public class Application {

    public void run() {
        CassandraConnector connector = new CassandraConnector();
        connector.connect("127.0.0.1", 9042, "datacenter1");
        CqlSession session = connector.getSession();

        KeyspaceRepository keyspaceRepository = new KeyspaceRepository(session);

        keyspaceRepository.createKeyspace("testKeyspace", 1);
        keyspaceRepository.useKeyspace("testKeyspace");

        VideoRepository videoRepository = new VideoRepository(session);

        videoRepository.createTable();

        videoRepository.insertVideo(new Video("Video Title 1", Instant.now()));
        videoRepository.insertVideo(new Video("Video Title 2",
          Instant.now().minus(1, ChronoUnit.DAYS)));

        List<Video> videos = videoRepository.selectAll();

        videos.forEach(x -> LOG.info(x.toString()));

        connector.close();
    }
}
```

在我们执行我们的示例之后，我们可以在日志中看到数据被正确地存储在数据库中:

```java
INFO com.baeldung.datastax.cassandra.Application - [id:733249eb-914c-4153-8698-4f58992c4ad4, title:Video Title 1, creationDate: 2019-07-10T19:43:35.112Z]
INFO com.baeldung.datastax.cassandra.Application - [id:a6568236-77d7-42f2-a35a-b4c79afabccf, title:Video Title 2, creationDate: 2019-07-09T19:43:35.181Z]
```

## 4.结论

在本教程中，我们介绍了 Apache Cassandra 的 DataStax Java 驱动程序的基本概念。我们连接到数据库并创建了一个键空间和表。此外，我们将数据插入到表中，并运行一个查询来检索它。

和往常一样，本教程的源代码可以在 Github 上的[处获得。](https://web.archive.org/web/20221129021453/https://github.com/eugenp/tutorials/tree/master/persistence-modules/java-cassandra)