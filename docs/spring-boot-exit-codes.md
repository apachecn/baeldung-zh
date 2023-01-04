# Spring Boot 出口代码

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-boot-exit-codes>

## 1.概观

每个应用程序在退出时返回一个退出代码；该代码可以是任何整数值，包括负值。

在这个快速教程中，我们将了解如何从 Spring Boot 应用程序中返回退出代码。

## 2.Spring Boot 和退出代码

如果启动时出现异常，Spring Boot 应用程序将退出，代码为`1`。否则，在干净退出时，它提供`0`作为退出代码。

Spring 向 JVM 注册关闭挂钩，以确保`ApplicationContext` 在退出时优雅地关闭。除此之外，Spring 还提供了接口`org.springframework.boot.ExitCodeGenerator`。该接口可以在调用`System.exit()` 时返回具体代码。

## 3.实现退出代码

Spring Boot 提供了四种方法，允许我们使用退出代码。

`ExitCodeGenerator`接口和`ExitCodeExceptionMapper`允许我们指定自定义退出代码，而`ExitCodeEvent`允许我们在退出时读取退出代码。此外，异常甚至有可能实现`ExitCodeGenerator`接口。

### 3.1.`ExitCodeGenerator`

让我们创建一个实现`ExitCodeGenerator` 接口`.` 的类。我们必须实现返回整数值的方法`getExitCode()`:

```java
@SpringBootApplication
public class ExitCodeGeneratorDemoApplication implements ExitCodeGenerator {

    public static void main(String[] args) {
        System.exit(SpringApplication
          .exit(SpringApplication.run(DemoApplication.class, args)));
    }

    @Override
    public int getExitCode() {
        return 42;
    }
} 
```

这里，`ExitCodeGeneratorDemoApplication`类实现了`ExitCodeGenerator`接口。同样，**我们用** `**SpringApplication.exit()**.` 包装了对`SpringApplication.run()` 的呼叫

退出时，退出代码现在将是 42。

### 3.2.`ExitCodeExceptionMapper`

现在让我们看看如何让**基于运行时异常**返回一个退出代码。为此，我们实现了一个总是抛出一个`NumberFormatException`的`CommandLineRunner` ，然后注册了一个`ExitCodeExceptionMapper`类型的 bean:

```java
@Bean
CommandLineRunner createException() {
    return args -> Integer.parseInt("test") ;
}

@Bean
ExitCodeExceptionMapper exitCodeToexceptionMapper() {
    return exception -> {
        // set exit code based on the exception type
        if (exception.getCause() instanceof NumberFormatException) {
            return 80;
        }
        return 1;
    };
} 
```

在`ExitCodeExceptionMapper,` 中，我们简单地将异常映射到某个退出代码。

### 3.3.`ExitCodeEvent`

接下来，我们将捕获一个`ExitCodeEvent`来读取我们的应用程序`.` 的退出代码。为此，我们只需**注册一个订阅** **`ExitCodeEvent` s** (在本例中命名为`DemoListener` )的事件监听器:

```java
@Bean
DemoListener demoListenerBean() {
    return new DemoListener();
}

private static class DemoListener {
    @EventListener
    public void exitEvent(ExitCodeEvent event) {
        System.out.println("Exit code: " + event.getExitCode());
    }
}
```

现在，当应用程序退出时，方法`exitEvent()`将被调用，我们可以从事件中读取退出代码。

### 3.4.退出代码异常

一个异常也可以实现`ExitCodeGenerator`接口。当抛出这样的异常时，Spring Boot 返回由实现的`getExitCode()`方法提供的退出代码。例如:

```java
public class FailedToStartException extends RuntimeException implements ExitCodeGenerator {

    @Override
    public int getExitCode() {
        return 42;
    }
} 
```

如果我们抛出一个`FailedToStartException`的实例，Spring Boot 会将这个异常检测为一个`ExitCodeGenerator`，并报告 42 为退出代码。

## 4.结论

在本文中，我们已经讨论了 Spring Boot 提供的处理退出代码的多个选项。

对于任何应用程序来说，在退出时返回正确的错误代码是非常重要的。退出代码决定了退出发生时应用程序的状态。除此之外，它还有助于故障排除。

代码样本可以在 GitHub 上找到[。](https://web.archive.org/web/20220627083949/https://github.com/eugenp/tutorials/tree/master/spring-boot-modules/spring-boot-basic-customization-2)