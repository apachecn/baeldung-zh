# 弹簧重试指南

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-retry>

## 1。概述

Spring Retry 提供了自动重新调用失败操作的能力。这在错误可能是暂时的(如瞬时网络故障)时很有帮助。

在本教程中，我们将看到使用 [Spring Retry](https://web.archive.org/web/20220821110620/https://github.com/spring-projects/spring-retry) 的各种方式:注释、`RetryTemplate`和回调。

## 延伸阅读:

## [具有指数补偿和抖动的更好重试](/web/20220821110620/https://www.baeldung.com/resilience4j-backoff-jitter)

Learn how to better control your application retries using backoff and jitter from Resilience4j.[Read more](/web/20220821110620/https://www.baeldung.com/resilience4j-backoff-jitter) →

## [弹性指南 4j](/web/20220821110620/https://www.baeldung.com/resilience4j)

Learn how to use the most useful modules from the Resilience4j library to build resilient systems.[Read more](/web/20220821110620/https://www.baeldung.com/resilience4j) →

## [在 Spring 批处理中配置重试逻辑](/web/20220821110620/https://www.baeldung.com/spring-batch-retry-logic)

Spring Batch allows us to set retry strategies on tasks so that they are automatically repeated when there is an error. Here we see how to configure it.[Read more](/web/20220821110620/https://www.baeldung.com/spring-batch-retry-logic) →

## 2。Maven 依赖关系

让我们从**将 `spring-retry`依赖项添加到我们的`pom.xml` 文件**开始:

```java
<dependency>
    <groupId>org.springframework.retry</groupId>
    <artifactId>spring-retry</artifactId>
    <version>1.2.5.RELEASE</version>
</dependency>
```

我们还需要将 Spring AOP 添加到我们的项目中:

```java
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-aspects</artifactId>
    <version>5.2.8.RELEASE</version>
</dependency>
```

看看 Maven Central 上的最新版本的 [spring-retry](https://web.archive.org/web/20220821110620/https://search.maven.org/search?q=spring-retry) 和 [spring-aspects](https://web.archive.org/web/20220821110620/https://search.maven.org/search?q=a:spring-aspects) 依赖项。

## 3。启用弹簧重试

为了在应用程序中启用 Spring Retry，**我们需要将`@EnableRetry`注释**添加到我们的`@Configuration` 类中:

```java
@Configuration
@EnableRetry
public class AppConfig { ... }
```

## 4。使用弹簧重试

### 4.1。`@Retryable`没有恢复

**我们可以使用`@Retryable`注释为方法**添加重试功能:

```java
@Service
public interface MyService {
    @Retryable(value = RuntimeException.class)
    void retryService(String sql);

}
```

这里，当抛出`.`时，尝试重试

根据`@Retryable`的默认行为，**重试最多可能发生三次，两次重试之间有一秒钟的延迟。**

### 4.2。`@Retryable` **`@Recover`**

**现在让我们使用`@Recover`注释**添加一个恢复方法:

```java
@Service
public interface MyService {
    @Retryable(value = SQLException.class)
    void retryServiceWithRecovery(String sql) throws SQLException;

    @Recover
    void recover(SQLException e, String sql);
}
```

这里，当一个`SQLException` 被抛出`.`**`@Recover`注释定义了一个单独的恢复方法，当一个`@Retryable`方法因指定的异常而失败时。**

因此，如果`retryServiceWithRecovery` 方法在三次尝试后继续抛出一个`SqlException`，那么`recover()`方法将被调用。

恢复处理程序应该有第一个类型为 *Throwable* (可选)的参数和相同的返回类型。下面的 参数以相同的顺序从失败方法的参数列表中填充。

### 4.3。定制`@Retryable's`行为

为了定制一个重试的行为，**我们可以使用参数`maxAttempts`和** `**backoff**`:

```java
@Service
public interface MyService {
    @Retryable( value = SQLException.class, 
      maxAttempts = 2, backoff = @Backoff(delay = 100))
    void retryServiceWithCustomization(String sql) throws SQLException;
}
```

最多会有两次尝试和 100 毫秒的延迟。

### 4.4。使用弹簧属性

我们也可以在`@Retryable`注释中使用属性。

为了演示这一点，**我们将看到如何将`delay`和`maxAttempts`的值具体化到一个属性文件中。**

首先，让我们在一个名为`retryConfig.` `properties`的文件中定义属性:

```java
retry.maxAttempts=2
retry.maxDelay=100
```

然后，我们指示我们的`@Configuration`类加载这个文件:

```java
// ...
@PropertySource("classpath:retryConfig.properties")
public class AppConfig { ... }
```

最后，**我们可以在我们的`@Retryable`定义**中注入`retry.maxAttempts`和`retry.maxDelay`的值:

```java
@Service 
public interface MyService { 
  @Retryable( value = SQLException.class, maxAttemptsExpression = "${retry.maxAttempts}",
            backoff = @Backoff(delayExpression = "${retry.maxDelay}")) 
  void retryServiceWithExternalConfiguration(String sql) throws SQLException; 
}
```

请注意**我们现在用的是`maxAttemptsExpression`和`delayExpression`T5，而不是`maxAttempts`和`delay`。**

## 5。`RetryTemplate`

### 5.1。`RetryOperations`

Spring Retry 提供了`RetryOperations`接口，该接口提供了一组`execute()`方法:

```java
public interface RetryOperations {
    <T> T execute(RetryCallback<T> retryCallback) throws Exception;

    ...
}
```

`RetryCallback`是`execute()`的参数，是允许插入失败后需要重试的业务逻辑的接口:

```java
public interface RetryCallback<T> {
    T doWithRetry(RetryContext context) throws Throwable;
}
```

### 5.2。`RetryTemplate`配置

`RetryTemplate`是`RetryOperations`的一个实现。

让我们在我们的`@Configuration`类中配置一个`RetryTemplate` bean:

```java
@Configuration
public class AppConfig {
    //...
    @Bean
    public RetryTemplate retryTemplate() {
        RetryTemplate retryTemplate = new RetryTemplate();

        FixedBackOffPolicy fixedBackOffPolicy = new FixedBackOffPolicy();
        fixedBackOffPolicy.setBackOffPeriod(2000l);
        retryTemplate.setBackOffPolicy(fixedBackOffPolicy);

        SimpleRetryPolicy retryPolicy = new SimpleRetryPolicy();
        retryPolicy.setMaxAttempts(2);
        retryTemplate.setRetryPolicy(retryPolicy);

        return retryTemplate;
    }
} 
```

`RetryPolicy` 决定何时应该重试操作。

`SimpleRetryPolicy` 用于重试固定次数。另一方面，`BackOffPolicy` 用于控制重试尝试之间的回退。

最后，`FixedBackOffPolicy` 在继续之前暂停一段固定的时间。

### 5.3。使用`RetryTemplate`

为了运行带有重试处理的代码，我们可以调用`r` `etryTemplate.execute()` 方法 :

```java
retryTemplate.execute(new RetryCallback<Void, RuntimeException>() {
    @Override
    public Void doWithRetry(RetryContext arg0) {
        myService.templateRetryService();
        ...
    }
});
```

我们可以使用 lambda 表达式来代替匿名类:

```java
retryTemplate.execute(arg0 -> {
    myService.templateRetryService();
    return null;
}); 
```

## 6。听众

侦听器在重试时提供额外的回调。我们可以在不同的重试中使用这些来处理各种交叉问题。

### 6.1。添加回调

回调在一个`RetryListener`接口中提供:

```java
public class DefaultListenerSupport extends RetryListenerSupport {
    @Override
    public <T, E extends Throwable> void close(RetryContext context,
      RetryCallback<T, E> callback, Throwable throwable) {
        logger.info("onClose);
        ...
        super.close(context, callback, throwable);
    }

    @Override
    public <T, E extends Throwable> void onError(RetryContext context,
      RetryCallback<T, E> callback, Throwable throwable) {
        logger.info("onError"); 
        ...
        super.onError(context, callback, throwable);
    }

    @Override
    public <T, E extends Throwable> boolean open(RetryContext context,
      RetryCallback<T, E> callback) {
        logger.info("onOpen);
        ...
        return super.open(context, callback);
    }
}
```

`open`和`close`回调出现在整个重试之前和之后，而`onError` 适用于单个`RetryCallback` 调用。

### 6.2。注册监听器

接下来，我们将侦听器(`DefaultListenerSupport)` 注册到我们的`RetryTemplate` bean:

```java
@Configuration
public class AppConfig {
    ...

    @Bean
    public RetryTemplate retryTemplate() {
        RetryTemplate retryTemplate = new RetryTemplate();
        ...
        retryTemplate.registerListener(new DefaultListenerSupport());
        return retryTemplate;
    }
}
```

## 7。测试结果

为了结束我们的示例，让我们验证结果:

```java
@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration(
  classes = AppConfig.class,
  loader = AnnotationConfigContextLoader.class)
public class SpringRetryIntegrationTest {

    @Autowired
    private MyService myService;

    @Autowired
    private RetryTemplate retryTemplate;

    @Test(expected = RuntimeException.class)
    public void givenTemplateRetryService_whenCallWithException_thenRetry() {
        retryTemplate.execute(arg0 -> {
            myService.templateRetryService();
            return null;
        });
    }
}
```

从测试日志中我们可以看到，我们已经正确地配置了`RetryTemplate`和`RetryListener`:

```java
2020-01-09 20:04:10 [main] INFO  o.b.s.DefaultListenerSupport - onOpen 
2020-01-09 20:04:10 [main] INFO  o.baeldung.springretry.MyServiceImpl
- throw RuntimeException in method templateRetryService() 
2020-01-09 20:04:10 [main] INFO  o.b.s.DefaultListenerSupport - onError 
2020-01-09 20:04:12 [main] INFO  o.baeldung.springretry.MyServiceImpl
- throw RuntimeException in method templateRetryService() 
2020-01-09 20:04:12 [main] INFO  o.b.s.DefaultListenerSupport - onError 
2020-01-09 20:04:12 [main] INFO  o.b.s.DefaultListenerSupport - onClose
```

## 8。 **结论**

在本文中，我们看到了如何使用注释、`RetryTemplate`和回调监听器来使用 Spring Retry。

GitHub 上的[提供了示例的源代码。](https://web.archive.org/web/20220821110620/https://github.com/eugenp/tutorials/tree/master/spring-scheduling)