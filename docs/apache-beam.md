# Apache Beam 简介

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/apache-beam>

## 1.概观

在本教程中，我们将介绍 Apache Beam 并探索它的基本概念。

我们将从演示使用 Apache Beam 的用例及好处开始，然后我们将介绍基本概念和术语。之后，我们将通过一个简单的例子来说明 Apache Beam 的所有重要方面。

## 2.什么是阿帕奇光束？

**Apache Beam (Batch + strEAM)是用于批处理和流数据处理作业的统一编程模型。**它提供了一个软件开发工具包来定义和构建数据处理管道以及执行它们的运行器。

Apache Beam 旨在提供一个可移植的编程层。实际上，Beam 管道运行器将数据处理管道翻译成与用户选择的后端兼容的 API。目前，支持这些分布式处理后端:

*   阿帕奇 Apex
*   Apache 很不错
*   阿帕奇齿轮泵(孵化)
*   Apache Samza
*   阿帕奇火花
*   谷歌云数据流
*   黑兹尔卡斯特喷气机

## 3.为什么是阿帕奇光束？

Apache Beam 融合了批处理和流数据处理，而其他人通常通过单独的 API 来实现。因此，当需求发生变化时，很容易将流式流程转换为批处理流程，反之亦然。

Apache Beam 提高了便携性和灵活性。我们关注我们的逻辑，而不是潜在的细节。此外，我们可以随时更改数据处理后端。

Apache Beam 有 Java、Python、Go 和 Scala SDKs 可用。事实上，团队中的每个人都可以用他们选择的语言来使用它。

## 4.基本概念

使用 Apache Beam，我们可以构建工作流图(管道)并执行它们。编程模型中的关键概念是:

*   `PCollection`–代表一个数据集，可以是固定的批处理或数据流
*   `PTransform`–采用一个或多个`PCollection`并输出零个或多个`PCollection`的数据处理操作
*   `Pipeline`–表示`PCollection`和`PTransform`的有向无环图，因此封装了整个数据处理作业
*   `PipelineRunner`–在指定的分布式处理后端执行`Pipeline`

简单来说，一个`PipelineRunner`执行一个`Pipeline,`，一个`Pipeline`由`PCollection`和`PTransform`组成。

## 5.字数统计示例

现在我们已经学习了 Apache Beam 的基本概念，让我们设计并测试一个字数统计任务。

### 5.1.构建波束管道

设计工作流图是每个 Apache Beam 作业的第一步。让我们定义字数统计任务的步骤:

1.  阅读原文。
2.  将文本拆分成单词列表。
3.  所有单词小写。
4.  修剪标点符号。
5.  过滤停用词。
6.  数一数每个独特的单词。

为了实现这一点，我们需要使用`PCollection`和`PTransform`抽象将上述步骤转换成一个单独的`Pipeline`。

### 5.2.属国

在实现我们的工作流图之前，我们应该将 [Apache Beam 的核心依赖关系](https://web.archive.org/web/20220629004535/https://search.maven.org/artifact/org.apache.beam/beam-sdks-java-core)添加到我们的项目中:

```java
<dependency>
    <groupId>org.apache.beam</groupId>
    <artifactId>beam-sdks-java-core</artifactId>
    <version>${beam.version}</version>
</dependency>
```

波束管道运行程序依靠分布式处理后端来执行任务。让我们添加 [`DirectRunner`](https://web.archive.org/web/20220629004535/https://search.maven.org/artifact/org.apache.beam/beam-runners-direct-java) 作为运行时依赖:

```java
<dependency>
    <groupId>org.apache.beam</groupId>
    <artifactId>beam-runners-direct-java</artifactId>
    <version>${beam.version}</version>
    <scope>runtime</scope>
</dependency>
```

与其他管道运行程序不同，`DirectRunner`不需要任何额外的设置，这使它成为初学者的一个好选择。

### 5.3.履行

Apache Beam 利用了 Map-Reduce 编程范式(与 [Java Streams](/web/20220629004535/https://www.baeldung.com/java-8-streams-introduction) 相同)。事实上，在我们继续之前，有一个关于[`reduce()`](/web/20220629004535/https://www.baeldung.com/java-stream-reduce)[`filter()`](/web/20220629004535/https://www.baeldung.com/java-stream-filter-lambda)[`count()`](/web/20220629004535/https://www.baeldung.com/java-stream-filter-count)[`map()``flatMap()`](/web/20220629004535/https://www.baeldung.com/java-difference-map-and-flatmap)的基本概念是个不错的主意。

创建一个`Pipeline`是我们要做的第一件事:

```java
PipelineOptions options = PipelineOptionsFactory.create();
Pipeline p = Pipeline.create(options);
```

现在我们应用我们的六步字数统计任务:

```java
PCollection<KV<String, Long>> wordCount = p
    .apply("(1) Read all lines", 
      TextIO.read().from(inputFilePath))
    .apply("(2) Flatmap to a list of words", 
      FlatMapElements.into(TypeDescriptors.strings())
      .via(line -> Arrays.asList(line.split("\\s"))))
    .apply("(3) Lowercase all", 
      MapElements.into(TypeDescriptors.strings())
      .via(word -> word.toLowerCase()))
    .apply("(4) Trim punctuations", 
      MapElements.into(TypeDescriptors.strings())
      .via(word -> trim(word)))
    .apply("(5) Filter stopwords", 
      Filter.by(word -> !isStopWord(word)))
    .apply("(6) Count words", 
      Count.perElement());
```

`apply()`的第一个(可选)参数是一个`String`,它只是为了提高代码的可读性。下面是上面代码中每个`apply()`做的事情:

1.  首先，我们使用`TextIO`逐行读取一个输入文本文件。
2.  用空格分割每一行，我们把它平面映射成一个单词列表。
3.  字数不区分大小写，所以我们将所有单词小写。
4.  之前，我们用空白分隔行，以类似“word！”以及“word？”，所以我们去掉标点符号。
5.  像“is”和“by”这样的停用词在几乎每个英语文本中都很常见，所以我们将其删除。
6.  最后，我们使用内置函数`Count.perElement()`计算唯一的单词。

如前所述，管道是在分布式后端处理的。不可能遍历内存中的`PCollection`,因为它分布在多个后端。相反，我们将结果写入外部数据库或文件。

首先，我们将我们的`PCollection`转换成`String`。然后，我们用`TextIO`写出输出:

```java
wordCount.apply(MapElements.into(TypeDescriptors.strings())
    .via(count -> count.getKey() + " --> " + count.getValue()))
    .apply(TextIO.write().to(outputFilePath));
```

现在我们的`Pipeline`定义已经完成，我们可以运行并测试它了。

### 5.4.运行和测试

到目前为止，我们已经为字数统计任务定义了一个`Pipeline`。此时，让我们运行`Pipeline`:

```java
p.run().waitUntilFinish();
```

在这一行代码中，Apache Beam 会将我们的任务发送给多个`DirectRunner`实例。因此，最后会生成几个输出文件。它们将包含以下内容:

```java
...
apache --> 3
beam --> 5
rocks --> 2
...
```

在 Apache Beam 中定义和运行分布式作业就像这样简单明了。作为比较，字数统计实现也可以在[阿帕奇火花](/web/20220629004535/https://www.baeldung.com/apache-spark)、[阿帕奇弗林克](/web/20220629004535/https://www.baeldung.com/apache-flink)和[黑兹尔卡斯特喷气机](/web/20220629004535/https://www.baeldung.com/hazelcast-jet)上实现。

## 6.我们将何去何从？

我们成功地统计了输入文件中的每个单词，但是我们还没有最常用单词的报告。当然，排序一个`PCollection`是我们下一步要解决的好问题。

稍后，我们可以学习更多关于窗口、触发器、度量和更复杂的转换的知识。 [Apache Beam 文档](https://web.archive.org/web/20220629004535/https://beam.apache.org/documentation/)提供了深入的信息和参考资料。

## 7.结论

在本教程中，我们学习了什么是 Apache Beam，以及为什么它优于其他选择。我们还通过一个字数统计示例演示了 Apache Beam 的基本概念。

本教程的代码可以在 GitHub 的[上找到。](https://web.archive.org/web/20220629004535/https://github.com/eugenp/tutorials/tree/master/apache-libraries)