# Hazelcast Jet 简介

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/hazelcast-jet>

## 1.介绍

在本教程中，我们将了解黑兹尔卡斯特喷气机。它是由 Hazelcast，Inc .提供的分布式数据处理引擎，建立在 Hazelcast IMDG 的基础上。

如果你想了解 IMDG 哈泽卡斯特，这里有一篇文章可以帮助你入门。

## 2.什么是 Hazelcast Jet？

Hazelcast Jet 是一个分布式数据处理引擎，将数据视为流。它可以处理存储在数据库或文件中的数据，也可以处理 Kafka 服务器传输的数据。

此外，它可以通过将数据流划分为子集并对每个子集应用聚合来对无限数据流执行聚合功能。这个概念在 Jet 术语中称为窗口。

我们可以在一个机器集群中部署 Jet，然后将我们的数据处理任务提交给它。Jet 将使集群的所有成员自动处理数据。集群的每个成员都会消耗一部分数据，这使得扩展到任何级别的吞吐量都很容易。

以下是 Hazelcast Jet 的典型使用案例:

*   实时流处理
*   快速批处理
*   以分布式方式处理 Java 8 流
*   微服务中的数据处理

## 3.设置

要在我们的环境中设置 Hazelcast Jet，我们只需要向我们的`pom.xml`添加一个 Maven 依赖项。

我们是这样做的:

```java
<dependency>
    <groupId>com.hazelcast.jet</groupId>
    <artifactId>hazelcast-jet</artifactId>
    <version>4.2</version>
</dependency>
```

包含这个依赖项将下载一个 10 Mb 的 jar 文件，它为我们提供了构建分布式数据处理管道所需的所有基础设施。

Hazelcast Jet 的最新版本可以在[这里](https://web.archive.org/web/20220626205850/https://search.maven.org/classic/#search%7Cga%7C1%7Chazelcast%20jet)找到。

## 4.示例应用程序

为了了解更多关于 Hazelcast Jet 的信息，我们将创建一个示例应用程序，它输入句子和要在这些句子中查找的单词，并返回这些句子中指定单词的数量。

### 4.1.管道

管道构成了 Jet 应用程序的基本结构。**流水线内的处理遵循以下步骤:**

*   从源中读取数据
*   转换数据
*   将数据写入接收器

对于我们的应用程序，管道将从分布式`List`读取，应用分组和聚合的转换，最后写入分布式`Map`。

以下是我们编写管道的方式:

```java
private Pipeline createPipeLine() {
    Pipeline p = Pipeline.create();
    p.readFrom(Sources.<String>list(LIST_NAME))
      .flatMap(word -> traverseArray(word.toLowerCase().split("\\W+")))
      .filter(word -> !word.isEmpty())
      .groupingKey(wholeItem())
      .aggregate(counting())
      .writeTo(Sinks.map(MAP_NAME));
    return p;
}
```

一旦我们从源中读取了数据，我们就遍历数据并使用正则表达式在空间上分割它。之后，我们过滤掉空白。

最后，我们将单词分组，聚合它们并将结果写入 a `Map.`

### 4.2.工作

现在我们的管道已经定义好了，我们创建一个作业来执行管道。

下面是我们如何编写一个接受参数并返回计数的`countWord` 函数:

```java
public Long countWord(List<String> sentences, String word) {
    long count = 0;
    JetInstance jet = Jet.newJetInstance();
    try {
        List<String> textList = jet.getList(LIST_NAME);
        textList.addAll(sentences);
        Pipeline p = createPipeLine();
        jet.newJob(p).join();
        Map<String, Long> counts = jet.getMap(MAP_NAME);
        count = counts.get(word);
        } finally {
            Jet.shutdownAll();
      }
    return count;
}
```

我们首先创建一个 Jet 实例，以便创建我们的作业并使用管道。接下来，我们将输入`List` 复制到一个分布式列表，这样它在所有实例上都可用。

然后，我们使用上面构建的管道提交一个作业。方法`newJob()` 返回一个由 Jet 异步启动的可执行作业。`join`方法等待作业完成，如果作业完成时出现错误，则抛出一个`exception`。

当作业完成时，结果被检索到一个分布式的`Map,` 中，正如我们在管道中定义的那样。因此，我们从 Jet 实例中获取`Map`,并获取单词的计数。

最后，我们关闭了 Jet 实例。在我们的执行结束后关闭它是很重要的，因为 **Jet 实例启动了它自己的线程**。否则，即使在我们的方法退出后，我们的 Java 进程仍将是活动的。

下面是一个测试我们为 Jet 编写的代码的单元测试:

```java
@Test
public void whenGivenSentencesAndWord_ThenReturnCountOfWord() {
    List<String> sentences = new ArrayList<>();
    sentences.add("The first second was alright, but the second second was tough.");
    WordCounter wordCounter = new WordCounter();
    long countSecond = wordCounter.countWord(sentences, "second");
    assertEquals(3, countSecond);
}
```

## 5.结论

在本文中，我们了解了黑兹尔卡斯特喷气机公司。要了解更多关于它及其特性的信息，请参考[手册](https://web.archive.org/web/20220626205850/https://docs.hazelcast.com/imdg/4.2/)。

像往常一样，本文中使用的示例代码可以在 Github 上找到[。](https://web.archive.org/web/20220626205850/https://github.com/eugenp/tutorials/tree/master/hazelcast)