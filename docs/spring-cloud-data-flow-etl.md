# 基于 Spring Cloud 数据流的 ETL

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-cloud-data-flow-etl>

## 1.概观

[Spring Cloud Data Flow](https://web.archive.org/web/20221126222258/https://cloud.spring.io/spring-cloud-dataflow/) 是一个用于构建实时数据管道和批处理的云原生工具包。Spring Cloud 数据流可用于一系列数据处理用例，如简单的导入/导出、ETL 处理、事件流和预测分析。

在本教程中，我们将学习一个使用流管道的实时提取、转换和加载(ETL)的示例，该示例从 JDBC 数据库中提取数据，将其转换为简单的 POJOs，并将其加载到 MongoDB 中。

## 2.ETL 和事件流处理

ETL——提取、转换和加载——通常指的是将数据从几个数据库和系统批量加载到一个公共数据仓库的过程。在这个数据仓库中，可以进行繁重的数据分析处理，而不会影响系统的整体性能。

然而，新的趋势正在改变这种方式。ETL 仍然在将数据传输到数据仓库和数据湖中发挥作用。

如今，在 Spring Cloud 数据流的帮助下，这可以通过事件流架构中的**流来完成。**

## 3.春季云数据流

借助 Spring Cloud Data Flow (SCDF ),开发人员可以创建两种风格的数据管道:

*   使用 Spring Cloud Stream 的长寿实时流应用程序
*   使用 Spring 云任务的短期批处理任务应用程序

在本文中，我们将讨论第一个，一个基于 Spring Cloud Stream 的长期流媒体应用程序。

### 3.1.Spring 云流应用程序

SCDF 流管道由多个步骤组成，**其中** **每个步骤都是使用 Spring Cloud Stream 微框架以 Spring Boot 风格构建的应用程序。**这些应用通过 Apache Kafka 或 RabbitMQ 等消息中间件进行集成。

这些应用程序分为源、处理器和接收器。与 ETL 过程相比，我们可以说源是“提取”，处理器是“转换器”，接收器是“加载”部分。

在某些情况下，我们可以在管道的一个或多个步骤中使用应用程序启动器。这意味着我们不需要为一个步骤实现一个新的应用程序，而是配置一个已经实现的现有应用程序启动器。

**可在[此处](https://web.archive.org/web/20221126222258/https://cloud.spring.io/spring-cloud-stream-app-starters/)找到应用启动者列表。**

### 3.2.Spring 云数据流服务器

**架构的最后一块是 Spring Cloud 数据流服务器**。SCDF 服务器使用 Spring Cloud Deployer 规范来部署应用程序和管道流。该规范通过部署到一系列现代运行时，如 Kubernetes、Apache Mesos、Yarn 和 Cloud Foundry，支持 SCDF 的云原生风格。

此外，我们可以将流作为本地部署来运行。

更多关于 SCDF 建筑的信息可以在这里找到。

## 4.环境设置

在我们开始之前，我们需要**选择这个复杂部署的各个部分**。首先要定义的是 SCDF 服务器。

对于测试，**我们将使用 SCDF 本地服务器进行本地开发**。对于生产部署，我们可以稍后选择一个云原生运行时，如 [SCDF 服务器 Kubernetes](https://web.archive.org/web/20221126222258/https://cloud.spring.io/spring-cloud-dataflow-server-kubernetes/) 。我们可以在这里找到服务器运行时列表。

现在，让我们检查运行该服务器的系统要求。

### 4.1.系统需求

要运行 SCDF 服务器，我们必须定义并设置两个依赖项:

*   消息中间件，以及
*   关系数据库管理系统。

对于消息中间件，**我们将使用 RabbitMQ，我们选择 PostgreSQL 作为 RDBMS** 来存储我们的管道流定义。

要运行 RabbitMQ，请在此下载最新版本[并使用默认配置启动 RabbitMQ 实例，或者运行以下 Docker 命令:](https://web.archive.org/web/20221126222258/https://www.rabbitmq.com/download.html)

```java
docker run --name dataflow-rabbit -p 15672:15672 -p 5672:5672 -d rabbitmq:3-management
```

作为最后一个设置步骤，在默认端口 5432 上安装并运行 PostgreSQL RDBMS。之后，使用以下脚本创建一个数据库，SCDF 可以在其中存储其流定义:

```java
CREATE DATABASE dataflow;
```

### 4.2.Spring Cloud 数据流服务器本地

为了在本地运行 SCDF 服务器，我们可以选择使用`docker-compose`、来启动服务器[，或者我们可以将它作为一个 Java 应用程序来启动。](https://web.archive.org/web/20221126222258/https://docs.spring.io/spring-cloud-dataflow/docs/current/reference/htmlsingle/#getting-started-deploying-spring-cloud-dataflow-docker)

在这里，我们将 SCDF 服务器作为 Java 应用程序在本地运行。为了配置应用程序，我们必须将配置定义为 Java 应用程序参数。我们需要在系统路径中安装 Java 8。

为了托管 jar 和依赖项，我们需要为我们的 SCDF 服务器创建一个主文件夹，并将 SCDF 服务器本地发行版下载到这个文件夹中。你可以在这里下载 SCDF 服务器本地[的最新发行版。](https://web.archive.org/web/20221126222258/https://cloud.spring.io/spring-cloud-dataflow/)

此外，我们需要创建一个 lib 文件夹，并在其中放置一个 JDBC 驱动程序。最新版本的 PostgreSQL 驱动可以在[这里](https://web.archive.org/web/20221126222258/https://jdbc.postgresql.org/download.html#current)获得。

最后，让我们运行 SCDF 本地服务器:

```java
$java -Dloader.path=lib -jar spring-cloud-dataflow-server-local-1.6.3.RELEASE.jar \
    --spring.datasource.url=jdbc:postgresql://127.0.0.1:5432/dataflow \
    --spring.datasource.username=postgres_username \
    --spring.datasource.password=postgres_password \
    --spring.datasource.driver-class-name=org.postgresql.Driver \
    --spring.rabbitmq.host=127.0.0.1 \
    --spring.rabbitmq.port=5672 \
    --spring.rabbitmq.username=guest \
    --spring.rabbitmq.password=guest
```

我们可以通过查看以下 URL 来检查它是否正在运行:

[http://localhost:9393/dashboard](https://web.archive.org/web/20221126222258/http://localhost:9393/dashboard)

### 4.3.Spring 云数据流外壳

SCDF Shell 是一个**命令行工具，它使得组合和部署我们的应用程序和管道**变得容易。这些 Shell 命令在 Spring Cloud 数据流服务器 [REST API](https://web.archive.org/web/20221126222258/https://docs.spring.io/spring-cloud-dataflow/docs/current-SNAPSHOT/reference/htmlsingle/#api-guide-overview) 上运行。

将 jar 的最新版本下载到你的 SCDF 主文件夹中，可以在这里找到。完成后，运行以下命令(根据需要更新版本):

```java
$ java -jar spring-cloud-dataflow-shell-1.6.3.RELEASE.jar
  ____                              ____ _                __
 / ___| _ __  _ __(_)_ __   __ _   / ___| | ___  _   _  __| |
 \___ \| '_ \| '__| | '_ \ / _` | | |   | |/ _ \| | | |/ _` |
  ___) | |_) | |  | | | | | (_| | | |___| | (_) | |_| | (_| |
 |____/| .__/|_|  |_|_| |_|\__, |  \____|_|\___/ \__,_|\__,_|
  ____ |_|    _          __|___/                 __________
 |  _ \  __ _| |_ __ _  |  ___| | _____      __  \ \ \ \ \ \
 | | | |/ _` | __/ _` | | |_  | |/ _ \ \ /\ / /   \ \ \ \ \ \
 | |_| | (_| | || (_| | |  _| | | (_) \ V  V /    / / / / / /
 |____/ \__,_|\__\__,_| |_|   |_|\___/ \_/\_/    /_/_/_/_/_/

Welcome to the Spring Cloud Data Flow shell. For assistance hit TAB or type "help".
dataflow:>
```

如果在最后一行得到的不是"`dataflow:>”`，而是"`server-unknown:>”` ，那么您没有在本地主机上运行 SCDF 服务器。在这种情况下，运行以下命令连接到另一台主机:

```java
server-unknown:>dataflow config server http://{host}
```

现在，Shell 连接到了 SCDF 服务器，我们可以运行我们的命令了。

在 Shell 中，我们需要做的第一件事是导入应用程序启动器。在这里找到 Spring Boot 2.0.x 中 RabbitMQ+Maven 的最新版本[，运行下面的命令(再次更新版本，这里`Darwin-SR1`，根据需要):](https://web.archive.org/web/20221126222258/https://cloud.spring.io/spring-cloud-stream-app-starters/)

```java
$ dataflow:>app import --uri http://bit.ly/Darwin-SR1-stream-applications-rabbit-maven
```

要检查已安装的应用程序，请运行以下 Shell 命令:

```java
$ dataflow:> app list
```

因此，我们应该会看到一个包含所有已安装应用程序的表格。

另外，SCDF 提供了一个名为`Flo`的图形界面，我们可以通过这个地址`http://localhost:9393/dashboard`访问它。然而，它的使用不在本文的讨论范围之内。

## 5.组成 ETL 管道

现在让我们创建我们的流管道。为此，我们将使用 JDBC 源应用程序启动器从关系数据库中提取信息。

此外，我们将创建一个定制的处理器来转换信息结构，并创建一个定制的接收器来将数据加载到 MongoDB 中。

### 5.1.提取–为提取准备关系数据库

让我们创建一个名为`crm`的数据库和一个名为`customer`的表:

```java
CREATE DATABASE crm;
```

```java
CREATE TABLE customer (
    id bigint NOT NULL,
    imported boolean DEFAULT false,
    customer_name character varying(50),
    PRIMARY KEY(id)
)
```

注意，我们使用了一个标志`imported`，它将存储已经导入的记录。如果需要，我们还可以将这些信息存储在另一个表中。

现在，让我们插入一些数据:

```java
INSERT INTO customer(id, customer_name, imported) VALUES (1, 'John Doe', false);
```

### 5.2.转换——将`JDBC`字段映射到`MongoDB`字段结构

对于转换步骤，我们将简单地将源表中的字段`customer_name`转换成新的字段`name`。这里还可以进行其他转换，但是让我们保持例子简短。

**为此，我们将创建一个名为`customer-transform`的新项目。**最简单的方法是使用 [Spring Initializr](https://web.archive.org/web/20221126222258/https://start.spring.io/) 站点创建项目。到达网站后，选择一个组和一个工件名称。我们将分别使用`com.customer`和`customer-transform,`。

完成后，点击“生成项目”按钮下载项目。然后，解压缩该项目并将其导入到您喜欢的 IDE 中，并将以下依赖项添加到`pom.xml`:

```java
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-stream-binder-rabbit</artifactId>
</dependency>
```

现在我们开始对字段名转换进行编码。为此，我们将创建`Customer`类作为适配器。这个类将通过`setName()`方法接收`customer_name`，并通过`getName`方法`.` 输出它的值

`@JsonProperty `注释将在从 JSON 到 Java 的反序列化过程中进行转换:

```java
public class Customer {

    private Long id;

    private String name;

    @JsonProperty("customer_name")
    public void setName(String name) {
        this.name = name;
    }

    @JsonProperty("name")
    public String getName() {
        return name;
    }

    // Getters and Setters
}
```

处理器需要从输入端接收数据，进行转换，并将结果绑定到输出通道。让我们创建一个类来完成这项工作:

```java
import org.springframework.cloud.stream.annotation.EnableBinding;
import org.springframework.cloud.stream.messaging.Processor;
import org.springframework.integration.annotation.Transformer;

@EnableBinding(Processor.class)
public class CustomerProcessorConfiguration {

    @Transformer(inputChannel = Processor.INPUT, outputChannel = Processor.OUTPUT)
    public Customer convertToPojo(Customer payload) {

        return payload;
    }
}
```

在上面的代码中，我们可以观察到转换是自动发生的。输入接收 JSON 形式的数据，Jackson 使用`set`方法将其反序列化为一个`Customer`对象。

与输出相反，使用`get`方法将数据序列化为 JSON。

### 5.3.MongoDB 中的加载-接收

类似于转换步骤，**我们将创建另一个 maven 项目，现在命名为`customer-` `mongodb` `-sink`** 。再次访问 [Spring Initializr](https://web.archive.org/web/20221126222258/https://start.spring.io/) ，为组选择`com.customer`，为工件选择`customer-mongodb-sink`。然后，在 dependencies 搜索框中键入 **"** MongoDB **"** 并下载项目。

接下来，将其解压缩并导入到您喜欢的 IDE 中。

然后，添加与`customer-transform`项目中相同的额外依赖项。

现在我们将创建另一个`Customer`类，用于在这一步接收输入:

```java
import org.springframework.data.mongodb.core.mapping.Document;

@Document(collection="customer")
public class Customer {

    private Long id;
    private String name;

    // Getters and Setters
}
```

为了接收`Customer`，我们将创建一个监听器类，它将使用`CustomerRepository`保存客户实体:

```java
@EnableBinding(Sink.class)
public class CustomerListener {

    @Autowired
    private CustomerRepository repository;

    @StreamListener(Sink.INPUT)
    public void save(Customer customer) {
        repository.save(customer);
    }
}
```

在这种情况下，`CustomerRepository`是来自 Spring 数据的`MongoRepository`:

```java
import org.springframework.data.mongodb.repository.MongoRepository;
import org.springframework.stereotype.Repository;

@Repository
public interface CustomerRepository extends MongoRepository<Customer, Long> {

} 
```

### 5.4.流定义

现在，**两个定制应用程序都准备好在 SCDF 服务器上注册了。**要做到这一点，使用 Maven 命令`mvn install`编译这两个项目。

然后，我们使用 Spring Cloud 数据流 Shell 注册它们:

```java
app register --name customer-transform --type processor --uri maven://com.customer:customer-transform:0.0.1-SNAPSHOT
```

```java
app register --name customer-mongodb-sink --type sink --uri maven://com.customer:customer-mongodb-sink:jar:0.0.1-SNAPSHOT
```

最后，让我们检查应用程序是否存储在 SCDF，在 shell 中运行应用程序列表命令:

```java
app list
```

因此，我们应该在结果表中看到这两个应用程序。

### 5.4.1.流管道领域特定语言–DSL

DSL 定义了应用程序之间的配置和数据流。SCDF DSL 很简单。在第一个词中，我们定义了应用程序的名称，然后是配置。

此外，该语法是受 Unix 启发的[管道语法](https://web.archive.org/web/20221126222258/https://en.wikipedia.org/wiki/Pipeline_(Unix))，它使用竖线(也称为“管道”)来连接多个应用程序:

```java
http --port=8181 | log
```

这将创建一个在端口 8181 上提供服务的 HTTP 应用程序，它将任何接收到的主体有效负载发送到一个日志中。

现在，让我们看看如何创建 JDBC 源的 DSL 流定义。

### 5.4.2.JDBC 源流定义

JDBC 源的关键配置是`query`和`update`。 **`query`将选择未读记录，而`update`将改变一个标志，以防止当前记录被重读。**

此外，我们将定义 JDBC 源以 30 秒的固定延迟进行轮询，最多轮询 1000 行。最后，我们将定义连接的配置，如驱动程序、用户名、密码和连接 URL:

```java
jdbc 
    --query='SELECT id, customer_name FROM public.customer WHERE imported = false'
    --update='UPDATE public.customer SET imported = true WHERE id in (:id)'
    --max-rows-per-poll=1000
    --fixed-delay=30 --time-unit=SECONDS
    --driver-class-name=org.postgresql.Driver
    --url=jdbc:postgresql://localhost:5432/crm
    --username=postgres
    --password=postgres
```

更多 JDBC 源配置属性可在[这里](https://web.archive.org/web/20221126222258/https://docs.spring.io/spring-cloud-stream-app-starters/docs/Celsius.SR2/reference/htmlsingle/#spring-cloud-stream-modules-jdbc-source)找到。

### 5.4.3.客户 MongoDB 接收器流定义

由于我们没有在`customer-mongodb-sink`的`application.properties`中定义连接配置，我们将通过 DSL 参数进行配置。

我们的应用完全基于`MongoDataAutoConfiguration.` 你可以在这里查看其他可能的配置[。](https://web.archive.org/web/20221126222258/https://docs.spring.io/spring-boot/docs/current/reference/html/boot-features-nosql.html#boot-features-mongodb)基本上，我们将定义`spring.data.mongodb.uri`:

```java
customer-mongodb-sink --spring.data.mongodb.uri=mongodb://localhost/main
```

### 5.4.4.创建和部署流

首先，为了创建最终的流定义，返回到 Shell 并执行下面的命令(没有换行符，它们只是为了可读性而插入的):

```java
stream create --name jdbc-to-mongodb 
  --definition "jdbc 
  --query='SELECT id, customer_name FROM public.customer WHERE imported=false' 
  --fixed-delay=30 
  --max-rows-per-poll=1000 
  --update='UPDATE customer SET imported=true WHERE id in (:id)' 
  --time-unit=SECONDS 
  --password=postgres 
  --driver-class-name=org.postgresql.Driver 
  --username=postgres 
  --url=jdbc:postgresql://localhost:5432/crm | customer-transform | customer-mongodb-sink 
  --spring.data.mongodb.uri=mongodb://localhost/main" 
```

这个流 DSL 定义了一个名为 jdbc `-to-` mongodb 的流。接下来，**我们将按照它的名字**来部署这个流:

```java
stream deploy --name jdbc-to-mongodb 
```

最后，我们应该在日志输出中看到所有可用日志的位置:

```java
Logs will be in {PATH_TO_LOG}/spring-cloud-deployer/jdbc-to-mongodb/jdbc-to-mongodb.customer-mongodb-sink

Logs will be in {PATH_TO_LOG}/spring-cloud-deployer/jdbc-to-mongodb/jdbc-to-mongodb.customer-transform

Logs will be in {PATH_TO_LOG}/spring-cloud-deployer/jdbc-to-mongodb/jdbc-to-mongodb.jdbc
```

## 6.结论

在本文中，我们看到了使用 Spring Cloud 数据流的 ETL 数据管道的完整示例。

最值得注意的是，我们看到了一个应用程序启动器的配置，使用 Spring Cloud 数据流外壳创建了一个 ETL 流管道，并实现了用于读取、转换和写入数据的定制应用程序。

和往常一样，示例代码可以在 GitHub 项目中找到[。](https://web.archive.org/web/20221126222258/https://github.com/eugenp/tutorials/tree/master/spring-cloud-modules/spring-cloud-data-flow/spring-cloud-data-flow-etl)