# 网飞螳螂简介

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-netflix-mantis>

## 1.概观

在这篇文章中，我们将看看由网飞开发的螳螂平台。

我们将通过创建、运行和研究流处理作业来探索 Mantis 的主要概念。

## 2.螳螂是什么？

Mantis 是一个构建流处理应用程序的平台。它提供了一种简单的方式来**管理作业的部署和生命周期。**此外，it **促进了这些工作之间的资源分配、发现和交流。**

因此，开发人员可以专注于实际的业务逻辑，同时拥有一个**健壮且可扩展的平台来运行其高容量、低延迟、无阻塞的应用程序。**

螳螂作业由三个不同的部分组成:

*   `source`，负责从外部来源检索数据
*   一个或多个`stages`，负责处理传入的事件流
*   以及收集处理后的数据的`sink`

现在，让我们逐一探讨一下。

## 3.设置和依赖关系

让我们从添加`[mantis-runtime](https://web.archive.org/web/20220627180555/https://search.maven.org/classic/#search%7Cga%7C1%7Cg%3A%22io.mantisrx%22%20a%3A%22mantis-runtime%22)`和`[jackson-databind](https://web.archive.org/web/20220627180555/https://search.maven.org/classic/#search%7Cga%7C1%7Cg%3A%22com.fasterxml.jackson.core%22%20a%3A%22jackson-databind%22)`依赖关系开始:

```java
<dependency>
    <groupId>io.mantisrx</groupId>
    <artifactId>mantis-runtime</artifactId>
</dependency>

<dependency>
    <groupId>com.fasterxml.jackson.core</groupId>
    <artifactId>jackson-databind</artifactId>
</dependency>
```

现在，为了设置我们作业的数据源，让我们实现 Mantis `Source` 接口:

```java
public class RandomLogSource implements Source<String> {

    @Override
    public Observable<Observable<String>> call(Context context, Index index) {
        return Observable.just(
          Observable
            .interval(250, TimeUnit.MILLISECONDS)
            .map(this::createRandomLogEvent));
    }

    private String createRandomLogEvent(Long tick) {
        // generate a random log entry string
        ...
    }

}
```

正如我们所看到的，它只是每秒钟生成多次随机日志条目。

## 4.我们的第一份工作

现在让我们创建一个 Mantis 作业，它只是从我们的`RandomLogSource`中收集日志事件。稍后，我们将添加分组和聚合转换，以获得更复杂和有趣的结果。

首先，让我们创建一个`LogEvent`实体:

```java
public class LogEvent implements JsonType {
    private Long index;
    private String level;
    private String message;

    // ...
}
```

然后，让我们添加我们的`TransformLogStage.`

这是一个简单的阶段，它实现了 ScalarComputation 接口，并拆分一个日志条目来构建一个`LogEvent`。此外，它还会过滤掉任何格式错误的字符串:

```java
public class TransformLogStage implements ScalarComputation<String, LogEvent> {

    @Override
    public Observable<LogEvent> call(Context context, Observable<String> logEntry) {
        return logEntry
          .map(log -> log.split("#"))
          .filter(parts -> parts.length == 3)
          .map(LogEvent::new);
    }

}
```

### 4.1.运行作业

此时，我们已经有足够的构件来组合我们的螳螂工作:

```java
public class LogCollectingJob extends MantisJobProvider<LogEvent> {

    @Override
    public Job<LogEvent> getJobInstance() {
        return MantisJob
          .source(new RandomLogSource())
          .stage(new TransformLogStage(), new ScalarToScalar.Config<>())
          .sink(Sinks.eagerSubscribe(Sinks.sse(LogEvent::toJsonString)))
          .metadata(new Metadata.Builder().build())
          .create();
    }

}
```

让我们仔细看看我们的工作。

正如我们所看到的，它首先扩展了`MantisJobProvider.`，从我们的`RandomLogSource` 获取数据，并将`TransformLogStage`应用于获取的数据。最后，它将处理后的数据发送到内置的接收器，接收器急切地订阅并通过 [SSE](https://web.archive.org/web/20220627180555/https://en.wikipedia.org/wiki/Server-sent_events) 传递数据。

现在，让我们将作业配置为在启动时本地执行:

```java
@SpringBootApplication
public class MantisApplication implements CommandLineRunner {

    // ...

    @Override
    public void run(String... args) {
        LocalJobExecutorNetworked.execute(new LogCollectingJob().getJobInstance());
    }
}
```

让我们运行应用程序。我们将看到一条日志消息，如下所示:

```java
...
Serving modern HTTP SSE server sink on port: 86XX
```

现在让我们使用`curl`连接到水槽:

```java
$ curl localhost:86XX
data: {"index":86,"level":"WARN","message":"login attempt"}
data: {"index":87,"level":"ERROR","message":"user created"}
data: {"index":88,"level":"INFO","message":"user created"}
data: {"index":89,"level":"INFO","message":"login attempt"}
data: {"index":90,"level":"INFO","message":"user created"}
data: {"index":91,"level":"ERROR","message":"user created"}
data: {"index":92,"level":"WARN","message":"login attempt"}
data: {"index":93,"level":"INFO","message":"user created"}
...
```

### 4.2.配置接收器

到目前为止，我们已经使用内置的接收器来收集我们处理过的数据。让我们看看是否可以通过提供一个定制的接收器来为我们的场景增加更多的灵活性。

例如，如果我们想要通过`message`来过滤日志呢？

让我们创建一个实现`Sink<LogEvent>` 接口的`LogSink`:

```java
public class LogSink implements Sink<LogEvent> {
    @Override
    public void call(Context context, PortRequest portRequest, Observable<LogEvent> logEventObservable) {
        SelfDocumentingSink<LogEvent> sink = new ServerSentEventsSink.Builder<LogEvent>()
          .withEncoder(LogEvent::toJsonString)
          .withPredicate(filterByLogMessage())
          .build();
        logEventObservable.subscribe();
        sink.call(context, portRequest, logEventObservable);
    }
    private Predicate<LogEvent> filterByLogMessage() {
        return new Predicate<>("filter by message",
          parameters -> {
            if (parameters != null && parameters.containsKey("filter")) {
                return logEvent -> logEvent.getMessage().contains(parameters.get("filter").get(0));
            }
            return logEvent -> true;
        });
    }
}
```

在这个 sink 实现中，我们配置了一个谓词，它使用`filter` 参数只检索包含在`filter`参数中设置的文本的日志:

```java
$ curl localhost:8874?filter=login
data: {"index":93,"level":"ERROR","message":"login attempt"}
data: {"index":95,"level":"INFO","message":"login attempt"}
data: {"index":97,"level":"ERROR","message":"login attempt"}
...
```

**Note Mantis 还提供了一种强大的查询语言， [MQL](https://web.archive.org/web/20220627180555/https://netflix.github.io/mantis/reference/mql/) ，可以用于以 SQL 方式查询、转换和分析流数据。**

## 5.阶段链接

现在假设我们想知道在给定的时间间隔内我们有多少个`ERROR`、`WARN,`或`INFO`日志条目。为此，我们将在我们的工作中再添加两个阶段，并将它们链接在一起。

### 5.1.分组

首先，让我们创建一个`GroupLogStage.`

这个阶段是一个从现有的`TransformLogStage`接收`LogEvent`流数据的`ToGroupComputation`实现。之后，它按日志记录级别对条目进行分组，并将它们发送到下一阶段:

```java
public class GroupLogStage implements ToGroupComputation<LogEvent, String, LogEvent> {

    @Override
    public Observable<MantisGroup<String, LogEvent>> call(Context context, Observable<LogEvent> logEvent) {
        return logEvent.map(log -> new MantisGroup<>(log.getLevel(), log));
    }

    public static ScalarToGroup.Config<LogEvent, String, LogEvent> config(){
        return new ScalarToGroup.Config<LogEvent, String, LogEvent>()
          .description("Group event data by level")
          .codec(JacksonCodecs.pojo(LogEvent.class))
          .concurrentInput();
    }

}
```

我们还通过提供描述、用于序列化输出的编解码器创建了一个定制的 stage 配置，并通过使用`  concurrentInput().`允许这个 stage 的 call 方法并发运行

需要注意的一点是，这个阶段是可横向扩展的。这意味着我们可以根据需要运行这个阶段的任意多个实例。另外值得一提的是，当部署在螳螂集群中时，**这个阶段将数据发送到下一个阶段，以便属于特定组的所有事件将落在下一个阶段的同一个工作器上。**

### 5.2.聚集

在我们继续并创建下一个阶段之前，让我们首先添加一个`LogAggregate`实体:

```java
public class LogAggregate implements JsonType {

    private final Integer count;
    private final String level;

}
```

现在，让我们创建链中的最后一个阶段。

这个阶段实现`GroupToScalarComputation`并将日志组流转换为标量`LogAggregate`。它通过计算每种类型的日志在流中出现的次数来做到这一点。此外，它还有一个`LogAggregationDuration`参数，可以用来控制聚合窗口的大小:

```java
public class CountLogStage implements GroupToScalarComputation<String, LogEvent, LogAggregate> {

    private int duration;

    @Override
    public void init(Context context) {
        duration = (int)context.getParameters().get("LogAggregationDuration", 1000);
    }

    @Override
    public Observable<LogAggregate> call(Context context, Observable<MantisGroup<String, LogEvent>> mantisGroup) {
        return mantisGroup
          .window(duration, TimeUnit.MILLISECONDS)
          .flatMap(o -> o.groupBy(MantisGroup::getKeyValue)
            .flatMap(group -> group.reduce(0, (count, value) ->  count = count + 1)
              .map((count) -> new LogAggregate(count, group.getKey()))
            ));
    }

    public static GroupToScalar.Config<String, LogEvent, LogAggregate> config(){
        return new GroupToScalar.Config<String, LogEvent, LogAggregate>()
          .description("sum events for a log level")
          .codec(JacksonCodecs.pojo(LogAggregate.class))
          .withParameters(getParameters());
    }

    public static List<ParameterDefinition<?>> getParameters() {
        List<ParameterDefinition<?>> params = new ArrayList<>();

        params.add(new IntParameter()
          .name("LogAggregationDuration")
          .description("window size for aggregation in milliseconds")
          .validator(Validators.range(100, 10000))
          .defaultValue(5000)
          .build());

        return params;
    }

}
```

### 5.3.配置并运行作业

现在唯一要做的就是配置我们的作业:

```java
public class LogAggregationJob extends MantisJobProvider<LogAggregate> {

    @Override
    public Job<LogAggregate> getJobInstance() {

        return MantisJob
          .source(new RandomLogSource())
          .stage(new TransformLogStage(), TransformLogStage.stageConfig())
          .stage(new GroupLogStage(), GroupLogStage.config())
          .stage(new CountLogStage(), CountLogStage.config())
          .sink(Sinks.eagerSubscribe(Sinks.sse(LogAggregate::toJsonString)))
          .metadata(new Metadata.Builder().build())
          .create();
    }
}
```

当我们运行应用程序并执行新作业时，我们可以看到每隔几秒钟就检索一次日志计数:

```java
$ curl localhost:8133
data: {"count":3,"level":"ERROR"}
data: {"count":13,"level":"INFO"}
data: {"count":4,"level":"WARN"}

data: {"count":8,"level":"ERROR"}
data: {"count":5,"level":"INFO"}
data: {"count":7,"level":"WARN"}
...
```

## 6.结论

总之，在这篇文章中，我们已经看到了网飞螳螂是什么，它可以用来做什么。此外，我们研究了主要概念，用它们来构建作业，并探索了不同场景的定制配置。

和往常一样，完整的代码可以在 GitHub 上找到[。](https://web.archive.org/web/20220627180555/https://github.com/eugenp/tutorials/tree/master/netflix-modules)