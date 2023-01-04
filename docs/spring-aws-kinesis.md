# 集成 Spring 和 AWS Kinesis

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-aws-kinesis>

## 1.介绍

Kinesis 是亚马逊开发的实时收集、处理和分析数据流的工具。它的主要优点之一是有助于事件驱动应用程序的开发。

在本教程中，我们将探索几个库，**使我们的 Spring 应用程序能够从 Kinesis 流**中产生和使用记录。代码示例将展示基本功能，但不代表生产就绪代码。

## 2.先决条件

在我们继续之前，我们需要做两件事。

第一个是[创建一个 Spring 项目](/web/20220627180237/https://www.baeldung.com/spring-boot-start)，因为这里的目标是与 Spring 项目中的 Kinesis 进行交互。

第二个是创建一个 Kinesis 数据流。我们可以在我们的 AWS 帐户中的 web 浏览器中完成此操作。对于我们当中的 AWS CLI 爱好者来说，另一种选择是[使用命令行](https://web.archive.org/web/20220627180237/https://docs.aws.amazon.com/cli/latest/reference/kinesis/create-stream.html)。因为我们将从代码中与它交互，所以我们手头还必须有 AWS [IAM 凭证](https://web.archive.org/web/20220627180237/https://docs.aws.amazon.com/sdk-for-java/v1/developer-guide/credentials.html)，访问密钥和秘密密钥，以及区域。

我们所有的生产者将创建虚拟 IP 地址记录，而消费者将读取这些值并在应用程序控制台中列出它们。

## 3.用于 Java 的 AWS SDK

我们将使用的第一个库是 AWS SDK for Java。它的优点是，它允许我们管理与 Kinesis 数据流工作的许多部分。我们可以**读取数据，产生数据，创建数据流，重新共享数据流**。缺点是，为了有生产就绪的代码，我们将不得不编码方面，如重散列，错误处理，或守护进程，以保持消费者活着。

### 3.1.Maven 依赖性

amazon-kinesis-client Maven 依赖将带来我们需要的一切工作示例。我们现在将它添加到我们的`pom.xml`文件中:

```java
<dependency>
    <groupId>com.amazonaws</groupId>
    <artifactId>amazon-kinesis-client</artifactId>
    <version>1.11.2</version>
</dependency>
```

### 3.2.弹簧设置

让我们重用与我们的 Kinesis 流交互所需的`AmazonKinesis`对象。我们将在我们的`@SpringBootApplication`类中创建一个`@Bean`:

```java
@Bean
public AmazonKinesis buildAmazonKinesis() {
    BasicAWSCredentials awsCredentials = new BasicAWSCredentials(accessKey, secretKey);
    return AmazonKinesisClientBuilder.standard()
      .withCredentials(new AWSStaticCredentialsProvider(awsCredentials))
      .withRegion(Regions.EU_CENTRAL_1)
      .build();
}
```

接下来，让我们在`application.properties`中定义本地机器所需的`aws.access.key`和`aws.secret.key`:

```java
aws.access.key=my-aws-access-key-goes-here
aws.secret.key=my-aws-secret-key-goes-here
```

我们将使用`@Value`注释来读取它们:

```java
@Value("${aws.access.key}")
private String accessKey;

@Value("${aws.secret.key}")
private String secretKey;
```

为了简单起见，我们将依靠`@Scheduled`方法来创建和消费记录。

### 3.3.消费者

**AWS SDK Kinesis 消费者使用拉模型**，这意味着我们的代码将从 Kinesis 数据流的碎片中提取记录:

```java
GetRecordsRequest recordsRequest = new GetRecordsRequest();
recordsRequest.setShardIterator(shardIterator.getShardIterator());
recordsRequest.setLimit(25);

GetRecordsResult recordsResult = kinesis.getRecords(recordsRequest);
while (!recordsResult.getRecords().isEmpty()) {
    recordsResult.getRecords().stream()
      .map(record -> new String(record.getData().array()))
      .forEach(System.out::println);

    recordsRequest.setShardIterator(recordsResult.getNextShardIterator());
    recordsResult = kinesis.getRecords(recordsRequest);
}
```

**`GetRecordsRequest`对象建立对流数据**的请求。在我们的例子中，我们已经定义了每个请求 25 条记录的限制，并且我们一直读取，直到没有更多要读取的内容。

我们还可以注意到，对于我们的迭代，我们使用了一个`GetShardIteratorResult`对象。我们在一个`@PostConstruc` t 方法中创建了这个对象，因此我们将立即开始跟踪记录:

```java
private GetShardIteratorResult shardIterator;

@PostConstruct
private void buildShardIterator() {
    GetShardIteratorRequest readShardsRequest = new GetShardIteratorRequest();
    readShardsRequest.setStreamName(IPS_STREAM);
    readShardsRequest.setShardIteratorType(ShardIteratorType.LATEST);
    readShardsRequest.setShardId(IPS_SHARD_ID);

    this.shardIterator = kinesis.getShardIterator(readShardsRequest);
}
```

### 3.4.生产者

现在让我们看看**如何为我们的 Kinesis 数据流**处理记录的创建。

**我们使用一个`PutRecordsRequest`对象**插入数据。对于这个新对象，我们添加了一个包含多个`PutRecordsRequestEntry`对象的列表:

```java
List<PutRecordsRequestEntry> entries = IntStream.range(1, 200).mapToObj(ipSuffix -> {
    PutRecordsRequestEntry entry = new PutRecordsRequestEntry();
    entry.setData(ByteBuffer.wrap(("192.168.0." + ipSuffix).getBytes()));
    entry.setPartitionKey(IPS_PARTITION_KEY);
    return entry;
}).collect(Collectors.toList());

PutRecordsRequest createRecordsRequest = new PutRecordsRequest();
createRecordsRequest.setStreamName(IPS_STREAM);
createRecordsRequest.setRecords(entries);

kinesis.putRecords(createRecordsRequest);
```

我们创造了一个基本的消费者和一个模拟 IP 记录的生产者。现在剩下要做的就是运行我们的 Spring 项目，并在我们的应用程序控制台中看到列出的 IP。

## 4.KCL 和 KPL

**Kinesis 客户端库(KCL)是一个简化记录使用的库**。它也是 AWS SDK Java APIs 之上的一个抽象层，用于 Kinesis 数据流。在幕后，库处理许多实例之间的负载平衡，响应实例故障，检查已处理的记录，并对重散列作出反应。

Kinesis Producer Library (KPL)是一个用于写入 Kinesis 数据流的库。它还为 Kinesis 数据流提供了一个位于 AWS SDK Java APIs 之上的抽象层。为了获得更好的性能，该库自动处理批处理、多线程和重试逻辑。

KCL 和 KPL 都有易于使用的主要优势，这样我们就可以专注于生产和消费唱片。

### 4.1.Maven 依赖性

如果需要，这两个库可以在我们的项目中单独引入。为了在我们的 Maven 项目中包含 [KPL](https://web.archive.org/web/20220627180237/https://search.maven.org/search?q=amazon-kinesis-producer) 和 [KCL](https://web.archive.org/web/20220627180237/https://search.maven.org/search?q=a:amazon-kinesis-client%20g:com.amazonaws) ，我们需要更新我们的 pom.xml 文件:

```java
<dependency>
    <groupId>com.amazonaws</groupId>
    <artifactId>amazon-kinesis-producer</artifactId>
    <version>0.13.1</version>
</dependency>
<dependency>
    <groupId>com.amazonaws</groupId>
    <artifactId>amazon-kinesis-client</artifactId>
    <version>1.11.2</version>
</dependency>
```

### 4.2.弹簧设置

我们需要做的唯一春季准备就是确保手头有 IAM 证书。在我们的`application.properties`文件中定义了`aws.access.key`和`aws.secret.key`的值，因此我们可以在需要时用`@Value`读取它们。

### 4.3.消费者

首先，我们将**创建一个实现`IRecordProcessor`接口的类，并为如何处理 Kinesis 数据流记录**定义我们的逻辑，即在控制台中打印它们:

```java
public class IpProcessor implements IRecordProcessor {
    @Override
    public void initialize(InitializationInput initializationInput) { }

    @Override
    public void processRecords(ProcessRecordsInput processRecordsInput) {
        processRecordsInput.getRecords()
          .forEach(record -> System.out.println(new String(record.getData().array())));
    }

    @Override
    public void shutdown(ShutdownInput shutdownInput) { }
}
```

下一步是**定义一个实现`IRecordProcessorFactory`接口**的工厂类，并返回先前创建的`IpProcessor`对象:

```java
public class IpProcessorFactory implements IRecordProcessorFactory {
    @Override
    public IRecordProcessor createProcessor() {
        return new IpProcessor();
    }
}
```

现在是最后一步，**我们将使用一个`Worker`对象来定义我们的消费者管道**。我们需要一个`KinesisClientLibConfiguration`对象，如果需要的话，它将定义 IAM 凭证和 AWS 区域。

我们将把`KinesisClientLibConfiguration`和我们的`IpProcessorFactory`对象传递给我们的`Worker`，然后在一个单独的线程中启动它。我们通过使用`Worker`类来保持消费记录的逻辑，所以我们现在不断地读取新记录:

```java
BasicAWSCredentials awsCredentials = new BasicAWSCredentials(accessKey, secretKey);
KinesisClientLibConfiguration consumerConfig = new KinesisClientLibConfiguration(
  APP_NAME, 
  IPS_STREAM,
  new AWSStaticCredentialsProvider(awsCredentials), 
  IPS_WORKER)
    .withRegionName(Regions.EU_CENTRAL_1.getName());

final Worker worker = new Worker.Builder()
  .recordProcessorFactory(new IpProcessorFactory())
  .config(consumerConfig)
  .build();
CompletableFuture.runAsync(worker.run());
```

### 4.4.生产者

现在让我们定义`KinesisProducerConfiguration`对象，添加 IAM 凭证和 AWS 区域:

```java
BasicAWSCredentials awsCredentials = new BasicAWSCredentials(accessKey, secretKey);
KinesisProducerConfiguration producerConfig = new KinesisProducerConfiguration()
  .setCredentialsProvider(new AWSStaticCredentialsProvider(awsCredentials))
  .setVerifyCertificate(false)
  .setRegion(Regions.EU_CENTRAL_1.getName());

this.kinesisProducer = new KinesisProducer(producerConfig);
```

我们将在一个`@Scheduled`作业中包含先前创建的`kinesisProducer`对象，并为我们的 Kinesis 数据流连续生成记录:

```java
IntStream.range(1, 200).mapToObj(ipSuffix -> ByteBuffer.wrap(("192.168.0." + ipSuffix).getBytes()))
  .forEach(entry -> kinesisProducer.addUserRecord(IPS_STREAM, IPS_PARTITION_KEY, entry));
```

## 5.春季云流粘合剂运动学

我们已经看到了两个库，都是在 Spring 生态系统之外创建的。我们现在将**看看 Spring Cloud Stream Binder Kinesis 如何在 [Spring Cloud Stream](/web/20220627180237/https://www.baeldung.com/spring-cloud-stream) 的基础上进一步简化我们的生活**。

### 5.1.Maven 依赖性

我们需要在应用程序中为[Spring Cloud Stream Binder Kinesis](https://web.archive.org/web/20220627180237/https://search.maven.org/search?q=a:spring-cloud-stream-binder-kinesis)定义的 Maven 依赖性是:

```java
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-stream-binder-kinesis</artifactId>
    <version>1.2.1.RELEASE</version>
</dependency>
```

### 5.2.弹簧设置

在 EC2 上运行时，会自动发现所需的 AWS 属性，因此不需要定义它们。因为我们在本地机器上运行我们的示例，所以我们需要为我们的 AWS 帐户定义 IAM 访问密钥、秘密密钥和区域。我们还禁用了应用程序的自动云形成堆栈名称检测:

```java
cloud.aws.credentials.access-key=my-aws-access-key
cloud.aws.credentials.secret-key=my-aws-secret-key
cloud.aws.region.static=eu-central-1
cloud.aws.stack.auto=false
```

**Spring Cloud Stream 捆绑了三个接口，我们可以在我们的流绑定中使用:**

*   `Sink`用于数据接收
*   `Source`用于发布记录
*   `Processor`是两者的结合

如果需要，我们也可以定义自己的接口。

### 5.3.消费者

定义消费者的工作分为两部分。首先，我们将在`application.properties`中定义我们将消费的数据流:

```java
spring.cloud.stream.bindings.input.destination=live-ips
spring.cloud.stream.bindings.input.group=live-ips-group
spring.cloud.stream.bindings.input.content-type=text/plain
```

接下来，让我们定义一个 Spring `@Component`类。注释 **`@EnableBinding(Sink.class)`将允许我们使用用`@StreamListener(Sink.INPUT)`** 注释的方法从 Kinesis 流中读取:

```java
@EnableBinding(Sink.class)
public class IpConsumer {

    @StreamListener(Sink.INPUT)
    public void consume(String ip) {
        System.out.println(ip);
    }
}
```

### 5.4.生产者

生产者也可以一分为二。首先，我们必须在`application.properties`中定义我们的流属性:

```java
spring.cloud.stream.bindings.output.destination=live-ips
spring.cloud.stream.bindings.output.content-type=text/plain
```

然后**我们在弹簧`@Component`上添加`@EnableBinding(Source.class)`并每隔几秒创建新的测试消息**:

```java
@Component
@EnableBinding(Source.class)
public class IpProducer {

    @Autowired
    private Source source;

    @Scheduled(fixedDelay = 3000L)
    private void produce() {
        IntStream.range(1, 200).mapToObj(ipSuffix -> "192.168.0." + ipSuffix)
          .forEach(entry -> source.output().send(MessageBuilder.withPayload(entry).build()));
    }
}
```

这就是我们需要春云流绑定器 Kinesis 工作。我们现在可以简单地启动应用程序。

## 6.结论

在本文中，我们看到了如何将 Spring 项目与两个 AWS 库集成，以便与 Kinesis 数据流进行交互。我们还看到了如何使用 Spring Cloud Stream Binder Kinesis 库来使实现变得更加容易。

这篇文章的源代码可以在 Github 上找到[。](https://web.archive.org/web/20220627180237/https://github.com/eugenp/tutorials/tree/master/spring-cloud-modules/spring-cloud-stream)