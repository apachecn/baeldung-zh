# 用 Spring Cloud 数据流进行批处理

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-cloud-data-flow-batch-processing>

## 1。概述

在本系列的第一篇文章中，我们介绍了`Spring Cloud Data Flow`的架构组件以及如何使用它来创建流数据管道。

与处理无限量数据的流管道相反，**批处理使创建按需执行任务的短期服务变得容易**。

## 2。本地数据流服务器和外壳

`Local Data Flow Server`是负责部署应用程序的组件，而`Data Flow Shell`允许我们 执行与服务器交互所需的 DSL 命令。

在[之前的文章](/web/20220807183327/https://www.baeldung.com/spring-cloud-data-flow-stream-processing)中，我们使用 [Spring Initilizr](https://web.archive.org/web/20220807183327/https://start.spring.io/) 将它们都设置为 Spring Boot 应用程序。

分别将`@EnableDataFlowServer`注释添加到`server's`主类中，将`@`注释添加到 shell 的主类中后，可以通过执行以下命令来启动它们:

```
mvn spring-boot:run
```

服务器将在端口 9393 上启动，一个 shell 将准备好与它进行交互。

关于如何获得和使用`Local Data Flow Server`及其 shell 客户端的详细信息，您可以参考上一篇文章。

## 3。批量申请

与服务器和 shell 一样，我们可以使用 [Spring Initilizr](https://web.archive.org/web/20220807183327/https://start.spring.io/) 来建立一个 `root Spring Boot`批处理应用程序。

到达网站后，只需选择一个`Group`、一个`Artifact`名称，并从依赖项搜索框中选择`Cloud Task`。

一旦完成，点击`Generate Project`按钮开始下载 Maven 工件。

工件是预先配置好的，并带有基本代码。让我们看看如何编辑它，以便构建我们的批处理应用程序。

### 3.1。Maven 依赖关系

首先，让我们添加几个 Maven 依赖项。由于这是一个批处理应用程序，我们需要从`Spring Batch Project`导入库:

```
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-batch</artifactId>
</dependency>
```

此外，由于 Spring Cloud 任务使用关系数据库来存储执行任务的结果，我们需要向 RDBMS 驱动程序添加一个依赖项:

```
<dependency>
    <groupId>com.h2database</groupId>
    <artifactId>h2</artifactId>
</dependency>
```

我们选择使用 Spring 提供的 H2 内存数据库。这给了我们一个简单的引导开发的方法。然而，在生产环境中，您会想要配置自己的`DataSource`。

请记住，工件的版本将从 Spring Boot 的父文件`pom.xml`中继承。

### 3.2。主类

启用所需功能的关键点是将`@EnableTask`和`@EnableBatchProcessing` 注释添加到`Spring Boot's`主类中。这个类级注释告诉 Spring Cloud Task 引导所有内容:

```
@EnableTask
@EnableBatchProcessing
@SpringBootApplication
public class BatchJobApplication {

    public static void main(String[] args) {
        SpringApplication.run(BatchJobApplication.class, args);
    }
}
```

### 3.3。作业配置

最后，让我们配置一个作业——在本例中，简单地将`String`打印到一个日志文件中:

```
@Configuration
public class JobConfiguration {

    private static Log logger
      = LogFactory.getLog(JobConfiguration.class);

    @Autowired
    public JobBuilderFactory jobBuilderFactory;

    @Autowired
    public StepBuilderFactory stepBuilderFactory;

    @Bean
    public Job job() {
        return jobBuilderFactory.get("job")
          .start(stepBuilderFactory.get("jobStep1")
          .tasklet(new Tasklet() {

              @Override
              public RepeatStatus execute(StepContribution contribution, 
                ChunkContext chunkContext) throws Exception {

                logger.info("Job was run");
                return RepeatStatus.FINISHED;
              }
        }).build()).build();
    }
}
```

D 关于如何配置和定义作业的详细信息不在本文的讨论范围之内。更多信息，可以看我们的[Spring Batch 介绍](/web/20220807183327/https://www.baeldung.com/introduction-to-spring-batch)文章。

最后，我们的应用程序准备好了。让我们将它安装在本地 Maven 存储库中。要做到这一点,`cd`进入项目的根目录并发出命令:

```
mvn clean install
```

现在是时候将应用程序放入`Data Flow Server.`中了

## 4。注册应用程序

为了在应用注册表中注册应用程序，我们需要提供一个惟一的名称、一个应用程序类型和一个可以解析为应用程序工件的 URI。

转到`Spring Cloud Data Flow Shell`并在提示符下发出命令:

```
app register --name batch-job --type task 
  --uri maven://com.baeldung.spring.cloud:batch-job:jar:0.0.1-SNAPSHOT
```

## 5。创建任务

可以使用以下命令创建任务定义:

```
task create myjob --definition batch-job
```

这将创建一个名为`myjob` 的新任务，指向先前注册的批处理作业应用程序。

可以使用以下命令获得当前任务定义的列表:

```
task list
```

## 6。启动任务

要启动一项任务，我们可以使用以下命令:

```
task launch myjob
```

一旦任务启动，任务的状态就存储在关系数据库中。我们可以使用以下命令检查任务执行的状态:

```
task execution list
```

## 7。查看结果

在本例中，作业只是在日志文件中打印一个字符串。日志文件位于`Data Flow Server`日志输出中显示的目录下。

要查看结果，我们可以跟踪日志:

```
tail -f PATH_TO_LOG\spring-cloud-dataflow-2385233467298102321\myjob-1472827120414\myjob
[...] --- [main] o.s.batch.core.job.SimpleStepHandler: Executing step: [jobStep1]
[...] --- [main] o.b.spring.cloud.JobConfiguration: Job was run
[...] --- [main] o.s.b.c.l.support.SimpleJobLauncher:
  Job: [SimpleJob: [name=job]] completed with the following parameters: 
    [{}] and the following status: [COMPLETED]
```

## 8。结论

在本文中，我们已经展示了如何通过使用`Spring Cloud Data Flow`来处理批处理。

示例代码可以在 [GitHub 项目](https://web.archive.org/web/20220807183327/https://github.com/eugenp/tutorials/tree/master/spring-cloud-modules/spring-cloud-data-flow/batch-job)中找到。