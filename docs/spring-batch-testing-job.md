# 测试 Spring 批处理作业

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-batch-testing-job>

## 1.介绍

与其他基于 Spring 的应用程序不同，测试批处理作业面临一些特殊的挑战，主要是由于作业执行方式的异步性。

在本教程中，我们将探索测试 [Spring 批处理](/web/20220627185434/https://www.baeldung.com/introduction-to-spring-batch)作业的各种替代方法。

## 2.必需的依赖关系

**我们正在使用 [`spring-boot-starter-batch`](https://web.archive.org/web/20220627185434/https://search.maven.org/classic/#search%7Cga%7C1%7Cspring-boot-starter-batch)** ，所以首先让我们在`pom.xml`中设置所需的依赖项:

```java
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-batch</artifactId>
    <version>2.6.1</version>
</dependency>
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-test</artifactId>
    <version>2.6.1</version>
    <scope>test</scope>
</dependency>
<dependency>
    <groupId>org.springframework.batch</groupId>
    <artifactId>spring-batch-test</artifactId>
    <version>4.3.0.RELEASE</version>
    <scope>test</scope>
</dependency>
```

**我们包含了 [`spring-boo` `t-starter-test`](https://web.archive.org/web/20220627185434/https://search.maven.org/classic/#search%7Cga%7C1%7Ca%3A%22spring-boot-starter-test%22) 和`[spring-batch-test](https://web.archive.org/web/20220627185434/https://search.maven.org/classic/#search%7Cga%7C1%7Ca%3A%22spring-batch-test%22) `** ，它们为测试 Spring 批处理应用程序带来了一些必要的帮助器方法、监听器和运行器。

## 3.定义春季批处理作业

让我们创建一个简单的应用程序来展示 Spring Batch 如何解决一些测试挑战。

**我们的应用程序使用两步`Job`读取包含结构化图书信息的 CSV 输入文件，并输出图书和图书详细信息。**

### 3.1.定义作业步骤

两个随后的`Step`从`BookRecord`提取特定信息，然后将这些信息映射到`Book` s(步骤 1)和`BookDetail` s(步骤 2):

```java
@Bean
public Step step1(
  ItemReader<BookRecord> csvItemReader, ItemWriter<Book> jsonItemWriter) throws IOException {
    return stepBuilderFactory
      .get("step1")
      .<BookRecord, Book> chunk(3)
      .reader(csvItemReader)
      .processor(bookItemProcessor())
      .writer(jsonItemWriter)
      .build();
}

@Bean
public Step step2(
  ItemReader<BookRecord> csvItemReader, ItemWriter<BookDetails> listItemWriter) {
    return stepBuilderFactory
      .get("step2")
      .<BookRecord, BookDetails> chunk(3)
      .reader(csvItemReader)
      .processor(bookDetailsItemProcessor())
      .writer(listItemWriter)
      .build();
}
```

### 3.2.定义输入读取器和输出写入器

现在让我们**配置 CSV 文件输入阅读器，使用`FlatFileItemReader`** 将结构化书籍信息反序列化为`BookRecord`对象:

```java
private static final String[] TOKENS = { 
  "bookname", "bookauthor", "bookformat", "isbn", "publishyear" };

@Bean
@StepScope
public FlatFileItemReader<BookRecord> csvItemReader(
  @Value("#{jobParameters['file.input']}") String input) {
    FlatFileItemReaderBuilder<BookRecord> builder = new FlatFileItemReaderBuilder<>();
    FieldSetMapper<BookRecord> bookRecordFieldSetMapper = new BookRecordFieldSetMapper();
    return builder
      .name("bookRecordItemReader")
      .resource(new FileSystemResource(input))
      .delimited()
      .names(TOKENS)
      .fieldSetMapper(bookRecordFieldSetMapper)
      .build();
}
```

这个定义中有几个重要的东西，这将对我们的测试方式产生影响。

首先，**我们用** `**@StepScope**,`注释了`FlatItemReader` bean，结果，**这个对象将与** `**StepExecution**.`共享它的生存期

**这也允许我们在运行时注入动态值，这样我们就可以从第 4 行**的`JobParameter`中传递我们的输入文件。相反，用于`BookRecordFieldSetMapper`的令牌是在编译时配置的。

然后我们类似地定义`JsonFileItemWriter`输出编写器:

```java
@Bean
@StepScope
public JsonFileItemWriter<Book> jsonItemWriter(
  @Value("#{jobParameters['file.output']}") String output) throws IOException {
    JsonFileItemWriterBuilder<Book> builder = new JsonFileItemWriterBuilder<>();
    JacksonJsonObjectMarshaller<Book> marshaller = new JacksonJsonObjectMarshaller<>();
    return builder
      .name("bookItemWriter")
      .jsonObjectMarshaller(marshaller)
      .resource(new FileSystemResource(output))
      .build();
} 
```

对于第二个`Step`，我们使用 Spring 批处理提供的`ListItemWriter`，它只是将内容转储到内存列表中。

### 3.3.定义自定义`JobLauncher`

接下来，让我们通过在我们的`application.properties.`中设置`spring.batch.job.enabled=false`来禁用 Spring Boot 批处理的默认`Job`启动配置

**当启动`Job` :** 时，我们配置我们自己的`JobLauncher`来传递一个自定义的`JobParameters`实例

```java
@SpringBootApplication
public class SpringBatchApplication implements CommandLineRunner {

    // autowired jobLauncher and transformBooksRecordsJob

    @Value("${file.input}")
    private String input;

    @Value("${file.output}")
    private String output;

    @Override
    public void run(String... args) throws Exception {
        JobParametersBuilder paramsBuilder = new JobParametersBuilder();
        paramsBuilder.addString("file.input", input);
        paramsBuilder.addString("file.output", output);
        jobLauncher.run(transformBooksRecordsJob, paramsBuilder.toJobParameters());
   }

   // other methods (main etc.)
} 
```

## 4.测试 Spring 批处理作业

`spring-batch-test` 依赖项提供了一组有用的助手方法和监听器，可以用来在测试期间配置 Spring 批处理上下文。

让我们为测试创建一个基本结构:

```java
@RunWith(SpringRunner.class)
@SpringBatchTest
@EnableAutoConfiguration
@ContextConfiguration(classes = { SpringBatchConfiguration.class })
@TestExecutionListeners({ DependencyInjectionTestExecutionListener.class, 
  DirtiesContextTestExecutionListener.class})
@DirtiesContext(classMode = ClassMode.AFTER_CLASS)
public class SpringBatchIntegrationTest {

    // other test constants

    @Autowired
    private JobLauncherTestUtils jobLauncherTestUtils;

    @Autowired
    private JobRepositoryTestUtils jobRepositoryTestUtils;

    @After
    public void cleanUp() {
        jobRepositoryTestUtils.removeJobExecutions();
    }

    private JobParameters defaultJobParameters() {
        JobParametersBuilder paramsBuilder = new JobParametersBuilder();
        paramsBuilder.addString("file.input", TEST_INPUT);
        paramsBuilder.addString("file.output", TEST_OUTPUT);
        return paramsBuilder.toJobParameters();
   } 
```

**`@SpringBatchTest` 注释提供了`JobLauncherTestUtils` 和`JobRepositoryTestUtils `助手类。在我们的测试中，我们用它们来触发`Job`和`Step`。**

我们的应用程序使用了 **[Spring Boot 自动配置](/web/20220627185434/https://www.baeldung.com/spring-boot-annotations)，它启用了一个默认的内存中配置`JobRepository`。**结果，**在同一个类中运行多个测试需要在每个测试运行之后进行清理**。

最后，**如果我们想要从几个测试类中运行多个测试，我们需要将我们的[上下文标记为脏的](/web/20220627185434/https://www.baeldung.com/spring-dirtiescontext)** 。这是为了避免使用相同数据源的几个`JobRepository `实例的冲突。

### 4.1.端到端测试`Job`

我们要测试的第一件事是一个完整的端到端`Job`和一个小的数据集输入。

然后，我们可以将结果与预期的测试输出进行比较:

```java
@Test
public void givenReferenceOutput_whenJobExecuted_thenSuccess() throws Exception {
    // given
    FileSystemResource expectedResult = new FileSystemResource(EXPECTED_OUTPUT);
    FileSystemResource actualResult = new FileSystemResource(TEST_OUTPUT);

    // when
    JobExecution jobExecution = jobLauncherTestUtils.launchJob(defaultJobParameters());
    JobInstance actualJobInstance = jobExecution.getJobInstance();
    ExitStatus actualJobExitStatus = jobExecution.getExitStatus();

    // then
    assertThat(actualJobInstance.getJobName(), is("transformBooksRecords"));
    assertThat(actualJobExitStatus.getExitCode(), is("COMPLETED"));
    AssertFile.assertFileEquals(expectedResult, actualResult);
}
```

Spring Batch Test 为使用`AssertFile` 类验证输出提供了一个有用的**文件比较方法。**

### 4.2.测试单个步骤

有时端到端测试整个`Job`非常昂贵，因此测试单个`Steps`是有意义的:

```java
@Test
public void givenReferenceOutput_whenStep1Executed_thenSuccess() throws Exception {
    // given
    FileSystemResource expectedResult = new FileSystemResource(EXPECTED_OUTPUT);
    FileSystemResource actualResult = new FileSystemResource(TEST_OUTPUT);

    // when
    JobExecution jobExecution = jobLauncherTestUtils.launchStep(
      "step1", defaultJobParameters()); 
    Collection actualStepExecutions = jobExecution.getStepExecutions();
    ExitStatus actualJobExitStatus = jobExecution.getExitStatus();

    // then
    assertThat(actualStepExecutions.size(), is(1));
    assertThat(actualJobExitStatus.getExitCode(), is("COMPLETED"));
    AssertFile.assertFileEquals(expectedResult, actualResult);
}

@Test
public void whenStep2Executed_thenSuccess() {
    // when
    JobExecution jobExecution = jobLauncherTestUtils.launchStep(
      "step2", defaultJobParameters());
    Collection actualStepExecutions = jobExecution.getStepExecutions();
    ExitStatus actualExitStatus = jobExecution.getExitStatus();

    // then
    assertThat(actualStepExecutions.size(), is(1));
    assertThat(actualExitStatus.getExitCode(), is("COMPLETED"));
    actualStepExecutions.forEach(stepExecution -> {
        assertThat(stepExecution.getWriteCount(), is(8));
    });
}
```

注意**我们使用`launchStep` 方法来触发特定的步骤**。

记住**我们还设计了我们的`ItemReader`和`ItemWriter `在运行时使用动态值**，这意味着**我们可以将我们的 I/O 参数传递给`JobExecution`** (第 9 和 23 行)。

对于第一个`Step`测试，我们将实际输出与预期输出进行比较。

另一方面，**在第二个测试中，我们为预期的书面项目**验证`StepExecution` 。

### 4.3.测试步骤范围的组件

现在让我们测试一下`FlatFileItemReader` `.` **，回想一下我们将它公开为`@StepScope` bean，所以我们想使用 Spring Batch 对这个**的专用支持:

```java
// previously autowired itemReader

@Test
public void givenMockedStep_whenReaderCalled_thenSuccess() throws Exception {
    // given
    StepExecution stepExecution = MetaDataInstanceFactory
      .createStepExecution(defaultJobParameters());

    // when
    StepScopeTestUtils.doInStepScope(stepExecution, () -> {
        BookRecord bookRecord;
        itemReader.open(stepExecution.getExecutionContext());
        while ((bookRecord = itemReader.read()) != null) {

            // then
            assertThat(bookRecord.getBookName(), is("Foundation"));
            assertThat(bookRecord.getBookAuthor(), is("Asimov I."));
            assertThat(bookRecord.getBookISBN(), is("ISBN 12839"));
            assertThat(bookRecord.getBookFormat(), is("hardcover"));
            assertThat(bookRecord.getPublishingYear(), is("2018"));
        }
        itemReader.close();
        return null;
    });
}
```

**`MetadataInstanceFactory`创建一个自定义`StepExecution` ，需要它来注入我们的步骤作用域`ItemReader.`**

正因为如此，**我们可以借助`doInTestScope` 方法来检查读者的行为。**

接下来，让我们测试`JsonFileItemWriter`并验证它的输出:

```java
@Test
public void givenMockedStep_whenWriterCalled_thenSuccess() throws Exception {
    // given
    FileSystemResource expectedResult = new FileSystemResource(EXPECTED_OUTPUT_ONE);
    FileSystemResource actualResult = new FileSystemResource(TEST_OUTPUT);
    Book demoBook = new Book();
    demoBook.setAuthor("Grisham J.");
    demoBook.setName("The Firm");
    StepExecution stepExecution = MetaDataInstanceFactory
      .createStepExecution(defaultJobParameters());

    // when
    StepScopeTestUtils.doInStepScope(stepExecution, () -> {
        jsonItemWriter.open(stepExecution.getExecutionContext());
        jsonItemWriter.write(Arrays.asList(demoBook));
        jsonItemWriter.close();
        return null;
    });

    // then
    AssertFile.assertFileEquals(expectedResult, actualResult);
} 
```

与之前的测试不同，**我们现在完全控制了我们的测试对象**。因此，**我们负责打开和关闭 I/O 流**。

## 5.结论

在本教程中，我们探索了测试 Spring 批处理作业的各种方法。

端到端测试验证作业的完整执行。在复杂的情况下，测试单个步骤可能会有所帮助。

最后，当涉及到 Step 范围的组件时，我们可以使用由`spring-batch-test.`提供的一系列帮助方法，它们将帮助我们清除和模仿 Spring Batch 域对象。

像往常一样，我们可以在 GitHub 上探索完整的代码库[。](https://web.archive.org/web/20220627185434/https://github.com/eugenp/tutorials/tree/master/spring-batch)