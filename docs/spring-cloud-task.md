# Spring 云任务简介

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-cloud-task>

## 1。概述

**Spring Cloud Task 的目标是为 Spring Boot 应用**提供创建短命微服务的功能。

在 Spring Cloud Task 中，我们可以灵活地动态运行任何任务，按需分配资源，并在任务完成后检索结果。

**Tasks 是 Spring Cloud 数据流中的一个新原语，允许用户将几乎任何 Spring Boot 应用作为一个短期任务执行**。

## 2。开发一个简单的任务应用程序

### 2.1。添加相关依赖关系

首先，我们可以用`spring-cloud-task-dependencies:`添加依赖管理部分

```
<dependencyManagement>
    <dependencies>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-task-dependencies</artifactId>
            <version>2.2.3.RELEASE</version>
            <type>pom</type>
            <scope>import</scope>
        </dependency>
    </dependencies>
</dependencyManagement>
```

这种依赖关系管理通过导入范围管理依赖关系的版本。

我们需要添加以下依赖项:

```
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-task</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-task-core</artifactId>
</dependency>
```

[这个](https://web.archive.org/web/20220926183253/https://search.maven.org/classic/#search%7Cga%7C1%7Ca%3A%22spring-cloud-task-core%22)是 `spring-cloud-task-core`的 Maven Central 的链接。

现在，要启动我们的 Spring Boot 应用程序，我们需要`spring-boot-starter` 相关的父类。

我们将使用 Spring Data JPA 作为 ORM 工具，因此我们也需要为其添加依赖关系:

```
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-jpa</artifactId>
    <version>2.6.1</version>
</dependency>
```

用 Spring Data JPA 引导一个简单的 Spring Boot 应用程序的细节可以在这里[找到。](/web/20220926183253/https://www.baeldung.com/spring-boot-start)

我们可以查看最新版本的`spring-boot-starter-parent` o `n` [Maven Central](https://web.archive.org/web/20220926183253/https://search.maven.org/classic/#search%7Cga%7C1%7Ca%3A%22spring-boot-starter-parent%22) 。

### 2.2。`@EnableTask`注解

为了引导 Spring Cloud Task 的功能，我们需要添加`@EnableTask`注释:

```
@SpringBootApplication
@EnableTask
public class TaskDemo {
    // ...
} 
```

**注释带来了图片中的`SimpleTaskConfiguration`类，该类依次注册了`TaskRepository`及其基础设施**。默认情况下，内存映射用于存储`TaskRepository`的状态。

`TaskRepository`的主要信息在`TaskExecution`类中建模。该类中值得注意的字段有`taskName`、`startTime`、`endTime`、`exitMessage`。`exitMessage`存储退出时的可用信息。

如果退出是由应用程序的任何事件中的失败引起的，那么完整的异常堆栈跟踪将存储在这里。

**Spring Boot 提供了一个接口`ExitCodeExceptionMapper` ，该接口将未被捕获的异常映射到允许仔细调试的退出代码**。云任务将信息存储在数据源中，以供将来分析。

### 2.3。为`TaskRepository` 配置一个`DataSource`

一旦任务结束，存储`TaskRepository` 的内存映射将消失，我们将丢失与任务事件相关的数据。为了存储在永久存储器中，我们将使用 MySQL 作为 Spring Data JPA 的数据源。

数据源在`application.yml`文件中配置。要配置 Spring Cloud Task 使用提供的数据源作为`TaskRepository`的存储，我们需要创建一个扩展 `DefaultTaskConfigurer`的类。

现在，我们可以将配置好的`Datasource`作为构造函数参数发送给超类的构造函数:

```
@Autowired
private DataSource dataSource;

public class HelloWorldTaskConfigurer extends DefaultTaskConfigurer{
    public HelloWorldTaskConfigurer(DataSource dataSource){
        super(dataSource);
    }
}
```

为了让上面的配置生效，我们需要用`@Autowired` 注释来注释`DataSource` 的实例，并将该实例作为上面定义的`HelloWorldTaskConfigurer` bean 的构造函数参数注入:

```
@Bean
public HelloWorldTaskConfigurer getTaskConfigurer() {
    return new HelloWorldTaskConfigurer(dataSource);
}
```

这就完成了将`TaskRepository`存储到 MySQL 数据库的配置。

### 2.4。实施

在 Spring Boot，**我们可以在应用程序完成启动之前执行任何任务。**我们可以使用`ApplicationRunner`或`CommandLineRunner`接口来创建一个简单的任务。

我们需要实现这些接口的`run` 方法，并将实现类声明为 bean:

```
@Component
public static class HelloWorldApplicationRunner 
  implements ApplicationRunner {

    @Override
    public void run(ApplicationArguments arg0) throws Exception {
        System.out.println("Hello World from Spring Cloud Task!");
    }
}
```

现在，如果我们运行我们的应用程序，我们应该让我们的任务产生必要的输出，在我们的 MySQL 数据库中创建所需的表，记录任务的事件数据。

## 3。Spring 云任务的生命周期

一开始，我们在`TaskRepository`中创建一个条目。这表明所有的 beans 都准备好在应用程序中使用了，Runner 接口的`run`方法也准备好执行了。

当`run`方法执行完成或`ApplicationContext`事件失败时， `TaskRepository`将被另一个条目更新。

**在任务生命周期中，我们可以从`TaskExecutionListener`接口**注册可用的监听器。我们需要一个实现接口的类，它有三个方法，分别在任务的事件中触发`onTaskEnd`、`onTaksFailed`和`onTaskStartup`。

我们需要在我们的`TaskDemo`类中声明实现类的 bean:

```
@Bean
public TaskListener taskListener() {
    return new TaskListener();
}
```

## 4。与 Spring 批处理的集成

我们可以将 Spring 批处理作业作为一个任务来执行，并使用 Spring Cloud Task 来记录作业执行的事件。要启用此功能，我们需要添加与引导和云相关的批处理依赖关系:

```
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-batch</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-task-batch</artifactId>
</dependency>
```

[这里的](https://web.archive.org/web/20220926183253/https://search.maven.org/classic/#search%7Cga%7C1%7Cspring-cloud-task-batch)是`spring-cloud-task-batch`的 Maven Central 的链接。

要将作业配置为任务，我们需要在`JobConfiguration`类中注册作业 bean:

```
@Bean
public Job job2() {
    return jobBuilderFactory.get("job2")
      .start(stepBuilderFactory.get("job2step1")
      .tasklet(new Tasklet(){
          @Override
          public RepeatStatus execute(
            StepContribution contribution,
            ChunkContext chunkContext) throws Exception {
            System.out.println("This job is from Baeldung");
                return RepeatStatus.FINISHED;
          }
    }).build()).build();
}
```

**我们需要用`@EnableBatchProcessing`注释**来修饰`TaskDemo`类:

```
//..Other Annotation..
@EnableBatchProcessing
public class TaskDemo {
    // ...
}
```

**`@EnableBatchProcessing`注释通过设置批处理作业所需的基本配置启用 Spring 批处理特性。**

现在，如果我们运行应用程序，`@EnableBatchProcessing`注释将触发 Spring 批处理作业执行，Spring Cloud Task 将记录所有批处理作业的执行事件以及在`springcloud` 数据库中执行的其他任务。

## 5。从流启动任务

我们可以从春云流中触发任务。为了达到这个目的，我们使用了`@EnableTaskLaucnher`注释。一旦我们用 Spring Boot 应用程序添加了注释，TaskSink 就可用了:

```
@SpringBootApplication
@EnableTaskLauncher
public class StreamTaskSinkApplication {
    public static void main(String[] args) {
        SpringApplication.run(TaskSinkApplication.class, args);
    }
}
```

`TaskSink`从包含`GenericMessage` 的流中接收消息，该流包含`TaskLaunchRequest` 作为有效载荷。然后，它触发任务启动请求中提供的基于坐标的任务。

**为了让`TaskSink`起作用，我们需要一个配置为实现 `TaskLauncher`接口**的 bean。出于测试目的，我们在这里模拟实现:

```
@Bean
public TaskLauncher taskLauncher() {
    return mock(TaskLauncher.class);
}
```

**这里需要注意的是`TaskLauncher` 界面只有在添加了 [`spring-cloud-deployer-local`](https://web.archive.org/web/20220926183253/https://search.maven.org/classic/#search%7Cga%7C1%7Ca%3A%22spring-cloud-deployer-local%22) 依赖:**后才可用

```
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-deployer-local</artifactId>
    <version>2.3.1.RELEASE</version>
</dependency>
```

我们可以通过调用`Sink` 接口的`input` 来测试任务是否启动:

```
public class StreamTaskSinkApplicationTests {

    @Autowired
    private Sink sink; 

    //
}
```

现在，我们创建一个`TaskLaunchRequest` 的实例，并将其作为`GenericMessage<TaskLaunchRequest>` 对象的有效负载发送。然后我们可以调用`Sink` 的`input` 通道，将`GenericMessage`对象保留在通道中。

## 6。结论

在本教程中，我们探索了 Spring Cloud Task 是如何执行的，以及如何将其配置为在数据库中记录事件。我们还观察了 Spring 批处理作业是如何在`TaskRepository`中定义和存储的。最后，我们解释了如何从 Spring Cloud Stream 中触发任务。

和往常一样，代码可以在 GitHub 的[上获得。](https://web.archive.org/web/20220926183253/https://github.com/eugenp/tutorials/tree/master/spring-cloud-modules/spring-cloud-task)