# Spring Data MongoDB 可定制游标

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-data-mongodb-tailable-cursors>

## 1.介绍

在本教程中，我们将讨论如何通过利用带有 [Spring 数据 MongoDB](/web/20220625230955/https://www.baeldung.com/spring-data-mongodb-tutorial) 的可定制游标来将 MongoDB 用作无限数据流。

## 2.可定制光标

当我们执行查询时，数据库驱动程序打开一个游标来提供匹配的文档。默认情况下，当客户端读取所有结果时，MongoDB 会自动关闭游标。因此，旋转会产生有限的数据流。

然而，**我们可以使用带有可定制游标的上限集合，该游标保持打开，即使在客户机消耗了所有最初返回的数据之后——这就产生了无限的数据流。**这种方法对于处理事件流的应用程序很有用，比如聊天消息或股票更新。

Spring Data MongoDB 项目帮助我们利用反应式数据库功能，包括可定制的游标。

## 3.设置

为了演示上述特性，我们将实现一个简单的日志计数器应用程序。让我们假设有一个日志聚合器将所有日志收集并保存到一个中心位置——我们的 MongoDB capped 集合。

首先，我们将使用简单的`Log`实体:

```java
@Document
public class Log {
    private @Id String id;
    private String service;
    private LogLevel level;
    private String message;
}
```

其次，我们将日志存储在我们的 MongoDB capped 集合中。 [Capped 集合](https://web.archive.org/web/20220625230955/https://docs.mongodb.com/manual/core/capped-collections/)是固定大小的集合，根据插入顺序插入和检索文档。我们可以用`MongoOperations.createCollection`来创建它们:

```java
db.createCollection(COLLECTION_NAME, new CreateCollectionOptions()
  .capped(true)
  .sizeInBytes(1024)
  .maxDocuments(5));
```

对于有上限的集合，我们必须定义`sizeInBytes`属性。此外，`maxDocuments`指定了一个集合可以拥有的最大文档数。一旦到达，旧文档将从集合中移除。

第三，我们将使用适当的 [Spring Boot 启动器依赖关系](https://web.archive.org/web/20220625230955/https://search.maven.org/search?q=a:spring-boot-starter-data-mongodb-reactive):

```java
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-mongodb-reactive</artifactId>
    <versionId>2.2.2.RELEASE</versionId>
</dependency>
```

## 4.反应式可定制光标

我们可以使用[命令式](#messagelistener)和反应式 MongoDB API 来消费可定制的游标。强烈建议**使用反应型**。

让我们使用反应式方法实现`WARN`级日志计数器。**我们能够用`ReactiveMongoOperations.tail`方法创建无限流查询。**

当新文档到达一个有上限的集合并匹配[过滤查询](/web/20220625230955/https://www.baeldung.com/queries-in-spring-data-mongodb)时，一个可定制的游标保持打开并发出数据——一个`Flux`实体:

```java
private Disposable subscription;

public WarnLogsCounter(ReactiveMongoOperations template) {
    Flux<Log> stream = template.tail(
      query(where("level").is(LogLevel.WARN)), 
      Log.class);
    subscription = stream.subscribe(logEntity -> 
      counter.incrementAndGet()
    );
}
```

一旦具有`WARN`日志级别的新文档被保存在集合中，订阅者(lambda 表达式)将递增计数器。

最后，我们应该释放订阅以关闭流:

```java
public void close() {
    this.subscription.dispose();
}
```

另外，**请注意，如果查询最初没有返回匹配，可定制的游标可能会失效。**换句话说，即使新的持久化文档匹配过滤器查询，订阅者也不能接收它们。这是 MongoDB 可定制游标的一个已知限制。在创建可定制的游标之前，我们必须确保在封顶的集合中有匹配的文档。

## 5.具有反应式存储库的可定制游标

Spring 数据项目为不同的数据存储提供了存储库抽象，包括反应版本。

MongoDB 也不例外。请查看关于 MongoDB 的 Spring Data Reactive Repositories 的文章，了解更多细节。

此外， **MongoDB reactive repositories 通过用`@Tailable`注释查询方法来支持无限流。**我们可以注释任何返回`Flux`的存储库方法或其他能够发出多个元素的反应类型:

```java
public interface LogsRepository extends ReactiveCrudRepository<Log, String> {
    @Tailable
    Flux<Log> findByLevel(LogLevel level);
}
```

让我们使用这个可定制的存储库方法来统计`INFO`日志:

```java
private Disposable subscription;

public InfoLogsCounter(LogsRepository repository) {
    Flux<Log> stream = repository.findByLevel(LogLevel.INFO);
    this.subscription = stream.subscribe(logEntity -> 
      counter.incrementAndGet()
    );
}
```

同样，对于`WarnLogsCounter`，我们应该处理订阅以关闭流:

```java
public void close() {
    this.subscription.dispose();
}
```

## 6.带有`MessageListener`的可定制光标

然而，如果我们不能使用反应式 API，我们可以利用 Spring 的消息传递概念。

首先，我们需要创建一个`MessageListenerContainer`来处理发送的`SubscriptionRequest`对象。同步 MongoDB 驱动程序创建了一个长时间运行的阻塞任务，该任务监听 capped 集合中的新文档。

Spring Data MongoDB 与**一起提供了一个默认实现，能够为`TailableCursorRequest:`创建和执行`Task`实例**

```java
private String collectionName;
private MessageListenerContainer container;
private AtomicInteger counter = new AtomicInteger();

public ErrorLogsCounter(MongoTemplate mongoTemplate,
  String collectionName) {
    this.collectionName = collectionName;
    this.container = new DefaultMessageListenerContainer(mongoTemplate);

    container.start();
    TailableCursorRequest<Log> request = getTailableCursorRequest();
    container.register(request, Log.class);
}

private TailableCursorRequest<Log> getTailableCursorRequest() {
    MessageListener<Document, Log> listener = message -> 
      counter.incrementAndGet();

    return TailableCursorRequest.builder()
      .collection(collectionName)
      .filter(query(where("level").is(LogLevel.ERROR)))
      .publishTo(listener)
      .build();
}
```

`TailableCursorRequest`创建仅过滤`ERROR`级日志的查询。每个匹配的文档都将被发布到`MessageListener`,它将递增计数器。

注意，我们仍然需要确保初始查询返回一些结果。否则，可定制光标将立即关闭。

此外，我们不应该忘记在不再需要容器时停止使用它:

```java
public void close() {
    container.stop();
}
```

## 7.结论

MongoDB 带有可定制游标的 capped 集合帮助我们以连续的方式从数据库接收信息。我们可以运行一个查询，该查询将一直给出结果，直到被显式关闭。Spring Data MongoDB 为我们提供了利用可定制游标的阻塞和反应方式。

GitHub 上的[提供了完整示例的源代码。](https://web.archive.org/web/20220625230955/https://github.com/eugenp/tutorials/tree/master/persistence-modules/spring-data-mongodb-reactive)