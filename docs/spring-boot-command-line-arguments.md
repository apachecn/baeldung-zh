# Spring Boot 的命令行参数

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-boot-command-line-arguments>

## 1。概述

在这个快速教程中，我们将讨论如何向 Spring Boot 应用程序传递命令行参数。

我们可以使用命令行参数来配置我们的应用程序，覆盖应用程序属性或传递自定义参数。

## 2。Maven 命令行参数

首先，让我们看看如何在使用 Maven 插件运行应用程序时传递参数。

稍后，我们将看到如何访问代码中的参数。

### 2.1。Spring Boot 1.x

对于 Spring Boot 1.x，我们可以使用`-Drun.arguments`将参数传递给我们的应用程序:

```java
mvn spring-boot:run -Drun.arguments=--customArgument=custom
```

我们还可以向我们的应用程序传递多个参数:

```java
mvn spring-boot:run -Drun.arguments=--spring.main.banner-mode=off,--customArgument=custom
```

请注意:

*   参数应该用逗号分隔
*   每个参数都应该带有前缀—
*   我们也可以传递配置属性，就像上面例子中显示的`spring.main.banner-mode`

### 2.2。Spring Boot 2.x

对于 Spring Boot 2.x，我们可以使用 `-Dspring-boot.run.arguments` : 来传递参数

```java
mvn spring-boot:run -Dspring-boot.run.arguments=--spring.main.banner-mode=off,--customArgument=custom
```

## 3。Gradle 命令行参数

接下来，让我们看看如何在使用 Gradle 插件运行我们的应用程序时传递参数。

我们需要在`build.gradle`文件中配置我们的`bootRun`任务:

```java
bootRun {
    if (project.hasProperty('args')) {
        args project.args.split(',')
    }
}
```

现在，我们可以如下传递命令行参数:

```java
./gradlew bootRun -Pargs=--spring.main.banner-mode=off,--customArgument=custom
```

## 4。覆盖系统属性

除了传递自定义参数，我们还可以覆盖系统属性。

例如，下面是我们的`application.properties`文件:

```java
server.port=8081
spring.application.name=SampleApp
```

为了覆盖`server.port`值，我们需要以如下方式传递新值(对于 Spring Boot 1.x):

```java
mvn spring-boot:run -Drun.arguments=--server.port=8085
```

Spring Boot 2.x 也是如此:

```java
mvn spring-boot:run -Dspring-boot.run.arguments=--server.port=8085
```

请注意:

*   Spring Boot 将命令行参数转换为属性，并将它们作为环境变量添加
*   通过在`application.properties` :

    ```java
    server.port=${port:8080}
    ```

    中使用占位符，我们可以使用简短的命令行参数`–port=8085`来代替`–server.port=8085`
*   命令行参数优先于`application.properties`值

如果需要，我们可以阻止应用程序将命令行参数转换为属性:

```java
@SpringBootApplication
public class Application extends SpringBootServletInitializer {
    public static void main(String[] args) {
        SpringApplication application = new SpringApplication(Application.class);
        application.setAddCommandLineProperties(false);
        application.run(args);
    }
}
```

## 5。访问命令行参数

让我们看看如何从应用程序的`main()`方法中访问命令行参数:

```java
@SpringBootApplication
public class Application extends SpringBootServletInitializer {
    public static void main(String[] args) {
        for(String arg:args) {
            System.out.println(arg);
        }
        SpringApplication.run(Application.class, args);
    }
}
```

这将打印我们从命令行传递给应用程序的参数，但我们也可以在应用程序中稍后使用它们。

## 6.将**命令行参数传递给`SpringBootTest`**

随着 Spring Boot 2.2 的发布，我们获得了在测试期间使用`@SpringBootTest`及其`args`属性注入命令行参数的可能性:

```java
@SpringBootTest(args = "--spring.main.banner-mode=off")
public class ApplicationTest {

    @Test
    public void whenUsingSpringBootTestArgs_thenCommandLineArgSet(@Autowired Environment env) {
        Assertions.assertThat(env.getProperty("spring.main.banner-mode")).isEqualTo("off");
    }
}
```

## 7。结论

在本文中，我们学习了如何从命令行向我们的 Spring Boot 应用程序传递参数，以及如何使用 Maven 和 Gradle 来完成。

我们还展示了如何从代码中访问这些参数，以便配置应用程序。