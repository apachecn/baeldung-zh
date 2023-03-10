# Apache Spark 简介

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/apache-spark>

## 1。简介

Apache Spark 是一个开源的集群计算框架。它为 Scala、Java、Python 和 R 提供了优雅的开发 API，允许开发人员跨不同的数据源执行各种数据密集型工作负载，包括 HDFS、卡珊德拉、HBase、S3 等。

历史上，Hadoop 的 MapReduce 被证明对于一些迭代和交互式计算工作效率低下，这最终导致了 Spark 的开发。**有了 Spark，我们在内存中运行逻辑的速度比 Hadoop 快两个数量级，在磁盘上快一个数量级**。

## 2。火花架构

Spark 应用程序在集群上作为独立的进程集运行，如下图[所示](https://web.archive.org/web/20220926185023/https://spark.apache.org/docs/latest/cluster-overview.html):

[![cluster overview](img/4d8b83d3c560f6e72e37897cab114579.png)](/web/20220926185023/https://www.baeldung.com/wp-content/uploads/2017/10/cluster-overview.png)

这些进程由主程序中的`SparkContext`对象协调(称为驱动程序)。`SparkContext` 连接到几种类型的集群管理器(Spark 自己的独立集群管理器、Mesos 或 YARN)，这些管理器跨应用程序分配资源。

一旦连接上，Spark 就会获得集群中节点上的执行器，这些执行器是为应用程序运行计算和存储数据的进程。

接下来，它将您的应用程序代码(由传递给`SparkContext`的 JAR 或 Python 文件定义)发送给执行器。最后，**T1 发送任务给执行者运行**。

## 3。核心部件

下图[给出了 Spark 不同组件的清晰图片:](https://web.archive.org/web/20220926185023/https://intellipaat.com/tutorial/spark-tutorial/apache-spark-components/)

[![Components of Spark](img/a4e6d2c26396d93d9c771e934d6f7e95.png)](/web/20220926185023/https://www.baeldung.com/wp-content/uploads/2017/10/Components-of-Spark.jpg)

### 3.1。火花核心

Spark 核心组件负责所有基本的 I/O 功能，调度和监控 spark 集群上的作业、任务调度、与不同存储系统联网、故障恢复以及高效的内存管理。

与 Hadoop 不同，Spark 通过使用一种称为 RDD(弹性分布式数据集)的特殊数据结构，避免了将共享数据存储在亚马逊 S3 或 HDFS 等中间存储中。

**弹性分布式数据集是不可变的，是可以并行操作的记录的分区集合，并允许容错的“内存中”计算**。

rdd 支持两种操作:

*   转换——火花 RDD 转换是从现有 RDD 产生新 RDD 的功能。变压器将 RDD 作为输入，并产生一个或多个 RDD 作为输出。转换本质上是懒惰的，也就是说，当我们调用一个动作时，它们就会被执行

*   动作**–**转换从彼此创建 rdd，但是当我们想要处理实际的数据集时，就在那时执行动作。因此， **`Actions`是给出非 RDD 值的火花 RDD 运算。**动作值存储到驱动程序或外部存储系统

动作是从执行器向驱动程序发送数据的方式之一。

执行者是负责执行任务的代理。而驱动程序是一个 JVM 进程，它协调工人和任务的执行。Spark 的一些操作是计数和收集。

### 3.2。火花 SQL

Spark SQL 是用于结构化数据处理的 Spark 模块。它主要用于执行 SQL 查询。构成了 Spark SQL 的主要抽象。在 Spark 中，被排序到指定列中的数据的分布式集合被称为`DataFrame`。

Spark SQL 支持从不同的数据源获取数据，比如 Hive、Avro、Parquet、ORC、JSON 和 JDBC。它还使用 Spark 引擎扩展到数千个节点和数小时的查询——提供完全的中间查询容错。

### 3.3。火花流

Spark Streaming 是核心 Spark API 的扩展，支持实时数据流的可伸缩、高吞吐量、容错流处理。可以从许多来源获取数据，如 Kafka、Flume、Kinesis 或 TCP 套接字。

最后，经过处理的数据可以被推送到文件系统、数据库和实时控制面板。

### 3.4。火花 Mlib

MLlib 是 Spark 的机器学习(ML)库。它的目标是让实用的机器学习变得可扩展和简单。在高层次上，它提供了如下工具:

*   ML 算法——常见的学习算法，如分类、回归、聚类和协同过滤
*   特征化——特征提取、转换、维度缩减和选择
*   管道——用于构建、评估和调整 ML 管道的工具
*   持久性——保存和加载算法、模型和管道
*   实用工具——线性代数、统计学、数据处理等。

### 3.5。火花图 X

GraphX 是用于图形和图形并行计算的组件。在高层次上，GraphX 通过引入一个新的图形抽象扩展了 Spark RDD:一个有向多图，每个顶点和边都有属性。

为了支持图形计算，GraphX 公开了一组基本操作符(例如，`subgraph`、`joinVertices`和`aggregateMessages`)。

此外，GraphX 还包含了越来越多的图形算法和构建器来简化图形分析任务。

## 4。《Hello World》中的 Spark

现在我们已经了解了核心组件，我们可以继续进行简单的基于 Maven 的 Spark 项目——**来计算字数**。

我们将演示 Spark 在本地模式下运行，其中所有组件都在同一台机器上本地运行，它是主节点、执行器节点或 Spark 的独立集群管理器。

### 4.1。Maven 设置

让我们用`pom.xml`文件中的 [Spark 相关依赖项](https://web.archive.org/web/20220926185023/https://search.maven.org/classic/#search%7Cgav%7C1%7Cg%3A%22org.apache.spark%22%20AND%20a%3A%22spark-core_2.10%22)建立一个 Java Maven 项目:

```java
<dependencies>
    <dependency>
        <groupId>org.apache.spark</groupId>
	<artifactId>spark-core_2.10</artifactId>
	<version>1.6.0</version>
    </dependency>
</dependencies>
```

### 4.2。字数-火花工作

现在让我们编写 Spark job 来处理包含句子的文件，并输出文件中不同的单词及其计数:

```java
public static void main(String[] args) throws Exception {
    if (args.length < 1) {
        System.err.println("Usage: JavaWordCount <file>");
        System.exit(1);
    }
    SparkConf sparkConf = new SparkConf().setAppName("JavaWordCount");
    JavaSparkContext ctx = new JavaSparkContext(sparkConf);
    JavaRDD<String> lines = ctx.textFile(args[0], 1);

    JavaRDD<String> words 
      = lines.flatMap(s -> Arrays.asList(SPACE.split(s)).iterator());
    JavaPairRDD<String, Integer> ones 
      = words.mapToPair(word -> new Tuple2<>(word, 1));
    JavaPairRDD<String, Integer> counts 
      = ones.reduceByKey((Integer i1, Integer i2) -> i1 + i2);

    List<Tuple2<String, Integer>> output = counts.collect();
    for (Tuple2<?, ?> tuple : output) {
        System.out.println(tuple._1() + ": " + tuple._2());
    }
    ctx.stop();
}
```

注意，我们将本地文本文件的路径作为一个参数传递给 Spark 作业。

一个`SparkContext`对象是 Spark 的主要入口点，它表示到一个已经运行的 Spark 集群的连接。它使用`SparkConf` 对象来描述应用程序配置。`SparkContext` 用于读取内存中的文本文件作为`JavaRDD` 对象。

接下来，我们使用`flatmap`方法将 lines `JavaRDD`对象转换为 words `JavaRDD`对象，首先将每行转换为空格分隔的单词，然后展平每行处理的输出。

我们再次应用变换操作`mapToPair` ,它基本上将单词的每个出现映射到单词元组和计数 1。

然后，我们应用`reduceByKey` 操作将计数为 1 的任意单词的多次出现分组到一个单词元组中，并对计数求和。

最后，我们执行 c `ollect` RDD 动作来获得最终结果。

### 4.3。执行-火花作业

现在让我们使用 Maven 在目标文件夹中生成`apache-spark-1.0-SNAPSHOT.jar`来构建项目。

接下来，我们需要将这个字数统计作业提交给 Spark:

```java
${spark-install-dir}/bin/spark-submit --class com.baeldung.WordCount 
  --master local ${WordCount-MavenProject}/target/apache-spark-1.0-SNAPSHOT.jar
  ${WordCount-MavenProject}/src/main/resources/spark_example.txt
```

在运行上述命令之前，需要更新 Spark 安装目录和 WordCount Maven 项目目录。

提交时，后台会发生几个步骤:

1.  从驱动程序代码中，`SparkContext`连接到集群管理器(在我们的例子中，spark 是本地运行的独立集群管理器)
2.  集群管理器在其他应用程序之间分配资源
3.  Spark 获取集群中节点上的执行器。在这里，我们的字数统计应用程序将获得自己的执行器进程
4.  应用程序代码(jar 文件)被发送给执行者
5.  任务由`SparkContext`发送给执行者。

最后，spark 作业的结果被返回给驱动程序，我们将看到文件中的字数作为输出:

```java
Hello 1
from 2
Baledung 2
Keep 1
Learning 1
Spark 1
Bye 1
```

## 5。结论

在本文中，我们讨论了 Apache Spark 的架构和不同组件。我们还演示了一个 Spark 作业的工作示例，它给出了文件的字数。

和往常一样，完整的源代码可以在 GitHub 上找到[。](https://web.archive.org/web/20220926185023/https://github.com/eugenp/tutorials/tree/master/apache-spark)