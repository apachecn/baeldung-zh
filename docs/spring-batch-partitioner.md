# 使用分割器的春季批次

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-batch-partitioner>

## 1。概述

在我们之前对 Spring Batch 的[介绍中，我们介绍了作为批处理工具的框架。我们还探讨了单线程、单进程作业执行的配置细节和实现。](/web/20220628052606/https://www.baeldung.com/introduction-to-spring-batch)

为了实现具有一些并行处理的作业，提供了一系列选项。在更高层次上，有两种并行处理模式:

1.  单进程、多线程
2.  多进程

在这篇简短的文章中，我们将讨论`Step`的分区，它可以为单进程和多进程作业实现。

## 2。划分一个步骤

Spring Batch with partitioning 为我们提供了划分 [`Step`](https://web.archive.org/web/20220628052606/https://docs.spring.io/spring-batch/trunk/reference/html/scalability.html) 执行的工具:

[![partitioning overview](img/b1cd7af9b00f73bd423fa1b63fab880a.png)](/web/20220628052606/https://www.baeldung.com/wp-content/uploads/2017/08/partitioning-overview.png.pagespeed.ce_.Wezsdp3QOx.png)

[分区概述](https://web.archive.org/web/20220628052606/https://docs.spring.io/spring-batch/trunk/reference/htimg/partitioning-overview.png)

上图显示了一个带有分区`Step`的`Job`的实现。

有一种`Step`叫做“主”，它的执行被分成一些“从”步骤。这些奴隶可以代替一个主人，结局仍然不会改变。主设备和从设备都是`Step`的实例。从线程可以是远程服务，也可以只是本地执行的线程。

如果需要，我们可以将数据从主机传递到从机。元数据(即`JobRepository`)确保每个从机在`Job.`的单次执行中仅被执行一次

下面是显示其工作原理的序列图:

[![partitioning spi](img/d493ed942a8edf7cc5b8dc905fd85519.png)](/web/20220628052606/https://www.baeldung.com/wp-content/uploads/2017/08/partitioning-spi.png.pagespeed.ce_.2kv9RO_vXW.png)

[分割步骤](https://web.archive.org/web/20220628052606/https://docs.spring.io/spring-batch/trunk/reference/htimg/partitioning-spi.png)

如图所示，`PartitionStep`正在驱动执行。`PartitionHandler`负责将“主”的工作拆分成“从”的工作。最右边的`Step`是奴隶。

## 3。Maven POM

Maven 依赖项与我们之前的[文章](/web/20220628052606/https://www.baeldung.com/introduction-to-spring-batch)中提到的相同。即 Spring Core、Spring Batch 和数据库的依赖关系(在我们的例子中是`SQLite`)。

## 4。配置

在我们的介绍性文章中，我们看到了一个将一些财务数据从 CSV 转换成 XML 文件的例子。让我们扩展同样的例子。

这里，我们将使用多线程实现将财务信息从 5 个 CSV 文件转换为相应的 XML 文件。

我们可以使用单个`Job`和`Step`分区来实现这一点。我们将有五个线程，每个 CSV 文件一个。

首先，让我们创建一个作业:

```java
@Bean(name = "partitionerJob")
public Job partitionerJob() 
  throws UnexpectedInputException, MalformedURLException, ParseException {
    return jobs.get("partitioningJob")
      .start(partitionStep())
      .build();
}
```

正如我们所看到的，这个`Job`从`PartitioningStep`开始。这是我们的主步骤，将被分为多个从步骤:

```java
@Bean
public Step partitionStep() 
  throws UnexpectedInputException, MalformedURLException, ParseException {
    return steps.get("partitionStep")
      .partitioner("slaveStep", partitioner())
      .step(slaveStep())
      .taskExecutor(taskExecutor())
      .build();
}
```

在这里，我们将创建`PartitioningStep using the StepBuilderFactory`。为此，我们需要给出关于`SlaveSteps`和`Partitioner`的信息。

`Partitioner`是一个接口，它提供了为每个从机定义一组输入值的功能。换句话说，将任务划分到各个线程的逻辑在这里开始。

让我们创建一个名为`CustomMultiResourcePartitioner`的实现，其中我们将把输入和输出文件名放在`ExecutionContext`中，以传递给每个从属步骤:

```java
public class CustomMultiResourcePartitioner implements Partitioner {

    @Override
    public Map<String, ExecutionContext> partition(int gridSize) {
        Map<String, ExecutionContext> map = new HashMap<>(gridSize);
        int i = 0, k = 1;
        for (Resource resource : resources) {
            ExecutionContext context = new ExecutionContext();
            Assert.state(resource.exists(), "Resource does not exist: " 
              + resource);
            context.putString(keyName, resource.getFilename());
            context.putString("opFileName", "output"+k+++".xml");
            map.put(PARTITION_KEY + i, context);
            i++;
        }
        return map;
    }
}
```

我们还将为这个类创建 bean，其中我们将给出输入文件的源目录:

```java
@Bean
public CustomMultiResourcePartitioner partitioner() {
    CustomMultiResourcePartitioner partitioner 
      = new CustomMultiResourcePartitioner();
    Resource[] resources;
    try {
        resources = resoursePatternResolver
          .getResources("file:src/main/resources/input/*.csv");
    } catch (IOException e) {
        throw new RuntimeException("I/O problems when resolving"
          + " the input file pattern.", e);
    }
    partitioner.setResources(resources);
    return partitioner;
}
```

我们将定义 slave 步骤，就像读者和作者的任何其他步骤一样。读取器和写入器将与我们在介绍性示例中看到的相同，除了它们将从`StepExecutionContext.` 接收文件名参数

请注意，这些 beans 需要有步骤范围，以便它们能够在每一步接收`stepExecutionContext`参数。如果它们不在步骤范围内，它们的 beans 将在最初创建，并且不接受步骤级别的文件名:

```java
@StepScope
@Bean
public FlatFileItemReader<Transaction> itemReader(
  @Value("#{stepExecutionContext[fileName]}") String filename)
  throws UnexpectedInputException, ParseException {

    FlatFileItemReader<Transaction> reader 
      = new FlatFileItemReader<>();
    DelimitedLineTokenizer tokenizer = new DelimitedLineTokenizer();
    String[] tokens 
      = {"username", "userid", "transactiondate", "amount"};
    tokenizer.setNames(tokens);
    reader.setResource(new ClassPathResource("input/" + filename));
    DefaultLineMapper<Transaction> lineMapper 
      = new DefaultLineMapper<>();
    lineMapper.setLineTokenizer(tokenizer);
    lineMapper.setFieldSetMapper(new RecordFieldSetMapper());
    reader.setLinesToSkip(1);
    reader.setLineMapper(lineMapper);
    return reader;
} 
```

```java
@Bean
@StepScope
public ItemWriter<Transaction> itemWriter(Marshaller marshaller, 
  @Value("#{stepExecutionContext[opFileName]}") String filename)
  throws MalformedURLException {
    StaxEventItemWriter<Transaction> itemWriter 
      = new StaxEventItemWriter<Transaction>();
    itemWriter.setMarshaller(marshaller);
    itemWriter.setRootTagName("transactionRecord");
    itemWriter.setResource(new ClassPathResource("xml/" + filename));
    return itemWriter;
}
```

在从属步骤中提到读取器和写入器时，我们可以将参数作为 null 传递，因为这些文件名不会被使用，因为它们将从`stepExecutionContext`接收文件名:

```java
@Bean
public Step slaveStep() 
  throws UnexpectedInputException, MalformedURLException, ParseException {
    return steps.get("slaveStep").<Transaction, Transaction>chunk(1)
      .reader(itemReader(null))
      .writer(itemWriter(marshaller(), null))
      .build();
}
```

## 5。结论

在本教程中，我们讨论了如何使用 Spring Batch 实现并行处理作业。

和往常一样，这个例子的完整实现可以在 GitHub 的[上找到。](https://web.archive.org/web/20220628052606/https://github.com/eugenp/tutorials/tree/master/spring-batch)