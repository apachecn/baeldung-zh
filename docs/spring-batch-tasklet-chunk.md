# 春季批次–小任务与大块任务

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-batch-tasklet-chunk>

## 1。简介

**[Spring Batch](https://web.archive.org/web/20220526050235/https://projects.spring.io/spring-batch/) 提供了两种实现作业的不同方式:使用微线程和块**。

在本文中，我们将通过一个简单的实际例子来学习如何配置和实现这两种方法。

## 2。依赖性

让我们从**添加所需的依赖关系**开始:

```java
<dependency>
    <groupId>org.springframework.batch</groupId>
    <artifactId>spring-batch-core</artifactId>
    <version>4.3.0</version>
</dependency>
<dependency>
    <groupId>org.springframework.batch</groupId>
    <artifactId>spring-batch-test</artifactId>
    <version>4.3.0</version>
    <scope>test</scope>
</dependency>
```

要获得最新版本的[弹簧批芯](https://web.archive.org/web/20220526050235/https://search.maven.org/classic/#search%7Cgav%7C1%7Cg%3A%22org.springframework.batch%22%20AND%20a%3A%22spring-batch-core%22)和[弹簧批测试](https://web.archive.org/web/20220526050235/https://search.maven.org/classic/#search%7Cgav%7C1%7Cg%3A%22org.springframework.batch%22%20AND%20a%3A%22spring-batch-test%22)，请参考 Maven Central。

## 3。我们的用例

让我们考虑一个包含以下内容的 CSV 文件:

```java
Mae Hodges,10/22/1972
Gary Potter,02/22/1953
Betty Wise,02/17/1968
Wayne Rose,04/06/1977
Adam Caldwell,09/27/1995
Lucille Phillips,05/14/1992
```

每行的第一个位置**代表一个人的名字，第二个位置代表他/她的出生日期**。

我们的用例是**生成另一个 CSV 文件，其中包含每个人的姓名和年龄**:

```java
Mae Hodges,45
Gary Potter,64
Betty Wise,49
Wayne Rose,40
Adam Caldwell,22
Lucille Phillips,25
```

现在我们的领域已经很清楚了，让我们继续使用这两种方法构建一个解决方案。我们将从微线程开始。

## 4。微线程接近

### 4.1。介绍和设计

微线程意味着在一个步骤中执行一个任务。我们的工作将由一个接一个执行的几个步骤组成。**每个步骤应该只执行一个定义的任务**。

我们的工作将包括三个步骤:

1.  从输入 CSV 文件中读取行。
2.  计算输入 CSV 文件中每个人的年龄。
3.  将每个人的姓名和年龄写入一个新的输出 CSV 文件。

现在大图已经准备好了，让我们每一步创建一个类。

`LinesReader` 将负责从输入文件中读取数据:

```java
public class LinesReader implements Tasklet {
    // ...
}
```

`LinesProcessor` 将计算文件中每个人的年龄:

```java
public class LinesProcessor implements Tasklet {
    // ...
}
```

最后，`LinesWriter` 将负责将姓名和年龄写入输出文件:

```java
public class LinesWriter implements Tasklet {
    // ...
}
```

至此，**的所有步骤都实现了`Tasklet` 接口**。这将迫使我们实现它的`execute`方法:

```java
@Override
public RepeatStatus execute(StepContribution stepContribution, 
  ChunkContext chunkContext) throws Exception {
    // ...
}
```

这个方法是我们为每一步添加逻辑的地方。在开始编写代码之前，让我们配置一下我们的作业。

### 4.2。配置

我们需要**给 Spring 的应用上下文**添加一些配置。为上一节中创建的类添加标准 bean 声明后，我们就可以创建作业定义了:

```java
@Configuration
@EnableBatchProcessing
public class TaskletsConfig {

    @Autowired 
    private JobBuilderFactory jobs;

    @Autowired 
    private StepBuilderFactory steps;

    @Bean
    protected Step readLines() {
        return steps
          .get("readLines")
          .tasklet(linesReader())
          .build();
    }

    @Bean
    protected Step processLines() {
        return steps
          .get("processLines")
          .tasklet(linesProcessor())
          .build();
    }

    @Bean
    protected Step writeLines() {
        return steps
          .get("writeLines")
          .tasklet(linesWriter())
          .build();
    }

    @Bean
    public Job job() {
        return jobs
          .get("taskletsJob")
          .start(readLines())
          .next(processLines())
          .next(writeLines())
          .build();
    }

    // ...

}
```

这意味着我们的`“taskletsJob”`将由三个步骤组成。第一个(`readLines`)将执行 bean `linesReader`中定义的小任务，进入下一步:`processLines. ProcessLines` 将执行 bean `linesProcessor` 中定义的小任务，进入最后一步:`writeLines`。

我们的工作流已经定义好了，我们准备添加一些逻辑！

### 4.3。型号和用途

因为我们将操作 CSV 文件中的行，所以我们将创建一个类`Line:`

```java
public class Line implements Serializable {

    private String name;
    private LocalDate dob;
    private Long age;

    // standard constructor, getters, setters and toString implementation

}
```

请注意，`Line`实现了`Serializable.`,这是因为`Line`将作为 DTO 在步骤之间传输数据。根据 Spring Batch，**在步骤之间传递的对象必须是可序列化的**。

另一方面，我们可以开始考虑读写台词。

为此，我们将利用 OpenCSV:

```java
<dependency>
    <groupId>com.opencsv</groupId>
    <artifactId>opencsv</artifactId>
    <version>4.1</version>
</dependency>
```

在 Maven Central 中寻找最新的 [OpenCSV](https://web.archive.org/web/20220526050235/https://search.maven.org/classic/#search%7Cgav%7C1%7Cg%3A%22com.opencsv%22%20AND%20a%3A%22opencsv%22) 版本。

一旦包含了 OpenCSV，**我们还将创建一个`FileUtils`类**。它将提供读取和写入 CSV 行的方法:

```java
public class FileUtils {

    public Line readLine() throws Exception {
        if (CSVReader == null) 
          initReader();
        String[] line = CSVReader.readNext();
        if (line == null) 
          return null;
        return new Line(
          line[0], 
          LocalDate.parse(
            line[1], 
            DateTimeFormatter.ofPattern("MM/dd/yyyy")));
    }

    public void writeLine(Line line) throws Exception {
        if (CSVWriter == null) 
          initWriter();
        String[] lineStr = new String[2];
        lineStr[0] = line.getName();
        lineStr[1] = line
          .getAge()
          .toString();
        CSVWriter.writeNext(lineStr);
    }

    // ...
}
```

注意，`readLine` 作为 OpenCSV 的`readNext` 方法的包装器，并返回一个`Line`对象。

同样，`writeLine` 包装 OpenCSV 的`writeNext` 接收一个`Line`对象。这个类的完整实现可以在 GitHub 项目中找到。

此时，我们已经准备好开始每一步的实现。

### 4.4。`LinesReader`

让我们继续完成我们的`LinesReader`课程:

```java
public class LinesReader implements Tasklet, StepExecutionListener {

    private final Logger logger = LoggerFactory
      .getLogger(LinesReader.class);

    private List<Line> lines;
    private FileUtils fu;

    @Override
    public void beforeStep(StepExecution stepExecution) {
        lines = new ArrayList<>();
        fu = new FileUtils(
          "taskletsvschunks/input/tasklets-vs-chunks.csv");
        logger.debug("Lines Reader initialized.");
    }

    @Override
    public RepeatStatus execute(StepContribution stepContribution, 
      ChunkContext chunkContext) throws Exception {
        Line line = fu.readLine();
        while (line != null) {
            lines.add(line);
            logger.debug("Read line: " + line.toString());
            line = fu.readLine();
        }
        return RepeatStatus.FINISHED;
    }

    @Override
    public ExitStatus afterStep(StepExecution stepExecution) {
        fu.closeReader();
        stepExecution
          .getJobExecution()
          .getExecutionContext()
          .put("lines", this.lines);
        logger.debug("Lines Reader ended.");
        return ExitStatus.COMPLETED;
    }
}
```

`LinesReader's execute` 方法通过输入文件路径创建一个`FileUtils`实例。然后，**将行添加到一个列表中，直到不再有行可读为止**。

我们的类**也实现了`StepExecutionListener`** ，它提供了两个额外的方法:`beforeStep` 和`afterStep`。我们将使用这些方法在`execute` 运行前后初始化和关闭东西。

如果我们看一下`afterStep` 代码，我们会注意到结果列表(`lines)` 放在作业上下文中的那一行，使其可用于下一步:

```java
stepExecution
  .getJobExecution()
  .getExecutionContext()
  .put("lines", this.lines);
```

此时，我们的第一步已经完成了它的职责:将 CSV 行加载到内存中的一个`List`中。让我们进入第二步，处理它们。

### 4.5。`LinesProcessor`

**`LinesProcessor`也会实现`StepExecutionListener`当然还有`Tasklet`。**这意味着它也将实现`beforeStep`、`execute`和`afterStep`方法:

```java
public class LinesProcessor implements Tasklet, StepExecutionListener {

    private Logger logger = LoggerFactory.getLogger(
      LinesProcessor.class);

    private List<Line> lines;

    @Override
    public void beforeStep(StepExecution stepExecution) {
        ExecutionContext executionContext = stepExecution
          .getJobExecution()
          .getExecutionContext();
        this.lines = (List<Line>) executionContext.get("lines");
        logger.debug("Lines Processor initialized.");
    }

    @Override
    public RepeatStatus execute(StepContribution stepContribution, 
      ChunkContext chunkContext) throws Exception {
        for (Line line : lines) {
            long age = ChronoUnit.YEARS.between(
              line.getDob(), 
              LocalDate.now());
            logger.debug("Calculated age " + age + " for line " + line.toString());
            line.setAge(age);
        }
        return RepeatStatus.FINISHED;
    }

    @Override
    public ExitStatus afterStep(StepExecution stepExecution) {
        logger.debug("Lines Processor ended.");
        return ExitStatus.COMPLETED;
    }
}
```

不难理解，**从工作环境中加载`lines`列表，并计算每个人**的年龄。

不需要将另一个结果列表放入上下文中，因为修改发生在来自上一步的同一个对象上。

我们准备好了最后一步。

### 4.6。`LinesWriter`

**`LinesWriter`的任务是检查`lines` 列表，将姓名和年龄写入输出文件**:

```java
public class LinesWriter implements Tasklet, StepExecutionListener {

    private final Logger logger = LoggerFactory
      .getLogger(LinesWriter.class);

    private List<Line> lines;
    private FileUtils fu;

    @Override
    public void beforeStep(StepExecution stepExecution) {
        ExecutionContext executionContext = stepExecution
          .getJobExecution()
          .getExecutionContext();
        this.lines = (List<Line>) executionContext.get("lines");
        fu = new FileUtils("output.csv");
        logger.debug("Lines Writer initialized.");
    }

    @Override
    public RepeatStatus execute(StepContribution stepContribution, 
      ChunkContext chunkContext) throws Exception {
        for (Line line : lines) {
            fu.writeLine(line);
            logger.debug("Wrote line " + line.toString());
        }
        return RepeatStatus.FINISHED;
    }

    @Override
    public ExitStatus afterStep(StepExecution stepExecution) {
        fu.closeWriter();
        logger.debug("Lines Writer ended.");
        return ExitStatus.COMPLETED;
    }
}
```

我们完成了我们工作的实施！让我们创建一个测试来运行它并查看结果。

### 4.7。运行作业

为了运行该作业，我们将创建一个测试:

```java
@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration(classes = TaskletsConfig.class)
public class TaskletsTest {

    @Autowired 
    private JobLauncherTestUtils jobLauncherTestUtils;

    @Test
    public void givenTaskletsJob_whenJobEnds_thenStatusCompleted()
      throws Exception {

        JobExecution jobExecution = jobLauncherTestUtils.launchJob();
        assertEquals(ExitStatus.COMPLETED, jobExecution.getExitStatus());
    }
}
```

`ContextConfiguration`注释指向 Spring 上下文配置类，它有我们的作业定义。

在运行测试之前，我们需要添加一些额外的 beans:

```java
@Bean
public JobLauncherTestUtils jobLauncherTestUtils() {
    return new JobLauncherTestUtils();
}

@Bean
public JobRepository jobRepository() throws Exception {
    JobRepositoryFactoryBean factory = new JobRepositoryFactoryBean();
    factory.setDataSource(dataSource());
    factory.setTransactionManager(transactionManager());
    return factory.getObject();
}

@Bean
public DataSource dataSource() {
    DriverManagerDataSource dataSource = new DriverManagerDataSource();
    dataSource.setDriverClassName("org.sqlite.JDBC");
    dataSource.setUrl("jdbc:sqlite:repository.sqlite");
    return dataSource;
}

@Bean
public PlatformTransactionManager transactionManager() {
    return new ResourcelessTransactionManager();
}

@Bean
public JobLauncher jobLauncher() throws Exception {
    SimpleJobLauncher jobLauncher = new SimpleJobLauncher();
    jobLauncher.setJobRepository(jobRepository());
    return jobLauncher;
}
```

一切准备就绪！继续运行测试！

作业完成后，`output.csv` 有预期的内容，日志显示执行流程:

```java
[main] DEBUG o.b.t.tasklets.LinesReader - Lines Reader initialized.
[main] DEBUG o.b.t.tasklets.LinesReader - Read line: [Mae Hodges,10/22/1972]
[main] DEBUG o.b.t.tasklets.LinesReader - Read line: [Gary Potter,02/22/1953]
[main] DEBUG o.b.t.tasklets.LinesReader - Read line: [Betty Wise,02/17/1968]
[main] DEBUG o.b.t.tasklets.LinesReader - Read line: [Wayne Rose,04/06/1977]
[main] DEBUG o.b.t.tasklets.LinesReader - Read line: [Adam Caldwell,09/27/1995]
[main] DEBUG o.b.t.tasklets.LinesReader - Read line: [Lucille Phillips,05/14/1992]
[main] DEBUG o.b.t.tasklets.LinesReader - Lines Reader ended.
[main] DEBUG o.b.t.tasklets.LinesProcessor - Lines Processor initialized.
[main] DEBUG o.b.t.tasklets.LinesProcessor - Calculated age 45 for line [Mae Hodges,10/22/1972]
[main] DEBUG o.b.t.tasklets.LinesProcessor - Calculated age 64 for line [Gary Potter,02/22/1953]
[main] DEBUG o.b.t.tasklets.LinesProcessor - Calculated age 49 for line [Betty Wise,02/17/1968]
[main] DEBUG o.b.t.tasklets.LinesProcessor - Calculated age 40 for line [Wayne Rose,04/06/1977]
[main] DEBUG o.b.t.tasklets.LinesProcessor - Calculated age 22 for line [Adam Caldwell,09/27/1995]
[main] DEBUG o.b.t.tasklets.LinesProcessor - Calculated age 25 for line [Lucille Phillips,05/14/1992]
[main] DEBUG o.b.t.tasklets.LinesProcessor - Lines Processor ended.
[main] DEBUG o.b.t.tasklets.LinesWriter - Lines Writer initialized.
[main] DEBUG o.b.t.tasklets.LinesWriter - Wrote line [Mae Hodges,10/22/1972,45]
[main] DEBUG o.b.t.tasklets.LinesWriter - Wrote line [Gary Potter,02/22/1953,64]
[main] DEBUG o.b.t.tasklets.LinesWriter - Wrote line [Betty Wise,02/17/1968,49]
[main] DEBUG o.b.t.tasklets.LinesWriter - Wrote line [Wayne Rose,04/06/1977,40]
[main] DEBUG o.b.t.tasklets.LinesWriter - Wrote line [Adam Caldwell,09/27/1995,22]
[main] DEBUG o.b.t.tasklets.LinesWriter - Wrote line [Lucille Phillips,05/14/1992,25]
[main] DEBUG o.b.t.tasklets.LinesWriter - Lines Writer ended.
```

这就是微线程。现在我们可以转到语块法了。

## 5T2。大块接近

### 5.1。介绍和设计

顾名思义，这种方法**对数据块**执行操作。也就是说，它不是一次读取、处理和写入所有行，而是一次读取、处理和写入固定数量的记录(块)。

然后，它会重复这个循环，直到文件中不再有数据。

因此，流量会略有不同:

1.  当有台词的时候:
    *   X 行的 Do:
        *   读一行
        *   处理一行
    *   写 X 行。

因此，我们还需要为面向块的方法创建**三个 beans:**

```java
public class LineReader {
     // ...
}
```

```java
public class LineProcessor {
    // ...
}
```

```java
public class LinesWriter {
    // ...
}
```

在开始实施之前，让我们配置一下我们的工作。

### 5.2。配置

工作定义看起来也会不同:

```java
@Configuration
@EnableBatchProcessing
public class ChunksConfig {

    @Autowired 
    private JobBuilderFactory jobs;

    @Autowired 
    private StepBuilderFactory steps;

    @Bean
    public ItemReader<Line> itemReader() {
        return new LineReader();
    }

    @Bean
    public ItemProcessor<Line, Line> itemProcessor() {
        return new LineProcessor();
    }

    @Bean
    public ItemWriter<Line> itemWriter() {
        return new LinesWriter();
    }

    @Bean
    protected Step processLines(ItemReader<Line> reader,
      ItemProcessor<Line, Line> processor, ItemWriter<Line> writer) {
        return steps.get("processLines").<Line, Line> chunk(2)
          .reader(reader)
          .processor(processor)
          .writer(writer)
          .build();
    }

    @Bean
    public Job job() {
        return jobs
          .get("chunksJob")
          .start(processLines(itemReader(), itemProcessor(), itemWriter()))
          .build();
    }

}
```

在这种情况下，只有一个步骤只执行一个小任务。

然而，那个小任务**定义了一个读取器、一个写入器和一个处理器，它们将对大块数据**进行操作。

注意，**提交间隔表示一个块**中要处理的数据量。我们的工作将一次读取、处理和写入两行。

现在我们准备添加我们的块逻辑！

### 5.3。`LineReader`

`LineReader`将负责读取一条记录并返回一个包含其内容的`Line` 实例。

为了成为一个读者，**我们的类必须实现`ItemReader` 接口**:

```java
public class LineReader implements ItemReader<Line> {
     @Override
     public Line read() throws Exception {
         Line line = fu.readLine();
         if (line != null) 
           logger.debug("Read line: " + line.toString());
         return line;
     }
}
```

代码很简单，它只读取一行并返回它。我们还将为这个类的最终版本实现`StepExecutionListener` :

```java
public class LineReader implements 
  ItemReader<Line>, StepExecutionListener {

    private final Logger logger = LoggerFactory
      .getLogger(LineReader.class);

    private FileUtils fu;

    @Override
    public void beforeStep(StepExecution stepExecution) {
        fu = new FileUtils("taskletsvschunks/input/tasklets-vs-chunks.csv");
        logger.debug("Line Reader initialized.");
    }

    @Override
    public Line read() throws Exception {
        Line line = fu.readLine();
        if (line != null) logger.debug("Read line: " + line.toString());
        return line;
    }

    @Override
    public ExitStatus afterStep(StepExecution stepExecution) {
        fu.closeReader();
        logger.debug("Line Reader ended.");
        return ExitStatus.COMPLETED;
    }
}
```

需要注意的是`beforeStep`和`afterStep` 分别在整个步骤的前后执行。

### 5.4。`LineProcessor`

`LineProcessor`遵循与`LineReader`几乎相同的逻辑。

然而，在本例中，**我们将实现`ItemProcessor` 及其方法`process()`** :

```java
public class LineProcessor implements ItemProcessor<Line, Line> {

    private Logger logger = LoggerFactory.getLogger(LineProcessor.class);

    @Override
    public Line process(Line line) throws Exception {
        long age = ChronoUnit.YEARS
          .between(line.getDob(), LocalDate.now());
        logger.debug("Calculated age " + age + " for line " + line.toString());
        line.setAge(age);
        return line;
    }

}
```

**`process()`方法获取一个输入行，处理它并返回一个输出行**。同样，我们也将实现`StepExecutionListener:`

```java
public class LineProcessor implements 
  ItemProcessor<Line, Line>, StepExecutionListener {

    private Logger logger = LoggerFactory.getLogger(LineProcessor.class);

    @Override
    public void beforeStep(StepExecution stepExecution) {
        logger.debug("Line Processor initialized.");
    }

    @Override
    public Line process(Line line) throws Exception {
        long age = ChronoUnit.YEARS
          .between(line.getDob(), LocalDate.now());
        logger.debug(
          "Calculated age " + age + " for line " + line.toString());
        line.setAge(age);
        return line;
    }

    @Override
    public ExitStatus afterStep(StepExecution stepExecution) {
        logger.debug("Line Processor ended.");
        return ExitStatus.COMPLETED;
    }
}
```

### 5.5。`LinesWriter`

与阅读器和处理器不同， **`LinesWriter`将写一整块行**，以便它接收`Lines:`的`List`

```java
public class LinesWriter implements 
  ItemWriter<Line>, StepExecutionListener {

    private final Logger logger = LoggerFactory
      .getLogger(LinesWriter.class);

    private FileUtils fu;

    @Override
    public void beforeStep(StepExecution stepExecution) {
        fu = new FileUtils("output.csv");
        logger.debug("Line Writer initialized.");
    }

    @Override
    public void write(List<? extends Line> lines) throws Exception {
        for (Line line : lines) {
            fu.writeLine(line);
            logger.debug("Wrote line " + line.toString());
        }
    }

    @Override
    public ExitStatus afterStep(StepExecution stepExecution) {
        fu.closeWriter();
        logger.debug("Line Writer ended.");
        return ExitStatus.COMPLETED;
    }
}
```

代码不言自明。同样，我们准备测试我们的工作。

### 5.6。运行作业

我们将创建一个新的测试，与我们为微线程方法创建的测试相同:

```java
@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration(classes = ChunksConfig.class)
public class ChunksTest {

    @Autowired
    private JobLauncherTestUtils jobLauncherTestUtils;

    @Test
    public void givenChunksJob_whenJobEnds_thenStatusCompleted() 
      throws Exception {

        JobExecution jobExecution = jobLauncherTestUtils.launchJob();

        assertEquals(ExitStatus.COMPLETED, jobExecution.getExitStatus()); 
    }
}
```

按照上面对`TaskletsConfig`的解释配置好`ChunksConfig` 之后，我们就可以开始运行测试了！

一旦工作完成，我们可以看到`output.csv` 再次包含了预期的结果，并且日志描述了流程:

```java
[main] DEBUG o.b.t.chunks.LineReader - Line Reader initialized.
[main] DEBUG o.b.t.chunks.LinesWriter - Line Writer initialized.
[main] DEBUG o.b.t.chunks.LineProcessor - Line Processor initialized.
[main] DEBUG o.b.t.chunks.LineReader - Read line: [Mae Hodges,10/22/1972]
[main] DEBUG o.b.t.chunks.LineReader - Read line: [Gary Potter,02/22/1953]
[main] DEBUG o.b.t.chunks.LineProcessor - Calculated age 45 for line [Mae Hodges,10/22/1972]
[main] DEBUG o.b.t.chunks.LineProcessor - Calculated age 64 for line [Gary Potter,02/22/1953]
[main] DEBUG o.b.t.chunks.LinesWriter - Wrote line [Mae Hodges,10/22/1972,45]
[main] DEBUG o.b.t.chunks.LinesWriter - Wrote line [Gary Potter,02/22/1953,64]
[main] DEBUG o.b.t.chunks.LineReader - Read line: [Betty Wise,02/17/1968]
[main] DEBUG o.b.t.chunks.LineReader - Read line: [Wayne Rose,04/06/1977]
[main] DEBUG o.b.t.chunks.LineProcessor - Calculated age 49 for line [Betty Wise,02/17/1968]
[main] DEBUG o.b.t.chunks.LineProcessor - Calculated age 40 for line [Wayne Rose,04/06/1977]
[main] DEBUG o.b.t.chunks.LinesWriter - Wrote line [Betty Wise,02/17/1968,49]
[main] DEBUG o.b.t.chunks.LinesWriter - Wrote line [Wayne Rose,04/06/1977,40]
[main] DEBUG o.b.t.chunks.LineReader - Read line: [Adam Caldwell,09/27/1995]
[main] DEBUG o.b.t.chunks.LineReader - Read line: [Lucille Phillips,05/14/1992]
[main] DEBUG o.b.t.chunks.LineProcessor - Calculated age 22 for line [Adam Caldwell,09/27/1995]
[main] DEBUG o.b.t.chunks.LineProcessor - Calculated age 25 for line [Lucille Phillips,05/14/1992]
[main] DEBUG o.b.t.chunks.LinesWriter - Wrote line [Adam Caldwell,09/27/1995,22]
[main] DEBUG o.b.t.chunks.LinesWriter - Wrote line [Lucille Phillips,05/14/1992,25]
[main] DEBUG o.b.t.chunks.LineProcessor - Line Processor ended.
[main] DEBUG o.b.t.chunks.LinesWriter - Line Writer ended.
[main] DEBUG o.b.t.chunks.LineReader - Line Reader ended.
```

**我们有相同的结果和不同的流程**。日志清楚地表明了作业是如何按照这种方法执行的。

## 6。结论

不同的环境会显示出对这种或那种方法的需要。虽然微线程在“一个任务接一个任务”的情况下感觉更自然，但块提供了一种简单的解决方案来处理分页读取或我们不想在内存中保留大量数据的情况。

这个例子的完整实现可以在 GitHub 项目[](https://web.archive.org/web/20220526050235/https://github.com/eugenp/tutorials/tree/master/spring-batch)**中找到。**