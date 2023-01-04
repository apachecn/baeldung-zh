# 春季测试执行监听器

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-testexecutionlistener>

## 1。概述

通常，我们使用像`@BeforeEach, @AfterEach, @BeforeAll,`和`@AfterAll,` 这样的 JUnit 注释来编排测试的生命周期，但有时这还不够——尤其是当我们使用 Spring 框架时。

这就是 Spring `TestExecutionListener`派上用场的地方。

在本教程中，**我们将看到`TestExecutionListener`提供了什么，Spring 提供的默认监听器，以及如何实现自定义的`TestExecutionListener`。**

## 2。`TestExecutionListener`界面

首先，我们来访问一下`TestExecutionListener`界面:

```
public interface TestExecutionListener {
    default void beforeTestClass(TestContext testContext) throws Exception {};
    default void prepareTestInstance(TestContext testContext) throws Exception {};
    default void beforeTestMethod(TestContext testContext) throws Exception {};
    default void afterTestMethod(TestContext testContext) throws Exception {};
    default void afterTestClass(TestContext testContext) throws Exception {};
}
```

该接口的实现可以在不同的测试执行阶段接收事件。因此，接口中的每个方法都被传递了一个`TestContext`对象。

这个`TestContext`对象包含了 Spring 上下文以及目标测试类和方法的信息。这些信息可以用来改变测试的行为或者扩展它们的功能。

现在，让我们快速浏览一下这些方法:

*   `afterTestClass`–在类内所有测试执行后，后处理一个测试类
***   `afterTestExecution`–在所提供的测试环境中执行测试方法后，立即对测试**进行后处理***   `afterTestMethod`–在执行底层测试框架的生命周期后回调之后，对测试**进行后处理***   `beforeTestClass`–在执行类内所有测试之前，预处理测试类***   `beforeTestExecution`–在所提供的测试环境中执行测试方法之前，立即预处理测试***   `beforeTestMethod`–在执行底层测试框架的生命周期前回调之前，预处理测试***   `prepareTestInstance`–准备所提供的测试上下文的测试实例********

 ******值得注意的是，这个接口为所有方法提供了空的默认实现。因此，具体的实现可以选择只覆盖那些适合手头任务的方法。

## 3。`TestExecutionListeners`春天的默认

默认情况下，Spring 提供了一些现成的`TestExecutionListener`实现。

让我们快速看一下其中的每一项:

*   `ServletTestExecutionListener`–为`WebApplicationContext`配置 Servlet API 模拟
*   `DirtiesContextBeforeModesTestExecutionListener`–处理“之前”模式的`@DirtiesContext`注释
*   `DependencyInjectionTestExecutionListener`–为测试实例提供依赖注入
*   `DirtiesContextTestExecutionListener`–处理“之后”模式的`@DirtiesContext`注释
*   `TransactionalTestExecutionListener`–提供带有默认回滚语义的事务性测试执行
*   `SqlScriptsTestExecutionListener`–运行使用`@Sql`注释配置的 SQL 脚本

这些侦听器是按照列出的顺序预先注册的。当我们创建一个自定义`TestExecutionListener`时，我们将看到更多关于订单的信息。

## 4。`TestExecutionListener`使用自定义

现在，让我们定义一个自定义`TestExecutionListener`:

```
public class CustomTestExecutionListener implements TestExecutionListener, Ordered {
    private static final Logger logger = LoggerFactory.getLogger(CustomTestExecutionListener.class);

    public void beforeTestClass(TestContext testContext) throws Exception {
        logger.info("beforeTestClass : {}", testContext.getTestClass());
    }; 

    public void prepareTestInstance(TestContext testContext) throws Exception {
        logger.info("prepareTestInstance : {}", testContext.getTestClass());
    }; 

    public void beforeTestMethod(TestContext testContext) throws Exception {
        logger.info("beforeTestMethod : {}", testContext.getTestMethod());
    }; 

    public void afterTestMethod(TestContext testContext) throws Exception {
        logger.info("afterTestMethod : {}", testContext.getTestMethod());
    }; 

    public void afterTestClass(TestContext testContext) throws Exception {
        logger.info("afterTestClass : {}", testContext.getTestClass());
    }

    @Override
    public int getOrder() {
        return Integer.MAX_VALUE;
    };
}
```

为了简单起见，这个类所做的只是记录一些`TestContext`信息。

### 4.1。使用`@TestExecutionListeners` 注册定制监听器

现在，让我们在测试类中使用这个监听器。为此，我们将使用`@TestExecutionListeners`注释来注册它:

```
@RunWith(SpringRunner.class)
@TestExecutionListeners(value = {
  CustomTestExecutionListener.class,
  DependencyInjectionTestExecutionListener.class
})
@ContextConfiguration(classes = AdditionService.class)
public class AdditionServiceUnitTest {
    // ...
}
```

需要注意的是，使用注释的**将注销所有默认的监听器**。因此，我们已经明确地添加了`DependencyInjectionTestExecutionListener`，这样我们就可以在我们的测试类中使用自动连接。

如果我们需要任何其他的默认侦听器，我们必须指定它们中的每一个。但是，我们也可以使用注释的`mergeMode`属性:

```
@TestExecutionListeners(
  value = { CustomTestExecutionListener.class }, 
  mergeMode = MergeMode.MERGE_WITH_DEFAULTS)
```

这里，`MERGE_WITH_DEFAULTS`表示本地声明的侦听器应该与默认侦听器合并。

现在，当我们运行上面的测试时，监听器将记录它接收到的每个事件:

```
[main] INFO  o.s.t.c.s.DefaultTestContextBootstrapper - Using TestExecutionListeners: 
[[[email protected]](/web/20220626203154/https://www.baeldung.com/cdn-cgi/l/email-protection)38364841, 
org.springframewor[[email protected]](/web/20220626203154/https://www.baeldung.com/cdn-cgi/l/email-protection)28c4711c]
[main] INFO  c.b.t.CustomTestExecutionListener - beforeTestClass : 
class com.baeldung.testexecutionlisteners.TestExecutionListenersWithoutMergeModeUnitTest
[main] INFO  c.b.t.CustomTestExecutionListener - prepareTestInstance : 
class com.baeldung.testexecutionlisteners.TestExecutionListenersWithoutMergeModeUnitTest
[main] INFO  o.s.c.s.GenericApplicationContext - 
Refreshing [[email protected]](/web/20220626203154/https://www.baeldung.com/cdn-cgi/l/email-protection)68ef40: startup date [XXX]; 
root of context hierarchy
[main] INFO  c.b.t.CustomTestExecutionListener - beforeTestMethod : 
public void com.baeldung.testexecutionlisteners.TestExecutionListenersWithoutMergeModeUnitTest
.whenValidNumbersPassed_thenReturnSum()
[main] INFO  c.b.t.CustomTestExecutionListener - afterTestMethod : 
public void com.baeldung.testexecutionlisteners.TestExecutionListenersWithoutMergeModeUnitTest
.whenValidNumbersPassed_thenReturnSum()
[main] INFO  c.b.t.CustomTestExecutionListener - afterTestClass : 
class com.baeldung.testexecutionlisteners.TestExecutionListenersWithoutMergeModeUnitTest
```

### 4.2。自动发现默认`TestExecutionListener`实现

如果在数量有限的测试类中使用，使用`@TestExecutionListener`来注册监听器是合适的。但是，将它添加到整个测试套件中会变得很麻烦。

我们可以通过利用由自动发现`TestExecutionListener`实现的`SpringFactoriesLoader`机制提供的支持来解决这个问题。

`spring-test`模块在其`META-INF/spring.factories`属性文件中的`org.springframework.test.context.TestExecutionListener`键下声明了所有的核心默认监听器。类似地，我们可以通过在我们自己的`META-INF/spring.factories`属性文件中使用上面的键来**注册我们的定制监听器:**

```
org.springframework.test.context.TestExecutionListener=\
com.baeldung.testexecutionlisteners.CustomTestExecutionListener
```

### 4.3。订购默认`TestExecutionListener`实施

当 Spring 通过`SpringFactoriesLoader`机制发现默认的`TestExecutionListener`实现时，它将使用 Spring 的`AnnotationAwareOrderComparator.`对它们进行排序，这支持 Spring 的`Ordered`接口和用于排序的`@Order`注释。

请注意，Spring 提供的所有默认`TestExecutionListener`实现都使用适当的值来实现`Ordered`。因此，我们必须确保我们的自定义`TestExecutionListener`实现以正确的顺序注册。因此，我们在自定义监听器中实现了`Ordered`:

```
public class CustomTestExecutionListener implements TestExecutionListener, Ordered {
    // ...
    @Override
    public int getOrder() {
        return Integer.MAX_VALUE;
    };
}
```

但是，我们可以使用`@Order`注释来代替`.`

## 5。结论

在本文中，我们看到了如何实现自定义的`TestExecutionListener`。我们还查看了 Spring 框架提供的默认监听器。

当然，本文附带的代码可以在 GitHub 上获得。******