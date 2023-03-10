# 使用石英进行春季调度

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-quartz-schedule>

## 1。概述

在本教程中，我们将在 Spring 中用 Quartz 构建一个简单的**调度器。**

我们将从一个简单的目标开始，即轻松配置一个新的计划作业。

### 1.1。石英 API 的关键组件

Quartz 具有模块化架构。它由几个基本组件组成，我们可以根据需要组合这些组件。在本教程中，我们将重点关注每个工作中常见的:**`Job`** **`JobDetail`** **`Trigger`**和 `**Scheduler**`。

尽管我们将使用 Spring 来管理应用程序，但是每个单独的组件都可以用两种方式来配置:Quartz 方式或 T2 方式(使用其便利类)。

为了完整起见，我们将尽可能涵盖这两个选项，但是我们可以采用其中任何一个。现在让我们开始构建，一次一个组件。

## 延伸阅读:

## Spring 任务调度器指南

A quick and practical guide to scheduling in Spring with Task Scheduler[Read more](/web/20220812055703/https://www.baeldung.com/spring-task-scheduler) →

## [雅加达的调度 EE](/web/20220812055703/https://www.baeldung.com/scheduling-in-java-enterprise-edition)

A demonstration of how to schedule tasks in Jakarta EE using the @Schedule annotation and the timer service.[Read more](/web/20220812055703/https://www.baeldung.com/scheduling-in-java-enterprise-edition) →

## [Drools 简介](/web/20220812055703/https://www.baeldung.com/drools)

Learn how to use Drools as a Business Rule Management System (BRMS).[Read more](/web/20220812055703/https://www.baeldung.com/drools) →

## 2。`Job`和`JobDetail`

### 2.1。`Job`

API 提供了一个只有一个方法的*作业*接口，`execute.`它必须由包含要完成的实际工作(即任务)的类来实现。当一个作业的触发器触发时，调度程序调用`execute`方法，传递给它一个`JobExecutionContext` 对象。

`JobExecutionContext`向作业实例提供关于其运行时环境的信息，包括调度程序的句柄、触发器的句柄和作业的`JobDetail`对象。

在这个简单的例子中，作业将任务委托给一个服务类:

```java
@Component
public class SampleJob implements Job {

    @Autowired
    private SampleJobService jobService;

    public void execute(JobExecutionContext context) throws JobExecutionException {
        jobService.executeSampleJob();
    }
} 
```

### 2.2。`JobDetail`

虽然作业是主要的工作，但 Quartz 并不存储作业类的实际实例。相反，我们可以使用`JobDetail`类定义一个`Job`的实例。必须向`JobDetail,`提供作业的类，以便它知道要执行的作业的**类型**。

### 2.3。石英`JobBuilder`

Quartz `JobBuilder` 为构建`JobDetail` 实体提供了一个构建器风格的 API:

```java
@Bean
public JobDetail jobDetail() {
    return JobBuilder.newJob().ofType(SampleJob.class)
      .storeDurably()
      .withIdentity("Qrtz_Job_Detail")  
      .withDescription("Invoke Sample Job service...")
      .build();
}
```

### 2.4。`JobDetailFactoryBean`春天

Spring 的 *JobDetailFactoryBean* 为配置`JobDetail` 实例提供了 Bean 风格的用法。如果没有另外指定，它使用 Spring bean 名称作为作业名称:

```java
@Bean
public JobDetailFactoryBean jobDetail() {
    JobDetailFactoryBean jobDetailFactory = new JobDetailFactoryBean();
    jobDetailFactory.setJobClass(SampleJob.class);
    jobDetailFactory.setDescription("Invoke Sample Job service...");
    jobDetailFactory.setDurability(true);
    return jobDetailFactory;
}
```

作业的每次执行都会创建一个新的`JobDetail`实例。`JobDetail`对象传达了作业的详细属性。一旦执行完成，对实例的引用将被删除。

## 3。触发器

一个 `Trigger`是调度一个`Job,`的机制，即一个`Trigger`实例“触发”一个作业的执行。在`Job` (任务的概念)和`Trigger` (调度机制)之间有明确的职责划分。

除了一个`Job`，触发器还需要一个**类型的**，我们可以根据调度需求来选择。

假设我们希望调度我们的任务，让它每小时无限期地执行一次**，那么我们可以使用 Quartz 的`TriggerBuilder` 或 Spring 的`SimpleTriggerFactoryBean` 来执行。**

 **### 3.1。石英`TriggerBuilder`

`TriggerBuilder`是一个构建器风格的 API，用于构建`Trigger`实体:

```java
@Bean
public Trigger trigger(JobDetail job) {
    return TriggerBuilder.newTrigger().forJob(job)
      .withIdentity("Qrtz_Trigger")
      .withDescription("Sample trigger")
      .withSchedule(simpleSchedule().repeatForever().withIntervalInHours(1))
      .build();
}
```

### 3.2。`SimpleTriggerFactoryBean`春天

*SimpleTriggerFactoryBean* 为配置`SimpleTrigger`提供了 Bean 风格的用法。它使用 Spring bean 名称作为触发器名称，如果没有另外指定，默认为无限重复:

```java
@Bean
public SimpleTriggerFactoryBean trigger(JobDetail job) {
    SimpleTriggerFactoryBean trigger = new SimpleTriggerFactoryBean();
    trigger.setJobDetail(job);
    trigger.setRepeatInterval(3600000);
    trigger.setRepeatCount(SimpleTrigger.REPEAT_INDEFINITELY);
    return trigger;
}
```

## 4。配置`JobStore`

`JobStore`为`Job`和`Trigger.` 提供存储机制，它还负责维护与作业调度器相关的所有数据。API 支持**内存**和**持久**存储。

### 4.1.内存中`JobStore`

对于我们的示例，我们将使用内存中的 ***RAMJobStore、*** ，它通过`quartz.properties`提供了极快的性能和简单的配置:

```java
org.quartz.jobStore.class=org.quartz.simpl.RAMJobStore
```

`RAMJobStore` 的明显缺点是它的**本质上是不稳定的**。所有调度信息在停机之间都会丢失。如果我们需要在关闭之间保留作业定义和调度，我们可以使用持久的`JDBCJobStore` 来代替。

为了在 Spring `,`中启用内存中的`JobStore`，我们将在我们的`application.properties`中设置这个属性:

```java
spring.quartz.job-store-type=memory
```

### 4.2\. JDBC `JobStore`

`JDBCJobStore`有两种:`**JobStoreTX**`和`**JobStoreCMT**`。它们都做同样的工作，在数据库中存储调度信息。

两者的区别在于它们如何管理提交数据的事务。`JobStoreCMT`类型需要一个应用程序事务来存储数据，而`JobStoreTX`类型启动并管理自己的事务。

有几个属性需要为`JDBCJobStore`设置。至少，我们必须指定`JDBCJobStore`的类型、数据源和数据库驱动程序类。大多数数据库都有驱动程序类，但是`StdJDBCDelegate`涵盖了大多数情况:

```java
org.quartz.jobStore.class=org.quartz.impl.jdbcjobstore.JobStoreTX
org.quartz.jobStore.driverDelegateClass=org.quartz.impl.jdbcjobstore.StdJDBCDelegate
org.quartz.jobStore.dataSource=quartzDataSource
```

在春天建立一个 JDBC 需要几个步骤。首先，我们将在我们的`application.properties`中设置商店类型:

```java
spring.quartz.job-store-type=jdbc
```

然后我们需要启用自动配置，并为 Spring 提供 Quartz 调度程序所需的数据源。`@QuartzDataSource`注释为我们完成了配置和初始化 Quartz 数据库的艰苦工作:

```java
@Configuration
@EnableAutoConfiguration
public class SpringQrtzScheduler {

    @Bean
    @QuartzDataSource
    public DataSource quartzDataSource() {
        return DataSourceBuilder.create().build();
    }
}
```

## 5。`Scheduler`

`Scheduler` 接口是与作业调度程序接口的主要 API。

一个`Scheduler`可以用一个`SchedulerFactory.`实例化。一旦创建，我们可以用它注册`Job`和`Trigger`。最初，`Scheduler`处于“待机”模式，我们必须调用它的 **`start`** 方法来启动触发作业执行的线程。

### 5.1。石英`StdSchedulerFactory`

通过简单地调用`StdSchedulerFactory`上的`getScheduler`方法，我们可以实例化`Scheduler`，初始化它(用配置好的`JobStore`和`ThreadPool`)，并返回其 API 的句柄:

```java
@Bean
public Scheduler scheduler(Trigger trigger, JobDetail job, SchedulerFactoryBean factory) 
  throws SchedulerException {
    Scheduler scheduler = factory.getScheduler();
    scheduler.scheduleJob(job, trigger);
    scheduler.start();
    return scheduler;
}
```

### 5.2。`SchedulerFactoryBean`春天

Spring 的 `SchedulerFactoryBean`提供了 bean 风格的用法来配置`Scheduler`，在应用程序上下文中管理它的生命周期，并将`Scheduler`公开为依赖注入的 bean:

```java
@Bean
public SchedulerFactoryBean scheduler(Trigger trigger, JobDetail job, DataSource quartzDataSource) {
    SchedulerFactoryBean schedulerFactory = new SchedulerFactoryBean();
    schedulerFactory.setConfigLocation(new ClassPathResource("quartz.properties"));

    schedulerFactory.setJobFactory(springBeanJobFactory());
    schedulerFactory.setJobDetails(job);
    schedulerFactory.setTriggers(trigger);
    schedulerFactory.setDataSource(quartzDataSource);
    return schedulerFactory;
}
```

### 5.3。配置`SpringBeanJobFactory`

`SpringBeanJobFactory`支持在创建实例时将调度器上下文、作业数据映射和触发器数据条目作为属性注入作业 bean。

然而，它缺乏对从**应用程序上下文**注入 bean 引用的支持。感谢[这篇博文](https://web.archive.org/web/20220812055703/http://www.btmatthews.com/blog/2011/inject-application-context+dependencies-in-quartz-job-beans.html)的作者，我们可以给 *SpringBeanJobFactory:* 添加**自动布线**支持

```java
@Bean
public SpringBeanJobFactory springBeanJobFactory() {
    AutoWiringSpringBeanJobFactory jobFactory = new AutoWiringSpringBeanJobFactory();
    jobFactory.setApplicationContext(applicationContext);
    return jobFactory;
}
```

## 6。结论

在本文中，我们使用 Quartz API 以及 Spring 的便利类构建了第一个基本调度程序。

关键的一点是，我们能够只用几行代码来配置作业，而不使用任何基于 XML 的配置。

在[github 项目](https://web.archive.org/web/20220812055703/https://github.com/eugenp/tutorials/tree/master/spring-quartz)中可以获得该示例的完整**源代码**。这是一个 Maven 项目，所以我们可以导入它并按原样运行它。默认设置使用 Spring 的便利类，但是我们可以很容易地用一个运行时参数将其切换到 Quartz API(参考存储库中的 README.md)。**