# 与乌特 2 和 JWT 一起处理祖尔城的安全问题

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-security-zuul-oauth-jwt>

## 1.介绍

简单地说，微服务架构允许我们将系统和 API 分解成一组独立的服务，这些服务可以完全独立地部署。

虽然从持续部署和管理的角度来看，这很好，但当涉及到 API 可用性时，它会很快变得复杂。由于要管理不同的端点，依赖应用程序将需要管理 CORS(跨源资源共享)和一组不同的端点。

Zuul 是一种边缘服务，允许我们将传入的 HTTP 请求路由到多个后端微服务。首先，这对于为我们后端资源的消费者提供统一的 API 非常重要。

基本上，Zuul 允许我们通过坐在他们前面并充当代理来统一我们所有的服务。它接收所有请求并将它们路由到正确的服务。对于外部应用程序，我们的 API 表现为一个统一的 API 表面区域。

在本教程中，我们将讨论如何使用它来实现这个确切的目的，并结合使用 [OAuth 2.0 和 JWTs](/web/20220630130959/https://www.baeldung.com/spring-security-oauth-jwt) ，作为保护我们的 web 服务的第一线。具体来说，我们将使用[密码授权](https://web.archive.org/web/20220630130959/https://oauth.net/2/grant-types/password/)流来获得受保护资源的访问令牌。

一个快速但重要的注意事项是，我们仅使用密码授权流来探索一个简单的场景；大多数客户端更有可能在生产场景中使用授权流。

## 2.添加 Zuul Maven 依赖项

让我们从给我们的项目添加 Zuul 开始。我们通过添加 [`spring-cloud-starter-netflix-zuul`](https://web.archive.org/web/20220630130959/https://search.maven.org/search?q=g:org.springframework.cloud%20AND%20a:spring-cloud-starter-netflix-zuul) 工件来做到这一点:

```java
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-netflix-zuul</artifactId>
    <version>2.0.2.RELEASE</version>
</dependency> 
```

## 3.启用 Zuul

我们希望通过 Zuul 路由的应用程序包含一个授予访问令牌的 OAuth 2.0 授权服务器和一个接受令牌的资源服务器。这些服务位于两个独立的端点上。

我们希望这些服务的所有外部客户端有一个单一的端点，不同的路径分支到不同的物理端点。为此，我们将引入 Zuul 作为边缘服务。

为此，我们将创建一个新的 Spring Boot 应用程序，名为`GatewayApplication`。然后，我们将简单地用`@EnableZuulProxy`注释来修饰这个应用程序类，这将导致一个 Zuul 实例的产生:

```java
@EnableZuulProxy
@SpringBootApplication
public class GatewayApplication {
    public static void main(String[] args) {
	SpringApplication.run(GatewayApplication.class, args);
    }
} 
```

## 4.配置 Zuul 路线

在我们继续之前，我们需要配置一些 Zuul 属性。我们首先要配置的是 Zuul 监听传入连接的端口。这需要放到`/src/main/resources/application.yml`文件中:

```java
server:
    port: 8080 
```

现在有趣的是，配置 Zuul 将转发到的实际路由。为此，我们需要注意以下服务、它们的路径以及它们监听的端口。

**授权服务器部署在:**[http://localhost:8081/spring-security-oauth-Server/oauth](https://web.archive.org/web/20220630130959/http://localhost:8081/spring-security-oauth-server/oauth)

**资源服务器部署在:**[http://localhost:8082/spring-security-oauth-Resource](https://web.archive.org/web/20220630130959/http://localhost:8082/spring-security-oauth-resource)

授权服务器是 OAuth 身份提供者。它的存在是为了向资源服务器提供授权令牌，资源服务器反过来提供一些受保护的端点。

授权服务器向客户端提供一个访问令牌，然后客户端使用该令牌代表资源所有者对资源服务器执行请求。快速浏览一下 [OAuth 术语](https://web.archive.org/web/20220630130959/https://oauth2.thephpleague.com/terminology/)将有助于我们理解这些概念。

现在让我们将一些路线映射到这些服务中的每一项:

```java
zuul:
  routes:
    spring-security-oauth-resource:
      path: /spring-security-oauth-resource/**
      url: http://localhost:8082/spring-security-oauth-resource
    oauth:
      path: /oauth/**
      url: http://localhost:8081/spring-security-oauth-server/oauth 
```

此时，任何在`localhost:8080/oauth/**`到达 Zuul 的请求都将被路由到在端口 8081 上运行的授权服务。任何对`localhost:8080/spring-security-oauth-resource/**`的请求都将被路由到运行在 8082 上的资源服务器。

## 5.保护 Zuul 外部通信路径

尽管我们的 Zuul edge 服务现在可以正确地路由请求，但它是在没有任何授权检查的情况下这样做的。位于`/oauth/*`之后的授权服务器为每次成功的认证创建一个 JWT。很自然，它可以匿名访问。

另一方面，位于`/spring-security-oauth-resource/**`的资源服务器应该始终通过 JWT 进行访问，以确保授权的客户端正在访问受保护的资源。

首先，我们将配置 Zuul 通过 JWT 到达位于其后的服务。在我们的例子中，这些服务本身需要验证令牌。

我们通过添加`sensitiveHeaders: Cookie,Set-Cookie`来做到这一点。

这就完成了我们的 Zuul 配置:

```java
server:
  port: 8080
zuul:
  sensitiveHeaders: Cookie,Set-Cookie
  routes:
    spring-security-oauth-resource:
      path: /spring-security-oauth-resource/**
      url: http://localhost:8082/spring-security-oauth-resource
    oauth:
      path: /oauth/**
      url: http://localhost:8081/spring-security-oauth-server/oauth 
```

在我们解决了这个问题之后，我们需要处理边缘的授权。现在，Zuul 在将 JWT 传递给我们的下游服务之前不会验证它。这些服务将验证 JWT 本身，但理想情况下，我们希望让边缘服务先验证，并在任何未经授权的请求深入传播到我们的架构之前拒绝它们。

让我们设置 Spring Security 来确保在 Zuul 中检查授权。

首先，我们需要将 Spring 安全依赖项引入我们的项目。我们想要 [spring-security-oauth2](https://web.archive.org/web/20220630130959/https://mvnrepository.com/artifact/org.springframework.security.oauth/spring-security-oauth2) 和 [spring-security-jwt:](https://web.archive.org/web/20220630130959/https://mvnrepository.com/artifact/org.springframework.security/spring-security-jwt)

```java
<dependency>
    <groupId>org.springframework.security.oauth</groupId>
    <artifactId>spring-security-oauth2</artifactId>
    <version>2.3.3.RELEASE</version>
</dependency>
<dependency>
    <groupId>org.springframework.security</groupId>
    <artifactId>spring-security-jwt</artifactId>
    <version>1.0.9.RELEASE</version>
</dependency> 
```

现在让我们通过扩展`ResourceServerConfigurerAdapter:`为我们想要保护的路由编写一个配置

```java
@Configuration
@Configuration
@EnableResourceServer
public class GatewayConfiguration extends ResourceServerConfigurerAdapter {
    @Override
    public void configure(final HttpSecurity http) throws Exception {
	http.authorizeRequests()
          .antMatchers("/oauth/**")
          .permitAll()
          .antMatchers("/**")
	  .authenticated();
    }
} 
```

`GatewayConfiguration`类定义了 Spring Security 应该如何处理通过 Zuul 传入的 HTTP 请求。在`configure`方法中，我们首先使用`antMatchers`匹配最严格的路径，然后通过`permitAll`允许匿名访问。

也就是说，所有进入`/oauth/**`的请求都应该被允许通过，而不检查任何授权令牌。这是有意义的，因为这是生成授权令牌的路径。

接下来，我们匹配了`all other paths with /**`，并通过对`authenticated`的调用坚持所有其他调用都应该包含访问令牌。

## 6.配置用于 JWT 验证的密钥

现在配置已经就绪，所有路由到`/oauth/**`路径的请求都将被允许匿名通过，而所有其他请求都需要认证。

但是，这里我们遗漏了一件事，那就是验证 JWT 有效所需的实际秘密。为此，我们需要提供用于签署 JWT 的密钥(在本例中是对称的)。我们可以使用 [`spring-security-oauth2-autoconfigure`](https://web.archive.org/web/20220630130959/https://mvnrepository.com/artifact/org.springframework.security.oauth.boot/spring-security-oauth2-autoconfigure) ，而不是手工编写配置代码。

让我们从将工件添加到我们的项目开始:

```java
<dependency>
    <groupId>org.springframework.security.oauth.boot</groupId>
    <artifactId>spring-security-oauth2-autoconfigure</artifactId>
    <version>2.1.2.RELEASE</version>
</dependency>
```

接下来，我们需要在我们的`application.yaml`文件中添加几行配置来定义用于签署 JWT 的密钥:

```java
security:
  oauth2:
    resource:
      jwt:
        key-value: 123 
```

第`key-value: 123`行设置授权服务器用来签署 JWT 的对称密钥。这个密钥将被`spring-security-oauth2-autoconfigure`用来配置令牌解析。

需要注意的是，在生产系统中，我们不应该使用应用程序源代码中指定的对称密钥。那自然需要外部配置。

## 7.测试边缘服务

### 7.1.获取访问令牌

现在让我们用几个 curl 命令来测试我们的 Zuul edge 服务的行为。

首先，我们将看看如何使用[密码授权](https://web.archive.org/web/20220630130959/https://oauth.net/2/grant-types/password/)从授权服务器获得新的 JWT。

这里我们用一个**用户名和密码交换一个访问令牌**。在这种情况下，我们使用'`john`'作为用户名，使用'`123`'作为密码:

```java
curl -X POST \
  http://localhost:8080/oauth/token \
  -H 'Authorization: Basic Zm9vQ2xpZW50SWRQYXNzd29yZDpzZWNyZXQ=' \
  -H 'Cache-Control: no-cache' \
  -H 'Content-Type: application/x-www-form-urlencoded' \
  -d 'grant_type=password&password;=123&username;=john' 
```

这个调用产生一个 JWT 令牌，然后我们可以使用它对我们的资源服务器发出经过验证的请求。

注意`“Authorization: Basic…”`标题字段。这是为了告诉授权服务器哪个客户端正在连接到它。

对于客户机(在本例中是 cURL 请求)来说，用户名和密码就像用户一样:

```java
{    
    "access_token":"eyJhbGciOiJIUzI1NiIsInR5cCI6IkpX...",
    "token_type":"bearer",    
    "refresh_token":"eyJhbGciOiJIUzI1NiIsInR5cCI6IkpX...",
    "expires_in":3599,
    "scope":"foo read write",
    "organization":"johnwKfc",
    "jti":"8e2c56d3-3e2e-4140-b120-832783b7374b"
} 
```

### 7.2.测试资源服务器请求

然后，我们可以使用从授权服务器检索到的 JWT 对资源服务器执行查询:

```java
curl -X GET \
curl -X GET \
  http:/localhost:8080/spring-security-oauth-resource/users/extra \
  -H 'Accept: application/json, text/plain, */*' \
  -H 'Accept-Encoding: gzip, deflate' \
  -H 'Accept-Language: en-US,en;q=0.9' \
  -H 'Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXV...' \
  -H 'Cache-Control: no-cache' \ 
```

Zuul edge 服务现在将在路由到资源服务器之前验证 JWT。

然后，它从 JWT 中提取关键字段，并在响应请求之前检查更细粒度的授权:

```java
{
    "user_name":"john",
    "scope":["foo","read","write"],
    "organization":"johnwKfc",
    "exp":1544584758,
    "authorities":["ROLE_USER"],
    "jti":"8e2c56d3-3e2e-4140-b120-832783b7374b",
    "client_id":"fooClientIdPassword"
} 
```

## 8.跨层安全性

值得注意的是，JWT 在被传递到资源服务器之前，将由 Zuul edge 服务进行验证。如果 JWT 无效，则请求将在边缘服务边界被拒绝。

另一方面，如果 JWT 确实有效，则请求被传递到下游。然后，资源服务器再次验证 JWT，并提取关键字段，如用户范围、组织(在这种情况下是一个定制字段)和权限。它使用这些字段来决定用户能做什么和不能做什么。

需要说明的是，在许多架构中，我们实际上不需要验证 JWT 两次——这是您必须根据您的流量模式做出的决定。

例如，在一些生产项目中，可以直接访问单独的资源服务器，也可以通过代理访问——我们可能希望在两个地方都验证令牌。在其他项目中，流量可能只通过代理，在这种情况下，验证令牌就足够了。

## 9.摘要

正如我们所见，Zuul 提供了一种简单、可配置的方式来抽象和定义服务的路由。与 Spring Security 一起，它允许我们在服务边界授权请求。

最后，和往常一样，代码可以在 Github 上[获得。](https://web.archive.org/web/20220630130959/https://github.com/Baeldung/spring-security-oauth/tree/master/oauth-legacy/oauth-zuul-gateway)