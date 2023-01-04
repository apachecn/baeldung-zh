# Kafka 连接器介绍

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/kafka-connectors-guide>

## 1。概述

Apache Kafka 是一个分布式流媒体平台。在之前的教程 中，我们讨论了[如何使用 Spring](/web/20220913030326/https://www.baeldung.com/spring-kafka) 实现 Kafka 消费者和生产者。

在本教程中，我们将学习如何使用 Kafka 连接器。

我们来看看:

*   不同类型的 Kafka 连接器
*   Kafka Connect 的特性和模式
*   连接器配置使用属性文件以及 REST API

## 2。Kafka Connect 和 Kafka 连接器的基础知识

**Kafka Connect 是一个使用所谓的`Connectors`将 Kafka 与外部系统**如数据库、键值存储、搜索索引和文件系统连接起来的框架。

**Kafka 连接器是现成的组件，可以帮助我们** **将外部系统的数据导入到 Kafka 主题中，** **将 Kafka 主题的数据导出到外部系统**。我们可以将现有的连接器实现用于公共数据源和接收器，或者实现我们自己的连接器。

A `source connector`从系统中收集数据。源系统可以是整个数据库、流表或消息代理。源连接器还可以从应用服务器收集指标到 Kafka 主题中，使数据可以用于低延迟的流处理。

A `sink connector`将 Kafka 主题中的数据传递到其他系统中，这些系统可能是索引(如 Elasticsearch)、批处理系统(如 Hadoop)或任何类型的数据库。

一些连接器由社区维护，而另一些则由 Confluent 或其合作伙伴支持。实际上，我们可以找到大多数流行系统的连接器，比如 S3、JDBC 和卡珊德拉等等。

## 3。功能

Kafka Connect 特性包括:

*   用 Kafka 连接外部系统的框架-**它简化了连接器的开发、部署和管理**
*   分布式和独立模式—**它帮助我们利用 Kafka 的分布式特性部署大型集群，以及开发、测试和小型生产部署的设置**
*   REST 接口——我们可以使用 REST API 管理连接器
*   自动偏移管理-**Kafka Connect 帮助我们处理** **偏移提交过程，**这为我们省去了手动实现连接器开发中这个容易出错的部分的麻烦
*   分布式，默认可扩展——**Kafka Connect 使用现有的组管理协议；我们可以增加更多的工人来扩大 Kafka Connect 集群**
*   流和批处理集成——Kafka Connect 是将流和批处理数据系统与 Kafka 现有功能连接起来的理想解决方案
*   转换——这使我们能够对单个消息进行简单和轻量级的修改

## 4。设置

我们不使用普通的 Kafka 发行版，而是下载 Confluent Platform，这是 Kafka 背后的公司 Confluent，Inc .提供的 Kafka 发行版。与普通的 Kafka 相比，融合平台附带了一些额外的工具和客户端，以及一些额外的预建连接器。

对于我们的情况，开源版本就足够了，可以在 [Confluent 的站点](https://web.archive.org/web/20220913030326/https://www.confluent.io/download/)找到。

## 5。快速启动 Kafka 连接

首先，我们将讨论 Kafka Connect 的原理，**使用其最基本的连接器，即文件`source`连接器和文件`sink`连接器**。

便利的是，融合平台带有这两种连接器以及参考配置。

### 5.1。源连接器配置

对于源连接器，参考配置可在`$CONFLUENT_HOME/etc/kafka/connect-file-source.properties`获得:

```
name=local-file-source
connector.class=FileStreamSource
tasks.max=1
topic=connect-test
file=test.txt
```

此配置具有一些所有源连接器共有的属性:

*   `name`是用户为连接器实例指定的名称
*   指定实现类，基本上是连接器的种类
*   `tasks.max`指定应该并行运行多少个源连接器实例，以及
*   `topic`定义连接器应该向其发送输出的主题

**在这种情况下，我们还有一个特定于连接器的属性:**

*   **`file`定义了连接器应该从哪个文件读取输入**

为此，让我们创建一个包含一些内容的基本文件:

```
echo -e "foo\nbar\n" > $CONFLUENT_HOME/test.txt
```

**注意，工作目录是$CONFLUENT_HOME。**

### 5.2。接收器连接器配置

对于我们的 sink 连接器，我们将使用位于`$CONFLUENT_HOME/etc/kafka/connect-file-sink.properties`的参考配置:

```
name=local-file-sink
connector.class=FileStreamSink
tasks.max=1
file=test.sink.txt
topics=connect-test
```

逻辑上，它包含完全相同的参数，**,不过这次`connector.class`指定了接收器连接器实现，而`file`是连接器应该写入内容的位置。**

### 5.3.工人配置

最后，我们必须配置 Connect worker，它将集成我们的两个连接器，并完成从源连接器读取和向接收器连接器写入的工作。

为此，我们可以使用`$CONFLUENT_HOME/etc/kafka/connect-standalone.properties`:

```
bootstrap.servers=localhost:9092
key.converter=org.apache.kafka.connect.json.JsonConverter
value.converter=org.apache.kafka.connect.json.JsonConverter
key.converter.schemas.enable=false
value.converter.schemas.enable=false
offset.storage.file.filename=/tmp/connect.offsets
offset.flush.interval.ms=10000
plugin.path=/share/java
```

注意,`plugin.path`可以保存一个路径列表，其中连接器实现是可用的

因为我们将使用 Kafka 捆绑的连接器，所以我们可以将`plugin.path`设置为`$CONFLUENT_HOME/share/java`。使用 Windows 时，可能有必要在这里提供一个绝对路径。

对于其他参数，我们可以保留默认值:

*   包含卡夫卡经纪人的地址
*   `key.converter`和`value.converter`定义了转换器类，当数据从源流到 Kafka，然后从 Kafka 流到接收器时，它们对数据进行序列化和反序列化
*   `key.converter.schemas.enable`和`value.converter.schemas.enable`是转换器的特定设置
*   **`offset.storage.file.filename`是在独立模式下运行 Connect 时最重要的设置:它定义了 Connect 应该在哪里存储其偏移数据**
*   `offset.flush.interval.ms`定义工人尝试提交任务补偿的时间间隔

并且参数列表已经相当成熟，可以查看[官方文档](https://web.archive.org/web/20220913030326/https://kafka.apache.org/documentation/#connectconfigs)获得完整列表。

### 5.4。独立模式下的 Kafka Connect

这样，我们就可以开始第一个连接器设置了:

```
$CONFLUENT_HOME/bin/connect-standalone \
  $CONFLUENT_HOME/etc/kafka/connect-standalone.properties \
  $CONFLUENT_HOME/etc/kafka/connect-file-source.properties \
  $CONFLUENT_HOME/etc/kafka/connect-file-sink.properties
```

首先，我们可以使用命令行检查主题的内容:

```
$CONFLUENT_HOME/bin/kafka-console-consumer --bootstrap-server localhost:9092 --topic connect-test --from-beginning
```

我们可以看到，源连接器从`test.txt`文件中获取数据，将其转换为 JSON，并发送给 Kafka:

```
{"schema":{"type":"string","optional":false},"payload":"foo"}
{"schema":{"type":"string","optional":false},"payload":"bar"}
```

如果我们看一下文件夹`$CONFLUENT_HOME`，我们可以看到这里创建了一个文件`test.sink.txt `:

```
cat $CONFLUENT_HOME/test.sink.txt
foo
bar
```

当接收器连接器从`payload`属性中提取值并将其写入目标文件时，`test.sink.txt`中的数据具有原始`test.txt `文件的内容。

现在让我们给`test.txt.`添加更多的行

**当我们这样做时，我们看到源连接器自动检测到这些变化。**

我们只需要确保在末尾插入一个新行，否则，源连接器不会考虑最后一行。

此时，让我们停止连接过程，因为我们将在`distributed mode`的几行中开始连接。

## 6。Connect 的 REST API

到目前为止，我们通过命令行传递属性文件来进行所有的配置。然而，由于 Connect 被设计为作为服务运行，所以也有 REST API 可用。

默认情况下，它在`http://localhost:8083`可用。一些终点是:

*   `GET /connectors` –返回所有正在使用的连接器列表
*   `GET /connectors/{name}`–返回特定连接器的详细信息
*   `POST /connectors`–创建一个新的连接器；请求体应该是一个 JSON 对象，包含一个字符串名称字段和一个带有连接器配置参数的对象配置字段
*   `GET /connectors/{name}/status`–返回连接器的当前状态–包括它是正在运行、失败还是暂停–它被分配给哪个工作线程，如果失败则返回错误信息，以及它的所有任务的状态
*   `DELETE /connectors/{name}`–删除连接器，正常停止所有任务并删除其配置
*   `GET /connector-plugins – `返回 Kafka Connect 集群中安装的连接器插件列表

[官方文档](https://web.archive.org/web/20220913030326/https://kafka.apache.org/documentation/#connect_rest)提供了所有终点的列表。

在下一节中，我们将使用 REST API 来创建新的连接器。

## 7 .**。分布式模式下的 Kafka 连接**

独立模式非常适合开发和测试，以及较小的设置。**然而，如果我们想充分利用 Kafka 的分布式特性，我们必须以分布式模式启动 Connect。**

通过这样做，连接器设置和元数据存储在 Kafka 主题中，而不是文件系统中。因此，工作节点实际上是无状态的。

### 7.1。开始连接

分布式模式的参考配置可在＄CONFLUENT _ HOME`/etc/kafka/connect-distributed.properties.`找到

**参数大多与独立模式相同。区别只有几个:**

*   `group.id`定义连接集群组的名称。该值必须不同于任何使用者组 ID
*   `offset.storage.topic`、`config.storage.topic`和`status.storage.topic`定义这些设置的主题。对于每个主题，我们还可以定义一个复制因子

同样，[官方文档](https://web.archive.org/web/20220913030326/https://kafka.apache.org/documentation/#connectconfigs)提供了所有参数的列表。

我们可以在分布式模式下启动连接，如下所示:

```
$CONFLUENT_HOME/bin/connect-distributed $CONFLUENT_HOME/etc/kafka/connect-distributed.properties
```

### 7.2.使用 REST API 添加连接器

现在，与独立的启动命令相比，我们没有传递任何连接器配置作为参数。相反，我们必须使用 REST API 创建连接器。

为了设置前面的例子，我们必须向`http://localhost:8083/connectors`发送两个包含以下 JSON 结构的 POST 请求。

首先，我们需要为源连接器 POST 创建一个 JSON 文件的主体。在这里，我们称之为`connect-file-source.json`:

```
{
    "name": "local-file-source",
    "config": {
        "connector.class": "FileStreamSource",
        "tasks.max": 1,
        "file": "test-distributed.txt",
        "topic": "connect-distributed"
    }
}
```

**请注意，这看起来与我们第一次使用的参考配置文件非常相似。**

然后我们发布它:

```
curl -d @"$CONFLUENT_HOME/connect-file-source.json" \
  -H "Content-Type: application/json" \
  -X POST http://localhost:8083/connectors
```

然后，我们将对 sink 连接器做同样的事情，调用文件`connect-file-sink.json`:

```
{
    "name": "local-file-sink",
    "config": {
        "connector.class": "FileStreamSink",
        "tasks.max": 1,
        "file": "test-distributed.sink.txt",
        "topics": "connect-distributed"
    }
}
```

并像以前一样执行 POST:

```
curl -d @$CONFLUENT_HOME/connect-file-sink.json \
  -H "Content-Type: application/json" \
  -X POST http://localhost:8083/connectors
```

如果需要，我们可以验证该设置是否正常工作:

```
$CONFLUENT_HOME/bin/kafka-console-consumer --bootstrap-server localhost:9092 --topic connect-distributed --from-beginning
{"schema":{"type":"string","optional":false},"payload":"foo"}
{"schema":{"type":"string","optional":false},"payload":"bar"}
```

如果我们看一下文件夹`$CONFLUENT_HOME`，我们可以看到这里创建了一个文件`test-distributed.sink.txt `:

```
cat $CONFLUENT_HOME/test-distributed.sink.txt
foo
bar
```

在我们测试了分布式设置之后，让我们通过移除两个连接器来进行清理:

```
curl -X DELETE http://localhost:8083/connectors/local-file-source
curl -X DELETE http://localhost:8083/connectors/local-file-sink
```

## 8。转换数据

### 8.1.支持的转换

转换使我们能够对单个消息进行简单和轻量级的修改。

Kafka Connect 支持以下内置转换:

*   `InsertField`–使用静态数据或记录元数据添加字段
*   `ReplaceField`–过滤或重命名字段
*   `MaskField`–将字段替换为该类型的有效空值(例如，零或空字符串)
*   `HoistField`–将整个事件包装为结构或映射中的单个字段
*   `ExtractField`–从结构和映射中提取特定字段，并在结果中仅包含该字段
*   `SetSchemaMetadata`–修改模式名称或版本
*   `TimestampRouter`–根据原始主题和时间戳修改记录的主题
*   `RegexRouter`–根据原始主题、替换字符串和正则表达式修改记录的主题

使用以下参数配置转换:

*   `transforms`–转换别名的逗号分隔列表
*   `transforms.$alias.type`–转换的类名
*   `transforms.$alias.$transformationSpecificConfig`–相应转换的配置

### 8.2.应用变压器

为了测试一些转换特性，让我们设置以下两个转换:

*   首先，让我们将整个消息包装成一个 JSON 结构
*   之后，让我们向该结构添加一个字段

在应用我们的转换之前，我们必须通过修改`connect-distributed.properties`来配置 Connect 使用无模式 JSON:

```
key.converter.schemas.enable=false
value.converter.schemas.enable=false
```

之后，我们必须重新启动 Connect，同样是在分布式模式下:

```
$CONFLUENT_HOME/bin/connect-distributed $CONFLUENT_HOME/etc/kafka/connect-distributed.properties
```

同样，我们需要为源连接器 POST 创建一个 JSON 文件的主体。在这里，我们称之为`connect-file-source-transform.json.`

除了已知的参数之外，我们为两个必需的转换添加了几行:

```
{
    "name": "local-file-source",
    "config": {
        "connector.class": "FileStreamSource",
        "tasks.max": 1,
        "file": "test-transformation.txt",
        "topic": "connect-transformation",
        "transforms": "MakeMap,InsertSource",
        "transforms.MakeMap.type": "org.apache.kafka.connect.transforms.HoistField$Value",
        "transforms.MakeMap.field": "line",
        "transforms.InsertSource.type": "org.apache.kafka.connect.transforms.InsertField$Value",
        "transforms.InsertSource.static.field": "data_source",
        "transforms.InsertSource.static.value": "test-file-source"
    }
}
```

之后，我们来执行 POST:

```
curl -d @$CONFLUENT_HOME/connect-file-source-transform.json \
  -H "Content-Type: application/json" \
  -X POST http://localhost:8083/connectors
```

让我们给我们的`test-transformation.txt`写几行:

```
Foo
Bar
```

如果我们现在检查`connect-transformation`主题，我们应该得到以下几行:

```
{"line":"Foo","data_source":"test-file-source"}
{"line":"Bar","data_source":"test-file-source"}
```

## 9。使用就绪连接器

使用完这些简单的连接器后，让我们看看更高级的现成连接器，以及如何安装它们。

### 9.1.哪里可以找到连接器

**预制连接器可从不同来源获得:**

*   一些连接器与普通的 Apache Kafka(文件和控制台的源和接收器)捆绑在一起
*   更多的连接器与融合平台捆绑在一起(ElasticSearch、HDFS、JDBC 和 AWS S3)
*   还可以看看[汇合枢纽](https://web.archive.org/web/20220913030326/https://www.confluent.io/hub/)，这是一种 Kafka 连接器的应用程序商店。提供的连接器数量持续增长:
    *   合流连接器(开发、测试、记录并由合流完全支持)
    *   经认证的连接器(由第三方实施并经合流认证)
    *   社区开发和支持的连接器
*   除此之外，Confluent 还提供了一个[连接器页面](https://web.archive.org/web/20220913030326/https://www.confluent.io/product/connectors/)，其中有些连接器在 Confluent Hub 上也是可用的，但也有更多的社区连接器
*   最后，还有一些供应商提供连接器作为他们产品的一部分。例如，Landoop 提供了一个名为[lens](https://web.archive.org/web/20220913030326/https://www.landoop.com/downloads/)的流库，其中也包含一组大约 25 个开源连接器(其中许多也在其他地方交叉列出)

### 9.2。从汇流毂安装连接器

Confluent 的企业版提供了从 Confluent Hub 安装连接器和其他组件的脚本(该脚本不包含在开源版本中)。如果我们使用企业版，我们可以使用以下命令安装连接器:

```
$CONFLUENT_HOME/bin/confluent-hub install confluentinc/kafka-connect-mqtt:1.0.0-preview
```

### 9.3。手动安装连接器

如果我们需要一个连接器，这在合流集线器上是不可用的，或者如果我们有合流的开源版本，我们可以手动安装所需的连接器。为此，我们必须下载并解压缩连接器，并将包含的库移动到指定为`plugin.path.`的文件夹中

对于每个连接器，归档应该包含两个我们感兴趣的文件夹:

*   `lib`文件夹包含连接器 jar，例如`kafka-connect-mqtt-1.0.0-preview.jar`，以及连接器需要的更多 jar
*   `etc`文件夹包含一个或多个参考配置文件

**我们必须将`lib`文件夹移动到`$CONFLUENT_HOME/share/java`，或者我们在`connect-standalone.properties`和`connect-distributed.properties`中指定为`plugin.path`的路径。这样做时，将文件夹重命名为有意义的名称也是有意义的。**

我们可以使用来自`etc`的配置文件，既可以在独立模式下启动时引用它们，也可以获取属性并从它们创建一个 JSON 文件。

## 10。结论

在本教程中，我们了解了如何安装和使用 Kafka Connect。

我们看了各种类型的连接器，包括源和接收器。我们还了解了 Connect 可以运行的一些特性和模式。然后，我们复习了《变形金刚》。最后，我们学习了从哪里获得以及如何安装定制连接器。

和往常一样，配置文件可以在 GitHub 的[中找到。](https://web.archive.org/web/20220913030326/https://github.com/eugenp/tutorials/tree/master/apache-kafka)