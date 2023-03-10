# 以编程方式重新启动 Spring Boot 应用程序

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-restart-spring-boot-app>

## 1。概述

在本教程中，我们将展示如何以编程方式****重启一个 Spring Boot 应用程序**。**

 **在某些情况下，重新启动我们的应用程序非常方便:

*   更改某些参数后重新加载配置文件
*   在运行时更改当前活动的配置文件
*   出于任何原因重新初始化应用程序上下文

虽然这篇文章介绍了重启 Spring Boot 应用程序的功能，但是请注意，我们还有一个关于关闭 Spring Boot 应用程序的很好的教程。

现在，让我们探索实现 Spring Boot 应用程序重启的不同方法。

## 2。通过创建新的上下文重新启动

我们可以通过关闭应用程序上下文并从头创建一个新的上下文来重新启动我们的应用程序。尽管这种方法非常简单，但要使它起作用，我们必须小心一些微妙的细节。

让我们看看如何在我们的 Spring Boot 应用程序的`main`方法中实现这一点:

```java
@SpringBootApplication
public class Application {

    private static ConfigurableApplicationContext context;

    public static void main(String[] args) {
        context = SpringApplication.run(Application.class, args);
    }

    public static void restart() {
        ApplicationArguments args = context.getBean(ApplicationArguments.class);

        Thread thread = new Thread(() -> {
            context.close();
            context = SpringApplication.run(Application.class, args.getSourceArgs());
        });

        thread.setDaemon(false);
        thread.start();
    }
}
```

正如我们在上面的例子中看到的，在一个单独的非守护线程中重新创建上下文是很重要的——这样我们可以防止由`close`方法触发的 JVM 关闭关闭我们的应用程序。否则，我们的应用程序将会停止，因为 JVM 不会在终止守护线程之前等待它们完成。

此外，让我们添加一个 REST 端点，通过它我们可以触发重启:

```java
@RestController
public class RestartController {

    @PostMapping("/restart")
    public void restart() {
        Application.restart();
    } 
}
```

这里，我们添加了一个带有映射方法的控制器，该方法调用我们的`restart`方法。

然后，我们可以调用新的端点来重启应用程序:

```java
curl -X POST localhost:port/restart
```

**当然，如果我们在实际应用中添加这样的端点，我们也必须保护它。**

## 3。执行器的重启端点

重启应用程序的另一种方法是使用`[Spring Boot Actuator](/web/20220626081405/https://www.baeldung.com/spring-boot-actuators).`中的内置`RestartEndpoint`

首先，让我们添加所需的 Maven 依赖项:

```java
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-actuator</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-cloud-starter</artifactId>
</dependency>
```

接下来，我们必须在我们的`application.properties`文件中启用内置重启端点:

```java
management.endpoint.restart.enabled=true
```

现在我们已经设置好了一切，我们可以将`Restart` `Endpoint `注入到我们的服务中:

```java
@Service
public class RestartService {

    @Autowired
    private RestartEndpoint restartEndpoint;

    public void restartApp() {
        restartEndpoint.restart();
    }
}
```

在上面的代码中，我们使用了`RestartEndpoint ` bean 来重启我们的应用程序。这是一种很好的重启方式，因为我们只需要调用一个方法就可以完成所有的工作。

正如我们所见，使用`RestartEndpoint`是重启应用程序的一种简单方式。另一方面，这种方法有一个缺点，因为它要求我们添加上面提到的库。如果我们还没有使用它们，那么对于这个功能来说，开销可能太大了。在这种情况下，我们可以坚持使用上一节中的手动方法，因为它只需要多几行代码。

## 4。刷新应用程序上下文

在某些情况下，我们可以通过调用其 [refresh](https://web.archive.org/web/20220626081405/https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/context/ConfigurableApplicationContext.html#refresh--) 方法来重新加载应用程序上下文。

尽管这种方法听起来很有前途，但是只有一些应用程序上下文类型支持刷新已经初始化的上下文。比如`FileSystemXmlApplicationContext`、`GroovyWebApplicationContext,` 等少数支持。

不幸的是，如果我们在 Spring Boot 的 web 应用程序中尝试这样做，我们将得到以下错误:

```java
java.lang.IllegalStateException: GenericApplicationContext does not support multiple refresh attempts:
just call 'refresh' once
```

最后，尽管有些上下文类型支持多次刷新，但我们应该避免这样做。原因是`refresh`方法被设计成框架用来初始化应用程序上下文的内部方法。

## 5。结论

在本文中，我们探索了多种不同的方式来以编程方式重启 Spring Boot 应用程序。

和往常一样，我们可以在 GitHub 上找到示例的源代码。**