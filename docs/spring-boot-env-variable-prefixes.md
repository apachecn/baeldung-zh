# Spring Boot 2.5 中的环境变量前缀

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-boot-env-variable-prefixes>

## 1.概观

本教程将讨论在 **Spring Boot 2.5 中增加的一个特性，该特性使得为系统环境变量**指定前缀成为可能。这样，我们可以在同一个环境中运行多个不同的 Spring Boot 应用程序，因为所有的属性都需要一个带前缀的版本。

## 2.环境变量前缀

我们可能需要在同一个环境中运行多个 [Spring Boot](/web/20220707143855/https://www.baeldung.com/spring-boot-start) 应用程序，并且**经常面临环境变量名分配给不同属性**的问题。

我们可以使用 Spring Boot [属性](/web/20220707143855/https://www.baeldung.com/properties-with-spring) ，这在某种程度上是相似的，但我们也可能希望在应用程序级别设置一个前缀，以利用环境端。

例如，让我们设置一个简单的 Spring Boot 应用程序，**通过设置这个前缀**来修改应用程序属性，例如 tomcat 服务器端口。

### 2.1.我们的 Spring Boot 应用程序

让我们创建一个 Spring Boot 应用程序来演示这个特性。**首先，让我们给应用程序添加一个前缀**。我们称之为“ `prefix”` 简单点说:

```java
@SpringBootApplication
public class PrefixApplication {

    public static void main(String[] args) {
        SpringApplication application = new SpringApplication(PrefixApplication.class);
        application.setEnvironmentPrefix("prefix");
        application.run(args);
    }
}
```

我们不能使用已经包含下划线字符(_)的单词作为前缀。否则，应用程序将抛出一个错误。

我们还想创建一个端点来检查我们的应用程序正在监听哪个端口:

```java
@Controller
public class PrefixController {

    @Autowired
    private Environment environment;

    @GetMapping("/prefix")
    public String getServerPortInfo(final Model model) {
        model.addAttribute("serverPort", environment.getProperty("server.port"));
        return "prefix";
    }
}
```

在这种情况下，我们使用[百里香叶](/web/20220707143855/https://www.baeldung.com/thymeleaf-in-spring-mvc)来解析我们的模板，同时设置我们的服务器端口，有一个简单的主体，如下所示:

```java
<html>
    // ...
<body>
It is working as we expected. Your server is running at port : <b th:text="${serverPort}"></b>
</body>
</html>
```

### 2.2.设置环境变量

**我们现在可以将环境变量`prefix_server_port`** 设置为 8085。我们可以看到如何设置系统环境变量，例如，在 [Linux](/web/20220707143855/https://www.baeldung.com/linux/environment-variables) 中。

一旦我们设置了环境变量，我们希望应用程序基于该前缀创建属性。

在从 IDE 运行的情况下，我们需要编辑启动配置并添加环境变量，或者从已经加载的环境变量中选择它。

### 2.3.运行应用程序

我们现在可以从命令行或用我们最喜欢的 IDE 启动应用程序。

如果我们用浏览器加载 URL `http://localhost:8085/prefix`，我们可以看到服务器正在运行，并且在端口处，我们之前添加了前缀:

```java
It is working as we expected. Your server is running at port : 8085
```

如果没有前缀，应用程序将开始使用默认的环境变量。

## 3.结论

在本教程中，我们已经看到了如何通过 Spring Boot 为环境变量使用前缀。例如，如果我们希望在同一个环境中运行多个 Spring Boot 应用程序，并为同名的属性(如服务器端口)分配不同的值，这会很有帮助。

和往常一样，本文中的代码可以从 GitHub 上的[处获得。](https://web.archive.org/web/20220707143855/https://github.com/eugenp/tutorials/tree/master/spring-boot-modules/spring-boot-environment)