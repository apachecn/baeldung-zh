# 夏洛克春季旅游指南

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/shedlock-spring>

## 1.概观

Spring 提供了一种简单的方法来实现调度作业的 API。在我们部署应用程序的多个实例之前，它一直工作得很好。

默认情况下，Spring 不能处理多个实例上的调度程序同步。而是在每个节点上同时执行作业。

在这个简短的教程中，我们将看看 shed lock——一个 Java 库，它确保我们的预定任务在同一时间只运行一次**,并且是 [Quartz](/web/20220626082247/https://www.baeldung.com/quartz) 的替代方案。**

## 2.Maven 依赖性

要在 Spring 中使用 ShedLock，我们需要添加 [和`shedlock-spring`依赖关系](https://web.archive.org/web/20220626082247/https://search.maven.org/search?q=g:net.javacrumbs.shedlock%20AND%20a:shedlock-spring&core=gav):

```java
<dependency>
    <groupId>net.javacrumbs.shedlock</groupId>
    <artifactId>shedlock-spring</artifactId>
    <version>2.2.0</version>
</dependency>
```

## 3.配置

**注意，ShedLock 只在具有共享数据库的环境中通过声明适当的`LockProvider`来工作。**它在数据库中创建一个表或文档，存储关于当前锁的信息。

目前，ShedLock 支持 Mongo，Redis，Hazelcast，ZooKeeper 和任何有 JDBC 驱动的软件。

对于这个例子，**我们将使用内存中的 H2 数据库。**

为了让它工作，我们需要提供 [H2 数据库](https://web.archive.org/web/20220626082247/https://search.maven.org/search?q=g:com.h2database%20AND%20a:h2)和[谢德洛克 JDBC 依赖关系](https://web.archive.org/web/20220626082247/https://search.maven.org/search?q=g:net.javacrumbs.shedlock%20AND%20a:shedlock-provider-jdbc-template&core=gav):

```java
<dependency>
    <groupId>net.javacrumbs.shedlock</groupId>
    <artifactId>shedlock-provider-jdbc-template</artifactId>
    <version>2.1.0</version>
</dependency>
<dependency>
     <groupId>com.h2database</groupId>
     <artifactId>h2</artifactId>
     <version>1.4.200</version>
</dependency>
```

接下来，我们需要为 ShedLock 创建一个数据库表来保存关于调度器锁的信息:

```java
CREATE TABLE shedlock (
  name VARCHAR(64),
  lock_until TIMESTAMP(3) NULL,
  locked_at TIMESTAMP(3) NULL,
  locked_by VARCHAR(255),
  PRIMARY KEY (name)
)
```

我们应该在 Spring Boot 应用程序的属性文件中声明数据源，这样`DataSource` bean 就可以是`Autowired`。

这里我们用`application.yml`来定义 H2 数据库的数据源:

```java
spring:
  datasource:
    driverClassName: org.h2.Driver
    url: jdbc:h2:mem:shedlock_DB;INIT=CREATE SCHEMA IF NOT EXISTS shedlock;DB_CLOSE_DELAY=-1;DB_CLOSE_ON_EXIT=FALSE
    username: sa
    password: 
```

让我们用上面的数据源配置来配置`LockProvider`。

Spring 可以让它变得非常简单:

```java
@Configuration
public class SchedulerConfiguration {
    @Bean
    public LockProvider lockProvider(DataSource dataSource) {
        return new JdbcTemplateLockProvider(dataSource);
    }
}
```

我们必须提供的其他配置需求是 Spring 配置类上的`@EnableScheduling`和`@EnableSchedulerLock`注释:

```java
@SpringBootApplication
@EnableScheduling
@EnableSchedulerLock(defaultLockAtMostFor = "PT30S")
public class Application {

    public static void main(String[] args) {
        SpringApplication.run(SpringApplication.class, args);
    }
}
```

`defaultLockAtMostFor`参数指定在执行节点死亡的情况下锁应该保持的默认时间量。它使用 [ISO8601 持续时间](https://web.archive.org/web/20220626082247/https://en.wikipedia.org/wiki/ISO_8601#Durations)的格式。

在下一节中，我们将看到如何覆盖这个缺省值。

## 4.创建任务

要创建一个由 ShedLock 处理的调度任务，我们只需在一个方法上添加`@Scheduled`和`@SchedulerLock`注释:

```java
@Component
class BaeldungTaskScheduler {

    @Scheduled(cron = "0 0/15 * * * ?")
    @SchedulerLock(name = "TaskScheduler_scheduledTask", 
      lockAtLeastForString = "PT5M", lockAtMostForString = "PT14M")
    public void scheduledTask() {
        // ...
    }
}
```

首先，我们来看看`@Scheduled`。它支持[的`cron`格式](https://web.archive.org/web/20220626082247/https://crontab.guru/)，这个表达的意思是“每 15 分钟”

接下来，看看`@SchedulerLock`,`name`参数必须是唯一的，而`ClassName_methodName`通常足以实现这一点。我们不希望这个方法同时运行不止一次，ShedLock 使用这个惟一的名称来实现这一点。

我们还添加了几个可选参数。

首先，我们添加了`lockAtLeastForString`,这样我们可以在方法调用之间保持一定的距离。使用`“PT5M” `意味着这个方法将保持锁至少五分钟。换句话说，**这意味着 ShedLock 运行该方法的频率不能超过每五分钟一次。**

接下来，我们添加了`lockAtMostForString`来指定锁应该保持多长时间，以防执行节点死亡。使用`“PT14M” `意味着锁定时间不会超过 14 分钟。

在正常情况下，ShedLock 会在任务完成后直接释放锁。现在，我们不必这样做，因为**在** `**@EnableSchedulerLock**`中提供了一个默认值，但是我们在这里选择了覆盖它。

## 5.结论

在本文中，我们学习了如何使用 ShedLock 创建和同步调度任务。

和往常一样，GitHub 上的所有源代码[都是可用的。](https://web.archive.org/web/20220626082247/https://github.com/eugenp/tutorials/tree/master/spring-boot-modules/spring-boot-libraries)