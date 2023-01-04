# Zuul 路线的回退

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-zuul-fallback-route>

## 1.概观

Zuul 是来自网飞的边缘服务(或 API 网关),提供动态路由、监控、弹性和安全性。

在本教程中，我们将看看如何**配置带有回退**的 Zuul 路由。

## 2.初始设置

首先，我们将首先设置两个 Spring Boot 应用程序。在第一个应用程序中，我们将创建一个简单的 REST 服务。然而，在第二个应用程序中，我们将使用 Zuul 代理为第一个应用程序的 REST 服务创建一个路由。

### 2.1.简单的休息服务

假设我们的应用程序需要向用户显示今天的天气信息。因此，我们将使用 [`spring-boot-starter-web`](https://web.archive.org/web/20221128041731/https://search.maven.org/search?q=a:spring-boot-starter-web) 启动器创建一个基于 Spring Boot 的天气服务应用程序:

```java
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
</dependency>
```

现在，我们将为天气服务创建一个控制器:

```java
@RestController
@RequestMapping("/weather")
public class WeatherController {

    @GetMapping("/today")
    public String getMessage() {
        return "It's a bright sunny day today!";
    }

}
```

现在，让我们运行天气服务并检查天气服务 API:

```java
$ curl -s localhost:8080/weather/today
It's a bright sunny day today!
```

### 2.2.API 网关应用程序

现在让我们创建第二个 Spring Boot 应用程序，API Gateway。在这个应用程序中，我们将为天气服务创建一条 Zuul 路线。

由于我们的天气服务和 Zuul 都希望默认使用 8080，我们将它配置为在另一个端口 7070 上运行。

所以，我们先在 pom.xml 中添加 [`spring-cloud-starter-netflix-zuul`](https://web.archive.org/web/20221128041731/https://search.maven.org/search?q=a:spring-cloud-starter-netflix-zuul) :

```java
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-netflix-zuul</artifactId>
</dependency>
```

接下来，我们将把`@EnableZuulProxy`注释添加到我们的 API 网关应用程序类中:

```java
@SpringBootApplication
@EnableZuulProxy
public class ApiGatewayApplication {

    public static void main(String[] args) {
        SpringApplication.run(ApiGatewayApplication.class, args);
    }

}
```

最后，我们将使用 Ribbon 为我们在`application.yml`中的天气服务 API 配置 Zuul 路线:

```java
spring:
   application:
      name: api-gateway
server:
   port: 7070

zuul:
   igoredServices: '*'
   routes:
      weather-service:
         path: /weather/**
         serviceId: weather-service
         strip-prefix: false

ribbon:
   eureka:
      enabled: false

weather-service:
   ribbon:
      listOfServers: localhost:8080
```

### 2.3.测试 Zuul 路线

至此，两个 Spring Boot 应用程序都已设置好使用 Zuul 代理公开天气服务 API。

因此，让我们运行这两个应用程序，并通过 Zuul 检查天气服务 API:

```java
$ curl -s localhost:7070/weather/today
It's a bright sunny day today!
```

### 2.4.测试没有回退的 Zuul 路由故障

现在，让我们停止天气服务应用程序，并再次通过 Zuul 检查天气服务。因此，我们将在响应中看到一条错误消息:

```java
$ curl -s localhost:7070/weather/today
{"timestamp":"2019-10-08T12:42:09.479+0000","status":500,
"error":"Internal Server Error","message":"GENERAL"}
```

显然，这不是用户希望看到的响应。因此，我们可以解决这个问题的方法之一是为天气服务 Zuul 路线创建一个备用路线。

## 3.路由的 Zuul 回退

Zuul 代理使用 Ribbon 进行负载平衡，请求在 Hystrix 命令中执行。因此，Zuul 路线中的**个故障出现在一个矩阵**中。

因此，要为 Zuul 路由创建自定义回退，我们将创建一个类型为 [`FallbackProvider`](https://web.archive.org/web/20221128041731/https://static.javadoc.io/org.springframework.cloud/spring-cloud-netflix-core/1.4.3.RELEASE/index.html?org/springframework/cloud/netflix/zuul/filters/route/FallbackProvider.html) 的 bean。

### 3.1.`WeatherServiceFallback`类

在本例中，我们希望从回退响应中返回一条消息，而不是我们之前看到的默认错误消息。因此，让我们为天气服务路线创建一个简单的`FallbackProvider`实现:

```java
@Component
class WeatherServiceFallback implements FallbackProvider {

    private static final String DEFAULT_MESSAGE = "Weather information is not available.";

    @Override
    public String getRoute() {
        return "weather-service";
    }

    @Override
    public ClientHttpResponse fallbackResponse(String route, Throwable cause) {
        if (cause instanceof HystrixTimeoutException) {
            return new GatewayClientResponse(HttpStatus.GATEWAY_TIMEOUT, DEFAULT_MESSAGE);
        } else {
            return new GatewayClientResponse(HttpStatus.INTERNAL_SERVER_ERROR, DEFAULT_MESSAGE);
        }
    }

}
```

正如我们所看到的，我们已经覆盖了方法`getRoute`和`fallbackResponse`。**`getRoute`方法返回路由**的`Id`，我们必须为其创建回退。然而，**`fallbackResponse`方法返回定制的回退响应**，在我们的例子中是一个类型为`GatewayClientResponse`的对象。`GatewayClientResponse`是 [`ClientHttpResponse`](https://web.archive.org/web/20221128041731/https://docs.spring.io/spring/docs/current/javadoc-api/org/springframework/http/client/ClientHttpResponse.html) 的简单实现。

### 3.2.测试 Zuul 回退

现在让我们测试我们为天气服务创建的回退。因此，我们将运行 API 网关应用程序，并确保天气服务应用程序已停止。

现在，让我们通过 Zuul 路由访问天气服务 API，并查看回退响应的运行情况:

```java
$ curl -s localhost:7070/weather/today
Weather information is not available.
```

## 4.所有路由的回退

到目前为止，我们已经看到了如何使用路由`Id`为 Zuul 路由创建回退。然而，让我们假设，我们还想**在我们的应用程序中为所有其他路由**创建一个通用回退。我们可以通过创建另一个从`getRoute`方法返回`*`或`null`的`FallbackProvider`和**的实现来做到这一点，而不是路由`Id`:**

```java
@Override
public String getRoute() {
    return "*"; // or return null;
}
```

## 5.结论

在本教程中，我们看到了一个为 Zuul 路由创建回退的示例。我们还看到了如何为所有 Zuul 路由创建通用回退。

像往常一样，所有这些例子和代码片段的实现都可以在 GitHub 上找到[。](https://web.archive.org/web/20221128041731/https://github.com/eugenp/tutorials/tree/master/spring-cloud-modules/spring-cloud-zuul-fallback)