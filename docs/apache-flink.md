# 用 Java 介绍 Apache Flink

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/apache-flink>

## 1。概述

Apache Flink 是一个大数据处理框架，允许程序员以非常高效和可扩展的方式处理大量数据。

在本文中，我们将介绍一些在`[Apache Flink](https://web.archive.org/web/20221128111931/https://ci.apache.org/projects/flink/flink-docs-release-1.2/index.html)` Java API 中可用的**核心 API 概念和标准数据转换。这个 API 的流畅风格使得使用 Flink 的核心构造——分布式集合变得很容易。**

首先，我们将看看 Flink 的`DataSet` API 转换，并使用它们来实现一个字数统计程序。然后我们将简单看一下 Flink 的`DataStream` API，它允许您以实时的方式处理事件流。

## 2。Maven 依赖关系

首先，我们需要将 Maven 依赖项添加到 [`flink-java`](https://web.archive.org/web/20221128111931/https://search.maven.org/classic/#search%7Cgav%7C1%7Cg%3A%22org.apache.flink%22%20AND%20a%3A%22flink-java%22) 和 [`flink-test-utils`](https://web.archive.org/web/20221128111931/https://search.maven.org/classic/#search%7Cgav%7C1%7Cg%3A%22org.apache.flink%22%20AND%20a%3A%22flink-test-utils%22) 库中:

```java
<dependency>
    <groupId>org.apache.flink</groupId>
    <artifactId>flink-java</artifactId>
    <version>1.2.0</version>
</dependency>
<dependency>
    <groupId>org.apache.flink</groupId>
    <artifactId>flink-test-utils_2.10</artifactId>
    <version>1.2.0</version>
    <scope>test<scope>
</dependency>
```

## 3。核心 API 概念

使用 Flink 时，我们需要了解一些与它的 API 相关的事情:

*   每个 Flink 程序都在分布式数据集合上执行转换。提供了多种转换数据的功能，包括过滤、映射、连接、分组和聚集
*   **Flink 中的一个`sink`操作触发一个流的执行，产生程序**想要的结果，比如将结果保存到文件系统或者打印到标准输出
*   **Flink 转换是惰性的，这意味着它们直到调用`sink`操作后才被执行**
*   **Apache Flink API 支持两种操作模式——批处理和实时。**如果您正在处理一个可以批处理的有限数据源，您将使用`DataSet` API。如果您想要实时处理无限的数据流，您将需要使用`DataStream` API

## 4。数据集 API 转换

Flink 程序的入口点是 [`ExecutionEnvironment`](https://web.archive.org/web/20221128111931/https://ci.apache.org/projects/flink/flink-docs-release-1.2/api/java/org/apache/flink/api/scala/ExecutionEnvironment.html) 类的一个实例——它定义了程序执行的上下文。

让我们创建一个`ExecutionEnvironment`来开始我们的处理:

```java
ExecutionEnvironment env
  = ExecutionEnvironment.getExecutionEnvironment();
```

**注意，当您在本地机器上启动应用程序时，它将在本地 JVM 上执行处理。**如果你想在一组机器上开始处理，你需要在那些机器上安装 [Apache Flink](https://web.archive.org/web/20221128111931/https://nightlies.apache.org/flink/flink-docs-release-1.14//docs/try-flink/local_installation/) ，并相应地配置`ExecutionEnvironment`。

### 4.1。创建数据集

为了开始执行数据转换，我们需要为我们的程序提供数据。

让我们使用我们的`ExecutionEnvironement`创建一个`DataSet` 类的实例:

```java
DataSet<Integer> amounts = env.fromElements(1, 29, 40, 50);
```

您可以从多个来源创建一个`DataSet` ，例如 Apache Kafka、CSV、文件或几乎任何其他数据源。

### 4.2。过滤并减少

一旦创建了`DataSet` 类的实例，就可以对其应用转换。

假设您想要过滤高于某个阈值的数字，然后对它们求和`.` ，您可以使用`filter()` 和`reduce()`转换来实现这一点:

```java
int threshold = 30;
List<Integer> collect = amounts
  .filter(a -> a > threshold)
  .reduce((integer, t1) -> integer + t1)
  .collect();

assertThat(collect.get(0)).isEqualTo(90); 
```

注意，`collect()` 方法是一个触发实际数据转换的`sink`操作。

### 4.3。地图

假设您有一个包含`Person` 个对象的`DataSet`:

```java
private static class Person {
    private int age;
    private String name;

    // standard constructors/getters/setters
}
```

接下来，让我们创建这些对象的`DataSet` :

```java
DataSet<Person> personDataSource = env.fromCollection(
  Arrays.asList(
    new Person(23, "Tom"),
    new Person(75, "Michael")));
```

假设您想从集合的每个对象中只提取`age`字段。您可以使用`map()` 转换来获得`Person` 类的特定字段:

```java
List<Integer> ages = personDataSource
  .map(p -> p.age)
  .collect();

assertThat(ages).hasSize(2);
assertThat(ages).contains(23, 75);
```

### 4.4。加入

当您有两个数据集时，您可能希望在某个`id`字段上连接它们。为此，您可以使用`join()`转换。

让我们创建用户的交易和地址集合:

```java
Tuple3<Integer, String, String> address
  = new Tuple3<>(1, "5th Avenue", "London");
DataSet<Tuple3<Integer, String, String>> addresses
  = env.fromElements(address);

Tuple2<Integer, String> firstTransaction 
  = new Tuple2<>(1, "Transaction_1");
DataSet<Tuple2<Integer, String>> transactions 
  = env.fromElements(firstTransaction, new Tuple2<>(12, "Transaction_2")); 
```

两个元组中的第一个字段是一个`Integer`类型，这是一个`id`字段，我们希望在其上连接两个数据集。

为了执行实际的加入逻辑，我们需要为地址和事务实现一个`[KeySelector](https://web.archive.org/web/20221128111931/https://ci.apache.org/projects/flink/flink-docs-release-1.2/api/java/org/apache/flink/api/java/functions/KeySelector.html)` 接口:

```java
private static class IdKeySelectorTransaction 
  implements KeySelector<Tuple2<Integer, String>, Integer> {
    @Override
    public Integer getKey(Tuple2<Integer, String> value) {
        return value.f0;
    }
}

private static class IdKeySelectorAddress 
  implements KeySelector<Tuple3<Integer, String, String>, Integer> {
    @Override
    public Integer getKey(Tuple3<Integer, String, String> value) {
        return value.f0;
    }
}
```

每个选择器只返回应该执行连接的字段。

不幸的是，这里不可能使用 lambda 表达式，因为 Flink 需要泛型类型信息。

接下来，让我们使用这些选择器实现合并逻辑:

```java
List<Tuple2<Tuple2<Integer, String>, Tuple3<Integer, String, String>>>
  joined = transactions.join(addresses)
  .where(new IdKeySelectorTransaction())
  .equalTo(new IdKeySelectorAddress())
  .collect();

assertThat(joined).hasSize(1);
assertThat(joined).contains(new Tuple2<>(firstTransaction, address)); 
```

### 4.5。排序

假设你有下面这些`Tuple2:`

```java
Tuple2<Integer, String> secondPerson = new Tuple2<>(4, "Tom");
Tuple2<Integer, String> thirdPerson = new Tuple2<>(5, "Scott");
Tuple2<Integer, String> fourthPerson = new Tuple2<>(200, "Michael");
Tuple2<Integer, String> firstPerson = new Tuple2<>(1, "Jack");
DataSet<Tuple2<Integer, String>> transactions = env.fromElements(
  fourthPerson, secondPerson, thirdPerson, firstPerson); 
```

如果您想按照元组的第一个字段对这个集合进行排序，您可以使用`sortPartitions()` 转换:

```java
List<Tuple2<Integer, String>> sorted = transactions
  .sortPartition(new IdKeySelectorTransaction(), Order.ASCENDING)
  .collect();

assertThat(sorted)
  .containsExactly(firstPerson, secondPerson, thirdPerson, fourthPerson);
```

## 5。字数

字数统计问题是一个常用来展示大数据处理框架能力的问题。基本的解决方案包括计算文本输入中的单词出现次数。让我们使用 Flink 来实现这个问题的解决方案。

作为我们解决方案的第一步，我们创建了一个`LineSplitter`类，它将我们的输入分割成令牌(单词)，为每个令牌收集一个`Tuple2` 的键-值对。在每个元组中，键是文本中的一个单词，值是整数一(1)。

这个类实现了将`String` 作为输入并产生`[Tuple2](https://web.archive.org/web/20221128111931/https://nightlies.apache.org/flink/flink-docs-release-1.3/api/java/org/apache/flink/api/java/tuple/Tuple2.html)<String, Integer>:`的`[FlatMapFunction](https://web.archive.org/web/20221128111931/https://ci.apache.org/projects/flink/flink-docs-release-1.1/api/java/org/apache/flink/api/common/functions/FlatMapFunction.html)` 接口

```java
public class LineSplitter implements FlatMapFunction<String, Tuple2<String, Integer>> {

    @Override
    public void flatMap(String value, Collector<Tuple2<String, Integer>> out) {
        Stream.of(value.toLowerCase().split("\\W+"))
          .filter(t -> t.length() > 0)
          .forEach(token -> out.collect(new Tuple2<>(token, 1)));
    }
}
```

我们调用 [`Collector`](https://web.archive.org/web/20221128111931/https://ci.apache.org/projects/flink/flink-docs-release-1.0/api/java/org/apache/flink/util/class-use/Collector.html) 类上的`collect()` 方法，在处理管道中向前推送数据。

我们的下一步也是最后一步是按照元组的第一个元素(单词)对元组进行分组，然后对第二个元素执行`sum`聚合，以产生单词出现的计数:

```java
public static DataSet<Tuple2<String, Integer>> startWordCount(
  ExecutionEnvironment env, List<String> lines) throws Exception {
    DataSet<String> text = env.fromCollection(lines);

    return text.flatMap(new LineSplitter())
      .groupBy(0)
      .aggregate(Aggregations.SUM, 1);
}
```

**我们使用了三种类型的 Flink 转换:`flatMap()`、`groupBy()`和`aggregate()`。**

让我们编写一个测试来断言字数统计实现按预期工作:

```java
List<String> lines = Arrays.asList(
  "This is a first sentence",
  "This is a second sentence with a one word");

DataSet<Tuple2<String, Integer>> result = WordCount.startWordCount(env, lines);

List<Tuple2<String, Integer>> collect = result.collect();

assertThat(collect).containsExactlyInAnyOrder(
  new Tuple2<>("a", 3), new Tuple2<>("sentence", 2), new Tuple2<>("word", 1),
  new Tuple2<>("is", 2), new Tuple2<>("this", 2), new Tuple2<>("second", 1),
  new Tuple2<>("first", 1), new Tuple2<>("with", 1), new Tuple2<>("one", 1));
```

## 6。数据流 API

### 6.1。创建数据流

Apache Flink 还通过其数据流 API 支持事件流的处理。如果我们想开始消费事件，我们首先需要使用`StreamExecutionEnvironment` 类:

```java
StreamExecutionEnvironment executionEnvironment
 = StreamExecutionEnvironment.getExecutionEnvironment();
```

接下来，我们可以使用来自各种来源的`executionEnvironment`创建一个事件流。它可以是像`Apache Kafka`这样的消息总线，但是在这个例子中，我们将简单地从几个字符串元素中创建一个源:

```java
DataStream<String> dataStream = executionEnvironment.fromElements(
  "This is a first sentence", 
  "This is a second sentence with a one word");
```

我们可以像在普通的`DataSet` 类中一样对`DataStream`的每个元素应用转换:

```java
SingleOutputStreamOperator<String> upperCase = text.map(String::toUpperCase);
```

为了触发执行，我们需要调用一个接收操作，比如`print()` ，它将把转换的结果打印到标准输出中，后面是`StreamExecutionEnvironment` 类上的`execute()` 方法:

```java
upperCase.print();
env.execute();
```

它将产生以下输出:

```java
1> THIS IS A FIRST SENTENCE
2> THIS IS A SECOND SENTENCE WITH A ONE WORD
```

### 6.2。事件窗口

当实时处理事件流时，您有时可能需要将事件分组在一起，并在这些事件的窗口上应用一些计算。

假设我们有一个事件流，其中每个事件都是由事件号和事件发送到我们系统时的时间戳组成的一对，并且我们可以容忍无序的事件，但前提是它们的延迟不超过 20 秒。

对于这个例子，让我们首先创建一个流来模拟相隔几分钟的两个事件，并定义一个时间戳提取器来指定我们的迟到阈值:

```java
SingleOutputStreamOperator<Tuple2<Integer, Long>> windowed
  = env.fromElements(
  new Tuple2<>(16, ZonedDateTime.now().plusMinutes(25).toInstant().getEpochSecond()),
  new Tuple2<>(15, ZonedDateTime.now().plusMinutes(2).toInstant().getEpochSecond()))
  .assignTimestampsAndWatermarks(
    new BoundedOutOfOrdernessTimestampExtractor
      <Tuple2<Integer, Long>>(Time.seconds(20)) {

        @Override
        public long extractTimestamp(Tuple2<Integer, Long> element) {
          return element.f1 * 1000;
        }
    });
```

接下来，让我们定义一个窗口操作，将我们的事件分组到五秒钟的窗口中，并对这些事件应用变换:

```java
SingleOutputStreamOperator<Tuple2<Integer, Long>> reduced = windowed
  .windowAll(TumblingEventTimeWindows.of(Time.seconds(5)))
  .maxBy(0, true);
reduced.print();
```

它将获得每五秒钟窗口的最后一个元素，因此它打印出:

```java
1> (15,1491221519)
```

注意，我们看不到第二个事件，因为它比指定的延迟阈值晚到达。

## 7 .**。结论**

在本文中，我们介绍了 Apache Flink 框架，并查看了其 API 提供的一些转换。

我们使用 Flink 的 fluent 和功能性数据集 API 实现了一个字数统计程序。然后我们查看了数据流 API，并在事件流上实现了一个简单的实时转换。

所有这些例子和代码片段的实现都可以在 GitHub 上找到[——这是一个 Maven 项目，所以它应该很容易导入和运行。](https://web.archive.org/web/20221128111931/https://github.com/eugenp/tutorials/tree/master/libraries-data-2)