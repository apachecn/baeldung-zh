# 阻止 ApplicationRunner 或 CommandLineRunner Beans 在 Junit 测试期间执行

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-junit-prevent-runner-beans-testing-execution>

## 1.概观

在本教程中，我们将展示如何在 Spring Boot 集成测试期间阻止类型为`[ApplicationRunner](https://web.archive.org/web/20221208143832/https://docs.spring.io/spring-boot/docs/current/api/org/springframework/boot/ApplicationRunner.html)` 或`[CommandLineRunner](https://web.archive.org/web/20221208143832/https://docs.spring.io/spring-boot/docs/current/api/org/springframework/boot/CommandLineRunner.html)`的 beans 运行。

## 2.示例应用程序

我们的示例应用程序由命令行运行程序、应用程序运行程序和任务服务 bean 组成。

命令行运行程序调用任务服务的`execute`方法，以便在应用程序启动时执行任务:

```java
@Component
public class CommandLineTaskExecutor implements CommandLineRunner {
    private TaskService taskService;

    public CommandLineTaskExecutor(TaskService taskService) {
        this.taskService = taskService;
    }

    @Override
    public void run(String... args) throws Exception {
        taskService.execute("command line runner task");
    }
} 
```

同样，应用程序运行器与任务服务交互以执行另一个任务:

```java
@Component
public class ApplicationRunnerTaskExecutor implements ApplicationRunner {
    private TaskService taskService;

    public ApplicationRunnerTaskExecutor(TaskService taskService) {
        this.taskService = taskService;
    }

    @Override
    public void run(ApplicationArguments args) throws Exception {
        taskService.execute("application runner task");
    }
} 
```

最后，任务服务负责执行其客户端的任务:

```java
@Service
public class TaskService {
    private static Logger logger = LoggerFactory.getLogger(TaskService.class);

    public void execute(String task) {
        logger.info("do " + task);
    }
} 
```

此外，我们还提供了一个 Spring Boot 应用程序类来实现这一切:

```java
@SpringBootApplication
public class ApplicationCommandLineRunnerApp {
    public static void main(String[] args) {
        SpringApplication.run(ApplicationCommandLineRunnerApp.class, args);
    }
}
```

## 3.测试预期行为

**`ApplicationRunnerTaskExecutor`和`CommandLineTaskExecutor`在 Spring Boot 加载应用上下文后运行。**

我们可以通过一个简单的测试来验证这一点:

```java
@SpringBootTest
class RunApplicationIntegrationTest {
    @SpyBean
    ApplicationRunnerTaskExecutor applicationRunnerTaskExecutor;
    @SpyBean
    CommandLineTaskExecutor commandLineTaskExecutor;

    @Test
    void whenContextLoads_thenRunnersRun() throws Exception {
        verify(applicationRunnerTaskExecutor, times(1)).run(any());
        verify(commandLineTaskExecutor, times(1)).run(any());
    }
}
```

正如我们所看到的，我们正在使用`[SpyBean](https://web.archive.org/web/20221208143832/https://docs.spring.io/spring-boot/docs/current/api/org/springframework/boot/test/mock/mockito/SpyBean.html)`注释将 [Mockito 间谍](/web/20221208143832/https://www.baeldung.com/mockito-spy)应用到`ApplicationRunnerTaskExecutor`和`CommandLineTaskExecutor`bean。通过这样做，我们可以验证每个 beans 的`run`方法都被调用了一次。

在接下来的部分中，我们将看到在 Spring Boot 集成测试中防止这种默认行为的各种方法和技术。

## 4.通过弹簧型材进行预防

我们可以防止这两者运行的一种方法是用 `@Profile`对它们进行注释:

```java
@Profile("!test")
@Component
public class CommandLineTaskExecutor implements CommandLineRunner {
    // same as before
}
```

```java
@Profile("!test")
@Component
public class ApplicationRunnerTaskExecutor implements ApplicationRunner {
    // same as before
}
```

完成上述更改后，我们继续进行集成测试:

```java
@ActiveProfiles("test")
@SpringBootTest
class RunApplicationWithTestProfileIntegrationTest {
    @Autowired
    private ApplicationContext context;

    @Test
    void whenContextLoads_thenRunnersAreNotLoaded() {
        assertNotNull(context.getBean(TaskService.class));
        assertThrows(NoSuchBeanDefinitionException.class, 
          () -> context.getBean(CommandLineTaskExecutor.class), 
          "CommandLineRunner should not be loaded during this integration test");
        assertThrows(NoSuchBeanDefinitionException.class, 
          () -> context.getBean(ApplicationRunnerTaskExecutor.class), 
          "ApplicationRunner should not be loaded during this integration test");
    }
}
```

正如我们所见，**我们用`@ActiveProfiles(“test”) `注释对上面的测试类进行了注释，这意味着它不会连接那些用`@Profile(“!test”)`注释的测试类。**结果，`CommandLineTaskExecutor` bean 和`ApplicationRunnerTaskExecutor` bean 都没有被加载。

## 5.通过`ConditionalOnProperty`注释预防

或者，我们可以按属性配置它们的接线，然后使用 [`ConditionalOnProperty`](/web/20221208143832/https://www.baeldung.com/spring-boot-annotations#condition-property) 标注:

```java
@ConditionalOnProperty(
  prefix = "application.runner", 
  value = "enabled", 
  havingValue = "true", 
  matchIfMissing = true)
@Component
public class ApplicationRunnerTaskExecutor implements ApplicationRunner {
    // same as before
} 
```

```java
@ConditionalOnProperty(
  prefix = "command.line.runner", 
  value = "enabled", 
  havingValue = "true", 
  matchIfMissing = true)
@Component
public class CommandLineTaskExecutor implements CommandLineRunner {
    // same as before
}
```

正如我们所看到的，**`ApplicationRunnerTaskExecutor`和`CommandLineTaskExecutor`在默认情况下是启用的，**，我们可以禁用它们，如果我们将以下属性设置为`false`:

*   `command.line.runner.enabled`
*   `application.runner.enabled`

因此，在我们的测试中，**我们将这些属性设置为`false`，并且`ApplicationRunnerTaskExecutor`和`CommandLineTaskExecutor`bean 都没有加载到应用程序上下文**:

```java
@SpringBootTest(properties = { 
  "command.line.runner.enabled=false", 
  "application.runner.enabled=false" })
class RunApplicationWithTestPropertiesIntegrationTest {
    // same as before
}
```

现在，尽管上面的技术帮助我们实现了我们的目标，但是有些情况下我们想要测试所有的 Spring beans 是否都被正确地加载和连接。

例如，我们可能想要测试`TaskService ` bean 是否被正确地注入到`CommandLineTaskExecutor,` 中，但是我们仍然不希望它的`run`方法在我们的测试中被执行。那么，让我们看看解释我们如何实现这一目标的最后一部分。

## 6.通过不引导整个容器进行预防

这里，我们将描述如何通过不引导整个应用程序容器来阻止`CommandLineTaskExecutor` 和`ApplicationRunnerTaskExecutor` bean 的执行。

在前面的章节中，我们使用了` [@SpringBootTest](/web/20221208143832/https://www.baeldung.com/spring-boot-testing#integration-testing-with-springboottest)`注释，这导致整个容器在我们的集成测试中被引导。`@SpringBootTest` [包括两个与最后一个解决方案相关的元注释](https://web.archive.org/web/20221208143832/https://docs.spring.io/spring-boot/docs/current/api/org/springframework/boot/test/context/SpringBootTest.html):

```java
@BootstrapWith(SpringBootTestContextBootstrapper.class)
@ExtendWith(SpringExtension.class) 
```

如果在我们的测试中不需要引导整个容器，那么就不要使用`@BootstrapWith`。

相反，**我们可以用`@ContextConfiguration`** 来代替:

```java
@ContextConfiguration(classes = {ApplicationCommandLineRunnerApp.class},
  initializers = ConfigDataApplicationContextInitializer.class)
```

通过`@ContextConfiguration,` ,我们决定如何为集成测试加载和配置应用程序上下文。通过设置`ContextConfiguration` `classes`属性，我们声明 Spring Boot 应该使用`ApplicationCommandLineRunnerApp`类来加载应用程序上下文。通过将初始化器定义为`ConfigDataApplicationContextInitializer`，应用程序加载其属性`.`

我们仍然需要 **`@ExtendWith(SpringExtension.class)`，因为它将 Spring TestContext 框架集成到了 JUnit 5 的 Jupiter 编程模型中。**

作为上述操作的结果，**Spring Boot 应用程序上下文加载应用程序的组件和属性，而不执行`CommandLineTaskExecutor`或`ApplicationRunnerTaskExecutor`bean:**

```java
@ExtendWith(SpringExtension.class)
@ContextConfiguration(classes = { ApplicationCommandLineRunnerApp.class }, 
  initializers = ConfigDataApplicationContextInitializer.class)
public class LoadSpringContextIntegrationTest {
    @SpyBean
    TaskService taskService;

    @SpyBean
    CommandLineRunner commandLineRunner;

    @SpyBean
    ApplicationRunner applicationRunner;

    @Test
    void whenContextLoads_thenRunnersDoNotRun() throws Exception {
        assertNotNull(taskService);
        assertNotNull(commandLineRunner);
        assertNotNull(applicationRunner);

        verify(taskService, times(0)).execute(any());
        verify(commandLineRunner, times(0)).run(any());
        verify(applicationRunner, times(0)).run(any());
    }
} 
```

此外，我们必须记住，**和`ConfigDataApplicationContextInitializer`单独使用时，不支持`@Value(“${…​}”)`注射。**如果我们想要支持它，我们必须配置一个`[PropertySourcesPlaceholderConfigurer](https://web.archive.org/web/20221208143832/https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/context/support/PropertySourcesPlaceholderConfigurer.html)`。

## 7。结论

在本文中，我们展示了在 Spring Boot 集成测试期间防止执行`ApplicationRunner`和`CommandLineRunner`bean 的各种方法。

和往常一样，代码可以在 GitHub 的[上获得。](https://web.archive.org/web/20221208143832/https://github.com/eugenp/tutorials/tree/master/spring-boot-modules/spring-boot-testing)