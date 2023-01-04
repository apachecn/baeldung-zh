# 探索新的 Spring 云网关

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-cloud-gateway>

## 1。概述

在本教程中，我们将探索 [Spring Cloud Gateway](https://web.archive.org/web/20221022212758/https://cloud.spring.io/spring-cloud-gateway/) 项目的主要特性，这是一个基于 Spring 5、Spring Boot 2 和 Project Reactor 的新 API。

该工具提供了在微服务应用程序中经常使用的开箱即用的路由机制，作为一种将多个服务隐藏在单个外观后面的方式。

对于没有 Spring Cloud Gateway 项目的 Gateway 模式的解释，请查看我们的[前一篇文章](/web/20221022212758/https://www.baeldung.com/spring-cloud-gateway-pattern)。

## 2。路由处理器

专注于路由请求，Spring Cloud Gateway 将请求转发到网关处理程序映射，该映射决定应该如何处理匹配特定路由的请求。

让我们从一个简单的例子开始，看看网关处理器如何使用`RouteLocator`解析路由配置:

```
@Bean
public RouteLocator customRouteLocator(RouteLocatorBuilder builder) {
    return builder.routes()
      .route("r1", r -> r.host("**.baeldung.com")
        .and()
        .path("/baeldung")
        .uri("http://baeldung.com"))
      .route(r -> r.host("**.baeldung.com")
        .and()
        .path("/myOtherRouting")
        .filters(f -> f.prefixPath("/myPrefix"))
        .uri("http://othersite.com")
        .id("myOtherID"))
    .build();
}
```

请注意我们是如何利用这个 API 的主要构件的:

*   `Route` —网关的主要 API。它由给定的标识(ID)、目的地(URI)以及一组谓词和过滤器定义。
*   `Predicate`—Java 8`Predicate`—用于匹配使用头、方法或参数的 HTTP 请求
*   `Filter` —标准弹簧`WebFilter`

## 3。动态路由

就像 [Zuul](/web/20221022212758/https://www.baeldung.com/spring-rest-with-zuul-proxy) 一样，Spring Cloud Gateway 提供了将请求路由到不同服务的方法。

路由配置可以通过使用纯 Java ( `RouteLocator`，如第 2 节中的示例所示)或使用属性配置来创建:

```
spring:
  application:
    name: gateway-service  
  cloud:
    gateway:
      routes:
      - id: baeldung
        uri: baeldung.com
      - id: myOtherRouting
        uri: localhost:9999
```

## 4。路由工厂

Spring Cloud Gateway 使用 Spring WebFlux `HandlerMapping`基础设施匹配路由。

它还包括许多内置的路由谓词工厂。所有这些谓词都匹配 HTTP 请求的不同属性。多个路由断言工厂可以通过逻辑“与”来组合。

路由匹配可以通过编程方式应用，也可以通过使用不同类型的路由谓词工厂的配置属性文件来应用。

我们的文章 [Spring Cloud Gateway 路由谓词工厂](/web/20221022212758/https://www.baeldung.com/spring-cloud-gateway-routing-predicate-factories)更详细地探讨了路由工厂。

## 5。WebFilter 工厂

路由过滤器使得修改传入的 HTTP 请求或传出的 HTTP 响应成为可能。

Spring Cloud Gateway 包括许多内置的 WebFilter 工厂以及创建自定义过滤器的可能性。

我们的文章[Spring Cloud Gateway WebFilter Factories](/web/20221022212758/https://www.baeldung.com/spring-cloud-gateway-webfilter-factories)更详细地探讨了 web filter Factories。

## 6。Spring Cloud DiscoveryClient 客户端支持

Spring Cloud Gateway 可以很容易地与服务发现和注册库集成，比如 Eureka Server 和 Consul:

```
@Configuration
@EnableDiscoveryClient
public class GatewayDiscoveryConfiguration {

    @Bean
    public DiscoveryClientRouteDefinitionLocator 
      discoveryClientRouteLocator(DiscoveryClient discoveryClient) {

        return new DiscoveryClientRouteDefinitionLocator(discoveryClient);
    }
}
```

### 6.1。`LoadBalancerClient`滤镜

`LoadBalancerClientFilter`使用`ServerWebExchangeUtils.GATEWAY_REQUEST_URL_ATTR`在交换属性中查找 URI。

如果 URL 有一个`lb`方案(例如`lb://baeldung-service`，它将使用 Spring Cloud `LoadBalancerClient`将名称(例如`baeldung-service`)解析为一个实际的主机和端口。

未修改的原始 URL 放在`ServerWebExchangeUtils.GATEWAY_ORIGINAL_REQUEST_URL_ATTR`属性中。

## 7。监控

Spring Cloud Gateway 利用了 Actuator API，这是一个著名的 Spring Boot 库，它提供了几个现成的服务来监控应用程序。

一旦安装和配置了执行器 API，就可以通过访问`/gateway/` 端点来可视化网关监控功能。

## 8。实施

我们现在将使用`path` 谓词创建一个使用 Spring Cloud Gateway 作为代理服务器的简单示例。

### 8.1。依赖性

Spring Cloud Gateway 目前在 2.0.0.RC2 版本的里程碑库中，这也是我们在这里使用的版本。

为了添加项目，我们将使用依赖关系管理系统:

```
<dependencyManagement>
    <dependencies>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-gateway</artifactId>
            <version>2.0.0.RC2</version>
            <type>pom</type>
            <scope>import</scope>
        </dependency>
    </dependencies>
</dependencyManagement>
```

接下来，我们将添加必要的依赖项:

```
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-actuator</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-webflux</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-gateway</artifactId>
</dependency> 
```

### 8.2。代码实现

现在我们在`application.yml`文件中创建一个简单的路由配置:

```
spring:
  cloud:
    gateway:
      routes:
      - id: baeldung_route
        uri: http://baeldung.com
        predicates:
        - Path=/baeldung/
management:
  endpoints:
    web:
      exposure:
        include: "*' 
```

这是网关应用程序代码:

```
@SpringBootApplication
public class GatewayApplication {
    public static void main(String[] args) {
        SpringApplication.run(GatewayApplication.class, args);
    }
}
```

在应用程序启动后，我们可以访问 url ` “http://localhost/actuator/gateway/routes/baeldung_route”`来检查所有创建的路由配置:

```
{
    "id":"baeldung_route",
    "predicates":[{
        "name":"Path",
        "args":{"_genkey_0":"/baeldung"}
    }],
    "filters":[],
    "uri":"http://baeldung.com",
    "order":0
}
```

我们看到相对 url `“/baeldung”`被配置为路由。因此，点击 url `“http://localhost/baeldung”`，我们将被重定向到`“http://baeldung.com”`，就像我们的例子中配置的那样。

## 9。结论

在本文中，我们探讨了 Spring Cloud Gateway 的一些特性和组件。这个新的 API 为网关和代理支持提供了现成的工具。

这里展示的例子可以在我们的 [GitHub 库](https://web.archive.org/web/20221022212758/https://github.com/eugenp/tutorials/tree/master/spring-cloud-modules/spring-cloud-gateway)中找到。