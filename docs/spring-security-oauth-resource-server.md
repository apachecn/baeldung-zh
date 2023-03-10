# 具有 Spring Security 5 的 OAuth 2.0 资源服务器

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-security-oauth-resource-server>

## 1.概观

在本教程中，我们将学习如何使用 Spring Security 5 设置 OAuth 2.0 资源服务器。

我们将使用 jwt 和不透明令牌来实现这一点，不透明令牌是 Spring Security 支持的两种无记名令牌。

在我们进入实现和代码示例之前，我们将首先建立一些背景。

## 2.一点背景

### 2.1.什么是 jwt 和不透明令牌？

JWT，即 [JSON Web Token](https://web.archive.org/web/20221018140149/https://tools.ietf.org/html/rfc7519) ，是一种以广泛接受的 JSON 格式安全传输敏感信息的方式。包含的信息可以是关于用户的，也可以是关于令牌本身的，比如它的有效期和颁发者。

另一方面，不透明令牌，顾名思义，就其携带的信息而言是不透明的。令牌只是一个标识符，它指向存储在授权服务器上的信息；它通过服务器端的自省得到验证。

### 2.2.什么是资源服务器？

在 OAuth 2.0 的上下文中，**资源服务器是一个通过 OAuth 令牌**保护资源的应用程序。这些令牌由授权服务器颁发，通常颁发给客户端应用程序。资源服务器的工作是在向客户机提供资源之前验证令牌。

令牌的有效性由几个因素决定:

*   该令牌来自已配置的授权服务器吗？
*   是未过期的吗？
*   这个资源服务器是它的目标用户吗？
*   令牌是否具有访问请求的资源所需的权限？

为了形象化这一点，让我们来看一下[授权代码流](https://web.archive.org/web/20221018140149/https://tools.ietf.org/html/rfc6749#section-1.3.1)的序列图，并看看所有的参与者:

[![](img/52d417027f28c0296fd8e88c0b796187.png)](/web/20221018140149/https://www.baeldung.com/wp-content/uploads/2020/08/AuthCodeFlowSequenceDiagram-1.png) 正如我们在第 8 步中看到的，当客户端应用程序调用资源服务器的 API 来访问受保护的资源时，它首先去授权服务器验证请求的`Authorization: Bearer`头中包含的令牌，然后响应客户端。

第 9 步是我们在本教程中重点关注的。

现在让我们进入代码部分。我们将使用 Keycloak 设置一个授权服务器，一个验证 JWT 令牌的资源服务器，另一个验证不透明令牌的资源服务器，以及几个 JUnit 测试来模拟客户端应用程序并验证响应。

## 3.授权服务器

首先，我们将设置一个授权服务器，它发布令牌。

为此，我们将使用嵌入在 Spring Boot 应用程序中的[key cloak。Keycloak 是一个开源的身份和访问管理解决方案。因为我们在本教程中关注的是资源服务器，所以我们不会对它进行更深入的研究。](/web/20221018140149/https://www.baeldung.com/keycloak-embedded-in-spring-boot-app)

我们的嵌入式 Keycloak 服务器定义了两个客户端，`fooClient`和`barClient,` ，分别对应于我们的两个资源服务器应用程序。

## 4.资源服务器–使用 JWTs

我们的资源服务器将有四个主要组件:

*   `Model`–要保护的资源
*   `API`–公开资源的 REST 控制器
*   `Security Configuration`–为 API 公开的受保护资源定义访问控制的类
*   `application.yml`–声明属性的配置文件，包括授权服务器的信息

在我们快速地看了一下依赖项之后，我们将为处理 JWT 令牌的资源服务器逐一检查这些组件。

### 4.1.Maven 依赖性

主要是，我们将需要 [`spring-boot-starter-oauth2-resource-server`](https://web.archive.org/web/20221018140149/https://search.maven.org/search?q=spring-boot-starter-oauth2-resource-server) ，Spring Boot 的资源服务器支持的启动器。这个 starter 默认包含 Spring Security，所以我们不需要显式地添加它:

```java
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
    <version>2.2.6.RELEASE</version>
</dependency>
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-oauth2-resource-server</artifactId>
    <version>2.2.6.RELEASE</version>
</dependency>
<dependency>
    <groupId>org.apache.commons</groupId>
    <artifactId>commons-lang3</artifactId>
    <version>3.12.0</version>
</dependency>
```

除此之外，我们还将增加网络支持。

出于演示的目的，我们将随机生成资源，而不是从数据库中获取，并借助 Apache 的 [`commons-lang3`](https://web.archive.org/web/20221018140149/https://search.maven.org/artifact/org.apache.commons/commons-lang3) 库。

### 4.2.模型

简单来说，我们将使用 POJO`Foo`作为我们的受保护资源:

```java
public class Foo {
    private long id;
    private String name;

    // constructor, getters and setters
} 
```

### 4.3.应用程序接口

这是我们的 rest 控制器，使`Foo`可用于操作:

```java
@RestController
@RequestMapping(value = "/foos")
public class FooController {

    @GetMapping(value = "/{id}")
    public Foo findOne(@PathVariable Long id) {
        return new Foo(Long.parseLong(randomNumeric(2)), randomAlphabetic(4));
    }

    @GetMapping
    public List findAll() {
        List fooList = new ArrayList();
        fooList.add(new Foo(Long.parseLong(randomNumeric(2)), randomAlphabetic(4)));
        fooList.add(new Foo(Long.parseLong(randomNumeric(2)), randomAlphabetic(4)));
        fooList.add(new Foo(Long.parseLong(randomNumeric(2)), randomAlphabetic(4)));
        return fooList;
    }

    @ResponseStatus(HttpStatus.CREATED)
    @PostMapping
    public void create(@RequestBody Foo newFoo) {
        logger.info("Foo created");
    }
}
```

很明显，我们有获取所有`Foo`的规定，通过 id 获取一个`Foo`，并发布一个`Foo`。

### 4.4.安全配置

在本配置课程中，我们将定义资源的访问级别:

```java
@Configuration
public class JWTSecurityConfig extends WebSecurityConfigurerAdapter {

    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http
          .authorizeRequests(authz -> authz
            .antMatchers(HttpMethod.GET, "/foos/**").hasAuthority("SCOPE_read")
            .antMatchers(HttpMethod.POST, "/foos").hasAuthority("SCOPE_write")
            .anyRequest().authenticated())
          .oauth2ResourceServer(oauth2 -> oauth2.jwt());
	}
} 
```

任何拥有具有`read`范围的访问令牌的人都可以获得`Foo` s。为了发布新的`Foo`，他们的令牌应该具有`write`范围。

此外，**我们将使用*oauth 2 resourceserver()*[DSL](https://web.archive.org/web/20221018140149/https://docs.spring.io/spring-integration/docs/5.1.0.M1/reference/html/java-dsl.html)添加对`jwt()`的调用，以指示我们的服务器支持的令牌类型**。

### 4.5.`application.yml`

在应用程序属性中，除了通常的端口号和上下文路径，**我们需要定义到我们的授权服务器的发布者 URI 的路径，以便资源服务器可以发现它的[提供者配置](https://web.archive.org/web/20221018140149/https://openid.net/specs/openid-connect-discovery-1_0.html#ProviderConfig)** :

```java
server: 
  port: 8081
  servlet: 
    context-path: /resource-server-jwt

spring:
  security:
    oauth2:
      resourceserver:
        jwt:
          issuer-uri: http://localhost:8083/auth/realms/baeldung
```

按照序列图的步骤 9，资源服务器使用这些信息来验证来自客户端应用程序的 JWT 令牌。

为了使用`issuer-uri`属性进行验证，授权服务器必须启动并运行。否则，资源服务器不会启动。

如果我们需要独立启动它，那么我们可以提供`jwk-set-uri`属性来指向授权服务器的端点，公开公钥:

```java
jwk-set-uri: http://localhost:8083/auth/realms/baeldung/protocol/openid-connect/certs
```

这就是我们让服务器验证 JWT 令牌所需的全部内容。

### 4.6.测试

为了测试，我们将设置一个 JUnit。为了执行这个测试，我们需要启动并运行授权服务器和资源服务器。

让我们验证一下，在我们的测试中，我们可以用一个`read`范围的令牌从`resource-server-jw` t 获得`Foo` s:

```java
@Test
public void givenUserWithReadScope_whenGetFooResource_thenSuccess() {
    String accessToken = obtainAccessToken("read");

    Response response = RestAssured.given()
      .header(HttpHeaders.AUTHORIZATION, "Bearer " + accessToken)
      .get("http://localhost:8081/resource-server-jwt/foos");
    assertThat(response.as(List.class)).hasSizeGreaterThan(0);
}
```

在上面的代码中，在第 3 行，我们从授权服务器获得了一个范围为`read`的访问令牌，涵盖了序列图中的步骤 1 到 7。

步骤 8 由`RestAssured`的`get()`调用执行。第 9 步是由资源服务器使用我们看到的配置执行的，对于用户来说是透明的。

## 5.资源服务器–使用不透明令牌

接下来，让我们看看处理不透明令牌的资源服务器的相同组件。

### 5.1.Maven 依赖性

为了支持不透明令牌，我们需要额外的`[oauth2-oidc-sdk](https://web.archive.org/web/20221018140149/https://search.maven.org/search?q=oauth2-oidc-sdk)` 依赖:

```java
<dependency>
    <groupId>com.nimbusds</groupId>
    <artifactId>oauth2-oidc-sdk</artifactId>
    <version>8.19</version>
    <scope>runtime</scope>
</dependency>
```

### 5.2.模型和控制器

对此，我们将添加一个`Bar`资源:

```java
public class Bar {
    private long id;
    private String name;

    // constructor, getters and setters
} 
```

我们还将有一个端点与之前的`FooController`相似的`BarController,`，来发布`Bar`

### 5.3.`application.yml`

在这里的`application.yml`中，我们需要添加一个对应于授权服务器自省端点的`introspection-uri` 。如前所述，这就是不透明令牌被验证的方式:

```java
server: 
  port: 8082
  servlet: 
    context-path: /resource-server-opaque

spring:
  security:
    oauth2:
      resourceserver:
        opaque:
          introspection-uri: http://localhost:8083/auth/realms/baeldung/protocol/openid-connect/token/introspect
          introspection-client-id: barClient
          introspection-client-secret: barClientSecret
```

### 5.4.安全配置

保持类似于`Bar`资源的`Foo`的访问级别，**这个配置类还使用 `oauth2ResourceServer()` DSL 调用`opaqueToken()`来指示使用不透明令牌类型**:

```java
@Configuration
public class OpaqueSecurityConfig extends WebSecurityConfigurerAdapter {

    @Value("${spring.security.oauth2.resourceserver.opaque.introspection-uri}")
    String introspectionUri;

    @Value("${spring.security.oauth2.resourceserver.opaque.introspection-client-id}")
    String clientId;

    @Value("${spring.security.oauth2.resourceserver.opaque.introspection-client-secret}")
    String clientSecret;

    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http
          .authorizeRequests(authz -> authz
            .antMatchers(HttpMethod.GET, "/bars/**").hasAuthority("SCOPE_read")
            .antMatchers(HttpMethod.POST, "/bars").hasAuthority("SCOPE_write")
            .anyRequest().authenticated())
          .oauth2ResourceServer(oauth2 -> oauth2
            .opaqueToken(token -> token.introspectionUri(this.introspectionUri)
              .introspectionClientCredentials(this.clientId, this.clientSecret)));
    }
} 
```

在这里，我们还将指定对应于我们将使用的授权服务器客户机的客户机凭证。我们在前面的`application.yml`中定义了这些。

### 5.5.测试

我们将为基于不透明令牌的资源服务器设置一个 JUnit，类似于我们为 JWT 服务器所做的。

在这种情况下，我们将检查一个`write`范围内的访问令牌是否可以向`resource-server-opaque`发送一个`Bar`:

```java
@Test
public void givenUserWithWriteScope_whenPostNewBarResource_thenCreated() {
    String accessToken = obtainAccessToken("read write");
    Bar newBar = new Bar(Long.parseLong(randomNumeric(2)), randomAlphabetic(4));

    Response response = RestAssured.given()
      .contentType(ContentType.JSON)
      .header(HttpHeaders.AUTHORIZATION, "Bearer " + accessToken)
      .body(newBar)
      .log()
      .all()
      .post("http://localhost:8082/resource-server-opaque/bars");
    assertThat(response.getStatusCode()).isEqualTo(HttpStatus.CREATED.value());
}
```

如果我们得到一个 CREATED back 状态，这意味着资源服务器成功地验证了不透明令牌并为我们创建了`Bar`。

## 6.结论

在本文中，我们学习了如何配置基于 Spring Security 的资源服务器应用程序来验证 jwt 以及不透明令牌。

正如我们所看到的，**通过最少的设置，Spring 使得与发行者**无缝验证令牌并向请求方发送资源成为可能(在我们的例子中，是一个 JUnit 测试)。

和往常一样，源代码可以在 GitHub 上的[处获得。](https://web.archive.org/web/20221018140149/https://github.com/Baeldung/spring-security-oauth/tree/master/oauth-resource-server)