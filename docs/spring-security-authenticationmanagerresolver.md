# Spring 安全中的 AuthenticationManagerResolver 指南

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-security-authenticationmanagerresolver>

## 1.介绍

在本教程中，我们将介绍`AuthenticationManagerResolver`，然后展示如何将它用于基本和 OAuth2 认证流。

## 2.什么是`AuthenticationManager`？

简单来说， [`AuthenticationManager`](https://web.archive.org/web/20221206111531/https://spring.io/guides/topicals/spring-security-architecture) 是认证的主要策略界面。

如果输入认证的主体是有效的并且经过验证，`AuthenticationManager#authenticate`返回一个`Authentication`实例，其中`authenticated`标志被设置为`true`。否则，如果主体无效，它将抛出一个`AuthenticationException`。对于最后一种情况，如果不能决定，它返回`null`。

`ProviderManager`是`AuthenticationManager`的默认实现。它将认证过程委托给一系列`AuthenticationProvider`实例。

如果我们创建一个 [`SecurityFilterChain`](/web/20221206111531/https://www.baeldung.com/spring-security-multiple-auth-providers) bean，我们可以设置全局或本地`AuthenticationManager`。对于本地的`AuthenticationManager`，我们可以创建一个`AuthenticationManager` bean *，通过 *`HttpSecurity`* 访问`AuthenticationManagerBuilder`* 。

[`AuthenticationManagerBuilder`](/web/20221206111531/https://www.baeldung.com/java-config-spring-security) 是一个助手类，它简化了`UserDetailService`、`AuthenticationProvider`和其他依赖项的设置，以构建一个`AuthenticationManager`。

对于一个全局`AuthenticationManager`，我们应该将一个`AuthenticationManager`定义为一个 bean。

## 3.为什么是`AuthenticationManagerResolver`？

`AuthenticationManagerResolver`让 Spring 为每个上下文选择一个`AuthenticationManager`。这是在 5.2.0 版本中添加到 Spring Security 的一个[新特性](https://web.archive.org/web/20221206111531/https://github.com/spring-projects/spring-security/issues/6722):

```java
public interface AuthenticationManagerResolver<C> {
    AuthenticationManager resolve(C context);
}
```

`AuthenticationManagerResolver#resolve`可以基于通用上下文返回`AuthenticationManager`的实例。换句话说，如果我们想根据它解析`AuthenticationManager`，我们可以设置一个类作为上下文。

**Spring Security 将`AuthenticationManagerResolver`集成到了以`HttpServletRequest`和`ServerWebExchange`为上下文的认证流程中。**

## 4.使用场景

让我们看看如何在实践中使用`AuthenticationManagerResolver`。

例如，假设一个系统有两组用户:雇员和客户。这两个组有特定的身份验证逻辑，并有独立的数据存储。此外，这两个组中的任何一个组的用户都只允许调用其相关的 URL。

## 5.`AuthenticationManagerResolver`是如何工作的？

我们可以在任何需要动态选择`AuthenticationManager`的地方使用`AuthenticationManagerResolver`，但是在本教程中，我们感兴趣的是在内置认证流中使用它。

首先，让我们设置一个`AuthenticationManagerResolver`，然后使用它进行基本认证和 OAuth2 认证。

### 5.1.设置`AuthenticationManagerResolver`

让我们从创建一个安全配置类开始。

```java
@Configuration
public class CustomWebSecurityConfigurer {
    // ...
}
```

然后，让我们添加一个为客户返回`AuthenticationManager`的方法:

```java
AuthenticationManager customersAuthenticationManager() {
    return authentication -> {
        if (isCustomer(authentication)) {
            return new UsernamePasswordAuthenticationToken(/*credentials*/);
        }
        throw new UsernameNotFoundException(/*principal name*/);
    };
}
```

员工的`AuthenticationManager`在逻辑上是相同的，只是我们将`isCustomer`替换为`isEmployee`:

```java
public AuthenticationManager employeesAuthenticationManager() {
    return authentication -> {
        if (isEmployee(authentication)) {
            return new UsernamePasswordAuthenticationToken(/*credentials*/);
        }
        throw new UsernameNotFoundException(/*principal name*/);
    };
}
```

最后，让我们添加一个根据请求的 URL 进行解析的`AuthenticationManagerResolver`:

```java
AuthenticationManagerResolver<HttpServletRequest> resolver() {
    return request -> {
        if (request.getPathInfo().startsWith("/employee")) {
            return employeesAuthenticationManager();
        }
        return customersAuthenticationManager();
    };
}
```

### 5.2.对于基本身份验证

我们可以使用`AuthenticationFilter`来动态解析每个请求的`AuthenticationManager`。`AuthenticationFilter`在 5.2 版本中加入了 Spring Security。

如果我们将它添加到我们的安全过滤器链中，那么对于每个匹配的请求，它首先检查是否可以提取任何身份验证对象。如果是，则它向`AuthenticationManagerResolver`请求合适的`AuthenticationManager`并继续流程。

首先，让我们在`CustomWebSecurityConfigurer`中添加一个方法来创建一个`AuthenticationFilter`:

```java
private AuthenticationFilter authenticationFilter() {
    AuthenticationFilter filter = new AuthenticationFilter(
      resolver(), authenticationConverter());
    filter.setSuccessHandler((request, response, auth) -> {});
    return filter;
}
```

**用 no-op `SuccessHandler`设置`AuthenticationFilter#successHandler`的原因是为了防止认证成功后重定向的默认行为。**

然后，我们可以通过在我们的 `CustomWebSecurityConfigurer`中创建一个`SecurityFilterChain` bean 来将这个过滤器添加到我们的安全过滤器链中:

```java
@Bean
public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
    http.addFilterBefore(authenticationFilter(), BasicAuthenticationFilter.class);
    return http.build();
}
```

### 5.3.对于 OAuth2 身份验证

`BearerTokenAuthenticationFilter`负责 OAuth2 认证。`BearerTokenAuthenticationFilter#doFilterInternal`方法检查请求中的`BearerTokenAuthenticationToken`，如果它可用，那么它解析适当的`AuthenticationManager`来认证令牌。

`OAuth2ResourceServerConfigurer`用于设置`BearerTokenAuthenticationFilter.`

因此，我们可以通过创建一个`SecurityFilterChain` bean 在我们的 `CustomWebSecurityConfigurer`中为我们的资源服务器设置`AuthenticationManagerResolver`:

```java
@Bean
public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
    http
      .oauth2ResourceServer()
      .authenticationManagerResolver(resolver());
    return http.build();
}
```

## 6.在反应式应用中解决`AuthenticationManager`

对于一个反应式 web 应用程序，我们仍然可以从根据上下文解析`AuthenticationManager`的概念中受益。但是这里我们用`ReactiveAuthenticationManagerResolver`代替:

```java
@FunctionalInterface
public interface ReactiveAuthenticationManagerResolver<C> {
    Mono<ReactiveAuthenticationManager> resolve(C context);
}
```

它返回一个`ReactiveAuthenticationManager`的`Mono`。`ReactiveAuthenticationManager`是`AuthenticationManager`的无功等价物，因此其`authenticate`方法返回`Mono`。

### 6.1.设置`ReactiveAuthenticationManagerResolver`

让我们首先为安全配置创建一个类:

```java
@EnableWebFluxSecurity
@EnableReactiveMethodSecurity
public class CustomWebSecurityConfig {
    // ...
}
```

接下来，让我们为该类客户定义`ReactiveAuthenticationManager`:

```java
ReactiveAuthenticationManager customersAuthenticationManager() {
    return authentication -> customer(authentication)
      .switchIfEmpty(Mono.error(new UsernameNotFoundException(/*principal name*/)))
      .map(b -> new UsernamePasswordAuthenticationToken(/*credentials*/));
} 
```

之后，我们将为员工定义`ReactiveAuthenticationManager`:

```java
public ReactiveAuthenticationManager employeesAuthenticationManager() {
    return authentication -> employee(authentication)
      .switchIfEmpty(Mono.error(new UsernameNotFoundException(/*principal name*/)))
      .map(b -> new UsernamePasswordAuthenticationToken(/*credentials*/));
}
```

最后，我们基于我们的场景设置了一个`ReactiveAuthenticationManagerResolver`:

```java
ReactiveAuthenticationManagerResolver<ServerWebExchange> resolver() {
    return exchange -> {
        if (match(exchange.getRequest(), "/employee")) {
            return Mono.just(employeesAuthenticationManager());
        }
        return Mono.just(customersAuthenticationManager());
    };
}
```

### 6.2.对于基本身份验证

在反应式 web 应用程序中，我们可以使用`AuthenticationWebFilter`进行认证。它验证请求并填充安全上下文。

`AuthenticationWebFilter`首先检查请求是否匹配。之后，如果请求中有一个认证对象，它从`ReactiveAuthenticationManagerResolver`获取适合请求的`ReactiveAuthenticationManager`,并继续认证流程。

因此，我们可以在安全配置中设置自定义的`AuthenticationWebFilter`:

```java
@Bean
public SecurityWebFilterChain securityWebFilterChain(ServerHttpSecurity http) {
    return http
      .authorizeExchange()
      .pathMatchers("/**")
      .authenticated()
      .and()
      .httpBasic()
      .disable()
      .addFilterAfter(
        new AuthenticationWebFilter(resolver()), 
        SecurityWebFiltersOrder.REACTOR_CONTEXT
      )
      .build();
}
```

**首先，我们禁用`ServerHttpSecurity#httpBasic`来阻止正常的认证流程，**然后手动用一个`AuthenticationWebFilter`来替换它，传入我们的自定义解析器。

### 6.3.对于 OAuth2 身份验证

**我们可以用`ServerHttpSecurity#oauth2ResourceServer`来配置`ReactiveAuthenticationManagerResolver`。** `ServerHttpSecurity#build`用我们的解析器将`AuthenticationWebFilter`的一个实例添加到安全过滤器链中。

因此，让我们在安全配置中为 OAuth2 身份验证过滤器设置我们的`AuthenticationManagerResolver`:

```java
@Bean
public SecurityWebFilterChain securityWebFilterChain(ServerHttpSecurity http) {
    return http
      // ...
      .and()
      .oauth2ResourceServer()
      .authenticationManagerResolver(resolver())
      .and()
      // ...;
}
```

## 7.结论

在本文中，我们在一个简单的场景中使用了`AuthenticationManagerResolver`进行基本认证和 OAuth2 认证。

我们还探索了在反应式 Spring web 应用程序中使用`ReactiveAuthenticationManagerResolver`进行基本认证和 OAuth2 认证。

和往常一样，源代码可以在 GitHub 上的[处获得。我们的反应例子也可以在 GitHub](https://web.archive.org/web/20221206111531/https://github.com/eugenp/tutorials/tree/master/spring-security-modules/spring-security-core-2) 的[上找到。](https://web.archive.org/web/20221206111531/https://github.com/eugenp/tutorials/tree/master/spring-reactive-modules/spring-5-reactive-security)