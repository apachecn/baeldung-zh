# 使用 Spring Cloud App Starter

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-cloud-app-starter>

## 1。简介

在本文中，我们将演示如何使用 Spring Cloud App starters——它提供了引导和现成的应用程序——可以作为未来开发的起点。

简单地说，任务应用启动器专用于数据库迁移和分布式测试等用例，而流应用启动器提供与外部系统的集成。

总的来说，首发人数超过 55 人；查看官方文档[这里](https://web.archive.org/web/20220626075034/https://cloud.spring.io/spring-cloud-task-app-starters/)和[这里](https://web.archive.org/web/20220626075034/https://cloud.spring.io/spring-cloud-stream-app-starters/)了解更多关于这两个的信息。

接下来，我们将构建一个小型分布式 Twitter 应用程序，该应用程序将 Twitter 帖子流式传输到 Hadoop 分布式文件系统中。

## 2.获取设置

我们将使用`consumer-key`和 `access-token` 来创建一个简单的 Twitter 应用程序。

然后，我们将设置 Hadoop，这样我们就可以为未来的大数据目的持久化我们的 Twitter 流。

最后，我们可以选择使用所提供的 Spring GitHub 库，通过 Maven 编译和组装`sources`–`processors-sinks`架构模式的独立组件，或者通过 Spring Stream 绑定接口组合`sources`、`processors`和`sinks`。

我们将看看这两种方法。

值得注意的是，以前，所有的流媒体应用启动器都在 github.com/spring-cloud/spring-cloud-stream-app-starters 的一个大型回购中进行整理。每个启动器都经过了简化和隔离。

## 3。Twitter 证书

首先，让我们设置我们的 Twitter 开发人员证书。要获得 Twitter 开发人员证书，请按照步骤设置一个应用程序，并根据官方 Twitter 开发人员文档创建一个访问令牌[。](https://web.archive.org/web/20220626075034/https://apps.twitter.com/)

具体来说，我们需要:

1.  消费者密钥
2.  消费者密钥秘密
3.  访问令牌秘密
4.  访问令牌

请确保打开窗口或记下它们，因为我们将在下面使用它们！

## 4.安装 Hadoop

接下来，我们来安装 Hadoop！我们可以遵循[官方文档](https://web.archive.org/web/20220626075034/https://hadoop.apache.org/docs/stable/hadoop-project-dist/hadoop-common/SingleCluster.html#Installing_Software)或者简单地利用 Docker:

```java
$ sudo docker run -p 50070:50070 sequenceiq/hadoop-docker:2.4.1
```

## 5.编译我们的应用启动器

为了使用独立的、完全独立的组件，我们可以从他们的 GitHub 存储库中下载并编译所需的 Spring Cloud Stream 应用启动器。

### 5.1。Twitter Spring Cloud Stream App Starter

让我们将 Twitter Spring Cloud Stream 应用 Starter ( `org.springframework.cloud.stream.app.twitterstream.source`)添加到我们的项目中:

```java
git clone https://github.com/spring-cloud-stream-app-starters/twitter.git
```

然后，我们运行 Maven:

```java
./mvnw clean install -PgenerateApps
```

最终编译的 Starter 应用程序将在本地项目根目录的“/target”中可用。

然后我们可以运行编译过的。jar 并传入相关的应用程序属性，如下所示:

```java
java -jar twitter_stream_source.jar --consumerKey=<CONSUMER_KEY> --consumerSecret=<CONSUMER_SECRET> \
    --accessToken=<ACCESS_TOKEN> --accessTokenSecret=<ACCESS_TOKEN_SECRET>
```

我们还可以使用熟悉的 Spring `application.properties:`传递我们的凭证

```java
twitter.credentials.access-token=...
twitter.credentials.access-token-secret=...
twitter.credentials.consumer-key=...
twitter.credentials.consumer-secret=...
```

### 5.2。HDFS 春天云流 App 首发

现在(Hadoop 已经设置好了)，让我们将 HDFS Spring Cloud Stream App Starter(`org.springframework.cloud.stream.app.hdfs.sink`)依赖项添加到我们的项目中。

首先，克隆相关的回购协议:

```java
git clone https://github.com/spring-cloud-stream-app-starters/hdfs.git
```

然后，运行 Maven 作业:

```java
./mvnw clean install -PgenerateApps
```

最终编译的 Starter 应用程序将在本地项目根目录的“/target”中可用。然后我们可以运行编译好的。jar 并传入相关的应用程序属性:

```java
java -jar hdfs-sink.jar --fsUri=hdfs://127.0.0.1:50010/
```

“`hdfs://127.0.0.1:50010/`”是 Hadoop 的默认端口，但您的默认 HDFS 端口可能会因您配置实例的方式而异。

根据之前传入的配置，我们可以在'`http://0.0.0.0:50070`'处看到数据节点(及其当前端口)的列表。

我们还可以在编译之前使用熟悉的 Spring `application.properties` 传递我们的凭证——这样我们就不必总是通过 CLI 传递这些凭证。

让我们配置我们的`application.properties` 来使用默认的 Hadoop 端口:

```java
hdfs.fs-uri=hdfs://127.0.0.1:50010/
```

## 6。使用`AggregateApplicationBuilder`

或者，我们可以通过`org.springframework.cloud.stream.aggregate.AggregateApplicationBuilder`将 Spring Stream `Source` 和 `Sink` 组合成一个简单的 Spring Boot 应用程序！

首先，我们将两个流应用启动器添加到我们的`pom.xml`:

```java
<dependencies>
    <dependency>
        <groupId>org.springframework.cloud.stream.app</groupId>
        <artifactId>spring-cloud-starter-stream-source-twitterstream</artifactId>
        <version>2.1.2.RELEASE</version>
    </dependency>
    <dependency>
        <groupId>org.springframework.cloud.stream.app</groupId>
        <artifactId>spring-cloud-starter-stream-sink-hdfs</artifactId>
        <version>2.1.2.RELEASE</version>
    </dependency>
</dependencies>
```

然后，我们将开始通过将两个流应用程序启动器依赖项包装到各自的子应用程序中来组合它们。

### 6.1。构建我们的应用组件

我们的`SourceApp`指定了要转换或消耗的`Source`:

```java
@SpringBootApplication
@EnableBinding(Source.class)
@Import(TwitterstreamSourceConfiguration.class)
public class SourceApp {
    @InboundChannelAdapter(Source.OUTPUT)
    public String timerMessageSource() {
        return new SimpleDateFormat().format(new Date());
    }
}
```

请注意，我们将`SourceApp`绑定到`org.springframework.cloud.stream.messaging.Source`，并注入适当的配置类，以从我们的环境属性中获取所需的设置。

接下来，我们设置一个简单的`org.springframework.cloud.stream.messaging.Processor` 绑定`:`

```java
@SpringBootApplication
@EnableBinding(Processor.class)
public class ProcessorApp {
    @Transformer(inputChannel = Processor.INPUT, outputChannel = Processor.OUTPUT)
    public String processMessage(String payload) {
        log.info("Payload received!");
        return payload;
    }
}
```

然后，我们创建我们的消费者(`Sink`):

```java
@SpringBootApplication
@EnableBinding(Sink.class)
@Import(HdfsSinkConfiguration.class)
public class SinkApp {
    @ServiceActivator(inputChannel= Sink.INPUT)
    public void loggerSink(Object payload) {
        log.info("Received: " + payload);
    }
}
```

在这里，我们将`SinkApp`绑定到`org.springframework.cloud.stream.messaging.Sink`，并再次注入正确的配置类来使用我们指定的 Hadoop 设置。

最后，我们在我们的`AggregateApp`主方法中使用`AggregateApplicationBuilder`来组合我们的`SourceApp`、 `ProcessorApp`和`SinkApp`:

```java
@SpringBootApplication
public class AggregateApp {
    public static void main(String[] args) {
        new AggregateApplicationBuilder()
          .from(SourceApp.class).args("--fixedDelay=5000")
          .via(ProcessorApp.class)
          .to(SinkApp.class).args("--debug=true")
          .run(args);
    }
}
```

与任何 Spring Boot 应用程序一样，我们可以通过`application.properties or`以编程方式注入指定的设置作为环境属性。

因为我们使用的是 Spring Stream 框架，所以我们也可以将参数传递给`AggregateApplicationBuilder`构造函数。

### 6.2。运行已完成的应用程序

然后，我们可以使用以下命令行指令编译并运行我们的应用程序:

```java
 $ mvn install
    $ java -jar twitterhdfs.jar
```

记住将每个`@SpringBootApplication`类保存在一个单独的包中(否则，将会抛出几个不同的绑定异常)！关于如何使用`AggregateApplicationBuilder`的更多信息，请看一下[官方文件](https://web.archive.org/web/20220626075034/https://github.com/spring-cloud/spring-cloud-stream-samples/)。

在我们编译并运行我们的应用程序后，我们应该会在我们的控制台中看到如下内容(自然，内容会因 Tweet 而异):

```java
2018-01-15 04:38:32.255  INFO 28778 --- [itterSource-1-1] 
c.b.twitterhdfs.processor.ProcessorApp   : Payload received!
2018-01-15 04:38:32.255  INFO 28778 --- [itterSource-1-1] 
com.baeldung.twitterhdfs.sink.SinkApp    : Received: {"created_at":
"Mon Jan 15 04:38:32 +0000 2018","id":952761898239385601,"id_str":
"952761898239385601","text":"RT @mighty_jimin: 180114 ...
```

这些演示了我们的`Processor`和`Sink`在从`Source`接收数据时的正确操作！在这个例子中，我们没有配置我们的 HDFS 接收器做太多事情——它将简单地打印消息“有效负载已接收！”

## 7。结论

在本教程中，我们学习了如何将两个令人敬畏的 Spring Stream 应用程序启动器合并成一个可爱的 Spring Boot 示例！

这里有一些关于 Spring Boot 首发和如何创建 T2 定制首发的官方文章！

和往常一样，本文中使用的代码可以在 GitHub 上找到[。](https://web.archive.org/web/20220626075034/https://github.com/eugenp/tutorials/tree/master/spring-cloud-modules/spring-cloud-stream-starters)