# Spring 5 中的并发测试执行

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-5-concurrent-tests>

## 1。简介

从`JUnit 4`开始，测试可以并行运行，以提高大型套件的速度。问题是在`Spring 5`之前的`Spring TestContext Framework`并不完全支持并发测试的执行。

在这篇简短的文章中，我们将展示如何**使用`Spring 5`在`Spring`项目中同时运行我们的测试**。

## 2。Maven 设置

提醒一下，要并行运行`JUnit`测试，我们需要配置 [`maven-surefire-plugin`](https://web.archive.org/web/20220908005809/https://search.maven.org/classic/#search%7Cgav%7C1%7Cg%3A%22org.apache.maven.plugins%22%20AND%20a%3A%22maven-surefire-plugin%22) 来启用这个特性:

```java
<build>
    <plugin>
        <groupId>org.apache.maven.plugins</groupId>
        <artifactId>maven-surefire-plugin</artifactId>
        <version>2.22.2</version>
        <configuration>
            <parallel>methods</parallel>
            <useUnlimitedThreads>true</useUnlimitedThreads>
        </configuration>
    </plugin>
</build>
```

关于并行测试执行的更详细的配置，您可以查看参考文档。

## 3。并发测试

以下示例测试在针对`Spring 5`之前的版本并行运行时会失败。

不过在`Spring 5`会运行的很流畅:

```java
@RunWith(SpringRunner.class)
@ContextConfiguration(classes = Spring5JUnit4ConcurrentTest.SimpleConfiguration.class)
public class Spring5JUnit4ConcurrentTest implements ApplicationContextAware, InitializingBean {

    @Configuration
    public static class SimpleConfiguration {}

    private ApplicationContext applicationContext;

    private boolean beanInitialized = false;

    @Override
    public void afterPropertiesSet() throws Exception {
        this.beanInitialized = true;
    }

    @Override
    public void setApplicationContext(
      final ApplicationContext applicationContext) throws BeansException {
        this.applicationContext = applicationContext;
    }

    @Test
    public void whenTestStarted_thenContextSet() throws Exception {
        TimeUnit.SECONDS.sleep(2);

        assertNotNull(
          "The application context should have been set due to ApplicationContextAware semantics.",
          this.applicationContext);
    }

    @Test
    public void whenTestStarted_thenBeanInitialized() throws Exception {
        TimeUnit.SECONDS.sleep(2);

        assertTrue(
          "This test bean should have been initialized due to InitializingBean semantics.",
          this.beanInitialized);
    }
}
```

当按顺序运行时，上述测试大约需要 6 秒钟才能通过。使用并发执行，只需要大约 4.5 秒，这是我们在更大的套件中可以节省的典型时间。

## 4。引擎盖下

以前版本的框架不支持并发运行测试的主要原因是由于由`TestContextManager`管理`TestContext`。

在`Spring 5`中， [`TestContextManager`](https://web.archive.org/web/20220908005809/https://docs.spring.io/spring/docs/5.0.0.M5/javadoc-api/org/springframework/test/context/TestContextManager.html) 使用一个线程 local——[`TestContext`](https://web.archive.org/web/20220908005809/https://docs.spring.io/spring/docs/5.0.0.M5/javadoc-api/org/springframework/test/context/TestContext.html)——来保证每个线程中`TestContexts`上的操作不会互相干扰。因此，对于大多数方法级和类级并发测试来说，线程安全是有保证的:

```java
public class TestContextManager {

    // ...
    private final TestContext testContext;

    private final ThreadLocal<TestContext> testContextHolder = new ThreadLocal<TestContext>() {
        protected TestContext initialValue() {
            return copyTestContext(TestContextManager.this.testContext);
        }
    };

    public final TestContext getTestContext() {
        return this.testContextHolder.get();
    }

    // ...
}
```

注意并发支持并不适用于所有类型的测试；我们需要排除那些测试:

*   更改外部共享状态，如缓存、数据库、消息队列等中的状态。
*   要求具体的执行顺序，例如，使用`JUnit`的 [`@FixMethodOrder`](https://web.archive.org/web/20220908005809/http://junit.org/junit4/javadoc/4.12/org/junit/FixMethodOrder.html) 的测试
*   修改`ApplicationContext`，一般用 [`@DirtiesContext`](https://web.archive.org/web/20220908005809/https://docs.spring.io/spring/docs/current/javadoc-api/org/springframework/test/annotation/DirtiesContext.html) 标注

## 5。总结

在这个快速教程中，我们展示了一个使用`Spring 5`并行运行测试的基本例子。

和往常一样，示例代码可以在 Github 上找到[。](https://web.archive.org/web/20220908005809/https://github.com/eugenp/tutorials/tree/master/testing-modules/spring-testing-2)