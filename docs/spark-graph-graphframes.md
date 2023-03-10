# 使用 GraphFrames 处理火花图简介

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spark-graph-graphframes>

## 1.介绍

从社交网络到广告，图形处理在许多应用中都很有用。在大数据场景中，我们需要一种工具来分配处理负载。

在本教程中，我们将使用 Java 中的 [Apache Spark](/web/20221208143839/https://www.baeldung.com/apache-spark) 加载和探索图形的可能性。为了避免复杂的结构，我们将使用一个简单的高级 Apache Spark graph API:graph frames API。

## 2.图表

首先，我们来定义一个图及其组成部分。图是具有边和顶点的数据结构。**边携带表示顶点之间关系的信息**。

顶点是一个`n`维空间中的点，边根据它们的关系连接顶点:

[![Graph Example 1](img/67c2befbeef1243fc1b9514ebf313774.png)](/web/20221208143839/https://www.baeldung.com/wp-content/uploads/2019/12/Graph-Example-1.png)

在上图中，我们有一个社交网络的例子。我们可以看到用字母表示的顶点和携带顶点之间关系的边。

## 3.Maven 设置

现在，让我们通过设置 Maven 配置来开始这个项目。

我们再加上`[spark-graphx 2.11](https://web.archive.org/web/20221208143839/https://search.maven.org/search?q=g:org.apache.spark%20AND%20a:spark-graphx_2.11),` `[graphframes](https://web.archive.org/web/20221208143839/https://mvnrepository.com/artifact/graphframes/graphframes?repo=spark-packages)`，和 [`spark-sql 2.11`](https://web.archive.org/web/20221208143839/https://search.maven.org/search?q=g:org.apache.spark%20AND%20a:spark-sql_2.11) :

```java
<dependency>
    <groupId>org.apache.spark</groupId>
    <artifactId>spark-graphx_2.11</artifactId>
    <version>2.4.4</version>
</dependency>
<dependency>
   <groupId>graphframes</groupId>
   <artifactId>graphframes</artifactId>
   <version>0.7.0-spark2.4-s_2.11</version>
</dependency>
<dependency>
   <groupId>org.apache.spark</groupId>
   <artifactId>spark-sql_2.11</artifactId>
   <version>2.4.4</version>
</dependency>
```

这些工件版本支持 Scala 2.11。

还有，碰巧 GraphFrames 不在 Maven Central 中。因此，让我们也添加所需的 Maven 存储库:

```java
<repositories>
     <repository>
          <id>SparkPackagesRepo</id>
          <url>http://dl.bintray.com/spark-packages/maven</url>
     </repository>
</repositories>
```

## 4.火花配置

为了使用 GraphFrames，我们需要下载 [Hadoop](https://web.archive.org/web/20221208143839/https://hadoop.apache.org/releases.html) 并定义`HADOOP_HOME`环境变量。

在 Windows 作为操作系统的情况下，我们还将把适当的`[winutils.exe](https://web.archive.org/web/20221208143839/https://github.com/steveloughran/winutils/blob/master/hadoop-3.0.0/bin/winutils.exe)`下载到`HADOOP_HOME/bin`文件夹中。

接下来，让我们通过创建基本配置来开始我们的代码:

```java
SparkConf sparkConf = new SparkConf()
  .setAppName("SparkGraphFrames")
  .setMaster("local[*]");
JavaSparkContext javaSparkContext = new JavaSparkContext(sparkConf);
```

我们还需要创建一个`SparkSession`:

```java
SparkSession session = SparkSession.builder()
  .appName("SparkGraphFrameSample")
  .config("spark.sql.warehouse.dir", "/file:C:/temp")
  .sparkContext(javaSparkContext.sc())
  .master("local[*]")
  .getOrCreate();
```

## 5.图形构造

现在，我们已经准备好开始编写主代码了。因此，让我们为顶点和边定义实体，并创建`GraphFrame`实例。

我们将研究一个假想的社交网络中用户之间的关系。

### 5.1.数据

首先，对于这个例子，让我们将两个实体都定义为`User`和`Relationship`:

```java
public class User {
    private Long id;
    private String name;
    // constructor, getters and setters
}

public class Relationship implements Serializable {
    private String type;
    private String src;
    private String dst;
    private UUID id;

    public Relationship(String type, String src, String dst) {
        this.type = type;
        this.src = src;
        this.dst = dst;
        this.id = UUID.randomUUID();
    }
    // getters and setters
}
```

接下来，让我们定义一些`User`和`Relationship`实例:

```java
List<User> users = new ArrayList<>();
users.add(new User(1L, "John"));
users.add(new User(2L, "Martin"));
users.add(new User(3L, "Peter"));
users.add(new User(4L, "Alicia"));

List<Relationship> relationships = new ArrayList<>();
relationships.add(new Relationship("Friend", "1", "2"));
relationships.add(new Relationship("Following", "1", "4"));
relationships.add(new Relationship("Friend", "2", "4"));
relationships.add(new Relationship("Relative", "3", "1"));
relationships.add(new Relationship("Relative", "3", "4"));
```

### 5.2.`GraphFrame`实例

现在，为了创建和操作我们的关系图，我们将创建一个`GraphFrame`的实例。`GraphFrame`构造函数需要两个`Dataset<Row>`实例，第一个代表顶点，第二个代表边:

```java
Dataset<Row> userDataset = session.createDataFrame(users, User.class);
Dataset<Row> relationshipDataset = session.createDataFrame(relationships, Relation.class);

GraphFrame graph = new GraphFrame(userDataframe, relationshipDataframe);
```

最后，我们将在控制台中记录我们的顶点和边，看看它看起来如何:

```java
graph.vertices().show();
graph.edges().show();
```

```java
+---+------+
| id|  name|
+---+------+
|  1|  John|
|  2|Martin|
|  3| Peter|
|  4|Alicia|
+---+------+

+---+--------------------+---+---------+
|dst|                  id|src|     type|
+---+--------------------+---+---------+
|  2|622da83f-fb18-484...|  1|   Friend|
|  4|c6dde409-c89d-490...|  1|Following|
|  4|360d06e1-4e9b-4ec...|  2|   Friend|
|  1|de5e738e-c958-4e0...|  3| Relative|
|  4|d96b045a-6320-4a6...|  3| Relative|
+---+--------------------+---+---------+
```

## 6.图形运算符

现在我们有了一个`GraphFrame`实例，让我们看看可以用它做什么。

### 6.1.过滤器

GraphFrames 允许我们通过查询过滤边和顶点。

接下来，让我们通过`User`上的`name `属性过滤顶点:

```java
graph.vertices().filter("name = 'Martin'").show();
```

在控制台上，我们可以看到结果:

```java
+---+------+
| id|  name|
+---+------+
|  2|Martin|
+---+------+
```

此外，我们可以通过调用`filterEdges`或`filterVertices`直接在图上过滤:

```java
graph.filterEdges("type = 'Friend'")
  .dropIsolatedVertices().vertices().show();
```

现在，由于我们过滤了边，我们可能仍然有一些孤立的顶点。所以，我们就叫`dropIsolatedVertices(). `

因此，我们有一个子图，仍然是一个`GraphFrame`实例，其中只有具有“朋友”状态的关系:

```java
+---+------+
| id|  name|
+---+------+
|  1|  John|
|  2|Martin|
|  4|Alicia|
+---+------+
```

### 6.2.度

另一个有趣的特性集是`degrees`操作集。这些操作返回每个顶点上的边数[事件](/web/20221208143839/https://www.baeldung.com/cs/graphs-incident-edge)。

`degrees`操作只返回每个顶点所有边的数量。另一方面，`inDegrees`只计算输入边，`outDegrees`只计算输出边。

让我们计算一下图中所有顶点的传入度:

```java
graph.inDegrees().show();
```

因此，我们有一个`GraphFrame`来显示每个顶点的传入边的数量，不包括那些没有传入边的:

```java
+---+--------+
| id|inDegree|
+---+--------+
|  1|       1|
|  4|       3|
|  2|       1|
+---+--------+
```

## 7.图形算法

GraphFrames 还提供了现成的流行算法——让我们来看看其中的一些。

### 7.1.页面等级

页面排名算法对顶点的传入边进行加权，并将其转换为分数。

这个想法是，每条进来的边代表一个认可，并使顶点在给定的图中更相关。

例如，在一个社交网络中，如果一个人被各种各样的人关注，他或她将被排名很高。

运行页面排名算法非常简单:

```java
graph.pageRank()
  .maxIter(20)
  .resetProbability(0.15)
  .run()
  .vertices()
  .show();
```

要配置该算法，我们只需提供:

*   `maxIter`–页面等级运行的迭代次数–建议 20 次，太少会降低质量，太多会降低性能
*   `resetProbability`–随机重置概率(alpha)–它越低，赢家和输家之间的分数差距就越大–有效范围从 0 到 1。通常，0.15 是一个很好的分数

响应是类似的`GraphFrame,` ,不过这次我们看到了一个额外的列，给出了每个顶点的页面排名:

```java
+---+------+------------------+
| id|  name|          pagerank|
+---+------+------------------+
|  4|Alicia|1.9393230468864597|
|  3| Peter|0.4848822786454427|
|  1|  John|0.7272991738542318|
|  2|Martin| 0.848495500613866|
+---+------+------------------+
```

在我们的图中，艾丽西娅是最相关的顶点，其次是马丁和约翰。

### 7.2.连接的组件

连通分量算法寻找孤立的聚类或孤立的子图。这些簇是图中的连接顶点的集合，其中每个顶点可以从同一集合中的任何其他顶点到达。

我们可以通过`connectedComponents()` 方法调用不带任何参数的算法:

```java
graph.connectedComponents().run().show();
```

该算法返回一个包含每个顶点和每个顶点所连接的组件的`GraphFrame` :

```java
+---+------+------------+
| id|  name|   component|
+---+------+------------+
|  1|  John|154618822656|
|  2|Martin|154618822656|
|  3| Peter|154618822656|
|  4|Alicia|154618822656|
+---+------+------------+
```

我们的图只有一个组件——这意味着我们没有孤立的子图。组件有一个自动生成的 id，在我们的例子中是 154618822656。

尽管我们在这里多了一列——组件 id——但我们的图表还是一样的。

### 7.3.三角形计数

三角形计数通常用作社交网络图中的社区检测和计数。三角形是由三个顶点组成的集合，其中每个顶点都与三角形中的其他两个顶点有关系。

在社交网络社区中，很容易找到数量可观的相互连接的三角形。

我们可以很容易地直接从我们的`GraphFrame`实例中执行三角形计数:

```java
graph.triangleCount().run().show();
```

算法还返回一个`GraphFrame`，其中包含经过每个顶点的三角形数量。

```java
+-----+---+------+
|count| id|  name|
+-----+---+------+
|    1|  3| Peter|
|    2|  1|  John|
|    2|  4|Alicia|
|    1|  2|Martin|
+-----+---+------+
```

## 8.结论

Apache Spark 是一个以优化和分布式方式计算相关数据量的优秀工具。并且，GraphFrames 库允许我们**轻松地在 Spark** 上分配图形操作。

与往常一样，该示例的完整源代码可以在 GitHub 上的[处获得。](https://web.archive.org/web/20221208143839/https://github.com/eugenp/tutorials/tree/master/apache-spark)