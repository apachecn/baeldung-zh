# Spring 安全和 OpenID 连接

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-security-openid-connect>

**注意，本文已经更新到新的 Spring Security OAuth 2.0 堆栈。使用传统堆栈的[教程仍然可用。](/web/20220926202321/https://www.baeldung.com/spring-security-openid-connect-legacy)**

## 1。概述

在本教程中，我们将重点介绍如何使用 Spring Security 设置 OpenID Connect (OIDC)。

我们将介绍该规范的不同方面，然后我们将看到 Spring Security 为在 OAuth 2.0 客户机上实现它所提供的支持。

## 2。快速 OpenID 连接简介

**[OpenID Connect](https://web.archive.org/web/20220926202321/https://openid.net/connect/) 是建立在 OAuth 2.0 协议之上的身份层。**

所以，在深入 OIDC 之前，了解 OAuth 2.0 真的很重要，尤其是授权代码流。

OIDC 规格套件非常广泛。它包括核心功能和其他几个可选功能，以不同的组呈现。以下是主要的几种:

*   核心–身份验证和使用声明来传达最终用户信息
*   发现——规定客户端如何动态确定 OpenID 提供者的信息
*   动态注册–规定客户如何向提供商注册
*   会话管理–定义如何管理 OIDC 会话

最重要的是，这些文档区分了为该规范提供支持的 OAuth 2.0 认证服务器，将它们称为 OpenID 提供者(OPs)和使用 OIDC 作为依赖方(RPs)的 OAuth 2.0 客户端。我们将在本文中使用这个术语。

同样值得注意的是，客户端可以通过在其授权请求中添加`openid `范围来请求使用该扩展。

最后，对于本教程，了解 OPs 将最终用户信息作为一个称为 ID 令牌的 JWT 发出是很有用的。

现在我们准备深入 OIDC 世界。

## 3。项目设置

在关注实际开发之前，我们必须向我们的 OpenID 提供者注册一个 OAuth 2.0 客户端。

在这种情况下，我们将使用 Google 作为 OpenID 提供者。我们可以按照[这些指令](https://web.archive.org/web/20220926202321/https://developers.google.com/identity/protocols/OpenIDConnect#appsetup)在他们的平台上注册我们的客户端应用。请注意，默认情况下会出现`openid `范围。

我们在这个过程中设置的重定向 URI 是我们服务中的一个端点:`http://localhost:8081/login/oauth2/code/google`。

我们应该从这个过程中获得一个客户端 ID 和一个客户端机密。

### 3.1。Maven 配置

我们首先将这些依赖项添加到我们的项目 pom 文件中:

```java
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-oauth2-client</artifactId>
    <version>2.2.6.RELEASE</version>
</dependency>
```

starter 工件聚合了所有与 Spring Security 客户端相关的依赖项，包括

*   OAuth 2.0 登录和客户端功能的`spring-security-oauth2-client`依赖性
*   支持 JWT 的何塞图书馆

像往常一样，我们可以使用[Maven 中央搜索引擎](https://web.archive.org/web/20220926202321/https://search.maven.org/search?q=a:spring-boot-starter-oauth2-client%20AND%20g:org.springframework.boot)找到这个工件的最新版本。

## 4。使用 Spring Boot 的基本配置

首先，我们将开始配置我们的应用程序，以使用我们刚刚用 Google 创建的客户端注册。

**使用 Spring Boot 使这变得非常简单，因为我们所要做的就是定义两个应用程序属性**:

```java
spring:
  security:
    oauth2:
      client:
        registration: 
          google: 
            client-id: <client-id>
            client-secret: <secret>
```

现在，让我们启动应用程序并尝试访问一个端点。我们将看到我们被重定向到 OAuth 2.0 客户端的 Google 登录页面。

这看起来真的很简单，但在这里有相当多的事情正在进行。接下来，我们将探索 Spring Security 如何实现这一点。

以前，在[我们的 WebClient 和 OAuth 2 支持帖子](/web/20220926202321/https://www.baeldung.com/spring-webclient-oauth2#springsecurity-internals)中，我们分析了 Spring Security 如何处理 OAuth 2.0 授权服务器和客户端的内部机制。

我们看到，除了客户机 ID 和客户机机密之外，我们还必须提供额外的数据，以便成功地配置一个`ClientRegistration` 实例。

那么，这是怎么回事？

Google 是一个知名的提供商，因此该框架提供了一些预定义的属性来简化事情。

我们可以在`CommonOAuth2Provider` 枚举中查看这些配置。

对于 Google，枚举类型定义了如下属性

*   将使用的默认范围
*   授权端点
*   令牌端点
*   UserInfo 端点，它也是 OIDC 核心规范的一部分

### 4.1.访问用户信息

**Spring Security 提供了一个向 OIDC 提供商注册的用户主体的有用表示,`OidcUser `实体。**

除了基本的`OAuth2AuthenticatedPrincipal` 方法，这个实体还提供了一些有用的功能:

*   检索 ID 标记值及其包含的声明
*   获取 UserInfo 端点提供的声明
*   生成两个集合的集合

我们可以很容易地在控制器中访问这个实体:

```java
@GetMapping("/oidc-principal")
public OidcUser getOidcUserPrincipal(
  @AuthenticationPrincipal OidcUser principal) {
    return principal;
}
```

或者我们可以在 bean 中使用`SecurityContextHolder`:

```java
Authentication authentication = SecurityContextHolder.getContext().getAuthentication();
if (authentication.getPrincipal() instanceof OidcUser) {
    OidcUser principal = ((OidcUser) authentication.getPrincipal());

    // ...
}
```

如果我们检查主体，我们会在这里看到许多有用的信息，如用户的姓名、电子邮件、个人资料图片和区域设置。

**此外，值得注意的是，Spring 根据从提供者那里收到的范围向主体添加权限，前缀为“`SCOPE_`”。例如，**的`openid`范围就变成了`SCOPE_openid `授予的权限。

这些权限可用于限制对某些资源的访问:

```java
@EnableWebSecurity
public class MappedAuthorities extends WebSecurityConfigurerAdapter {
    protected void configure(HttpSecurity http) {
        http
          .authorizeRequests(authorizeRequests -> authorizeRequests
            .mvcMatchers("/my-endpoint")
              .hasAuthority("SCOPE_openid")
            .anyRequest().authenticated()
          );
    }
}
```

## 5。OIDC 在行动

到目前为止，我们已经了解了如何使用 Spring Security 轻松实现 OIDC 登录解决方案。

我们已经看到了通过将用户识别过程委托给 OpenID 提供者所带来的好处，OpenID 提供者反过来提供详细有用的信息，甚至以可伸缩的方式。

但事实是，到目前为止，我们并不需要处理任何 OIDC 特有的问题。这意味着春天为我们做了大部分的工作。

因此，让我们看看幕后发生了什么，以便更好地理解该规范是如何付诸实施的，并能够从中获得最大收益。

### 5.1.登录过程

为了清楚地看到这一点，让我们启用`RestTemplate `日志来查看服务正在执行的请求:

```java
logging:
  level:
    org.springframework.web.client.RestTemplate: DEBUG
```

如果我们现在调用一个安全端点，我们将看到服务正在执行常规的 OAuth 2.0 授权代码流。正如我们所说，这是因为该规范是建立在 OAuth 2.0 之上的。

有一些不同。

首先，根据我们使用的提供者和我们配置的作用域，我们可能会看到服务正在调用我们在开始提到的 UserInfo 端点。

也就是说，如果授权响应检索到`profile`、`email`、`address`或`phone `范围中的至少一个，那么框架将调用 UserInfo 端点来获得附加信息。

尽管一切都表明 Google 应该检索`profile` 和`email `范围——因为我们在授权请求中使用它们——OP 反而检索它们的自定义对应项,`https://www.googleapis.com/auth/userinfo.email` 和`https://www.googleapis.com/auth/userinfo.profile`,所以 Spring 不调用端点。

这意味着我们获得的所有信息都是 ID 令牌的一部分。

我们可以通过创建和提供自己的`OidcUserService` 实例来适应这种行为:

```java
@Configuration
public class OAuth2LoginSecurityConfig
  extends WebSecurityConfigurerAdapter {

    @Override
    protected void configure(HttpSecurity http) throws Exception {
        Set<String> googleScopes = new HashSet<>();
        googleScopes.add(
          "https://www.googleapis.com/auth/userinfo.email");
        googleScopes.add(
          "https://www.googleapis.com/auth/userinfo.profile");

        OidcUserService googleUserService = new OidcUserService();
        googleUserService.setAccessibleScopes(googleScopes);

        http
          .authorizeRequests(authorizeRequests -> authorizeRequests
            .anyRequest().authenticated())
          .oauth2Login(oauthLogin -> oauthLogin
            .userInfoEndpoint()
              .oidcUserService(googleUserService));
    }
}
```

我们将观察到的第二个区别是对 JWK 集合 URI 的调用。正如我们在[我们的 JWS 和 JWK 帖子](/web/20220926202321/https://www.baeldung.com/spring-security-oauth2-jws-jwk)中解释的，这用于验证 JWT 格式的 ID 令牌签名。

接下来，我们将详细分析 ID 令牌。

### 5.2.ID 标记

自然，OIDC 规范涵盖并适应了许多不同的场景。**在这种情况下，我们使用授权代码流，协议表明访问令牌和 ID 令牌都将作为令牌端点响应的一部分被检索。**

正如我们之前所说的，`OidcUser`实体包含 id 令牌中包含的声明，以及实际的 JWT 格式的令牌，可以使用 [jwt.io](https://web.archive.org/web/20220926202321/https://jwt.io/) 来检查。

除此之外，Spring 提供了许多方便的 getters 来以简洁的方式获得规范定义的标准声明。

我们可以看到 ID 令牌包括一些强制声明:

*   格式化为 URL 的发行者标识符(例如，“`https://accounts.google.com`”)
*   主题 id，它是发行者包含的最终用户的参考
*   令牌的到期时间
*   颁发令牌的时间
*   受众，它将包含我们已经配置的 OAuth 2.0 客户端 ID

其中还包含了很多 [OIDC 标准主张](https://web.archive.org/web/20220926202321/https://openid.net/specs/openid-connect-core-1_0.html#StandardClaims)比如我们之前提到的(`name`、`locale`、`picture`、`email`)。

由于这些是标准的，我们可以预期许多提供者至少会检索其中的一些字段，从而促进更简单的解决方案的开发。

### 5.3.权利要求和范围

正如我们可以想象的那样，OP 检索到的声明与我们(或 Spring Security)配置的范围相对应。

OIDC 定义了一些可用于请求 OIDC 定义的权利要求的范围:

*   `profile`，可用于请求默认配置文件声明(如`name`、 `preferred_username`、`picture`等)。)
*   `email`，访问`email`和`email_verified`的债权
*   `address`
*   `phone`、请求`phone_number`和`phone_number_verified`索赔

尽管 Spring 还不支持它，但是规范允许通过在授权请求中指定来请求单个声明。

## 6。春天支持 OIDC 探索号

正如我们在引言中解释的，除了核心目的之外，OIDC 还包括许多不同的特性。

我们将在本节和以下内容中分析的功能在 OIDC 是可选的。因此，了解可能有不支持它们的操作是很重要的。

该规范为 RP 定义了一种发现机制，用于发现 OP 并获取与之交互所需的信息。

简而言之，OPs 提供了标准元数据的 JSON 文档。该信息必须由发行者所在地的众所周知的端点`/.well-known/openid-configuration`提供。

Spring 受益于此，它允许我们只使用一个简单的属性来配置一个`ClientRegistration` ,即发行者位置。

但是让我们直接看一个例子来清楚地了解这一点。

我们将定义一个定制的`ClientRegistration` 实例:

```java
spring:
  security:
    oauth2:
      client:
        registration: 
          custom-google: 
            client-id: <client-id>
            client-secret: <secret>
        provider:
          custom-google:
            issuer-uri: https://accounts.google.com
```

现在，我们可以重启我们的应用程序并检查日志，以确认应用程序正在启动过程中调用`openid-configuration `端点。

我们甚至可以浏览这个端点，看看谷歌提供的信息:

[https://accounts.google.com/.知名/openid 配置](https://web.archive.org/web/20220926202321/https://accounts.google.com/.well-known/openid-configuration)

例如，我们可以看到服务必须使用的授权、令牌和 UserInfo 端点，以及支持的范围。

**这里特别需要注意的是，如果服务启动时发现端点不可用，我们的应用将无法成功完成启动过程。**

## 7。OpenID 连接会话管理

该规范通过定义以下内容补充了核心功能:

*   持续监控终端用户在 OP 的登录状态的不同方法，以便 RP 可以注销已从 OpenID 提供者注销的终端用户
*   作为客户端注册的一部分，可以向 OP 注册 RP 注销 URIs，以便在最终用户注销 OP 时得到通知
*   **一种机制，用于通知操作员最终用户已从站点注销，并且可能也想从操作员处注销**

当然，并非所有的 OPs 都支持所有这些项目，其中一些解决方案只能通过用户代理在前端实现中实现。

在本教程中，我们将关注 Spring 为列表的最后一项——RP 发起的注销——提供的功能。

此时，如果我们登录到我们的应用程序，我们可以正常地访问每个端点。

如果我们注销(调用`/logout `端点),然后我们向一个安全的资源发出请求，我们将看到我们可以得到响应，而不必再次登录。

然而，这实际上并不正确。如果我们检查浏览器调试控制台中的 Network 选项卡，我们将看到当我们第二次点击安全端点时，我们被重定向到 OP 授权端点。因为我们仍然在那里登录，所以流程是透明地完成的，几乎立即到达安全的端点。

当然，在某些情况下，这可能不是我们想要的行为。让我们看看如何实现这个 OIDC 机制来处理这个问题。

### 7.1.OpenID 提供程序配置

在这种情况下，我们将配置并使用 Okta 实例作为我们的 OpenID 提供者。我们不会详细讨论如何创建实例，但是我们可以遵循本指南中的[步骤，记住 Spring Security 的默认回调端点将是`/login/oauth2/code/okta`。](https://web.archive.org/web/20220926202321/https://help.okta.com/en/prod/Content/Topics/Apps/apps-about-oidc.htm)

在我们的应用程序中，我们可以用属性定义客户端注册数据:

```java
spring:
  security:
    oauth2:
      client:
        registration: 
          okta: 
            client-id: <client-id>
            client-secret: <secret>
        provider:
          okta:
            issuer-uri: https://dev-123.okta.com
```

OIDC 表示 OP 注销端点可以在发现文档中指定为`end_session_endpoint `元素。

### 7.2.`LogoutSuccessHandler`配置

接下来，我们必须通过提供一个定制的`LogoutSuccessHandler` 实例来配置`HttpSecurity `注销逻辑:

```java
@Override
protected void configure(HttpSecurity http) throws Exception {
    http
      .authorizeRequests(authorizeRequests -> authorizeRequests
        .mvcMatchers("/home").permitAll()
        .anyRequest().authenticated())
      .oauth2Login(oauthLogin -> oauthLogin.permitAll())
      .logout(logout -> logout
        .logoutSuccessHandler(oidcLogoutSuccessHandler()));
}
```

现在让我们看看如何使用 Spring Security 提供的特殊类`OidcClientInitiatedLogoutSuccessHandler`创建一个`LogoutSuccessHandler` :

```java
@Autowired
private ClientRegistrationRepository clientRegistrationRepository;

private LogoutSuccessHandler oidcLogoutSuccessHandler() {
    OidcClientInitiatedLogoutSuccessHandler oidcLogoutSuccessHandler =
      new OidcClientInitiatedLogoutSuccessHandler(
        this.clientRegistrationRepository);

    oidcLogoutSuccessHandler.setPostLogoutRedirectUri(
      URI.create("http://localhost:8081/home"));

    return oidcLogoutSuccessHandler;
}
```

因此，我们需要在 OP 客户端配置面板中将这个 URI 设置为有效的注销重定向 URI。

显然，OP 注销配置包含在客户机注册设置中，因为我们用来配置处理程序的只是上下文中的`ClientRegistrationRepository ` bean。

那么，现在会发生什么呢？

登录应用程序后，我们可以向 Spring Security 提供的`/logout `端点发送请求。

如果我们在浏览器调试控制台中检查网络日志，**我们将看到在最终访问我们配置的重定向 URI 之前，我们被重定向到一个 OP 注销端点。**

下次我们访问应用中需要认证的端点时，我们必须再次登录我们的 OP 平台以获得许可。

## 8。结论

总之，在本文中，我们了解了很多关于 OpenID Connect 提供的解决方案，以及我们如何使用 Spring Security 实现其中的一些。

和往常一样，所有完整的例子都可以在 GitHub 上找到[。](https://web.archive.org/web/20220926202321/https://github.com/eugenp/tutorials/tree/master/spring-security-modules/spring-security-oidc)