# Spring Boot 错误 ApplicationContextException

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-boot-application-context-exception>

## 1.概观

在这个快速教程中，我们将仔细看看 [Spring Boot](/web/20220628095139/https://www.baeldung.com/spring-boot) 错误“`ApplicationContextException: Unable to start ServletWebServerApplicationContext due to missing ServletWebServerFactory bean`”。

首先，我们将阐明这个错误背后的主要原因。然后，我们将深入研究如何使用一个实际的例子来重现它，以及最后如何解决它。

## 2.可能的原因

首先，让我们试着理解错误信息的含义。`Unable to start ServletWebServerApplicationContext due to missing ServletWebServerFactory bean`“说明了一切。它简单地告诉我们，在 [`ApplicationContext`](/web/20220628095139/https://www.baeldung.com/spring-application-context) `.`中没有配置 [`ServletWebServerFactory`](https://web.archive.org/web/20220628095139/https://docs.spring.io/spring-boot/docs/current/api/org/springframework/boot/web/servlet/server/ServletWebServerFactory.html) bean

错误主要出现在 Spring Boot 无法启动 [`ServletWebServerApplicationContext`](https://web.archive.org/web/20220628095139/https://docs.spring.io/spring-boot/docs/current/api/org/springframework/boot/web/servlet/context/ServletWebServerApplicationContext.html) 的时候。为什么？**因为`ServletWebServerApplicationContext`使用包含的`ServletWebServerFactory` bean 来引导自己。**

一般来说，Spring Boot 提供了 [`SpringApplication.run`](https://web.archive.org/web/20220628095139/https://docs.spring.io/spring-boot/docs/1.0.0.RC5/reference/html/boot-features-spring-application.html) 的方法来引导 Spring 应用程序。

`SpringApplication` 类将尝试为我们创建正确的`ApplicationContext`，**，这取决于我们是否在开发 web 应用程序**。

例如，用于确定一个 web 应用程序是否来自一些依赖关系(如`spring-boot-starter-web.`)的算法，也就是说，缺乏这些依赖关系可能是我们错误背后的原因之一。

另一个原因是在 Spring Boot 入口点类中缺少 [`@SpringBootApplication`](/web/20220628095139/https://www.baeldung.com/spring-boot-start#application-configuration) 注释。

## 3.重现错误

现在，让我们看一个会产生 Spring Boot 误差的例子。最简单的方法是**创建一个没有`@SpringBootApplication` 注释**的主类。

首先，让我们创建一个入口点类，并故意忘记用`@SpringBootApplication`对其进行注释:

```java
public class MainEntryPoint {

    public static void main(String[] args) {
        SpringApplication.run(MainEntryPoint.class, args);
    }
}
```

现在，让我们运行我们的示例 Spring Boot 应用程序，看看会发生什么:

```java
22:20:39.134 [main] ERROR o.s.boot.SpringApplication - Application run failed
org.springframework.context.ApplicationContextException: Unable to start web server; nested exception is org.springframework.context.ApplicationContextException: Unable to start ServletWebServerApplicationContext due to missing ServletWebServerFactory bean.
	...
	at com.baeldung.applicationcontextexception.MainEntryPoint.main(MainEntryPoint.java:10)
<strong>Caused by: org.springframework.context.ApplicationContextException: Unable to start ServletWebServerApplicationContext due to missing ServletWebServerFactory bean.</strong>
	at org.springframework.boot.web.servlet.context.ServletWebServerApplicationContext.getWebServerFactory(ServletWebServerApplicationContext.java:209)
	at org.springframework.boot.web.servlet.context.ServletWebServerApplicationContext.createWebServer(ServletWebServerApplicationContext.java:179)
	... 
```

如上所示，我们得到“`ApplicationContextException: Unable to start ServletWebServerApplicationContext due to missing ServletWebServerFactory bean`”错误。

## 4.修复错误

修复错误的简单方法是用`@SpringBootApplication` 注释`.`来注释我们的`MainEntryPoint`类

通过使用这个注释，**我们告诉 Spring Boot 自动配置必要的 beans，并在上下文**中注册它们。

类似地，我们可以通过禁用 web 环境来避免非 web 应用程序的错误。为此，我们可以使用`spring.main.web-application-type` 属性。

在`application.properties`中:

```java
spring.main.web-application-type=none
```

同样，在我们的`application.yml`:

```java
spring: 
    main: 
        web-application-type: none
```

**`none`表示应用程序不应该作为 web 应用程序运行。它被用来禁用网络服务器**。

请记住，从 [Spring Boot 2.0](https://web.archive.org/web/20220628095139/https://spring.io/blog/2018/03/01/spring-boot-2-0-goes-ga) 开始，我们还可以使用`[SpringApplicationBuilder](https://web.archive.org/web/20220628095139/https://docs.spring.io/spring-boot/docs/2.0.x/api/org/springframework/boot/builder/SpringApplicationBuilder.html#web-org.springframework.boot.WebApplicationType-)`来明确定义特定类型的 web 应用程序:

```java
@SpringBootApplication
public class MainClass {

    public static void main(String[] args) {
        new SpringApplicationBuilder(MainClass.class)
          .web(WebApplicationType.NONE)
          .run(args);
    }
}
```

**对于一个 [WebFlux 项目，](/web/20220628095139/https://www.baeldung.com/spring-webflux)我们可以使用** `**WebApplicationType.REACTIVE**.` 另一个解决方案可以是排除`spring-webmvc` 依赖。

类路径中这种依赖性的存在告诉 Spring Boot 将该项目视为一个 servlet 应用程序，而不是一个被动的 web 应用程序。结果`,` Spring Boot 没能启动`ServletWebServerApplicationContext`。

## 5.结论

在这篇短文中，我们详细讨论了是什么原因导致 Spring Boot 在启动时失败，并出现以下错误:“`ApplicationContextException: Unable to start ServletWebServerApplicationContext due to missing ServletWebServerFactory bean`”。

在这个过程中，我们通过一个实例解释了如何产生错误以及如何修复它。

与往常一样，GitHub 上的[提供了示例的完整源代码。](https://web.archive.org/web/20220628095139/https://github.com/eugenp/tutorials/tree/master/spring-boot-modules/spring-boot-exceptions)