# 在春季有条件地启用计划作业

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-scheduled-enabled-conditionally>

## 1.介绍

[Spring Scheduling](/web/20221129021325/https://www.baeldung.com/spring-scheduled-tasks) 库允许应用程序在特定的时间间隔执行代码。因为时间间隔是使用`@Scheduled`注释指定的，**时间间隔通常是静态的，在应用程序的生命周期中不能改变**。

在本教程中，我们将研究有条件地启用 Spring 调度作业的各种方法。

## 2.使用布尔标志

**有条件地启用 Spring 调度作业的最简单方法是使用一个`boolean`变量**，我们在调度作业中检查这个变量。该变量可以用`@Value`进行注释，使其可以使用普通的[弹簧配置机制](/web/20221129021325/https://www.baeldung.com/properties-with-spring)进行配置:

```java
@Configuration
@EnableScheduling
public class ScheduledJobs {
  @Value("${jobs.enabled:true}")
  private boolean isEnabled;

  @Scheduled(fixedDelay = 60000)
  public void cleanTempDirectory() {
    if(isEnabled) {
      // do work here
    }
  }
}
```

缺点是**调度的作业总是由 Spring** 执行，这在某些情况下可能并不理想。

## 3.使用`@ConditionalOnProperty`

另一个选择是使用`@ConditionalOnProperty`注释。它采用一个 Spring 属性名，并且只有当属性的值为`true.`时才运行

首先，我们创建一个新的类，它封装了调度作业代码，包括调度间隔:

```java
public class ScheduledJob {
    @Scheduled(fixedDelay = 60000)
    public void cleanTempDir() {
        // do work here
  }
}
```

然后，我们有条件地创建该类型的 bean:

```java
@Configuration
@EnableScheduling
public class ScheduledJobs {
    @Bean
    @ConditionalOnProperty(value = "jobs.enabled", matchIfMissing = true, havingValue = "true")
    public ScheduledJob scheduledJob() {
        return new ScheduledJob();
    }
}
```

在这种情况下，如果属性`jobs.enabled`设置为`true,`或者根本不存在，作业将运行。缺点是这个注释**只在 Spring Boot 可用。**

## 4.使用弹簧轮廓

我们还可以根据应用程序运行时使用的[概要文件](/web/20221129021325/https://www.baeldung.com/spring-profiles)有条件地启用 Spring 调度作业。例如，当作业只应在生产环境中调度时，这种方法就很有用。

当时间表在所有环境中都相同，并且只需要在特定的配置文件中禁用或启用时，这种方法很有效**。**

这类似于使用`@ConditionalOnProperty`，除了我们在 bean 方法上使用`@Profile`注释:

```java
@Profile("prod")
@Bean
public ScheduledJob scheduledJob() {
    return new ScheduledJob();
}
```

只有当`prod`配置文件处于活动状态时，**才会创建工作。此外，它为我们提供了带有`@Profile`注释的全套选项:匹配多个概要文件、复杂的弹簧表达式等等。**

使用这种方法需要注意的一点是，如果根本没有指定概要文件，bean 方法就会被执行。

## 5.Cron 表达式中的值占位符

使用 Spring 值占位符，我们不仅可以有条件地启用一个作业，还可以更改它的调度:

```java
@Scheduled(cron = "${jobs.cronSchedule:-}")
public void cleanTempDirectory() {
    // do work here
}
```

在本例中，默认情况下禁用该作业(使用特殊的 Spring cron disable 表达式)。

如果我们想要启用这个任务，我们所要做的就是为`jobs.cronSchedule.`提供一个有效的 cron 表达式，我们可以像任何其他 Spring 配置一样做这件事:命令行参数、环境变量、属性文件等等。

与 cron 表达式不同，无法设置禁用作业的固定延迟或固定速率值。因此**这种方法只适用于 cron 调度作业**。

## 6.结论

在本教程中，我们看到了有条件地启用 Spring 调度作业的几种不同方式。有些方法比其他方法简单，但可能有局限性。

GitHub 上的[提供了示例的完整源代码。](https://web.archive.org/web/20221129021325/https://github.com/eugenp/tutorials/tree/master/spring-scheduling)