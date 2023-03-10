# 如何测试@Scheduled 注释

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-testing-scheduled-annotation>

## 1。简介

在 [Spring 框架](/web/20220628161001/https://www.baeldung.com/spring-intro)中可用的注释之一是`@Scheduled`。我们可以使用这个注释让[按照预定的方式](/web/20220628161001/https://www.baeldung.com/spring-scheduled-tasks)执行任务。

在本教程中，我们将探索如何测试`@Scheduled`注释。

## 2。依赖性

首先，让我们从 [Spring 初始化器](https://web.archive.org/web/20220628161001/https://start.spring.io/)开始创建一个基于 [Spring Boot](/web/20220628161001/https://www.baeldung.com/spring-boot) Maven 的应用程序:

```java
<parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>2.6.1</version>
    <relativePath/>
</parent>
```

我们还需要使用几个 Spring Boot 首发:

```java
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-test</artifactId>
    <scope>test</scope>
</dependency>
```

让我们将 [JUnit 5](/web/20220628161001/https://www.baeldung.com/junit-5) 的依赖项添加到我们的`pom.xml`:

```java
<dependency>
    <groupId>org.junit.jupiter</groupId>
    <artifactId>junit-jupiter-api</artifactId>
</dependency>
```

我们可以在 Maven Central 上找到 Spring Boot 的最新版本。

此外，为了在我们的测试中使用[可用性](https://web.archive.org/web/20220628161001/https://search.maven.org/search?q=g:org.awaitility%20AND%20a:awaitility)，我们需要添加它的依赖项:

```java
<dependency>
    <groupId>org.awaitility</groupId>
    <artifactId>awaitility</artifactId>
    <version>3.1.6</version>
    <scope>test</scope>
</dependency>
```

## 3。简单的`@Scheduled`样品

让我们从创建一个简单的`Counter`类开始:

```java
@Component
public class Counter {
    private AtomicInteger count = new AtomicInteger(0);

    @Scheduled(fixedDelay = 5)
    public void scheduled() {
        this.count.incrementAndGet();
    }

    public int getInvocationCount() {
        return this.count.get();
    }
}
```

我们将使用`scheduled`方法来增加我们的`count`。注意，我们还添加了`@Scheduled`注释，以便在 5 毫秒的固定时间内执行它。

同样，让我们创建一个`ScheduledConfig`类来使用`@EnableScheduling`注释启用调度任务:

```java
@Configuration
@EnableScheduling
@ComponentScan("com.baeldung.scheduled")
public class ScheduledConfig {
}
```

## 4。使用集成测试

测试我们的类的替代方法之一是使用[集成测试](/web/20220628161001/https://www.baeldung.com/integration-testing-in-spring)。为此，我们需要使用`@SpringJUnitConfig`注释在测试环境中启动应用程序上下文和 beans:

```java
@SpringJUnitConfig(ScheduledConfig.class)
public class ScheduledIntegrationTest {

    @Autowired 
    Counter counter;

    @Test
    public void givenSleepBy100ms_whenGetInvocationCount_thenIsGreaterThanZero() 
      throws InterruptedException {
        Thread.sleep(100L);

        assertThat(counter.getInvocationCount()).isGreaterThan(0);
    }
}
```

在这种情况下，我们启动我们的`Counter` bean 并等待 100 毫秒来检查调用计数。

## 5。使用意识

测试计划任务的另一种方法是使用[可用性](/web/20220628161001/https://www.baeldung.com/awaitlity-testing)。我们可以使用 Awaitility DSL 来使我们的测试更具声明性:

```java
@SpringJUnitConfig(ScheduledConfig.class)
public class ScheduledAwaitilityIntegrationTest {

    @SpyBean 
    private Counter counter;

    @Test
    public void whenWaitOneSecond_thenScheduledIsCalledAtLeastTenTimes() {
        await()
          .atMost(Duration.ONE_SECOND)
          .untilAsserted(() -> verify(counter, atLeast(10)).scheduled());
    }
}
```

在这种情况下，我们给 bean 注入了`[@SpyBean](https://web.archive.org/web/20220628161001/https://docs.spring.io/spring-boot/docs/current/api/org/springframework/boot/test/mock/mockito/SpyBean.html)`注释，以检查在一秒钟内调用`scheduled`方法的次数。

## 6。结论

在本教程中，我们展示了一些使用集成测试和可用性库来测试计划任务的方法。

我们需要考虑到，尽管集成测试很好，**通常更好的做法是关注预定方法**中逻辑的单元测试。

像往常一样，本教程中显示的所有代码示例都可以在 [GitHub](https://web.archive.org/web/20220628161001/https://github.com/eugenp/tutorials/tree/master/testing-modules/spring-testing) 上获得。