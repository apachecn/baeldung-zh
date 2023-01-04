# Apache Spark:数据帧、数据集和 rdd 之间的差异

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-spark-dataframe-dataset-rdd>

## 1.概观

**[Apache Spark](/web/20221208143856/https://www.baeldung.com/apache-spark) 是一个快速、分布式的数据处理系统。**它进行内存中数据处理，并使用内存中缓存和优化执行，从而实现快速性能。它为 Scala、Python、Java 和 r 等流行的编程语言提供了高级 API。

在这个快速教程中，我们将介绍 Spark 的三个基本概念:数据帧、数据集和 rdd。

## 2.数据帧

从 Spark 1.3 开始，Spark SQL 引入了一种称为 DataFrame 的表格数据抽象。从此，它成为了 Spark 中最重要的特性之一。当我们想要处理结构化和半结构化的分布式数据时，这个 API 非常有用。

在第 3 节中，我们将讨论弹性分布式数据集(RDD)。数据帧以比 rdd 更有效的方式存储数据，这是因为它们使用 rdd 的不可变、内存中、弹性、分布式和并行功能，而且它们还对数据应用模式。 DataFrames 还将 SQL 代码翻译成优化的低级 RDD 操作。

我们可以用三种方式创建数据帧:

*   转换现有 rdd
*   运行 SQL 查询
*   加载外部数据

Spark 团队在 2.0 版本中引入了`SparkSession`，它统一了所有不同的上下文，确保开发人员无需担心创建不同的上下文:

```java
SparkSession session = SparkSession.builder()
  .appName("TouristDataFrameExample")
  .master("local[*]")
  .getOrCreate();

DataFrameReader dataFrameReader = session.read();
```

我们将分析`Tourist.csv`文件:

```java
Dataset<Row> data = dataFrameReader.option("header", "true")
  .csv("data/Tourist.csv");
```

**由于 Spark 2.0 数据帧变成了`Row`类型的`Dataset`，所以我们可以使用数据帧作为**和`**Dataset<Row>**.`的别名

我们可以选择我们感兴趣的特定列。我们还可以根据给定的列进行筛选和分组:

```java
data.select(col("country"), col("year"), col("value"))
  .show();

data.filter(col("country").equalTo("Mexico"))
  .show();

data.groupBy(col("country"))
  .count()
  .show();
```

## 3.`Datasets`

**数据集是一组强类型的结构化数据**。它们提供了熟悉的面向对象的编程风格以及类型安全的好处，因为数据集可以在编译时检查语法并捕捉错误。

是 DataFrame 的扩展，因此我们可以认为 DataFrame 是数据集的非类型化视图。

Spark 团队在 Spark 1.6 中发布了`Dataset` API，正如他们提到的:“Spark 数据集的目标是提供一个 API，允许用户轻松表达对象域上的转换，同时还提供 Spark SQL 执行引擎的性能和健壮性优势”。

首先，我们需要创建一个类型为`TouristData`的类:

```java
public class TouristData {
    private String region;
    private String country;
    private String year;
    private String series;
    private Double value;
    private String footnotes;
    private String source;
    // ... getters and setters
}
```

为了将我们的每个记录映射到指定的类型，我们需要使用编码器。**编码器在 Java 对象和 Spark 的内部二进制格式之间进行转换**:

```java
// SparkSession initialization and data load
Dataset<Row> responseWithSelectedColumns = data.select(col("region"), 
  col("country"), col("year"), col("series"), col("value").cast("double"), 
  col("footnotes"), col("source"));

Dataset<TouristData> typedDataset = responseWithSelectedColumns
  .as(Encoders.bean(TouristData.class));
```

与 DataFrame 一样，我们可以按特定的列进行筛选和分组:

```java
typedDataset.filter((FilterFunction) record -> record.getCountry()
  .equals("Norway"))
  .show();

typedDataset.groupBy(typedDataset.col("country"))
  .count()
  .show();
```

我们还可以执行一些操作，如按匹配特定范围的列进行筛选，或者计算特定列的总和，以获得它的总值:

```java
typedDataset.filter((FilterFunction) record -> record.getYear() != null 
  && (Long.valueOf(record.getYear()) > 2010 
  && Long.valueOf(record.getYear()) < 2017)).show();

typedDataset.filter((FilterFunction) record -> record.getValue() != null 
  && record.getSeries()
    .contains("expenditure"))
    .groupBy("country")
    .agg(sum("value"))
    .show();
```

## 4.RDDs

弹性分布式数据集或 RDD 是 Spark 的主要编程抽象。它代表了元素的集合，这些元素是:**不可变的、有弹性的、分布式的**。

**RDD 封装了一个大型数据集，Spark 会自动将 RDD 中包含的数据分布到我们的集群中，并将我们对它们执行的操作并行化**。

我们只能通过稳定存储中的数据操作或其他 rdd 上的操作来创建 rdd。

当我们处理大数据集并且数据分布在集群机器上时，容错是必不可少的。由于 Spark 内置的故障恢复机制，rdd 具有弹性。Spark 依赖于 rdd 记住它们是如何被创建的这一事实，因此我们可以很容易地追溯血统来恢复分区。

我们可以在 rdd 上进行两种类型的操作:**转换和动作**。

### 4.1.转换

我们可以对 RDD 应用变换来操作其数据。执行这个操作后，我们将得到一个全新的 RDD，因为 rdd 是不可变的对象。

我们将检查如何实现 Map 和 Filter，这是两种最常见的转换。

首先，我们需要创建一个`JavaSparkContext`并从`Tourist.csv`文件中加载数据作为 RDD:

```java
SparkConf conf = new SparkConf().setAppName("uppercaseCountries")
  .setMaster("local[*]");
JavaSparkContext sc = new JavaSparkContext(conf);

JavaRDD<String> tourists = sc.textFile("data/Tourist.csv");
```

接下来，让我们应用 map 函数从每个记录中获取国家的名称，并将名称转换为大写。我们可以将这个新生成的数据集作为文本文件保存在磁盘上:

```java
JavaRDD<String> upperCaseCountries = tourists.map(line -> {
    String[] columns = line.split(COMMA_DELIMITER);
    return columns[1].toUpperCase();
}).distinct();

upperCaseCountries.saveAsTextFile("data/output/uppercase.txt");
```

如果我们只想选择一个特定的国家，我们可以对我们的原始游客 RDD 应用过滤函数:

```java
JavaRDD<String> touristsInMexico = tourists
  .filter(line -> line.split(COMMA_DELIMITER)[1].equals("Mexico"));

touristsInMexico.saveAsTextFile("data/output/touristInMexico.txt");
```

### 4.2.行动

在对数据进行一些计算后，操作将返回一个最终值或将结果保存到磁盘。

Spark 中经常使用的两个操作是 Count 和 Reduce。

让我们统计一下 CSV 文件中的国家总数:

```java
// Spark Context initialization and data load
JavaRDD<String> countries = tourists.map(line -> {
    String[] columns = line.split(COMMA_DELIMITER);
    return columns[1];
}).distinct();

Long numberOfCountries = countries.count();
```

现在，我们将按国家计算总支出。我们需要过滤描述中包含支出的记录。

我们将使用一个`JavaPairRDD`，而不是使用一个`JavaRDD`。**一对 RDD 是一种可以存储键值对的 RDD**。接下来我们来检查一下:

```java
JavaRDD<String> touristsExpenditure = tourists
  .filter(line -> line.split(COMMA_DELIMITER)[3].contains("expenditure"));

JavaPairRDD<String, Double> expenditurePairRdd = touristsExpenditure
  .mapToPair(line -> {
      String[] columns = line.split(COMMA_DELIMITER);
      return new Tuple2<>(columns[1], Double.valueOf(columns[6]));
});

List<Tuple2<String, Double>> totalByCountry = expenditurePairRdd
  .reduceByKey((x, y) -> x + y)
  .collect();
```

## 5.结论

总而言之，当我们需要特定领域的 API 时，我们应该使用数据框架或数据集，我们需要高级表达式，如聚合、求和或 SQL 查询。或者当我们在编译时需要类型安全时。

另一方面，当数据是非结构化的，我们不需要实现特定的模式，或者当我们需要低层次的转换和操作时，我们应该使用 rdd。

和往常一样，GitHub 上的所有代码示例[都是可用的。](https://web.archive.org/web/20221208143856/https://github.com/eugenp/tutorials/tree/master/apache-spark)