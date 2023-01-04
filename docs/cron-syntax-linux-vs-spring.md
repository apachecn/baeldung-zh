# Linux 和 Spring 中 Cron 语法的区别

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/cron-syntax-linux-vs-spring>

## 1.概观

Cron 表达式使我们能够安排任务在特定的日期和时间定期运行。在它被引入 Unix 之后，其他基于 Unix 的操作系统和软件库(包括 Spring 框架)都采用了它的任务调度方法。

在这个快速教程中，我们将看到基于 Unix 的操作系统中的 Cron 表达式和 Spring 框架之间的区别。

## 2.Unix Cron

[Cron](https://web.archive.org/web/20220525121552/https://www.manpagez.com/man/5/crontab/) 在大多数基于 Unix 的系统中有五个字段:**分钟(0-59)、小时(0-23)、一个月中的某天(1-31)、月份(1-12 或名称)、星期几(0-7 或名称)。**

我们可以在每个字段中输入一些特殊的值，比如星号(*):

```java
5 0 * * *
```

该作业将在每天午夜后 5 分钟执行。也可以使用一系列值:

```java
5 0-5 * * *
```

在这里，调度程序将在午夜后 5 分钟执行任务，并且在每天 1 点、2 点、3 点、4 点和 5 点后 5 分钟执行任务。

或者，我们可以使用一个值列表:

```java
5 0,3 * * *
```

现在调度程序每天在午夜后 5 分钟和 3 点后 5 分钟执行作业。最初的 Cron 表达式提供了比我们到目前为止所介绍的更多的特性。

然而，它有一个很大的限制:我们不能用秒精度调度作业，因为它没有专用的秒字段。

让我们看看 Spring 是如何解决这个限制的。

## 3.春天克隆

为了在 Spring 中调度周期性的后台任务，我们通常将一个 Cron 表达式传递给`[@Scheduled](/web/20220525121552/https://www.baeldung.com/spring-scheduled-tasks#schedule-a-task-using-cron-expressions) `注释。

与基于 Unix 的系统中的 Cron 表达式不同，**Spring 中的 Cron 表达式有六个空格分隔的字段:秒、分钟、小时、日、月和工作日**。

例如，要每十秒钟运行一个任务，我们可以这样做:

```java
*/10 * * * * *
```

另外，要在每天早上 8 点到晚上 10 点每 20 秒运行一次任务:

```java
*/20 * 8-10 * * *
```

如上例所示，**第一个字段代表表达式的第二部分。这就是两种实现的区别。**尽管第二个字段有所不同，但 Spring 支持许多来自原始 Cron 的特性，比如范围号或列表。

从实现的角度来看， [`CronSequenceGenerator`](https://web.archive.org/web/20220525121552/https://docs.spring.io/spring/docs/current/javadoc-api/org/springframework/scheduling/support/CronSequenceGenerator.html) 类负责解析 Spring 中的 Cron 表达式。

## 4.结论

在这个简短的教程中，我们看到了 Spring 和大多数基于 Unix 的系统之间的 Cron 实现差异。在这个过程中，我们看到了两种实现的一些例子。

为了看到更多 Cron 表达式的例子，强烈推荐查看我们的[Cron 表达式指南](/web/20220525121552/https://www.baeldung.com/cron-expressions)。此外，看一看 [`CronSequenceGenerator`](https://web.archive.org/web/20220525121552/https://github.com/spring-projects/spring-framework/blob/master/spring-context/src/main/java/org/springframework/scheduling/support/CronSequenceGenerator.java) 类的源代码可以让我们很好地了解 Spring 是如何实现这个特性的。