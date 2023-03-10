# 通过 OAuth 2.0 模式使用 Spring Cloud Gateway

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-cloud-gateway-oauth2>

## 1.介绍

Spring Cloud Gateway 是一个库，它允许我们快速创建基于 Spring Boot 的轻量级 API 网关，我们在之前的文章中已经介绍过了。

**这一次，我们将展示如何在其基础上快速实现 OAuth 2.0 模式**。

## 2.OAuth 2.0 快速回顾

OAuth 2.0 标准是一个成熟的标准，在整个互联网上作为一种安全机制使用，用户和应用程序可以通过它安全地访问资源。

尽管详细描述这一标准超出了本文的范围，但让我们先快速回顾一下几个关键术语:

*   `Resource`:任何只能由授权客户检索的信息
*   通常通过 REST API 消耗资源的应用程序
*   `Resource Server`:负责向授权客户提供资源的服务
*   `Resource Owner`:拥有资源的实体(人或应用程序),并最终负责向客户机授予对资源的访问权
*   `Token`:客户端获得的一条信息，作为认证请求的一部分发送给资源服务器
*   `Identity Provider (IdP)`:验证用户凭证并向客户端颁发访问令牌。
*   `Authentication Flow:`客户端获取有效令牌必须经历的一系列步骤。

对于该标准的全面描述，一个很好的起点是 Auth0 的关于这个主题的文档。

## 3.OAuth 2.0 模式

Spring Cloud Gateway 主要用于以下角色之一:

*   `OAuth Client`
*   `OAuth Resource Server`

让我们更详细地讨论每一种情况。

### 3.1.作为 OAuth 2.0 客户端的 Spring Cloud Gateway

**在这个场景中，任何未经认证的传入请求都将启动一个授权代码流**。网关获取令牌后，将在向后端服务发送请求时使用该令牌:

[![OAuth 2.0 Authorization Code Flow](img/7991bd84789033f4449e7301190e0c95.png)](/web/20220903190824/https://www.baeldung.com/wp-content/uploads/2022/01/oauth2_authorization_flow.png)

这种模式的一个很好的例子是社交网络提要聚合器应用程序:对于每个受支持的网络，网关将充当 OAuth 2.0 客户端。

因此，前端——通常是用 Angular、React 或类似 UI 框架构建的 SPA 应用——可以代表最终用户无缝访问这些网络上的数据。**更重要的是:它可以做到这一点，而无需用户向聚合器透露他们的凭证**。

### 3.2.作为 OAuth 2.0 资源服务器的 Spring Cloud Gateway

这里，网关充当看门人的角色，强制每个请求在发送到后端服务之前都有一个有效的访问令牌。此外，它还可以根据关联的范围检查令牌是否具有访问给定资源的适当权限:

[![Spring Gateway Resource Server](img/b60afae9db8c7739c1c2b216dad332b1.png)](/web/20220903190824/https://www.baeldung.com/wp-content/uploads/2022/01/spring_gateway_resource_server.png)

需要注意的是，这种权限检查主要是在粗略的级别上进行的。细粒度的访问控制(例如，对象/字段级权限)通常使用域逻辑在后端实现。在这个模式中要考虑的一件事是后端服务如何认证和授权任何转发的请求。主要有两种情况:

*   `Token propagation` : API 网关将收到的令牌按原样转发给后端
*   `Token replacement` : API Gateway 在发送请求之前用另一个令牌替换传入的令牌。

在本教程中，我们将只讨论令牌传播的情况，因为这是最常见的场景。第二种也是可能的，但需要额外的设置和编码，这会分散我们对这里要展示的要点的注意力。

## 4.示例项目概述

为了展示如何将 Spring Gateway 与我们到目前为止描述的 OAuth 模式一起使用，让我们构建一个示例项目，该项目公开一个端点:`/quotes/{symbol}`。**访问此端点需要由已配置的身份提供者颁发的有效访问令牌。**

在我们的例子中，我们将使用[嵌入式 Keycloak 身份提供者](/web/20220903190824/https://www.baeldung.com/keycloak-embedded-in-spring-boot-app)。唯一需要的更改是添加一个新的客户端应用程序和一些测试用户。

为了让事情变得有趣一点，我们的后端服务将根据与请求相关联的用户返回不同的报价。拥有黄金角色的用户获得较低的价格，而其他人获得普通价格(毕竟，生活是不公平的；^)).

我们将用 Spring Cloud Gateway 作为这项服务的前端，只需更改几行配置，我们就可以将其角色从 OAuth 客户机切换到资源服务器。

## 5.项目设置

### 5.1.Keycloak IdP

我们将在本教程中使用的嵌入式 Keycloak 只是一个普通的 SpringBoot 应用程序，我们可以从 [GitHub](https://web.archive.org/web/20220903190824/https://github.com/Baeldung/spring-security-oauth) 中克隆并使用 Maven 构建它:

```java
$ git clone https://github.com/Baeldung/spring-security-oauth
$ cd oauth-rest/oauth-authorization/server
$ mvn install
```

注意:这个项目目前以 Java 13+为目标，但是也可以在 Java 11 上构建和运行。我们只需要给 Maven 的命令加上`-Djava.version=11`。

接下来，我们将把[的`src/main/resources/baeldung-domain.json`换成这个](https://web.archive.org/web/20220903190824/https://raw.githubusercontent.com/eugenp/tutorials/master/spring-cloud/spring-cloud-gateway/baeldung-realm.json)。修改后的版本具有与原始版本相同的配置，外加一个额外的客户端应用程序(`quotes-client`)、两个用户组(`golden_`和`silver_customers`)以及两个角色(`gold`和`silver`)。

我们现在可以使用`spring-boot:run` maven 插件启动服务器:

```java
$ mvn spring-boot:run
... many, many log messages omitted
2022-01-16 10:23:20.318
  INFO 8108 --- [           main] c.baeldung.auth.AuthorizationServerApp   : Started AuthorizationServerApp in 23.815 seconds (JVM running for 24.488)
2022-01-16 10:23:20.334
  INFO 8108 --- [           main] c.baeldung.auth.AuthorizationServerApp   : Embedded Keycloak started: http://localhost:8083/auth to use keycloak
```

一旦服务器启动，我们可以通过将浏览器指向`http://localhost:8083/auth/admin/master/console/#/realms/baeldung`来访问它。一旦我们使用管理员的凭证(`bael-admin/pass`)登录，我们将得到该领域的管理屏幕:

[![Keycloak Baeldung Realm Administration Screen](img/b90553654f7f7767965f1995dd1ffa8c.png)](/web/20220903190824/https://www.baeldung.com/wp-content/uploads/2022/01/keycloak-baeldung-realm.png)

为了完成 IdP 设置，让我们添加几个用户。第一个是 Maxwell Smart，`golden_customer`组的成员。第二个是约翰·斯诺，我们不会把他加到任何组里。

**使用提供的配置，`golden_customers`组的成员将自动承担`gold`角色。**

### 5.2.后端服务

报价后端需要常规的 Spring Boot 反应式 MVC 依赖关系，加上[资源服务器启动器依赖关系](https://web.archive.org/web/20220903190824/https://search.maven.org/search?q=spring-boot-starter-oauth2-resource-server):

```java
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-oauth2-resource-server</artifactId>
    <version>2.6.2</version>
</dependency> 
```

请注意，我们有意省略了依赖项的版本。当使用 SpringBoot 的父 POM 或依赖管理部分中相应的 BOM 时，这是推荐的做法。

在主应用程序类中，我们必须使用`@EnableWebFluxSecurity`启用 web flux 安全性:

```java
@SpringBootApplication
@EnableWebFluxSecurity
public class QuotesApplication {    
    public static void main(String[] args) {
        SpringApplication.run(QuotesApplication.class);
    }
}
```

端点实现使用提供的`BearerAuthenticationToken`来检查当前用户是否拥有`gold`角色:

```java
@RestController
public class QuoteApi {
    private static final GrantedAuthority GOLD_CUSTOMER = new SimpleGrantedAuthority("gold");

    @GetMapping("/quotes/{symbol}")
    public Mono<Quote> getQuote(@PathVariable("symbol") String symbol,
      BearerTokenAuthentication auth ) {

        Quote q = new Quote();
        q.setSymbol(symbol);        
        if ( auth.getAuthorities().contains(GOLD_CUSTOMER)) {
            q.setPrice(10.0);
        }
        else {
            q.setPrice(12.0);
        }
        return Mono.just(q);
    }
} 
```

现在，Spring 是如何获得用户角色的？毕竟这不是像`scopes`或者`email`那样的标准索赔。**事实上，这里没有魔法:我们必须提供一个自定义`ReactiveOpaqueTokenIntrospection`，从 Keycloak** 返回的自定义字段中提取那些角色。这个 bean 可以在网上获得，基本上与 Spring 的关于这个主题的[文档中显示的相同，只是针对我们的定制字段做了一些小的改动。](https://web.archive.org/web/20220903190824/https://docs.spring.io/spring-security/reference/servlet/oauth2/resource-server/opaque-token.html#oauth2resourceserver-opaque-authorization-extraction)

我们还必须提供访问身份提供者所需的配置属性:

```java
spring.security.oauth2.resourceserver.opaquetoken.introspection-uri=http://localhost:8083/auth/realms/baeldung/protocol/openid-connect/token/introspect
spring.security.oauth2.resourceserver.opaquetoken.client-id=quotes-client
spring.security.oauth2.resourceserver.opaquetoken.client-secret=<CLIENT SECRET> 
```

最后，要运行我们的应用程序，我们可以在 IDE 中导入它，也可以从 Maven 中运行它。项目的 POM 包含一个用于此目的的概要:

```java
$ mvn spring-boot:run -Pquotes-application
```

应用程序现在将准备好在`http://localhost:8085/quotes`为请求提供服务。我们可以使用`curl`检查它是否正在响应:

```java
$ curl -v http://localhost:8085/quotes/BAEL
```

**不出所料，我们得到了一个`401 Unauthorized`响应，因为没有发送`Authorization`报头。**

## 6.作为 OAuth 2.0 资源服务器的 Spring Gateway

保护一个充当资源服务器的 Spring Cloud Gateway 应用程序与普通的资源服务没有什么不同。因此，毫不奇怪，我们必须添加与后端服务相同的启动器依赖项:

```java
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-gateway</artifactId>
    <version>3.1.0</version>
</dependency>
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-oauth2-resource-server</artifactId>
    <version>2.6.2</version>
</dependency> 
```

相应地，我们还必须将`@EnableWebFluxSecurity`添加到我们的启动类中:

```java
@SpringBootApplication
@EnableWebFluxSecurity
public class ResourceServerGatewayApplication {
    public static void main(String[] args) {
        SpringApplication.run(ResourceServerGatewayApplication.class,args);
    }
} 
```

与安全相关的配置属性与后端中使用的相同:

```java
spring:
  security:
    oauth2:
      resourceserver:
        opaquetoken:
          introspection-uri: http://localhost:8083/auth/realms/baeldung/protocol/openid-connect/token/introspect
          client-id: quotes-client
          client-secret: <code class="language-css"><CLIENT SECRET> 
```

接下来，我们只是添加路由声明，就像我们在上一篇关于 Spring Cloud Gateway setup 的文章中所做的一样:

```java
... other properties omitted
  cloud:
    gateway:
      routes:
      - id: quotes
        uri: http://localhost:8085
        predicates:
        - Path=/quotes/** 
```

**注意，除了安全依赖和属性，我们没有改变网关本身的任何东西**。为了运行网关应用程序，我们将使用`spring-boot:run`，使用具有所需设置的特定概要文件:

```java
$ mvn spring-boot:run -Pgateway-as-resource-server
```

### 6.1.测试资源服务器

既然我们已经有了拼图的所有碎片，让我们把它们拼在一起。首先，我们必须确保 Keycloak、报价后端和网关都在运行。

接下来，我们需要从 Keycloak 获得一个访问令牌。在这种情况下，获得密码的最直接方法是使用密码授权流(也称为“资源所有者”)。这意味着向 Keycloak 发出 POST 请求，传递其中一个用户的用户名/密码，以及客户机 id 和 quotes 客户机应用程序的密码:

```java
$ curl -L -X POST \
  'http://localhost:8083/auth/realms/baeldung/protocol/openid-connect/token' \
  -H 'Content-Type: application/x-www-form-urlencoded' \
  --data-urlencode 'client_id=quotes-client' \
  --data-urlencode 'client_secret=0e082231-a70d-48e8-b8a5-fbfb743041b6' \
  --data-urlencode 'grant_type=password' \
  --data-urlencode 'scope=email roles profile' \
  --data-urlencode 'username=john.snow' \
  --data-urlencode 'password=1234' 
```

响应将是一个 JSON 对象，包含访问令牌以及其他值:

```java
{
	"access_token": "...omitted",
	"expires_in": 300,
	"refresh_expires_in": 1800,
	"refresh_token": "...omitted",
	"token_type": "bearer",
	"not-before-policy": 0,
	"session_state": "7fd04839-fab1-46a7-a179-a2705dab8c6b",
	"scope": "profile email"
} 
```

我们现在可以使用返回的访问令牌来访问`/quotes` API:

```java
$ curl --location --request GET 'http://localhost:8086/quotes/BAEL' \
--header 'Accept: application/json' \
--header 'Authorization: Bearer xxxx...'
```

它生成 JSON 格式的报价:

```java
{
  "symbol":"BAEL",
  "price":12.0
}
```

让我们重复这个过程，这次使用 Maxwell Smart 的访问令牌:

```java
{
  "symbol":"BAEL",
  "price":10.0
}
```

我们看到我们有一个更低的价格，这意味着后端能够正确地识别相关用户。我们还可以使用没有`Authorization`头的 curl 请求来检查未经身份验证的请求不会传播到后端:

```java
$ curl  http://localhost:8086/quotes/BAEL
```

**检查网关日志，我们发现没有与请求转发过程相关的消息。**这表明响应是在网关上生成的。

## 7.作为 OAuth 2.0 客户端的 Spring Gateway

对于启动类，我们将使用与资源服务器版本相同的类。我们将用它来强调所有的安全行为都来自于可用的库和属性。

事实上，比较两个版本时，唯一值得注意的区别是配置属性。这里，我们需要使用`issuer-uri`属性或各个端点的单独设置(授权、令牌和自省)来配置提供者细节。

我们还需要定义我们的应用程序客户机注册细节，包括请求的范围。这些范围通知 IdP 哪组信息项将通过自省机制可用:

```java
... other propeties omitted
  security:
    oauth2:
      client:
        provider:
          keycloak:
            issuer-uri: http://localhost:8083/auth/realms/baeldung
        registration:
          quotes-client:
            provider: keycloak
            client-id: quotes-client
            client-secret: <CLIENT SECRET>
            scope:
            - email
            - profile
            - roles 
```

最后，路由定义部分有一个重要的变化。**我们必须将`TokenRelay`过滤器添加到任何需要传播接入令牌的路由:**

```java
spring:
  cloud:
    gateway:
      routes:
      - id: quotes
        uri: http://localhost:8085
        predicates:
        - Path=/quotes/**
        filters:
        - TokenRelay= 
```

或者，如果我们希望所有路由都启动一个授权流，我们可以将`TokenRelay`过滤器添加到`default-filters `部分:

```java
spring:
  cloud:
    gateway:
      default-filters:
      - TokenRelay=
      routes:
... other routes definition omitted
```

### 7.1.将 Spring Gateway 作为 OAuth 2.0 客户端进行测试

对于测试设置，我们还需要确保项目的三个部分都在运行。然而，这一次，我们将使用一个不同的 [Spring Profile](/web/20220903190824/https://www.baeldung.com/spring-profiles) 来运行网关，它包含了使其充当 OAuth 2.0 客户端所需的属性。示例项目的 POM 包含一个概要文件，允许我们在启用该概要文件的情况下启动它:

```java
$ mvn spring-boot:run -Pgateway-as-oauth-client
```

一旦网关运行，我们可以通过将浏览器指向 http://localhost:8087/quotes/BAEL 来测试它。如果一切正常，我们将被重定向到 IdP 的登录页面:

[![Login Page](img/884f868d367b572f7824152037e1dee0.png)](/web/20220903190824/https://www.baeldung.com/wp-content/uploads/2022/01/maxwell-login.png)

由于我们使用了 Maxwell Smart 的凭证，我们再次得到了一个更低的报价:

[![Maxwell's Quote](img/e5df4d8bece74bea43b685a2f8d17dd5.png)](/web/20220903190824/https://www.baeldung.com/wp-content/uploads/2022/01/maxwell-quote.png)

为了结束我们的测试，我们将使用一个匿名的浏览器窗口，并用 John Snow 的凭证测试这个端点。这次我们得到了常规报价:

[![Snow's Quote](img/753093e9fa5b28239bf89ec088c487d8.png)](/web/20220903190824/https://www.baeldung.com/wp-content/uploads/2022/01/snows-quote.png)

## 8.结论

在本文中，我们探索了一些 OAuth 2.0 安全模式，以及如何使用 Spring Cloud Gateway 实现它们。像往常一样，GitHub 上的所有代码[都是可用的。](https://web.archive.org/web/20220903190824/https://github.com/eugenp/tutorials/tree/master/spring-cloud-modules/spring-cloud-gateway)