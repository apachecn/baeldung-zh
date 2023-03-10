# 更改 Log4j2 配置文件在 Spring Boot 的默认位置

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-boot-change-log4j2-location>

## 1.概观

在我们之前关于 Spring Boot 的[日志的教程中，我们展示了如何在 Spring Boot 使用 Log4j2。](/web/20221208143835/https://www.baeldung.com/spring-boot-logging)

在这个简短的教程中，我们将学习如何**改变 Log4j2 配置文件**的默认位置。

## 2.使用属性文件

默认情况下，我们将 Log4j2 配置文件(`log4j2.xml/log4j2-spring.xml`)保留在项目类路径或 resources 文件夹中。

我们可以通过在我们的`application.properties`文件中添加/修改下面一行来改变这个文件的位置:

```java
logging.config=/path/to/log4j2.xml
```

## 3.使用虚拟机选项

在运行我们的程序时，我们还可以添加以下 VM 选项来实现相同的目标:

```java
-Dlogging.config=/path/to/log4j2.xml
```

## 4.程序配置

最后，我们可以通过改变我们的 Spring Boot `Application`类来编程配置这个文件的位置，如下所示:

```java
@SpringBootApplication
public class Application implements CommandLineRunner {

    public static void main(String[] args) {
        SpringApplication.run(Application.class, args);
    }

    @Override
    public void run(String... param) {
        Configurator.initialize(null, "/path/to/log4j2.xml");
    }
}
```

这种解决方案有一个缺点:应用程序引导过程不会使用 Log4j2 来记录。

## 5.结论

总之，我们已经学习了不同的方法来**改变 Log4j2 配置文件在 Spring Boot** 中的默认位置。希望这些东西对你的工作有帮助。