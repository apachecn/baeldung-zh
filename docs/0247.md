# Spring 云网关中客户端 IP 的速率限制

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-cloud-gateway-rate-limit-by-client-ip>

## 1.介绍

在这个快速教程中，我们将看到如何基于客户端的实际 IP 地址来限制我们的 [Spring Cloud Gateway](/web/20221209212526/https://www.baeldung.com/spring-cloud-gateway) 的传入请求率。

简而言之，我们将在路由上设置`RequestRateLimiter`过滤器，然后**我们将配置网关使用 IP 地址来限制唯一客户端**的请求。

## 2.路线配置

首先，我们需要配置 Spring Cloud Gateway 来限制特定路由的速率。为此，我们将使用由 [`spring-boot-starter-data-redis-reactive`](/web/20221209212526/https://www.baeldung.com/spring-data-redis-reactive) 实现的经典[令牌桶](https://web.archive.org/web/20221209212526/https://en.wikipedia.org/wiki/Token_bucket)速率限制器。简而言之，**速率限制器创建一个桶，该桶具有标识其自身的相关密钥和随着时间推移而得到补充的令牌的固定初始容量**。然后，对于每个请求，速率限制器检查其相关的桶，并在可能的情况下减少令牌。否则，它会拒绝传入的请求。

当我们使用分布式系统时，我们可能希望跟踪应用程序所有实例中的所有传入请求。出于这个原因，拥有一个分布式缓存系统可以方便地存储 bucket 的信息。在这种情况下，我们[预先配置了一个 Redis 实例](/web/20221209212526/https://www.baeldung.com/spring-data-redis-properties)来模拟真实世界的应用程序。

接下来，我们将配置一条带有速率限制器的路由。我们将监听`/example` 端点，并将请求转发给`http://example.org`:

```
@Bean
public RouteLocator myRoutes(RouteLocatorBuilder builder) {
    return builder.routes()
        .route("requestratelimiter_route", p -> p
            .path("/example")
            .filters(f -> f.requestRateLimiter(r -> r.setRateLimiter(redisRateLimiter())))
            .uri("http://example.org"))
        .build();
}
```

上面，我们通过使用`.setRateLimiter()`方法用一个`RequestRateLimiter`来配置路由。特别是，我们通过方法`redisRatelimiter()`定义了`RedisRateLimiter` bean 来管理速率限制器的状态:

```
@Bean
public RedisRateLimiter redisRateLimiter() {
    return new RedisRateLimiter(1, 1, 1);
}
```

举例来说，我们将所有的`replenishRate`、`burstCapacity`和`requestedToken`属性设置为 1 来配置速率限制。这使得多次调用`/example`端点和获取 HTTP 429 响应代码变得容易。

## 3.`KeyResolver`比恩

为了正确工作，**速率限制器必须通过密钥**识别每个到达端点的客户端。在下面，关键字标识了速率限制器将用于为每个请求消耗令牌的桶。因此，我们希望每个客户端的密钥都是唯一的。在这种情况下，我们将使用客户端的 IP 地址来监控他们的请求，并在他们发出太多请求时限制他们。

因此，我们之前配置的`RequestRateLimiter`将使用一个`KeyResolver` bean，该 bean 允许可插拔策略来获取限制请求的密钥。这意味着**我们可以配置如何从每个请求中提取密钥**。

## 4.`KeyResolver`中客户端的 IP 地址

目前，这个接口没有默认的实现，所以我们必须定义一个，记住我们需要客户端的 IP 地址:

```
@Component
public class SimpleClientAddressResolver implements KeyResolver {
    @Override
    public Mono<String> resolve(ServerWebExchange exchange) {
        return Optional.ofNullable(exchange.getRequest().getRemoteAddress())
            .map(InetSocketAddress::getAddress)
            .map(InetAddress::getHostAddress)
            .map(Mono::just)
            .orElse(Mono.empty());
    }
}
```

我们使用`ServerWebExchange`对象提取客户端的 IP 地址。如果我们不能获得 IP 地址，我们将返回`Mono.empty()`向速率限制器发出信号，并默认拒绝该请求。然而，我们可以通过将`.setDenyEmptyKey()`设置为`false`来配置速率限制器，以便在`KeyResolver`返回空键时允许请求。此外，通过为`.setKeyResolver()`方法提供一个定制的`KeyResolver`实现，我们还可以为每个不同的路由提供不同的`KeyResolver`:

```
builder.routes()
    .route("ipaddress_route", p -> p
        .path("/example2")
        .filters(f -> f.requestRateLimiter(r -> r.setRateLimiter(redisRateLimiter())
            .setDenyEmptyKey(false)
            .setKeyResolver(new SimpleClientAddressResolver())))
        .uri("http://example.org"))
.build();
```

### 4.1.位于代理后的始发 IP 地址

如果 Spring Cloud Gateway 直接监听客户机的请求，前面定义的实现就可以工作。但是，如果我们在代理后面部署应用程序，所有主机地址都将是相同的。因此，速率限制器会将所有请求视为来自同一个客户端，并限制它可以处理的请求数量。

为了解决这个问题，**我们依靠`X-Forwarded-For`报头来识别通过代理服务器**连接的客户机的原始 IP 地址。例如，让我们配置`KeyResolver`,以便它可以读取起始 IP 地址:

```
@Primary
@Component
public class ProxiedClientAddressResolver implements KeyResolver {
    @Override
    public Mono<String> resolve(ServerWebExchange exchange) {
        XForwardedRemoteAddressResolver resolver = XForwardedRemoteAddressResolver.maxTrustedIndex(1);
        InetSocketAddress inetSocketAddress = resolver.resolve(exchange);
        return Mono.just(inetSocketAddress.getAddress().getHostAddress());
    }
}
```

我们将值 1 传递给`maxTrustedIndex()`，假设我们只有一个代理服务器。否则，必须相应地设置该值。此外，我们用 [`@Primary`](/web/20221209212526/https://www.baeldung.com/spring-primary) 对这个`KeyResolver`进行注释，使其优先于前面的实现。

## 5.结论

在本文中，我们基于客户机的 IP 地址配置了一个 API 速率限制器。首先，我们配置了一个带有令牌桶速率限制器的路由。然后，我们研究了`KeyResolver`如何识别每个请求使用的桶。最后，我们探索了当直接访问我们的 API 或者当它部署在代理后面时，通过`KeyResolver`分配客户端 IP 地址的策略。

这些例子的实现可以在 GitHub 的[中找到。](https://web.archive.org/web/20221209212526/https://github.com/eugenp/tutorials/tree/master/spring-cloud-modules/spring-cloud-gateway)