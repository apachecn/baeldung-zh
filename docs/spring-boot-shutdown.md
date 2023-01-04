# 关闭 Spring Boot 应用程序

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-boot-shutdown>

## 1。概述

管理 Spring Boot 应用程序的生命周期对于生产就绪的系统非常重要。Spring 容器在`ApplicationContext.`的帮助下处理所有 Beans 的创建、初始化和销毁

这篇文章的重点是生命周期的销毁阶段。更具体地说，我们将看看关闭 Spring Boot 应用程序的不同方法。

要了解更多关于如何使用 Spring Boot 建立一个项目，请查看 [Spring Boot 启动器](/web/20220905213101/https://www.baeldung.com/spring-boot-starters)文章，或者浏览 [Spring Boot 配置](/web/20220905213101/https://www.baeldung.com/spring-boot-application-configuration)。

## 2。关机终点

默认情况下，除了`/shutdown`之外，Spring Boot 应用程序中的所有端点都是启用的；这自然是`Actuator`端点的一部分。

下面是设置这些的 Maven 依赖项:

```java
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-actuator</artifactId>
</dependency>
```

如果我们还想建立安全支持，我们需要:

```java
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-security</artifactId>
</dependency>
```

最后，我们在`application.properties`文件中启用关闭端点:

```java
management.endpoints.web.exposure.include=*
management.endpoint.shutdown.enabled=true
endpoints.shutdown.enabled=true
```

注意，我们还必须公开我们想要使用的任何执行器端点。在上面的例子中，我们已经暴露了所有的执行器端点，包括`/shutdown`端点。

**要关闭 Spring Boot 应用程序，我们只需像这样调用 POST 方法**:

```java
curl -X POST localhost:port/actuator/shutdown
```

在这个调用中，`port`代表执行器端口。

## 3.关闭应用程序上下文

我们也可以使用应用程序上下文直接调用`close()`方法。

让我们从创建上下文并关闭它的示例开始:

```java
ConfigurableApplicationContext ctx = new 
  SpringApplicationBuilder(Application.class).web(WebApplicationType.NONE).run();
System.out.println("Spring Boot application started");
ctx.getBean(TerminateBean.class);
ctx.close();
```

这销毁了所有的豆子，释放了锁，然后关闭了豆子工厂。为了验证应用程序的关闭，我们使用 Spring 的标准生命周期回调和`@PreDestroy`注释:

```java
public class TerminateBean {

    @PreDestroy
    public void onDestroy() throws Exception {
        System.out.println("Spring Container is destroyed!");
    }
}
```

我们还必须添加这种类型的 bean:

```java
@Configuration
public class ShutdownConfig {

    @Bean
    public TerminateBean getTerminateBean() {
        return new TerminateBean();
    }
}
```

以下是运行该示例后的输出:

```java
Spring Boot application started
Closing [[email protected]](/web/20220905213101/https://www.baeldung.com/cdn-cgi/l/email-protection)
DefaultLifecycleProcessor - Stopping beans in phase 0
Unregistering JMX-exposed beans on shutdown
Spring Container is destroyed!
```

这里需要记住的重要一点是:**当关闭应用程序上下文时，父上下文不会因为独立的生命周期而受到影响**。

### 3.1.关闭当前应用程序上下文

在上面的例子中，我们创建了一个子应用程序上下文，然后使用`close()`方法销毁它。

如果我们想要关闭当前上下文，一个解决方案是简单地调用执行器`/shutdown`端点。

但是，我们也可以创建自己的自定义端点:

```java
@RestController
public class ShutdownController implements ApplicationContextAware {

    private ApplicationContext context;

    @PostMapping("/shutdownContext")
    public void shutdownContext() {
        ((ConfigurableApplicationContext) context).close();
    }

    @Override
    public void setApplicationContext(ApplicationContext ctx) throws BeansException {
        this.context = ctx;

    }
}
```

这里，我们添加了一个控制器，它实现了`ApplicationContextAware`接口并覆盖了 setter 方法来获取当前的应用程序上下文。然后，在一个映射方法中，我们简单地调用`close()`方法。

然后，我们可以调用新端点来关闭当前上下文:

```java
curl -X POST localhost:port/shutdownContext
```

**当然，如果您在实际应用中添加这样的端点，您也会希望保护它。**

## 4.退出`SpringApplication`

`SpringApplication`向 JVM 注册一个`shutdown`钩子，以确保应用程序正确退出。

Beans 可以实现`ExitCodeGenerator`接口来返回特定的错误代码:

```java
ConfigurableApplicationContext ctx = new SpringApplicationBuilder(Application.class)
  .web(WebApplicationType.NONE).run();

int exitCode = SpringApplication.exit(ctx, new ExitCodeGenerator() {
@Override
public int getExitCode() {
        // return the error code
        return 0;
    }
});

System.exit(exitCode);
```

Java 8 lambdas 应用程序的相同代码:

```java
SpringApplication.exit(ctx, () -> 0);
```

**调用`System.exit(exitCode)`后，程序以 0 返回码**终止:

```java
Process finished with exit code 0
```

## 5。杀死 App 进程

最后，我们还可以使用 bash 脚本从应用程序外部关闭 Spring Boot 应用程序。该选项的第一步是让应用程序上下文将其 PID 写入文件:

```java
SpringApplicationBuilder app = new SpringApplicationBuilder(Application.class)
  .web(WebApplicationType.NONE);
app.build().addListeners(new ApplicationPidFileWriter("./bin/shutdown.pid"));
app.run();
```

接下来，创建一个包含以下内容的`shutdown.bat`文件:

```java
kill $(cat ./bin/shutdown.pid)
```

`shutdown.bat`的执行从`shutdown.pid`文件中提取进程 ID，并使用`kill`命令终止引导应用程序。

## 6。结论

在这篇快速的文章中，我们介绍了一些可以用来关闭正在运行的 Spring Boot 应用程序的简单方法。

而选择一个合适的方法则取决于开发人员；所有这些方法都应该有目的地使用。

例如，当我们需要将一个错误代码传递给另一个环境，比如 JVM 以进行进一步的操作时，`.exit()`是首选。使用 **`Application` `PID`提供了更多的灵活性，因为我们还可以使用 bash 脚本启动或重启应用程序**。

最后， **`/shutdown`** 在这里使**能够通过 HTTP** 从外部终止应用程序。对于所有其他情况，`.close()`将完美地工作。

像往常一样，这篇文章的完整代码可以在 GitHub 项目上找到。