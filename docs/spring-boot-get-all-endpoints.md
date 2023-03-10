# 获取 Spring Boot 的所有端点

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-boot-get-all-endpoints>

## 1.概观

当使用 REST API 时，通常会检索所有的 REST 端点。例如，我们可能需要在数据库中保存所有请求映射端点。在本教程中，我们将看看如何在一个 [Spring Boot](/web/20220628051659/https://www.baeldung.com/category/spring/spring-boot/) 应用程序中获得所有的 REST 端点。

## 2.映射端点

在一个 Spring Boot 应用程序中，我们通过在控制器类中使用`@RequestMapping`注释来公开一个 REST API 端点。要获得这些端点，有三个选项:事件监听器、Spring Boot 执行器或 Swagger 库。

## 3.事件监听器方法

为了创建 REST API 服务，我们在控制器类中使用了`[@RestController](/web/20220628051659/https://www.baeldung.com/building-a-restful-web-service-with-spring-and-java-based-configuration#controller)`和`@RequestMapping`。这些类在 spring 应用程序上下文中注册为 spring bean。因此，当应用程序上下文在启动时准备就绪时，我们可以通过使用事件侦听器来获取端点。定义侦听器有两种方法。我们既可以实现 [`ApplicationListener`](/web/20220628051659/https://www.baeldung.com/spring-events#listener) 接口，也可以使用 [@ `EventListener`](/web/20220628051659/https://www.baeldung.com/spring-events#annotation-driven) 注释。

### 3.1.`ApplicationListener`界面

当实现`ApplicationListener`时，我们必须定义`onApplicationEvent()`方法:

```java
@Override
public void onApplicationEvent(ContextRefreshedEvent event) {
    ApplicationContext applicationContext = event.getApplicationContext();
    RequestMappingHandlerMapping requestMappingHandlerMapping = applicationContext
        .getBean("requestMappingHandlerMapping", RequestMappingHandlerMapping.class);
    Map<RequestMappingInfo, HandlerMethod> map = requestMappingHandlerMapping
        .getHandlerMethods();
    map.forEach((key, value) -> LOGGER.info("{} {}", key, value));
}
```

这样，我们使用了 [`ContextRefreshedEvent`](/web/20220628051659/https://www.baeldung.com/spring-context-events#1-contextrefreshedevent) 类。当`ApplicationContext`被初始化或刷新时，该事件被发布。Spring Boot 提供了许多`HandlerMapping`实现。其中一个是`RequestMappingHandlerMapping`类，它检测请求映射并由`@RequestMapping`注释使用。因此，我们在`ContextRefreshedEvent`事件中使用这个 bean。

### 3.2.`@EventListener`注释

映射端点的另一种方法是使用`@EventListener`注释。我们在处理`ContextRefreshedEvent`的方法上直接使用这个注释:

```java
@EventListener
public void handleContextRefresh(ContextRefreshedEvent event) {
    ApplicationContext applicationContext = event.getApplicationContext();
    RequestMappingHandlerMapping requestMappingHandlerMapping = applicationContext
        .getBean("requestMappingHandlerMapping", RequestMappingHandlerMapping.class);
    Map<RequestMappingInfo, HandlerMethod> map = requestMappingHandlerMapping
        .getHandlerMethods();
    map.forEach((key, value) -> LOGGER.info("{} {}", key, value));
}
```

## 4.致动器方法

检索所有端点列表的第二种方法是通过 [Spring Boot 执行器](/web/20220628051659/https://www.baeldung.com/spring-boot-actuators)特性。

### 4.1.Maven 依赖性

为了启用这个特性，我们将把`[spring-boot-actuator](https://web.archive.org/web/20220628051659/https://search.maven.org/classic/#search%7Cga%7C1%7Cg%3A%22org.springframework.boot%22%20AND%20a%3A%22spring-boot-starter-actuator%22)` Maven 依赖项添加到我们的`pom.xml`文件中:

```java
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-actuator</artifactId>
</dependency>
```

### 4.2.配置

当我们添加`spring-boot-actuator`依赖项时，默认情况下只有`/health`和`/info`端点可用。为了启用所有的致动器[端点](https://web.archive.org/web/20220628051659/https://docs.spring.io/spring-boot/docs/current/reference/html/production-ready-features.html#production-ready-endpoints)，我们可以通过向我们的`application.properties`文件添加一个属性来公开它们:

```java
management.endpoints.web.exposure.include=*
```

或者，我们可以简单地通过**公开端点来检索`mappings`** :

```java
management.endpoints.web.exposure.include=mappings
```

一旦启用，我们的应用程序的 REST API 端点在`http://host/actuator/mappings`可用。

## 5.时髦的

还可以使用 [Swagger 库](/web/20220628051659/https://www.baeldung.com/swagger-2-documentation-for-spring-rest-api)来列出 REST API 的所有端点。

### 5.1.Maven 依赖性

要将它添加到我们的项目中，我们需要在`pom.xml`文件中有一个 [`springfox-boot-starter`](https://web.archive.org/web/20220628051659/https://search.maven.org/classic/#search%7Cga%7C1%7C%22springfox-boot-starter%22) 依赖项:

```java
<dependency>
    <groupId>io.springfox</groupId>
    <artifactId>springfox-boot-starter</artifactId>
    <version>3.0.0</version>
</dependency> 
```

### 5.2.配置

让我们通过定义 [`Docket`](/web/20220628051659/https://www.baeldung.com/swagger-2-documentation-for-spring-rest-api#1-java-configuration) bean 来创建配置类:

```java
@Bean
public Docket api() {
    return new Docket(DocumentationType.SWAGGER_2)
      .select()
      .apis(RequestHandlerSelectors.any())
      .paths(PathSelectors.any())
      .build();
}
```

`Docket`是一个构建器类，用于配置 Swagger 文档的生成。要访问 REST API 端点，我们可以在浏览器中访问以下 URL:

```java
http://host/v2/api-docs
```

## 6.结论

在本文中，我们描述了如何使用事件监听器、Spring Boot 执行器和 Swagger 库在 Spring Boot 应用程序中检索请求映射端点。

像往常一样，本教程中使用的所有代码示例都可以在 GitHub 上获得。