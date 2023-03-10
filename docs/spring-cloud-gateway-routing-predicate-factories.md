# Spring 云网关路由谓词工厂

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-cloud-gateway-routing-predicate-factories>

## 1.介绍

在上一篇文章中，我们已经介绍了什么是 [Spring Cloud Gateway](/web/20221117030334/https://www.baeldung.com/spring-cloud-gateway) 以及如何使用内置谓词来实现基本的路由规则。然而，有时这些内置谓词可能还不够。例如，出于某种原因，我们的路由逻辑可能需要数据库查找。

对于那些情况， **Spring Cloud Gateway 允许我们定义** **自定义谓词。**一旦定义完毕，我们可以将它们用作任何其他谓词，这意味着我们可以使用 fluent API 和/或 DSL 来定义路由。

## 2.谓词的剖析

简而言之，Spring Cloud Gateway 中的`Predicate`是一个测试给定请求是否满足给定条件的对象。对于每个路由，我们可以定义一个或多个谓词，如果满足这些谓词，那么在应用任何过滤器之后，这些谓词将接受对已配置后端的请求。

在编写我们的谓词之前，让我们看一下现有谓词的源代码，或者更准确地说，看一下现有`PredicateFactory.`的代码正如名称所暗示的，Spring Cloud Gateway 使用流行的[工厂方法模式](https://web.archive.org/web/20221117030334/https://en.wikipedia.org/wiki/Factory_method_pattern)作为一种机制，以可扩展的方式支持`Predicate` 实例的创建。

我们可以选择任意一个内置的谓词工厂，这些工厂在 [`spring-cloud-gateway-core`](https://web.archive.org/web/20221117030334/https://github.com/spring-cloud/spring-cloud-gateway/tree/v2.1.3.RELEASE/spring-cloud-gateway-core) 模块的 [`org.springframework.cloud.gateway.handler.predicate`](https://web.archive.org/web/20221117030334/https://github.com/spring-cloud/spring-cloud-gateway/tree/v2.1.3.RELEASE/spring-cloud-gateway-core/src/main/java/org/springframework/cloud/gateway/handler/predicate) 包中有。我们可以很容易地找到现存的，因为它们的名字都以`RoutePredicateFactory`结尾。 [`HeaderRouterPredicateFactory`](https://web.archive.org/web/20221117030334/https://github.com/spring-cloud/spring-cloud-gateway/blob/v2.1.3.RELEASE/spring-cloud-gateway-core/src/main/java/org/springframework/cloud/gateway/handler/predicate/HeaderRoutePredicateFactory.java) 就是一个很好的例子:

```java
public class HeaderRoutePredicateFactory extends 
  AbstractRoutePredicateFactory<HeaderRoutePredicateFactory.Config> {

    // ... setup code omitted
    @Override
    public Predicate<ServerWebExchange> apply(Config config) {
        return new GatewayPredicate() {
            @Override
            public boolean test(ServerWebExchange exchange) {
                // ... predicate logic omitted
            }
        };
    }

    @Validated
    public static class Config {
        public Config(boolean isGolden, String customerIdCookie ) {
          // ... constructor details omitted
        }
        // ...getters/setters omitted
    }
}
```

在实施过程中，我们可以观察到几个关键点:

*   它扩展了`AbstractRoutePredicateFactory<T>`，而 T0 又实现了网关使用的`RoutePredicateFactory`接口
*   在这种情况下，`apply`方法返回实际的`Predicate –` a `GatewayPredicate` 的一个实例
*   谓词定义了一个内部的`Config` 类，用来存储测试逻辑使用的静态配置参数

如果我们看看其他可用的`PredicateFactory, `,我们会发现基本模式基本相同:

1.  定义一个`Config`类来保存配置参数
2.  使用配置类作为模板参数，扩展`AbstractRoutePredicateFactory`
3.  覆盖`apply`方法，返回实现所需测试逻辑的`Predicate`

## 3.实现自定义谓词工厂

对于我们的实现，让我们假设以下场景:对于给定的 API，调用我们必须在两个可能的后端之间进行选择。“黄金”客户是我们最有价值的客户，他们应该被路由到一个强大的服务器，可以访问更多的内存、更多的 CPU 和高速磁盘。非金牌客户会选择功能较弱的服务器，这会导致响应速度较慢。

为了确定请求是否来自黄金客户，我们需要调用一个服务，该服务获取与请求相关联的`customerId`并返回其状态。至于`customerId`，在我们的简单场景中，我们假设它存在于 cookie 中。

有了这些信息，我们现在可以编写自定义谓词了。我们将保留现有的命名约定，并将我们的类命名为`GoldenCustomerRoutePredicateFactory`:

```java
public class GoldenCustomerRoutePredicateFactory extends 
  AbstractRoutePredicateFactory<GoldenCustomerRoutePredicateFactory.Config> {

    private final GoldenCustomerService goldenCustomerService;

    // ... constructor omitted

    @Override
    public Predicate<ServerWebExchange> apply(Config config) {        
        return (ServerWebExchange t) -> {
            List<HttpCookie> cookies = t.getRequest()
              .getCookies()
              .get(config.getCustomerIdCookie());

            boolean isGolden; 
            if ( cookies == null || cookies.isEmpty()) {
                isGolden = false;
            } else {                
                String customerId = cookies.get(0).getValue();                
                isGolden = goldenCustomerService.isGoldenCustomer(customerId);
            }              
            return config.isGolden() ? isGolden : !isGolden;           
        };        
    }

    @Validated
    public static class Config {        
        boolean isGolden = true;        
        @NotEmpty
        String customerIdCookie = "customerId";
        // ...constructors and mutators omitted   
    }    
}
```

正如我们所看到的，实现非常简单。我们的`apply`方法使用传递给它的`ServerWebExchange`返回一个实现所需逻辑的 lambda。首先，它检查是否存在`customerId` cookie。如果它找不到，那么这是一个正常的客户。否则，我们使用 cookie 值来调用`isGoldenCustomer`服务方法。

接下来，我们将客户机的类型与配置的`isGolden` 参数结合起来，以确定返回值。**这允许我们使用相同的谓词创建前面描述的两条路由，只需改变参数`isGolden`的值**。

## 4.注册自定义谓词工厂

一旦我们编写了自定义谓词工厂，我们需要一种方法来让 Spring Cloud Gateway 意识到 if。因为我们使用的是 Spring，所以这是以通常的方式完成的:我们声明一个类型为`GoldenCustomerRoutePredicateFactory` `.`的 bean

由于我们的类型通过 is 基类实现了`RoutePredicateFactory `,它将在上下文初始化时被 Spring 选中，并提供给 Spring Cloud Gateway。

这里，我们将使用一个`@Configuration`类来创建我们的 bean:

```java
@Configuration
public class CustomPredicatesConfig {
    @Bean
    public GoldenCustomerRoutePredicateFactory goldenCustomer(
      GoldenCustomerService goldenCustomerService) {
        return new GoldenCustomerRoutePredicateFactory(goldenCustomerService);
    }
}
```

我们假设在 Spring 的上下文中有一个合适的`GoldenCustomerService`实现。在我们的例子中，我们只有一个虚拟实现，它将`customerId`值与一个固定值进行比较——这并不现实，但对于演示来说很有用。

## 5.使用自定义谓词

既然我们已经实现了“黄金客户”谓词，并且可以用于 Spring Cloud Gateway，我们可以开始使用它来定义路由。首先，我们将使用 fluent API 来定义一条路线，然后我们将使用 YAML 以声明的方式来完成它。

### 5.1.使用 Fluent API 定义路线

当我们必须以编程方式创建复杂对象时，流畅的 API 是一种流行的设计选择。在我们的例子中，我们在一个 `@Bean`中定义路由，它使用一个`RouteLocatorBuilder`和我们的定制谓词工厂创建一个`RouteLocator`对象:

```java
@Bean
public RouteLocator routes(RouteLocatorBuilder builder, GoldenCustomerRoutePredicateFactory gf ) {
    return builder.routes()
      .route("golden_route", r -> r.path("/api/**")
        .uri("https://fastserver")
        .predicate(gf.apply(new Config(true, "customerId"))))
      .route("common_route", r -> r.path("/api/**")
        .uri("https://slowserver")
        .predicate(gf.apply(new Config(false, "customerId"))))                
      .build();
}
```

注意我们如何在每条路线中使用两种不同的`Config`配置。在第一种情况下，第一个参数是`true`，所以当我们有一个来自黄金客户的请求时，谓词也计算为`true` 。至于第二条路线，我们在构造函数中传递了`false`,因此我们的谓词将为非黄金客户返回`true `。

### 5.2.在 YAML 定义路线

我们可以使用属性或 YAML 文件以声明的方式实现与以前相同的结果。在这里，我们将使用 YAML，因为它更容易阅读:

```java
spring:
  cloud:
    gateway:
      routes:
      - id: golden_route
        uri: https://fastserver
        predicates:
        - Path=/api/**
        - GoldenCustomer=true
      - id: common_route
        uri: https://slowserver
        predicates:
        - Path=/api/**
        - name: GoldenCustomer
          args:
            golden: false
            customerIdCookie: customerId 
```

这里我们定义了和以前一样的路由，使用了两个可用的选项来定义谓词。第一个是`golden_route`，它使用了一个紧凑的表示形式`Predicate=[param[,param]+]`。`Predicate`这里是谓词的名称，它是通过去掉后缀`RoutePredicateFactory` 从工厂类名中自动派生出来的。在“=”符号之后，我们有用于填充关联的`Config`实例的参数。

当我们的谓词只需要简单的值时，这种紧凑的语法很好，但情况可能并不总是如此。对于这些场景，我们可以使用长格式，如第二条路线所示。在这种情况下，我们提供一个具有两个属性的对象:`name`和`args`。`name` 包含谓词名称，`args`用于填充`Config `实例。由于这次`args`是一个对象，我们的配置可以根据需要任意复杂。

## 6.测试

现在，让我们使用`curl`来测试我们的网关，检查是否一切正常。对于这些测试，我们已经像前面显示的那样设置了我们的路由，但是我们将使用公开可用的`[httpbin.org](https://web.archive.org/web/20221117030334/https://httpbin.org/)`服务作为我们的虚拟后端。这是一个非常有用的服务，我们可以使用它来快速检查我们的规则是否如预期的那样工作，既可以在线使用，也可以作为 docker 图像在本地使用。

我们的测试配置还包括标准的`AddRequestHeader`滤波器。我们用它向请求添加一个定制的`Goldencustomer`头，其值对应于谓词结果。我们还添加了一个`StripPrefix`过滤器，因为我们想在调用后端之前从请求 URI 中删除/ `api`。

首先，让我们测试“公共客户端”场景。随着网关的启动和运行，我们使用 curl 来调用`httpbin`的`headers` API，它将简单地回显所有接收到的头:

```java
$ curl http://localhost:8080/api/headers
{
  "headers": {
    "Accept": "*/*",
    "Forwarded": "proto=http;host=\"localhost:8080\";for=\"127.0.0.1:51547\"",
    "Goldencustomer": "false",
    "Host": "httpbin.org",
    "User-Agent": "curl/7.55.1",
    "X-Forwarded-Host": "localhost:8080",
    "X-Forwarded-Prefix": "/api"
  }
}
```

正如所料，我们看到发送的`Goldencustomer`报头带有一个`false`值。现在让我们来试试一位“黄金”客户:

```java
$ curl -b customerId=baeldung http://localhost:8080/api/headers
{
  "headers": {
    "Accept": "*/*",
    "Cookie": "customerId=baeldung",
    "Forwarded": "proto=http;host=\"localhost:8080\";for=\"127.0.0.1:51651\"",
    "Goldencustomer": "true",
    "Host": "httpbin.org",
    "User-Agent": "curl/7.55.1",
    "X-Forwarded-Host": "localhost:8080",
    "X-Forwarded-Prefix": "/api"
  }
}
```

这一次，`Goldencustomer`是`true`，因为我们已经发送了一个`customerId` cookie，其值被我们的虚拟服务识别为对黄金客户有效。

## 7.结论

在本文中，我们介绍了如何向 Spring Cloud Gateway 添加定制谓词工厂，并使用它们通过任意逻辑定义路由。

像往常一样，GitHub 上的所有代码[都是可用的。](https://web.archive.org/web/20221117030334/https://github.com/eugenp/tutorials/tree/master/spring-cloud-modules/spring-cloud-gateway)