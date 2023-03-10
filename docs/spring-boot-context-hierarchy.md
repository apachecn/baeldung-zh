# 使用 Spring Boot Fluent Builder API 的上下文层次结构

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-boot-context-hierarchy>

## 1。概述

在 Spring Boot 中，可以创建单独的上下文并按层次组织它们。

在 Spring Boot 应用程序中，可以用不同的方式定义上下文层次结构。在本文中，我们将了解如何使用 fluent builder API 创建多个上下文。

由于我们不会详细讨论如何建立一个 Spring Boot 应用程序，你可能想看看这篇文章。

## 2。应用上下文层次结构

我们可以拥有多个共享父子关系的应用程序上下文。

上下文层次结构允许多个子上下文共享驻留在父上下文中的 beans。每个子上下文都可以覆盖从父上下文继承的配置。

此外，我们可以使用上下文来防止在一个上下文中注册的 beans 在另一个上下文中被访问。这有助于创建松散耦合的模块。

这里值得注意的是，一个上下文只能有一个父上下文，而一个父上下文可以有多个子上下文。此外，子上下文可以访问父上下文中的 beans，但反之则不行。

## 3。使用`SpringApplicationBuilder` API

`SpringApplicationBuilder`类提供了一个流畅的 API，使用`parent()`、`child()` 和`sibling()`方法在上下文之间创建父子关系。

为了举例说明上下文层次结构，**我们将设置一个非 web 父应用程序上下文和两个子 web 上下文。**

为了演示这一点，我们将启动两个嵌入式 Tomcat 实例，每个实例都有自己的 web 应用程序上下文，并且都运行在单个 JVM 中。

### 3.1。父上下文

首先，让我们创建一个服务 bean 和一个位于父包中的 bean 定义类。我们希望这个 bean 返回一个问候语，显示给我们的 web 应用程序的客户端:

```java
@Service
public class HomeService implements IHomeService {

    public String getGreeting() {
        return "Welcome User";
    }
}
```

和 bean 定义类:

```java
@Configuration
@ComponentScan("com.baeldung.parent")
public class ServiceConfig {}
```

接下来，我们将为两个子上下文创建配置。

### 3.2。子上下文

因为所有的上下文都是使用默认配置文件配置的，所以我们需要为不能在上下文之间共享的属性(如服务器端口)提供单独的配置。

为了防止自动配置拾取冲突的配置，我们还将类保存在单独的包中。

让我们首先为第一个子上下文定义一个属性文件:

```java
server.port=8074
server.servlet.context-path=/ctx1

spring.application.admin.enabled=false
spring.application.admin.jmx-name=org.springframework.boot:type=Ctx1Rest,name=Ctx1Application
```

注意，我们已经配置了端口和上下文路径，以及一个 JMX 名称，这样应用程序名称就不会冲突。

现在让我们为这个上下文添加主配置类:

```java
@Configuration
@ComponentScan("com.baeldung.ctx1")
@EnableAutoConfiguration
public class Ctx1Config {

    @Bean
    public IHomeService homeService() {
        return new GreetingService();
    }
}
```

这个类为`homeService` bean 提供了一个新的定义，这个定义将覆盖父 bean 的定义。

让我们看看`GreetingService`类的定义:

```java
@Service
public class GreetingService implements IHomeService {

    public String getGreeting() {
        return "Greetings for the day";
    }
}
```

最后，我们将为这个 web 上下文添加一个控制器，它使用`homeService` bean 向用户显示一条消息:

```java
@RestController
public class Ctx1Controller {

    @Autowired
    private HomeService homeService;

    @GetMapping("/home")
    public String greeting() {
        return homeService.getGreeting();
    }
}
```

### 3.3.兄弟上下文

对于我们的第二个上下文，我们将创建一个控制器和配置类，它们与上一节中的非常相似。

这一次，我们不会创建一个`homeService`bean——因为我们将从父上下文中访问它。

首先，让我们为这个上下文添加一个属性文件:

```java
server.port=8075
server.servlet.context-path=/ctx2

spring.application.admin.enabled=false
spring.application.admin.jmx-name=org.springframework.boot:type=WebAdmin,name=SpringWebApplication
```

以及同级应用程序的配置类:

```java
@Configuration
@ComponentScan("com.baeldung.ctx2")
@EnableAutoConfiguration
@PropertySource("classpath:ctx2.properties")
public class Ctx2Config {}
```

让我们也添加一个控制器，它有`HomeService` 作为依赖项:

```java
@RestController
public class Ctx2Controller {

    @Autowired
    private IHomeService homeService;

    @GetMapping("/greeting")
    public String getGreeting() {
        return homeService.getGreeting();
    }
}
```

在这种情况下，我们的控制器应该从父上下文中获取`homeService` bean。

### 3.4.上下文层次结构

现在我们可以把所有东西放在一起，用`SpringApplicationBuilder:`定义上下文层次结构

```java
public class App {
    public static void main(String[] args) {
        new SpringApplicationBuilder()
          .parent(ParentConfig.class).web(WebApplicationType.NONE)
          .child(WebConfig.class).web(WebApplicationType.SERVLET)
          .sibling(RestConfig.class).web(WebApplicationType.SERVLET)
          .run(args);
    }
}
```

最后，在运行 Spring Boot 应用程序时，我们可以使用`localhost:8074/ctx1/home` 和`localhost:8075/ctx2/greeting.`在各自的端口访问这两个应用程序

## 4.结论

使用`SpringApplicationBuilder` API，我们首先在应用程序的两个上下文之间创建了父子关系。接下来，我们讲述了如何在子上下文中覆盖父配置。最后，我们添加了一个兄弟上下文来演示父上下文中的配置如何与其他子上下文共享。

这个例子的源代码可以在 GitHub 的[上找到。](https://web.archive.org/web/20221127165817/https://github.com/eugenp/tutorials/tree/master/spring-boot-modules/spring-boot-ctx-fluent)