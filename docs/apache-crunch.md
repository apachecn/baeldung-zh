# Apache Crunch 指南

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/apache-crunch>

## 1.介绍

在本教程中，我们将用一个示例数据处理应用程序来演示 [Apache Crunch](https://web.archive.org/web/20220627175026/https://crunch.apache.org/) 。我们将使用 [MapReduce](https://web.archive.org/web/20220627175026/https://en.wikipedia.org/wiki/MapReduce) 框架运行这个应用程序。

我们将首先简要介绍一些 Apache Crunch 概念。然后，我们将进入一个示例应用程序。在这个应用程序中，我们将进行文本处理:

*   首先，我们将读取文本文件中的行
*   稍后，我们将把它们拆分成单词，并删除一些常用单词
*   然后，我们将对剩余的单词进行分组，以获得唯一单词及其计数的列表
*   最后，我们将把这个列表写入一个文本文件

## 2.什么是嘎吱声？

MapReduce 是一个分布式并行编程框架，用于在服务器集群上处理大量数据。Hadoop、Spark 等软件框架实现了 MapReduce。

Crunch 提供了一个用 Java 编写、测试和运行 MapReduce 管道的框架。这里，我们不直接编写 MapReduce 作业。相反，我们使用 Crunch APIs 定义数据管道(即执行输入、处理和输出步骤的操作)。Crunch Planner 将它们映射到 MapReduce 作业，并在需要时执行它们。

**因此，每个 Crunch 数据管道都由一个`Pipeline`接口的实例来协调。**该接口还定义了通过`Source`实例将数据读入管道以及将数据从管道写出到`Target`实例的方法。

我们有 3 个接口来表示数据:

1.  `PCollection`–不可变的分布式元素集合
2.  `PTable<K`，V`>`–一个不可变的、分布式的、无序的键和值的多重映射
3.  `PGroupedTable<K`，V`>`–一个分布式的、排序的 K 类键到一个`Iterable` V 的映射，它可以被精确地迭代一次

`**DoFn**` **是所有数据处理函数**的基类。它对应于 MapReduce 中的`Mapper`、`Reducer`和`Combiner`类。我们把大部分开发时间花在使用它编写和测试逻辑计算上`.`

现在，我们对 Crunch 更加熟悉了，让我们使用它来构建示例应用程序。

## 3.建立一个危机项目

首先，我们用 Maven 建立一个 Crunch 项目。我们可以通过两种方式做到这一点:

1.  在现有项目的`pom.xml`文件中添加所需的依赖项
2.  使用原型生成起始项目

让我们快速浏览一下这两种方法。

### 3.1.Maven 依赖性

为了将 Crunch 添加到现有项目中，让我们在`pom.xml`文件中添加所需的依赖项。

首先，让我们添加`crunch-core`库:

```java
<dependency>
    <groupId>org.apache.crunch</groupId>
    <artifactId>crunch-core</artifactId>
    <version>0.15.0</version>
</dependency>
```

接下来，让我们添加`hadoop-client`库来与 Hadoop 通信。我们使用与 Hadoop 安装相匹配的版本:

```java
<dependency>
    <groupId>org.apache.hadoop</groupId>
    <artifactId>hadoop-client</artifactId>
    <version>2.2.0</version>
    <scope>provided</scope>
</dependency>
```

我们可以在 Maven Central 上查看最新版本的 [crunch-core](https://web.archive.org/web/20220627175026/https://search.maven.org/search?q=g:org.apache.crunch%20AND%20a:crunch-core&core=gav) 和 [hadoop-client](https://web.archive.org/web/20220627175026/https://search.maven.org/search?q=g:org.apache.hadoop%20AND%20a:hadoop-client&core=gav) 库。

### 3.2.Maven 原型

**另一种方法是使用 Crunch** 提供的 Maven 原型快速生成一个启动项目:

```java
mvn archetype:generate -Dfilter=org.apache.crunch:crunch-archetype 
```

当上面的命令提示时，我们提供 Crunch 版本和项目工件细节。

## 4.Crunch 管道设置

建立项目后，我们需要创建一个`Pipeline`对象。**嘎吱有 3 个`Pipeline`实现**:

*   `MRPipeline `–在 Hadoop MapReduce 中执行
*   `SparkPipeline `–作为一系列火花管道执行
*   `MemPipeline `–在客户端的内存中执行，对于单元测试非常有用

通常，我们使用`MemPipeline`的实例进行开发和测试。稍后，我们使用一个`MRPipeline`或`SparkPipeline`的实例进行实际执行。

如果我们需要内存管道，我们可以使用静态方法`getInstance`来获得`MemPipeline`实例:

```java
Pipeline pipeline = MemPipeline.getInstance();
```

但是现在，让我们创建一个`MRPipeline`的实例来用 Hadoop `:`执行应用程序

```java
Pipeline pipeline = new MRPipeline(WordCount.class, getConf());
```

## 5.读取输入数据

创建管道对象后，我们希望读取输入数据。**`Pipeline`接口提供了一种从文本文件**、`readTextFile(pathName).`中读取输入的便利方法

让我们调用这个方法来读取输入的文本文件:

```java
PCollection<String> lines = pipeline.readTextFile(inputPath);
```

上面的代码将文本文件作为`String`的集合读取。

下一步，让我们编写一个读取输入的测试用例:

```java
@Test
public void givenPipeLine_whenTextFileRead_thenExpectedNumberOfRecordsRead() {
    Pipeline pipeline = MemPipeline.getInstance();
    PCollection<String> lines = pipeline.readTextFile(INPUT_FILE_PATH);

    assertEquals(21, lines.asCollection()
      .getValue()
      .size());
}
```

在这个测试中，我们验证了在读取文本文件时我们得到了预期的行数。

## 6.数据处理步骤

在读取输入数据后，我们需要对其进行处理。 **Crunch API 包含了许多`DoFn`的子类来处理常见的数据处理场景**:

*   `FilterFn`–根据布尔条件过滤集合成员
*   `MapFn`–将每个输入记录映射到一个输出记录
*   `CombineFn`–将多个值组合成一个值
*   `JoinFn`–执行连接，如内连接、左外连接、右外连接和全外连接

让我们通过使用这些类来实现以下数据处理逻辑:

1.  将输入文件中的每一行拆分成单词
2.  删除停用词
3.  数数独特的单词

### 6.1.将一行文本拆分成单词

首先，让我们创建`Tokenizer`类来将一行拆分成单词。

我们将扩展`DoFn`类。这个类有一个名为`process`的抽象方法。该方法处理来自`PCollection`的输入记录，并将输出发送给`Emitter. `

我们需要在这个方法中实现拆分逻辑:

```java
public class Tokenizer extends DoFn<String, String> {
    private static final Splitter SPLITTER = Splitter
      .onPattern("\\s+")
      .omitEmptyStrings();

    @Override
    public void process(String line, Emitter<String> emitter) {
        for (String word : SPLITTER.split(line)) {
            emitter.emit(word);
        }
    }
} 
```

在上面的实现中，我们使用了来自[番石榴](https://web.archive.org/web/20220627175026/https://github.com/google/guava/wiki/StringsExplained#splitter)库的`Splitter`类从一行中提取单词。

接下来，让我们为`Tokenizer`类编写一个单元测试:

```java
@RunWith(MockitoJUnitRunner.class)
public class TokenizerUnitTest {

    @Mock
    private Emitter<String> emitter;

    @Test
    public void givenTokenizer_whenLineProcessed_thenOnlyExpectedWordsEmitted() {
        Tokenizer splitter = new Tokenizer();
        splitter.process("  hello  world ", emitter);

        verify(emitter).emit("hello");
        verify(emitter).emit("world");
        verifyNoMoreInteractions(emitter);
    }
}
```

上面的测试验证了是否返回了正确的单词。

最后，让我们使用这个类拆分从输入文本文件中读取的行。

**`PCollection`接口的`parallelDo`方法将给定的`DoFn`应用于所有元素，并返回一个新的`PCollection`。**

让我们在 lines 集合上调用这个方法，并传递一个`Tokenizer`的实例:

```java
PCollection<String> words = lines.parallelDo(new Tokenizer(), Writables.strings()); 
```

结果，我们得到了输入文本文件中的单词列表。我们将在下一步中删除停用词。

### 6.2.删除停用词

与上一步类似，让我们创建一个`StopWordFilter`类来过滤掉停用词。

然而，我们将扩展`FilterFn`而不是`DoFn`。`FilterFn`有一个抽象方法叫做`accept`。我们需要在这个方法中实现过滤逻辑:

```java
public class StopWordFilter extends FilterFn<String> {

    // English stop words, borrowed from Lucene.
    private static final Set<String> STOP_WORDS = ImmutableSet
      .copyOf(new String[] { "a", "and", "are", "as", "at", "be", "but", "by",
        "for", "if", "in", "into", "is", "it", "no", "not", "of", "on",
        "or", "s", "such", "t", "that", "the", "their", "then", "there",
        "these", "they", "this", "to", "was", "will", "with" });

    @Override
    public boolean accept(String word) {
        return !STOP_WORDS.contains(word);
    }
}
```

接下来，让我们为`StopWordFilter`类编写单元测试:

```java
public class StopWordFilterUnitTest {

    @Test
    public void givenFilter_whenStopWordPassed_thenFalseReturned() {
        FilterFn<String> filter = new StopWordFilter();

        assertFalse(filter.accept("the"));
        assertFalse(filter.accept("a"));
    }

    @Test
    public void givenFilter_whenNonStopWordPassed_thenTrueReturned() {
        FilterFn<String> filter = new StopWordFilter();

        assertTrue(filter.accept("Hello"));
        assertTrue(filter.accept("World"));
    }

    @Test
    public void givenWordCollection_whenFiltered_thenStopWordsRemoved() {
        PCollection<String> words = MemPipeline
          .collectionOf("This", "is", "a", "test", "sentence");
        PCollection<String> noStopWords = words.filter(new StopWordFilter());

        assertEquals(ImmutableList.of("This", "test", "sentence"),
         Lists.newArrayList(noStopWords.materialize()));
    }
}
```

该测试验证过滤逻辑是否正确执行。

最后，我们用`StopWordFilter`来过滤上一步生成的单词列表。**`PCollection`接口的`filter`方法将给定的`FilterFn`应用于所有元素并返回一个新的`PCollection`。**

让我们在单词集合上调用这个方法，并传递一个`StopWordFilter`的实例:

```java
PCollection<String> noStopWords = words.filter(new StopWordFilter());
```

结果，我们得到了过滤后的单词集合。

### 6.3.计算独特的单词

在获得过滤后的单词集合后，我们希望统计每个单词出现的频率。 **`PCollection`接口有许多方法来执行常见的聚合:**

*   `min`–返回集合的最小元素
*   `max`–返回集合中的最大元素
*   `length`–返回集合中元素的数量
*   `count`–返回包含集合中每个唯一元素的计数的`PTable`

让我们使用`count`方法获得唯一的单词及其计数:

```java
// The count method applies a series of Crunch primitives and returns
// a map of the unique words in the input PCollection to their counts.
PTable<String, Long> counts = noStopWords.count();
```

## 7.指定输出

作为前面步骤的结果，我们有了一个单词及其计数的表格。我们希望将这个结果写入一个文本文件。**`Pipeline`接口提供了方便的方法来编写输出:**

```java
void write(PCollection<?> collection, Target target);

void write(PCollection<?> collection, Target target,
  Target.WriteMode writeMode);

<T> void writeTextFile(PCollection<T> collection, String pathName);
```

因此，让我们调用`writeTextFile`方法:

```java
pipeline.writeTextFile(counts, outputPath); 
```

## 8.管理管道执行

到目前为止，所有步骤都只是定义了数据管道。未读取或处理任何输入。这是因为 **Crunch 使用了懒惰执行模型。**

在管道接口上调用控制作业计划和执行的方法之前，它不会运行 MapReduce 作业:

*   `run`–准备一个执行计划以创建所需的输出，然后同步执行
*   `done`–运行生成输出所需的任何剩余作业，然后清理创建的任何中间数据文件
*   `runAsync`–类似于 run 方法，但以非阻塞方式执行

因此，让我们调用`done`方法将管道作为 MapReduce 作业来执行:

```java
PipelineResult result = pipeline.done(); 
```

上面的语句运行 MapReduce 作业来读取输入，处理它们并将结果写入输出目录。

## 9.将管道组装在一起

到目前为止，我们已经开发并测试了读取输入数据、处理输入数据并写入输出文件的逻辑。

接下来，让我们将它们放在一起，构建整个数据管道:

```java
public int run(String[] args) throws Exception {
    String inputPath = args[0];
    String outputPath = args[1];

    // Create an object to coordinate pipeline creation and execution.
    Pipeline pipeline = new MRPipeline(WordCount.class, getConf());

    // Reference a given text file as a collection of Strings.
    PCollection<String> lines = pipeline.readTextFile(inputPath);

    // Define a function that splits each line in a PCollection of Strings into
    // a PCollection made up of the individual words in the file.
    // The second argument sets the serialization format.
    PCollection<String> words = lines.parallelDo(new Tokenizer(), Writables.strings());

    // Take the collection of words and remove known stop words.
    PCollection<String> noStopWords = words.filter(new StopWordFilter());

    // The count method applies a series of Crunch primitives and returns
    // a map of the unique words in the input PCollection to their counts.
    PTable<String, Long> counts = noStopWords.count();

    // Instruct the pipeline to write the resulting counts to a text file.
    pipeline.writeTextFile(counts, outputPath);

    // Execute the pipeline as a MapReduce.
    PipelineResult result = pipeline.done();

    return result.succeeded() ? 0 : 1;
}
```

## 10.Hadoop 启动配置

数据流水线就这样准备好了。

然而，我们需要启动它的代码。因此，让我们编写`main`方法来启动应用程序:

```java
public class WordCount extends Configured implements Tool {

    public static void main(String[] args) throws Exception {
        ToolRunner.run(new Configuration(), new WordCount(), args);
    }
```

**`ToolRunner.run`从命令行解析 Hadoop 配置并执行 MapReduce 作业。**

## 11.运行应用程序

完整的应用程序现在已经准备好了。让我们运行以下命令来构建它:

```java
mvn package 
```

作为上述命令的结果，我们在目标目录中获得了打包的应用程序和一个特殊的作业 jar。

让我们使用这个作业 jar 在 Hadoop 上执行应用程序:

```java
hadoop jar target/crunch-1.0-SNAPSHOT-job.jar <input file path> <output directory>
```

应用程序读取输入文件，并将结果写入输出文件。输出文件包含唯一的单词及其计数，如下所示:

```java
[Add,1]
[Added,1]
[Admiration,1]
[Admitting,1]
[Allowance,1]
```

除了 Hadoop，我们还可以在 IDE 中运行应用程序，作为独立应用程序或单元测试。

## 12.结论

在本教程中，我们创建了一个运行在 MapReduce 上的数据处理应用程序。Apache Crunch 使得用 Java 编写、测试和执行 MapReduce 管道变得很容易。

像往常一样，完整的源代码可以在 Github 上找到[。](https://web.archive.org/web/20220627175026/https://github.com/eugenp/tutorials/tree/master/libraries-data)