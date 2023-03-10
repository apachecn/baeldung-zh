# 使用 Spring Security OAuth2 实现简单的单点登录

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/sso-spring-security-oauth2>

## 1。概述

在本教程中，我们将讨论如何实现**[SSO——单点登录](/web/20220707095557/https://www.baeldung.com/cs/sso-guide)——使用 Spring Security OAuth 和 Spring Boot，使用 Keycloak** 作为授权服务器。

我们将使用 4 个独立的应用程序:

*   授权服务器——这是中央认证机制
*   资源服务器——的提供者
*   两个客户端应用程序–使用 SSO 的应用程序

非常简单地说，当用户试图通过一个客户端应用程序访问资源时，他们将首先通过授权服务器重定向到身份验证。Keycloak 将让用户登录，在登录第一个应用程序的同时，如果使用同一浏览器访问第二个客户端应用程序，用户将无需再次输入凭据。

我们将使用 OAuth2 中的`Authorization Code` 授权类型来驱动身份验证的委托。

我们将在 Spring Security 5 中使用 OAuth 堆栈。如果您想使用 Spring Security OAuth 遗留堆栈，可以看看之前的这篇文章:[使用 Spring Security OAuth 的简单单点登录 2(遗留堆栈)](/web/20220707095557/https://www.baeldung.com/sso-spring-security-oauth2-legacy)

根据[迁移指南](https://web.archive.org/web/20220707095557/https://github.com/spring-projects/spring-security/wiki/OAuth-2.0-Migration-Guide#changes-in-approach-1):

> Spring Security 把这个特性称为 OAuth 2.0 登录，而 Spring Security OAuth 把它称为 SSO

## 延伸阅读:

## [Spring Security 5–oauth 2 登录](/web/20220707095557/https://www.baeldung.com/spring-security-5-oauth2-login)

Learn how to authenticate users with Facebook, Google or other credentials using OAuth2 in Spring Security 5.[Read more](/web/20220707095557/https://www.baeldung.com/spring-security-5-oauth2-login) →

## [Spring Security oauth 2 中的新功能-验证声明](/web/20220707095557/https://www.baeldung.com/spring-security-oauth-2-verify-claims)

Quick practical intro to the new Claim verification support in Spring Security OAuth.[Read more](/web/20220707095557/https://www.baeldung.com/spring-security-oauth-2-verify-claims) →

## [二级脸书登录 Spring Social](/web/20220707095557/https://www.baeldung.com/facebook-authentication-with-spring-security-and-social)

A quick look at implementing a Facebook driven authentication next to a standard form-login Spring app.[Read more](/web/20220707095557/https://www.baeldung.com/facebook-authentication-with-spring-security-and-social) →

好吧，让我们开始吧。

## 2。授权服务器

以前，Spring Security OAuth 堆栈提供了将授权服务器设置为 Spring 应用程序的可能性。

然而，OAuth 栈已经被 Spring 弃用，现在我们将使用 Keycloak 作为我们的授权服务器。

**所以这一次，我们将把我们的授权服务器设置为[一个 Spring Boot 应用](/web/20220707095557/https://www.baeldung.com/keycloak-embedded-in-a-spring-boot-application/)** 中的嵌入式 Keycloak 服务器。

在我们的[预配置](/web/20220707095557/https://www.baeldung.com/keycloak-embedded-in-spring-boot-app#keycloak-config)、**中，我们将为每个客户端应用定义两个客户端`ssoClient-1`和** `**ssoClient-2**,`。

## 3.资源服务器

接下来，我们需要一个资源服务器，或者 REST API，它将为我们的客户端应用程序提供所需的资源。

它本质上与我们之前用于 Angular 客户端应用程序的[相同。](/web/20220707095557/https://www.baeldung.com/rest-api-spring-oauth2-angular#resource-server)

## 4。客户端应用程序

现在让我们看看我们的百里香客户端应用程序；当然，我们将使用 Spring Boot 来最小化配置。

请记住，**我们需要两个来演示单点登录功能**。

### 4.1。Maven 依赖关系

首先，在我们的`pom.xml`中，我们需要以下依赖项:

```java
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-oauth2-client</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-thymeleaf</artifactId>
</dependency>
<dependency>
    <groupId>org.thymeleaf.extras</groupId>
    <artifactId>thymeleaf-extras-springsecurity5</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-webflux</artifactId>
</dependency>
<dependency>
    <groupId>io.projectreactor.netty</groupId>
    <artifactId>reactor-netty</artifactId>
</dependency>
```

**为了包含我们需要的所有客户端支持，包括安全性，我们只需添加`spring-boot-starter-oauth2-client`** 。还有，既然旧的`RestTemplate`要被弃用了，我们就用`WebClient`，这也是我们加了`spring-webflux`和`reactor-netty`的原因。

### 4.2。安全配置

接下来，最重要的部分，我们的第一个客户端应用程序的安全配置:

```java
@EnableWebSecurity
public class UiSecurityConfig extends WebSecurityConfigurerAdapter {

    @Override
    public void configure(HttpSecurity http) throws Exception {
        http.antMatcher("/**")
          .authorizeRequests()
          .antMatchers("/")
          .permitAll()
          .anyRequest()
          .authenticated()
          .and()
          .oauth2Login();
    }

    @Bean
    WebClient webClient(ClientRegistrationRepository clientRegistrationRepository, 
      OAuth2AuthorizedClientRepository authorizedClientRepository) {
        ServletOAuth2AuthorizedClientExchangeFilterFunction oauth2 = 
          new ServletOAuth2AuthorizedClientExchangeFilterFunction(clientRegistrationRepository, 
          authorizedClientRepository);
        oauth2.setDefaultOAuth2AuthorizedClient(true);
        return WebClient.builder().apply(oauth2.oauth2Configuration()).build();
    }
}
```

**这个配置的核心部分是`oauth2Login()`方法，用来启用 Spring Security 的 OAuth 2.0 登录支持。**由于我们使用的是 Keycloak，这是默认的 web 应用和 RESTful web 服务的单点登录解决方案，我们不需要为 SSO 添加任何进一步的配置。

最后，我们还定义了一个`WebClient` bean 作为一个简单的 HTTP 客户端来处理发送到我们的资源服务器的请求。

这里是`application.yml`:

```java
spring:
  security:
    oauth2:
      client:
        registration:
          custom:
            client-id: ssoClient-1
            client-secret: ssoClientSecret-1
            scope: read,write
            authorization-grant-type: authorization_code
            redirect-uri: http://localhost:8082/ui-one/login/oauth2/code/custom
        provider:
          custom:
            authorization-uri: http://localhost:8083/auth/realms/baeldung/protocol/openid-connect/auth
            token-uri: http://localhost:8083/auth/realms/baeldung/protocol/openid-connect/token
            user-info-uri: http://localhost:8083/auth/realms/baeldung/protocol/openid-connect/userinfo
            user-name-attribute: preferred_username
  thymeleaf:
    cache: false

server: 
  port: 8082
  servlet: 
    context-path: /ui-one

resourceserver:
  api:
    project:
      url: http://localhost:8081/sso-resource-server/api/foos/ 
```

这里，`spring.security.oauth2.client.registration`是注册客户机的根名称空间。我们定义了一个注册 id 为`custom`的客户端。然后我们定义了它的`client-id`、`client-secret`、`scope`、`authorization-grant-type`和`redirect-uri`，当然这些应该和我们为授权服务器定义的一样。

之后，我们定义了我们的服务提供者或授权服务器，同样使用相同的 id`custom`，并列出不同的 URI 供 Spring Security 使用。这就是我们所需要定义的，框架为我们无缝地完成了整个登录过程，包括重定向到 key cloak。

还要注意，在我们这里的例子中，我们推出了我们的授权服务器，但是当然我们也可以使用其他第三方提供商，比如[脸书或者 GitHub](/web/20220707095557/https://www.baeldung.com/spring-security-5-oauth2-login) 。

### 4.3。控制器

现在让我们在客户端应用程序中实现我们的控制器，向我们的资源服务器请求`Foo`:

```java
@Controller
public class FooClientController {

    @Value("${resourceserver.api.url}")
    private String fooApiUrl;

    @Autowired
    private WebClient webClient;

    @GetMapping("/foos")
    public String getFoos(Model model) {
        List<FooModel> foos = this.webClient.get()
            .uri(fooApiUrl)
            .retrieve()
            .bodyToMono(new ParameterizedTypeReference<List<FooModel>>() {
            })
            .block();
        model.addAttribute("foos", foos);
        return "foos";
    }
}
```

正如我们所见，这里只有一个方法将资源分配给`foos`模板。**我们无需添加任何登录代码。**

### 4.4。前端

现在，让我们看看我们的客户端应用程序的前端配置。我们不会在这里集中讨论这个问题，主要是因为我们已经在网站上介绍过了[。](/web/20220707095557/https://www.baeldung.com/spring-thymeleaf-3)

我们的客户端应用程序有一个非常简单的前端；下面是`index.html`:

```java
<a class="navbar-brand" th:href="@{/foos/}">Spring OAuth Client Thymeleaf - 1</a>
<label>Welcome !</label> <br /> <a th:href="@{/foos/}">Login</a>
```

而`foos.html`:

```java
<a class="navbar-brand" th:href="@{/foos/}">Spring OAuth Client Thymeleaf -1</a>
Hi, <span sec:authentication="name">preferred_username</span>   

<h1>All Foos:</h1>
<table>
  <thead>
    <tr>
      <td>ID</td>
      <td>Name</td>                    
    </tr>
  </thead>
  <tbody>
    <tr th:if="${foos.empty}">
      <td colspan="4">No foos</td>
    </tr>
    <tr th:each="foo : ${foos}">
      <td><span th:text="${foo.id}"> ID </span></td>
      <td><span th:text="${foo.name}"> Name </span></td>                    
    </tr>
  </tbody>
</table>
```

`foos.html`页面需要对用户进行认证。**如果一个未经认证的用户试图访问`foos.html`，他们将首先被重定向到 Keycloak 的登录页面**。

### 4.5.第二客户端应用程序

我们将使用另一个`client_id` `ssoClient-2`配置第二个应用程序`Spring OAuth Client Thymeleaf -2`。

它基本上与我们刚刚描述的第一个应用程序相同。

**`application.yml`将有所不同，在其`spring.security.oauth2.client.registration:`** 中包含不同的`client_id`、`client_secret`和`redirect_uri`

```java
spring:
  security:
    oauth2:
      client:
        registration:
          custom:
            client-id: ssoClient-2
            client-secret: ssoClientSecret-2
            scope: read,write
            authorization-grant-type: authorization_code
            redirect-uri: http://localhost:8084/ui-two/login/oauth2/code/custom
```

当然，我们还需要一个不同的服务器端口，以便我们可以并行运行它们:

```java
server: 
  port: 8084
  servlet: 
    context-path: /ui-two
```

最后，我们将调整前端 HTMLs 的标题为`Spring OAuth Client Thymeleaf – 2`而不是`– 1`，这样我们就可以区分两者。

## 5.测试 SSO 行为

为了测试 SSO 行为，让我们运行我们的应用程序。

为此，我们需要启动并运行所有 4 个引导应用程序——授权服务器、资源服务器和两个客户端应用程序。

现在让我们打开一个浏览器，比如说 Chrome，使用凭证`[[email protected]](/web/20220707095557/https://www.baeldung.com/cdn-cgi/l/email-protection)/123`登录 [`Client-1`](https://web.archive.org/web/20220707095557/http://localhost:8082/ui-one) 。接下来，在另一个窗口或标签中，点击 URL 为 [`Client-2`](https://web.archive.org/web/20220707095557/http://localhost:8084/ui-two) 。点击登录按钮，我们将直接被重定向到`Foos`页面，绕过认证步骤。

类似地，如果用户首先登录到`Client-2`，他们不需要输入`Client-1`的用户名/密码。

## 6。结论

在本教程中，我们重点介绍了使用 Spring Security OAuth2 实现单点登录，以及使用 Keycloak 作为身份提供者实现 Spring Boot。

和往常一样，完整的源代码可以在 GitHub 上找到[。](https://web.archive.org/web/20220707095557/https://github.com/Baeldung/spring-security-oauth/tree/master/oauth-sso)