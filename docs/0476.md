# 编写定制的 Spring 云网关过滤器

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-cloud-custom-gateway-filters>

## 1。概述

在本教程中，我们将学习如何编写定制的 Spring 云网关过滤器。

我们在之前的文章[探索新的 Spring Cloud Gateway](/web/20221202171428/https://www.baeldung.com/spring-cloud-gateway) 中介绍了这个框架，在那里我们看到了许多内置的过滤器。

在这种情况下，我们将深入探讨，我们将编写自定义过滤器来充分利用我们的 API 网关。

首先，我们将了解如何创建全局过滤器来影响网关处理的每一个请求。然后，我们将编写网关过滤器工厂，它可以精确地应用于特定的路由和请求。

最后，我们将学习更高级的场景，学习如何修改请求或响应，甚至如何以一种被动的方式将请求与对其他服务的调用联系起来。

## 2。项目设置

我们将从设置一个基本应用程序开始，我们将使用它作为我们的 API 网关。

### 2.1。Maven 配置

当使用 Spring Cloud 库时，设置一个依赖管理配置来为我们处理依赖关系总是一个不错的选择:

```
<dependencyManagement>
    <dependencies>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-dependencies</artifactId>
            <version>Hoxton.SR4</version>
            <type>pom</type>
            <scope>import</scope>
        </dependency>
    </dependencies>
</dependencyManagement>
```

现在，我们可以添加我们的 Spring Cloud 库，而无需指定我们正在使用的实际版本:

```
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-gateway</artifactId>
</dependency>
```

最新的[春云发布列车版本](https://web.archive.org/web/20221202171428/https://search.maven.org/search?q=g:org.springframework.cloud%20AND%20a:spring-cloud-dependencies)可以使用 Maven 中央搜索引擎找到。当然，我们应该经常检查这个版本是否与我们在[Spring Cloud 文档](https://web.archive.org/web/20221202171428/https://spring.io/projects/spring-cloud)中使用的 Spring Boot 版本兼容。

### 2.2。API 网关配置

我们将假设在端口`8081`中有第二个本地运行的应用程序，它在点击`/resource`时公开一个资源(为了简单起见，只是一个简单的`String`)。

考虑到这一点，我们将配置网关来代理对该服务的请求。简而言之，当我们向 URI 路径中带有前缀`/service` 的网关发送请求时，我们将把调用转发给这个服务。

因此，当我们在网关中调用`/service/resource `时，我们应该会收到`String` 响应。

为了实现这一点，我们将使用`application properties`来配置这条路由:

```
spring:
  cloud:
    gateway:
      routes:
      - id: service_route
        uri: http://localhost:8081
        predicates:
        - Path=/service/**
        filters:
        - RewritePath=/service(?<segment>/?.*), $\{segment}
```

此外，为了能够正确跟踪网关进程，我们还将启用一些日志:

```
logging:
  level:
    org.springframework.cloud.gateway: DEBUG
    reactor.netty.http.client: DEBUG
```

## 3。创建全局过滤器

一旦网关处理器确定一个请求匹配一个路由，框架就通过过滤器链传递该请求。这些过滤器可以在发送请求之前或之后执行逻辑。

在这一节中，我们将从编写简单的全局过滤器开始。这意味着，它会影响每一个请求。

首先，我们将了解如何在发送代理请求之前执行逻辑(也称为“pre”过滤器)

### 3.1.编写全局“预”过滤器逻辑

正如我们所说的，我们将在这一点上创建简单的过滤器，因为这里的主要目标只是看到过滤器实际上在正确的时刻被执行；只需记录一条简单的消息就可以了。

**要创建一个定制的全局过滤器，我们所要做的就是实现 Spring Cloud Gateway `GlobalFilter `接口，并将其作为 bean 添加到上下文中:**

```
@Component
public class LoggingGlobalPreFilter implements GlobalFilter {

    final Logger logger =
      LoggerFactory.getLogger(LoggingGlobalPreFilter.class);

    @Override
    public Mono<Void> filter(
      ServerWebExchange exchange,
      GatewayFilterChain chain) {
        logger.info("Global Pre Filter executed");
        return chain.filter(exchange);
    }
}
```

我们可以很容易地看到这里发生了什么；一旦这个过滤器被调用，我们将记录一条消息，并继续执行过滤器链。

现在让我们定义一个“post”过滤器，如果我们不熟悉[反应式编程模型和 Spring Webflux API](/web/20221202171428/https://www.baeldung.com/spring-webflux) ，这个过滤器可能会稍微复杂一点。

### 3.2.编写全局“后”过滤器逻辑

关于我们刚刚定义的全局过滤器，需要注意的另一件事是,`GlobalFilter `接口只定义了一个方法。因此，它可以表示为一个[λ表达式](/web/20221202171428/https://www.baeldung.com/java-8-lambda-expressions-tips)，允许我们方便地定义过滤器。

例如，我们可以在配置类中定义“后”过滤器:

```
@Configuration
public class LoggingGlobalFiltersConfigurations {

    final Logger logger =
      LoggerFactory.getLogger(
        LoggingGlobalFiltersConfigurations.class);

    @Bean
    public GlobalFilter postGlobalFilter() {
        return (exchange, chain) -> {
            return chain.filter(exchange)
              .then(Mono.fromRunnable(() -> {
                  logger.info("Global Post Filter executed");
              }));
        };
    }
}
```

简单地说，这里我们在链完成执行后运行一个新的`Mono` 实例。

现在，让我们通过调用网关服务中的`/service/resource` URL 来尝试一下，并检查日志控制台:

```
DEBUG --- o.s.c.g.h.RoutePredicateHandlerMapping:
  Route matched: service_route
DEBUG --- o.s.c.g.h.RoutePredicateHandlerMapping:
  Mapping [Exchange: GET http://localhost/service/resource]
  to Route{id='service_route', uri=http://localhost:8081, order=0, predicate=Paths: [/service/**],
  match trailing slash: true, gatewayFilters=[[[RewritePath /service(?<segment>/?.*) = '${segment}'], order = 1]]}
INFO  --- c.b.s.c.f.global.LoggingGlobalPreFilter:
  Global Pre Filter executed
DEBUG --- r.netty.http.client.HttpClientConnect:
  [id: 0x58f7e075, L:/127.0.0.1:57215 - R:localhost/127.0.0.1:8081]
  Handler is being applied: {uri=http://localhost:8081/resource, method=GET}
DEBUG --- r.n.http.client.HttpClientOperations:
  [id: 0x58f7e075, L:/127.0.0.1:57215 - R:localhost/127.0.0.1:8081]
  Received response (auto-read:false) : [Content-Type=text/html;charset=UTF-8, Content-Length=16]
INFO  --- c.f.g.LoggingGlobalFiltersConfigurations:
  Global Post Filter executed
DEBUG --- r.n.http.client.HttpClientOperations:
  [id: 0x58f7e075, L:/127.0.0.1:57215 - R:localhost/127.0.0.1:8081] Received last HTTP packet
```

正如我们所看到的，过滤器在网关将请求转发给服务之前和之后被有效地执行。

自然，我们可以在一个过滤器中结合“前置”和“后置”逻辑:

```
@Component
public class FirstPreLastPostGlobalFilter
  implements GlobalFilter, Ordered {

    final Logger logger =
      LoggerFactory.getLogger(FirstPreLastPostGlobalFilter.class);

    @Override
    public Mono<Void> filter(ServerWebExchange exchange,
      GatewayFilterChain chain) {
        logger.info("First Pre Global Filter");
        return chain.filter(exchange)
          .then(Mono.fromRunnable(() -> {
              logger.info("Last Post Global Filter");
            }));
    }

    @Override
    public int getOrder() {
        return -1;
    }
}
```

**注意，如果我们关心过滤器在链中的位置，我们也可以实现`Ordered`接口。**

**由于过滤器链的性质，具有较低优先级(链中的较低顺序)的过滤器将在较早阶段执行其“前”逻辑，但其“后”实现将在稍后调用:**

[![SpringCloudGateway](img/0ea8d2907dfadabc24d85e207c3d9fd7.png)](/web/20221202171428/https://www.baeldung.com/wp-content/uploads/2019/11/SpringCloudGateway-01-768x1091.png)

## 4。创造`GatewayFilter`年代

全局过滤器非常有用，但是我们经常需要执行仅适用于某些路由的细粒度自定义网关过滤器操作。

### 4.1.定义`GatewayFilterFactory`

为了实现一个`GatewayFilter`，我们必须实现`GatewayFilterFactory` 接口。Spring Cloud Gateway 还提供了一个抽象类来简化这个过程，这个`AbstractGatewayFilterFactory `类:

```
@Component
public class LoggingGatewayFilterFactory extends 
  AbstractGatewayFilterFactory<LoggingGatewayFilterFactory.Config> {

    final Logger logger =
      LoggerFactory.getLogger(LoggingGatewayFilterFactory.class);

    public LoggingGatewayFilterFactory() {
        super(Config.class);
    }

    @Override
    public GatewayFilter apply(Config config) {
        // ...
    }

    public static class Config {
        // ...
    }
}
```

这里我们已经定义了我们的`GatewayFilterFactory`的基本结构。**我们将使用一个`Config `类来定制我们的过滤器。**

例如，在这种情况下，我们可以在配置中定义三个基本字段:

```
public static class Config {
    private String baseMessage;
    private boolean preLogger;
    private boolean postLogger;

    // contructors, getters and setters...
}
```

简单地说，这些字段是:

1.  将包含在日志条目中的自定义消息
2.  一个标志，指示筛选器是否应在转发请求前记录
3.  一个标志，指示筛选器在收到代理服务的响应后是否应记录日志

现在我们可以使用这些配置来检索一个`GatewayFilter`实例，这个实例也可以用 lambda 函数来表示:

```
@Override
public GatewayFilter apply(Config config) {
    return (exchange, chain) -> {
        // Pre-processing
        if (config.isPreLogger()) {
            logger.info("Pre GatewayFilter logging: "
              + config.getBaseMessage());
        }
        return chain.filter(exchange)
          .then(Mono.fromRunnable(() -> {
              // Post-processing
              if (config.isPostLogger()) {
                  logger.info("Post GatewayFilter logging: "
                    + config.getBaseMessage());
              }
          }));
    };
}
```

### 4.2.用属性注册`GatewayFilter`

现在，我们可以轻松地将过滤器注册到之前在应用程序属性中定义的路由:

```
...
filters:
- RewritePath=/service(?<segment>/?.*), $\{segment}
- name: Logging
  args:
    baseMessage: My Custom Message
    preLogger: true
    postLogger: true
```

我们只需指出配置参数。**这里很重要的一点是，我们需要在我们的`LoggingGatewayFilterFactory.Config `类中配置一个无参数的构造函数和 setters，这样这个方法才能正常工作。**

如果我们想使用紧凑符号来配置滤波器，我们可以这样做:

```
filters:
- RewritePath=/service(?<segment>/?.*), $\{segment}
- Logging=My Custom Message, true, true
```

我们需要对我们的工厂进行更多的调整。简而言之，我们必须覆盖`shortcutFieldOrder` 方法，以指示快捷方式属性将使用的顺序和多少个参数:

```
@Override
public List<String> shortcutFieldOrder() {
    return Arrays.asList("baseMessage",
      "preLogger",
      "postLogger");
}
```

### 4.3.订购`GatewayFilter`

**如果我们想要配置过滤器在过滤器链中的位置，我们可以从`AbstractGatewayFilterFactory#apply `方法中检索一个`OrderedGatewayFilter` 实例**，而不是普通的 lambda 表达式:

```
@Override
public GatewayFilter apply(Config config) {
    return new OrderedGatewayFilter((exchange, chain) -> {
        // ...
    }, 1);
}
```

### 4.4.以编程方式注册`GatewayFilter`

此外，我们也可以通过编程来注册我们的过滤器。让我们重新定义我们一直使用的路线，这次通过设置一个`RouteLocator ` bean:

```
@Bean
public RouteLocator routes(
  RouteLocatorBuilder builder,
  LoggingGatewayFilterFactory loggingFactory) {
    return builder.routes()
      .route("service_route_java_config", r -> r.path("/service/**")
        .filters(f -> 
            f.rewritePath("/service(?<segment>/?.*)", "$\\{segment}")
              .filter(loggingFactory.apply(
              new Config("My Custom Message", true, true))))
            .uri("http://localhost:8081"))
      .build();
}
```

## 5。高级场景

到目前为止，我们所做的只是在网关流程的不同阶段记录一条消息。

通常，我们需要过滤器来提供更高级的功能。例如，我们可能需要检查或操作我们收到的请求，修改我们正在检索的响应，或者甚至将反应流与对其他不同服务的调用联系起来。

接下来，我们将看到这些不同场景的示例。

### 5.1.检查和修改请求

让我们想象一个假设的场景。我们的服务曾经基于一个`locale `查询参数提供其内容。然后，我们将 API 改为使用`Accept-Language `头，但是一些客户端仍然在使用查询参数。

因此，我们希望将网关配置为遵循以下逻辑进行规范化:

1.  如果我们收到了`Accept-Language`头，我们希望保留它
2.  否则，使用`locale` 查询参数值
3.  如果不存在，使用默认的语言环境
4.  最后，我们想要删除`locale`查询参数

注意:为了简单起见，我们将只关注过滤器逻辑；为了查看整个实现，我们会在教程的末尾找到一个代码库的链接。

让我们将网关过滤器配置为“预”过滤器，然后:

```
(exchange, chain) -> {
    if (exchange.getRequest()
      .getHeaders()
      .getAcceptLanguage()
      .isEmpty()) {
        // populate the Accept-Language header...
    }

    // remove the query param...
    return chain.filter(exchange);
};
```

在这里，我们关注逻辑的第一个方面。我们可以看到检查`ServerHttpRequest `对象真的很简单。在这一点上，我们只访问了它的头，但是正如我们接下来将看到的，我们可以同样容易地获得其他属性:

```
String queryParamLocale = exchange.getRequest()
  .getQueryParams()
  .getFirst("locale");

Locale requestLocale = Optional.ofNullable(queryParamLocale)
  .map(l -> Locale.forLanguageTag(l))
  .orElse(config.getDefaultLocale());
```

现在我们已经讨论了行为的下两点。但是我们还没有修改请求。为此，**我们必须利用`mutate `功能。**

这样，框架将创建一个实体的`Decorator `，保持原来的对象不变。

修改头很简单，因为我们可以获得对`HttpHeaders` map 对象的引用:

```
exchange.getRequest()
  .mutate()
  .headers(h -> h.setAcceptLanguageAsLocales(
    Collections.singletonList(requestLocale)))
```

但是，另一方面，修改 URI 并不是一件小事。

我们必须从原始的`exchange `对象获得一个新的`ServerWebExchange `实例，修改原始的`ServerHttpRequest` 实例:

```
ServerWebExchange modifiedExchange = exchange.mutate()
  // Here we'll modify the original request:
  .request(originalRequest -> originalRequest)
  .build();

return chain.filter(modifiedExchange);
```

现在是时候通过删除查询参数来更新原始请求 URI 了:

```
originalRequest -> originalRequest.uri(
  UriComponentsBuilder.fromUri(exchange.getRequest()
    .getURI())
  .replaceQueryParams(new LinkedMultiValueMap<String, String>())
  .build()
  .toUri())
```

好了，我们现在可以试试了。在代码库中，我们在调用下一个链式过滤器之前添加了日志条目，以查看请求中到底发送了什么。

### 5.2.修改响应

继续同一个案例场景，我们现在将定义一个“post”过滤器。我们假想的服务过去常常检索一个定制的头来指示它最终选择的语言，而不是使用传统的`Content-Language` 头。

因此，我们希望我们的新过滤器添加这个响应头，但是只有当请求包含我们在上一节中介绍的`locale`头时。

```
(exchange, chain) -> {
    return chain.filter(exchange)
      .then(Mono.fromRunnable(() -> {
          ServerHttpResponse response = exchange.getResponse();

          Optional.ofNullable(exchange.getRequest()
            .getQueryParams()
            .getFirst("locale"))
            .ifPresent(qp -> {
                String responseContentLanguage = response.getHeaders()
                  .getContentLanguage()
                  .getLanguage();

                response.getHeaders()
                  .add("Bael-Custom-Language-Header", responseContentLanguage);
                });
        }));
}
```

我们可以很容易地获得对响应对象的引用，并且不需要像请求那样创建一个副本来修改它。

这是一个很好的例子，说明了过滤器在链中的顺序的重要性；如果我们在上一节创建的过滤器之后配置这个过滤器的执行，那么这里的`exchange `对象将包含一个对`ServerHttpRequest `的引用，这个引用将永远不会有任何查询参数。

在执行完所有“pre”过滤器之后，这一点也不重要，因为由于有了`mutate` 逻辑，我们仍然有对原始请求的引用。

### 5.3.将请求链接到其他服务

我们假设场景中的下一步是依靠第三个服务来指示我们应该使用哪个`Accept-Language` 头。

因此，我们将创建一个新的过滤器来调用这个服务，并使用它的响应体作为代理服务 API 的请求头。

在反应式环境中，这意味着链接请求以避免阻塞异步执行。

在我们的过滤器中，我们将首先向语言服务发出请求:

```
(exchange, chain) -> {
    return WebClient.create().get()
      .uri(config.getLanguageEndpoint())
      .exchange()
      // ...
}
```

请注意，我们正在返回这个流畅的操作，因为正如我们所说的，我们将把调用的输出与我们代理的请求链接起来。

下一步是提取语言——如果响应不成功，则从响应正文或配置中提取——并解析它:

```
// ...
.flatMap(response -> {
    return (response.statusCode()
      .is2xxSuccessful()) ? response.bodyToMono(String.class) : Mono.just(config.getDefaultLanguage());
}).map(LanguageRange::parse)
// ...
```

最后，我们将像以前一样将`LanguageRange` 值设置为请求头，并继续过滤链:

```
.map(range -> {
    exchange.getRequest()
      .mutate()
      .headers(h -> h.setAcceptLanguage(range))
      .build();

    return exchange;
}).flatMap(chain::filter);
```

就是这样，现在交互将以非阻塞的方式进行。

## 6。结论

既然我们已经学习了如何编写定制的 Spring 云网关过滤器，并了解了如何操作请求和响应实体，我们就可以充分利用这个框架了。

和往常一样，所有完整的例子都可以在 GitHub 的[中找到。请记住，为了测试它，我们需要通过](https://web.archive.org/web/20221202171428/https://github.com/eugenp/tutorials/tree/master/spring-cloud-modules/spring-cloud-gateway) [Maven](/web/20221202171428/https://www.baeldung.com/maven) 运行集成和现场测试。