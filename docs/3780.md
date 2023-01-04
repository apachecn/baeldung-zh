# 阿帕奇风暴简介

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/apache-storm>

## 1.概观

本教程将介绍[阿帕奇风暴](https://web.archive.org/web/20220926180436/https://storm.apache.org/)，**一个分布式实时计算系统。**

我们将重点关注并涵盖:

*   阿帕奇风暴到底是什么，它解决了什么问题
*   它的建筑，和
*   如何在项目中使用它

## 2.什么是阿帕奇风暴？

Apache Storm 是用于实时计算的免费开源分布式系统。

它提供容错、可伸缩性，并保证数据处理，尤其擅长处理无界数据流。

Storm 的一些好的用例可以是处理信用卡操作以检测欺诈，或者处理来自智能家居的数据以检测故障传感器。

Storm 允许与市场上的各种数据库和排队系统集成。

## 3.Maven 依赖性

在我们使用 Apache Storm 之前，我们需要在我们的项目中包含 storm-core 依赖关系:

```
<dependency>
    <groupId>org.apache.storm</groupId>
    <artifactId>storm-core</artifactId>
    <version>1.2.2</version>
    <scope>provided</scope>
</dependency>
```

**如果我们打算在 Storm 集群上运行我们的应用程序，我们应该只使用`provided scope `。**

要在本地运行应用程序，我们可以使用所谓的本地模式，在本地进程中模拟 Storm 集群，在这种情况下，我们应该删除`provided.`

## 4.数据模型

Apache Storm 的数据模型由两个元素组成:元组和流。

### 4.1.元组

`Tuple`是具有动态类型的命名字段的有序列表。这意味着我们不需要显式声明字段的类型。

Storm 需要知道如何序列化元组中使用的所有值。默认情况下，它已经可以序列化基本类型、`Strings`和`byte`数组。

由于 Storm 使用 Kryo 序列化，我们需要使用`Config`注册序列化程序来使用定制类型。我们可以通过以下两种方式之一实现这一点:

首先，我们可以使用全名注册要序列化的类:

```
Config config = new Config();
config.registerSerialization(User.class);
```

在这种情况下，Kryo 将默认使用`[FieldSerializer](https://web.archive.org/web/20220926180436/https://github.com/EsotericSoftware/kryo#fieldserializer). `来序列化该类，这将序列化该类的所有非瞬态字段，包括私有和公共字段。

或者，我们可以同时提供要序列化的类和我们希望 Storm 为该类使用的序列化程序:

```
Config config = new Config();
config.registerSerialization(User.class, UserSerializer.class);
```

为了创建定制的序列化程序，我们需要[来扩展通用类](/web/20220926180436/https://www.baeldung.com/kryo) `Serializer `，它有两个方法`write `和`read.`

### 4.2.溪流

一个*流*是风暴生态系统中的核心抽象。***流*是一个无界的元组序列。**

Storms 允许并行处理多个流。

每个流都有一个在声明期间提供和分配的 id。

## 5.拓扑学

实时 Storm 应用程序的逻辑被打包到拓扑中。拓扑结构由**喷口**和**螺栓**组成。

### 5.1.喷口

喷口是溪流的源头。它们向拓扑发出元组。

元组可以从各种外部系统读取，如 Kafka、Kestrel 或 ActiveMQ。

喷口可以是`reliable`或`unreliable`。`Reliable`表示 spout 可以回复 Storm 处理失败的元组。`Unreliable` 意味着 spout 不回复，因为它将使用一个“发射并忘记”机制来发出元组。

为了创建定制的 spout，我们需要实现`IRichSpout `接口或者扩展任何已经实现该接口的类，例如，抽象的`BaseRichSpout`类。

让我们创建一个`unreliable `喷口:

```
public class RandomIntSpout extends BaseRichSpout {

    private Random random;
    private SpoutOutputCollector outputCollector;

    @Override
    public void open(Map map, TopologyContext topologyContext,
      SpoutOutputCollector spoutOutputCollector) {
        random = new Random();
        outputCollector = spoutOutputCollector;
    }

    @Override
    public void nextTuple() {
        Utils.sleep(1000);
        outputCollector.emit(new Values(random.nextInt(), System.currentTimeMillis()));
    }

    @Override
    public void declareOutputFields(OutputFieldsDeclarer outputFieldsDeclarer) {
        outputFieldsDeclarer.declare(new Fields("randomInt", "timestamp"));
    }
}
```

我们的客户`RandomIntSpout`将每秒生成随机整数和时间戳。

### 5.2.拴住

**Bolts 处理流中的元组。**他们可以执行各种操作，如过滤、聚合或自定义功能。

有些操作需要多个步骤，因此在这种情况下我们需要使用多个螺栓。

为了创建自定义的`Bolt`，我们需要实现`IRichBolt `或者为了更简单的操作`IBasicBolt`接口。

还有多个助手类可用于实现`Bolt. `在这种情况下，我们将使用`BaseBasicBolt`:

```
public class PrintingBolt extends BaseBasicBolt {
    @Override
    public void execute(Tuple tuple, BasicOutputCollector basicOutputCollector) {
        System.out.println(tuple);
    }

    @Override
    public void declareOutputFields(OutputFieldsDeclarer outputFieldsDeclarer) {

    }
}
```

这个定制`PrintingBolt`将简单地将所有元组打印到控制台。

## 6.创建简单的拓扑

让我们把这些想法放到一个简单的拓扑中。我们的拓扑将有一个喷口和三个螺栓。

### 6.1.`RandomNumberSpout`

一开始，我们将创建一个不可靠的喷口。它将每秒生成(0，100)范围内的随机整数:

```
public class RandomNumberSpout extends BaseRichSpout {
    private Random random;
    private SpoutOutputCollector collector;

    @Override
    public void open(Map map, TopologyContext topologyContext, 
      SpoutOutputCollector spoutOutputCollector) {
        random = new Random();
        collector = spoutOutputCollector;
    }

    @Override
    public void nextTuple() {
        Utils.sleep(1000);
        int operation = random.nextInt(101);
        long timestamp = System.currentTimeMillis();

        Values values = new Values(operation, timestamp);
        collector.emit(values);
    }

    @Override
    public void declareOutputFields(OutputFieldsDeclarer outputFieldsDeclarer) {
        outputFieldsDeclarer.declare(new Fields("operation", "timestamp"));
    }
}
```

### 6.2.`FilteringBolt`

接下来，我们将创建一个 bolt 来过滤掉所有`operation`等于 0 的元素:

```
public class FilteringBolt extends BaseBasicBolt {
    @Override
    public void execute(Tuple tuple, BasicOutputCollector basicOutputCollector) {
        int operation = tuple.getIntegerByField("operation");
        if (operation > 0) {
            basicOutputCollector.emit(tuple.getValues());
        }
    }

    @Override
    public void declareOutputFields(OutputFieldsDeclarer outputFieldsDeclarer) {
        outputFieldsDeclarer.declare(new Fields("operation", "timestamp"));
    }
}
```

### 6.3.`AggregatingBolt`

接下来，让我们创建一个更复杂的`Bolt `来聚合每天的所有积极操作。

为此，我们将使用一个专门为实现在 windows 上操作而不是在单元组上操作的 bolts 而创建的特定类:`BaseWindowedBolt`。

`Windows`是流处理中的一个基本概念，将无限的流分割成有限的块。然后我们可以对每个块进行计算。通常有两种类型的窗口:

**时间窗口用于使用时间戳**对来自给定时间段的元素进行分组。时间窗可以具有不同数量的元素。

**计数窗口用于创建定义尺寸的窗口**。在这种情况下，所有的窗口都有相同的大小，如果元素少于定义的大小，窗口将不会发出**。**

我们的`AggregatingBolt`将从一个`time window`连同它的开始和结束时间戳产生所有正操作的总和:

```
public class AggregatingBolt extends BaseWindowedBolt {
    private OutputCollector outputCollector;

    @Override
    public void prepare(Map stormConf, TopologyContext context, OutputCollector collector) {
        this.outputCollector = collector;
    }

    @Override
    public void declareOutputFields(OutputFieldsDeclarer declarer) {
        declarer.declare(new Fields("sumOfOperations", "beginningTimestamp", "endTimestamp"));
    }

    @Override
    public void execute(TupleWindow tupleWindow) {
        List<Tuple> tuples = tupleWindow.get();
        tuples.sort(Comparator.comparing(this::getTimestamp));

        int sumOfOperations = tuples.stream()
          .mapToInt(tuple -> tuple.getIntegerByField("operation"))
          .sum();
        Long beginningTimestamp = getTimestamp(tuples.get(0));
        Long endTimestamp = getTimestamp(tuples.get(tuples.size() - 1));

        Values values = new Values(sumOfOperations, beginningTimestamp, endTimestamp);
        outputCollector.emit(values);
    }

    private Long getTimestamp(Tuple tuple) {
        return tuple.getLongByField("timestamp");
    }
}
```

注意，在这种情况下，直接获取列表的第一个元素是安全的。这是因为每个窗口都是使用`Tuple, `的`timestamp `字段计算的，所以**每个窗口中至少要有** **个元素。**

### 6.4.`FileWritingBolt`

最后，我们将创建一个 bolt，它将获取所有大于 2000 的元素，将它们序列化并写入文件:

```
public class FileWritingBolt extends BaseRichBolt {
    public static Logger logger = LoggerFactory.getLogger(FileWritingBolt.class);
    private BufferedWriter writer;
    private String filePath;
    private ObjectMapper objectMapper;

    @Override
    public void cleanup() {
        try {
            writer.close();
        } catch (IOException e) {
            logger.error("Failed to close writer!");
        }
    }

    @Override
    public void prepare(Map map, TopologyContext topologyContext, 
      OutputCollector outputCollector) {
        objectMapper = new ObjectMapper();
        objectMapper.setVisibility(PropertyAccessor.FIELD, JsonAutoDetect.Visibility.ANY);

        try {
            writer = new BufferedWriter(new FileWriter(filePath));
        } catch (IOException e) {
            logger.error("Failed to open a file for writing.", e);
        }
    }

    @Override
    public void execute(Tuple tuple) {
        int sumOfOperations = tuple.getIntegerByField("sumOfOperations");
        long beginningTimestamp = tuple.getLongByField("beginningTimestamp");
        long endTimestamp = tuple.getLongByField("endTimestamp");

        if (sumOfOperations > 2000) {
            AggregatedWindow aggregatedWindow = new AggregatedWindow(
                sumOfOperations, beginningTimestamp, endTimestamp);
            try {
                writer.write(objectMapper.writeValueAsString(aggregatedWindow));
                writer.newLine();
                writer.flush();
            } catch (IOException e) {
                logger.error("Failed to write data to file.", e);
            }
        }
    }

    // public constructor and other methods
}
```

**注意，我们不需要声明输出，因为这将是我们拓扑中的最后一个螺栓**

### 6.5.运行拓扑

最后，我们可以将所有内容整合在一起，运行我们的拓扑结构:

```
public static void runTopology() {
    TopologyBuilder builder = new TopologyBuilder();

    Spout random = new RandomNumberSpout();
    builder.setSpout("randomNumberSpout");

    Bolt filtering = new FilteringBolt();
    builder.setBolt("filteringBolt", filtering)
      .shuffleGrouping("randomNumberSpout");

    Bolt aggregating = new AggregatingBolt()
      .withTimestampField("timestamp")
      .withLag(BaseWindowedBolt.Duration.seconds(1))
      .withWindow(BaseWindowedBolt.Duration.seconds(5));
    builder.setBolt("aggregatingBolt", aggregating)
      .shuffleGrouping("filteringBolt"); 

    String filePath = "./src/main/resources/data.txt";
    Bolt file = new FileWritingBolt(filePath);
    builder.setBolt("fileBolt", file)
      .shuffleGrouping("aggregatingBolt");

    Config config = new Config();
    config.setDebug(false);
    LocalCluster cluster = new LocalCluster();
    cluster.submitTopology("Test", config, builder.createTopology());
}
```

为了使数据流经拓扑中的每一部分，我们需要指出如何连接它们。`shuffleGroup`允许我们声明`filteringBolt`的数据将来自`randomNumberSpout`。

**对于每个`Bolt`，我们需要添加`shuffleGroup`来定义这个螺栓的元素来源。**元素源可能是一个`Spout `或另一个`Bolt.` ，如果我们为一个以上的螺栓`, `设置相同的源，该源将发射所有元素给它们中的每一个。

在这种情况下，我们的拓扑将使用`LocalCluster`在本地运行作业。

## 7.结论

在本教程中，我们介绍了 Apache Storm，一个分布式实时计算系统。我们创建了一个喷口，一些螺栓，并把它们组合成一个完整的拓扑。

和往常一样，所有的代码样本都可以在 GitHub 上找到[。](https://web.archive.org/web/20220926180436/https://github.com/eugenp/tutorials/tree/master/libraries-data)