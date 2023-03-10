# 在 Java 中使用 InfluxDB

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-influxdb>

## 1。概述

**[InfluxDB](https://web.archive.org/web/20221222164641/https://www.influxdata.com/) 是时序数据的高性能存储。**通过类似 SQL 的查询语言，支持数据的插入和实时查询。

在这篇介绍性文章中，我们将演示如何连接到 InfluxDb 服务器，创建数据库，写入时间序列信息，然后查询数据库。

## 2。设置

为了连接到数据库，我们需要向我们的`pom.xml`文件添加一个条目:

```java
<dependency>
    <groupId>org.influxdb</groupId>
    <artifactId>influxdb-java</artifactId>
    <version>2.8</version>
</dependency> 
```

这个依赖关系的最新版本可以在 [Maven Central](https://web.archive.org/web/20221222164641/https://search.maven.org/classic/#search%7Cgav%7C1%7Cg%3A%22org.influxdb%22%20AND%20a%3A%22influxdb-java%22) 上找到。

我们还需要一个 InfluxDB 实例。关于[下载和安装数据库](https://web.archive.org/web/20221222164641/https://docs.influxdata.com/influxdb/v1.4/introduction/installation/)的说明可以在 [InfluxData](https://web.archive.org/web/20221222164641/https://www.influxdata.com/) 网站上找到。

## 3。连接到服务器

### 3.1。创建连接

创建数据库连接需要将 URL `String`和用户凭证传递给连接工厂:

```java
InfluxDB influxDB = InfluxDBFactory.connect(databaseURL, userName, password);
```

### 3.2。验证连接

与数据库的通信是通过 RESTful API 执行的，所以它们不是持久的。

API 提供了一个专用的“ping”服务来确认连接是否正常。如果连接良好，则响应包含数据库版本。如果不是，则包含`“unknown”.`

因此，在创建连接之后，我们可以通过执行以下操作来验证它:

```java
Pong response = this.influxDB.ping();
if (response.getVersion().equalsIgnoreCase("unknown")) {
    log.error("Error pinging server.");
    return;
} 
```

### 3.3。创建数据库

创建 InfluxDB 数据库类似于在大多数平台上创建数据库。但是在使用之前，我们需要至少创建一个保留策略。

**保留策略告诉数据库一段数据应该存储多长时间。**时间序列，如 CPU 或内存统计数据，往往会在大型数据集中积累。

控制时间序列数据库大小的典型策略是`downsampling`。“原始”数据被高速存储、汇总，然后在短时间内删除。

**保留策略通过将数据与到期时间相关联来简化这一过程。** InfluxData 在其网站上有一个[深度解释](https://web.archive.org/web/20221222164641/https://docs.influxdata.com/influxdb/v1.4/guides/downsampling_and_retention/)。

创建数据库后，我们将添加一个名为`defaultPolicy.`的策略，它将简单地将数据保留 30 天:

```java
influxDB.createDatabase("baeldung");
influxDB.createRetentionPolicy(
  "defaultPolicy", "baeldung", "30d", 1, true);
```

要创建一个保留策略，我们需要一个`name,`、`database,`、`interval,`、`replication factor`(对于单实例数据库应该是 1)和一个`boolean`，表示这是一个默认策略。

### 3.4。设置记录级别

在内部，InfluxDB API 通过一个[日志拦截器使用](/web/20221222164641/https://www.baeldung.com/retrofit#logging)[改型](/web/20221222164641/https://www.baeldung.com/retrofit)并公开一个接口给改型的日志工具。

因此，我们可以使用以下命令设置日志记录级别:

```java
influxDB.setLogLevel(InfluxDB.LogLevel.BASIC); 
```

现在，当我们打开一个连接并 ping 它时，我们可以看到消息:

```java
Dec 20, 2017 5:38:10 PM okhttp3.internal.platform.Platform log
INFO: --> GET http://127.0.0.1:8086/ping
```

可用的级别有`BASIC`、`FULL`、`HEADERS`和`NONE.`

## 4。添加和检索数据

### 4.1。点数

所以现在我们准备开始插入和检索数据。

InfluxDB 中的基本信息单元是一个`Point,`，它本质上是一个时间戳和一个键值映射。

让我们看一下保存内存利用率数据的点:

```java
Point point = Point.measurement("memory")
  .time(System.currentTimeMillis(), TimeUnit.MILLISECONDS)
  .addField("name", "server1")
  .addField("free", 4743656L)
  .addField("used", 1015096L)
  .addField("buffer", 1010467L)
  .build(); 
```

我们已经创建了一个条目，包含三个`Longs`作为内存统计数据、一个主机名和一个时间戳。

让我们看看如何将它添加到数据库中。

### 4.2。写批次

时间序列数据往往由许多小点组成，一次写一条记录效率很低。首选方法是批量收集记录。

InfluxDB API 提供了一个`BatchPoint` 对象:

```java
BatchPoints batchPoints = BatchPoints
  .database(dbName)
  .retentionPolicy("defaultPolicy")
  .build();

Point point1 = Point.measurement("memory")
  .time(System.currentTimeMillis(), TimeUnit.MILLISECONDS)
  .addField("name", "server1") 
  .addField("free", 4743656L)
  .addField("used", 1015096L) 
  .addField("buffer", 1010467L)
  .build();

Point point2 = Point.measurement("memory")
  .time(System.currentTimeMillis() - 100, TimeUnit.MILLISECONDS)
  .addField("name", "server1")
  .addField("free", 4743696L)
  .addField("used", 1016096L)
  .addField("buffer", 1008467L)
  .build();

batchPoints.point(point1);
batchPoints.point(point2);
influxDB.write(batchPoints);
```

我们创建一个`BatchPoint`,然后给它添加`Points`。我们将第二个条目的时间戳设置为过去 100 毫秒，因为时间戳是主索引。如果我们用相同的时间戳发送两个点，只有一个点会被保留。

注意，我们必须将`BatchPoints`与数据库和保留策略相关联。

### 4.3。一次写一个

对于某些用例，批处理可能不切实际。

让我们通过对 InfluxDB 连接的一个调用来启用批处理模式:

```java
influxDB.enableBatch(100, 200, TimeUnit.MILLISECONDS); 
```

我们启用了 100 的批处理来插入到服务器中，或者每 200 毫秒发送一次。

启用批处理模式后，我们仍然可以一次写一个。但是，需要一些额外的设置:

```java
influxDB.setRetentionPolicy("defaultPolicy");
influxDB.setDatabase(dbName); 
```

此外，现在我们可以编写个人积分，它们正被一个后台线程批量收集:

```java
influxDB.write(point); 
```

**在我们对各个点进行排队之前，我们需要设置一个数据库**(类似于 SQL 中的`use`命令)**并设置一个默认的保留策略。**因此，如果我们希望利用具有多种保留策略的缩减采样，那么创建批处理就是一种方法。

**批处理模式利用一个单独的线程池。因此，当不再需要它时，最好禁用它:**

```java
influxDB.disableBatch(); 
```

关闭连接也将关闭线程池:

```java
influxDB.close();
```

### 4.4。映射查询结果

**查询返回一个`QueryResult`，我们可以将它映射到 POJOs。**

在我们查看查询语法之前，让我们创建一个类来保存内存统计信息:

```java
@Measurement(name = "memory")
public class MemoryPoint {

    @Column(name = "time")
    private Instant time;

    @Column(name = "name")
    private String name;

    @Column(name = "free")
    private Long free;

    @Column(name = "used")
    private Long used;

    @Column(name = "buffer")
    private Long buffer;
} 
```

这个类用`@Measurement(name = “memory”)`标注，对应于我们用来创建`Points`的`Point.measurement(“memory”)`。

对于我们的`QueryResult`中的每个字段，我们添加带有相应字段名称的`@Column(name = “XXX”)`注释。

`QueryResults`映射到带有`InfluxDBResultMapper.`的 POJOs

### 4.5。查询 InfluxDB

因此，让我们将 POJO 与我们在两点批处理中添加到数据库的点一起使用:

```java
QueryResult queryResult = connection
  .performQuery("Select * from memory", "baeldung");

InfluxDBResultMapper resultMapper = new InfluxDBResultMapper();
List<MemoryPoint> memoryPointList = resultMapper
  .toPOJO(queryResult, MemoryPoint.class);

assertEquals(2, memoryPointList.size());
assertTrue(4743696L == memoryPointList.get(0).getFree()); 
```

该查询说明了如何将我们名为`memory`的度量存储为一个包含`Points`的表，我们可以从该表中`select`。

`InfluxDBResultMapper`用`QueryResult`接受对`MemoryPoint.class`的引用，并返回一个点列表。

映射结果后，我们通过检查从查询中收到的`List`的长度来验证我们收到了两个结果。然后我们查看列表中的第一个条目，并查看我们插入的第二个点的空闲内存大小。**influx db 查询结果的默认排序是时间戳升序。**

让我们改变这一点:

```java
queryResult = connection.performQuery(
  "Select * from memory order by time desc", "baeldung");
memoryPointList = resultMapper
  .toPOJO(queryResult, MemoryPoint.class);

assertEquals(2, memoryPointList.size());
assertTrue(4743656L == memoryPointList.get(0).getFree()); 
```

**加上`order by time desc`颠倒我们结果的顺序。**

InfluxDB 查询看起来非常类似于 SQL。他们的网站上有大量的参考指南[。](https://web.archive.org/web/20221222164641/https://docs.influxdata.com/influxdb/v1.4/guides/querying_data/)

## 5。结论

我们已经连接到一个 InfluxDB 服务器，创建了一个带有保留策略的数据库，然后从服务器插入和检索数据。

示例的完整源代码在 GitHub 上的[。](https://web.archive.org/web/20221222164641/https://github.com/eugenp/tutorials/tree/master/persistence-modules/influxdb)