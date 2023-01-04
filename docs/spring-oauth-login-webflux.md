# 使用 WebFlux 的 Spring Security OAuth 登录

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-oauth-login-webflux>

## 1。概述

Spring Security 从 5.1.x GA 开始增加了对 WebFlux 的 OAuth 支持。

我们将讨论**如何配置我们的 WebFlux 应用程序来使用 OAuth2 登录支持**。我们还将讨论如何使用`WebClient`来访问 OAuth2 安全资源。

Webflux 的 OAuth 登录配置类似于标准 Web MVC 应用程序的配置。关于这方面的更多细节，也可以看看我们关于[弹簧`OAuth2Login`元件](/web/20220628061843/https://www.baeldung.com/spring-security-5-oauth2-login)的文章。

## 2。Maven 配置

首先，我们将创建一个简单的 Spring Boot 应用程序，并将这些依赖项添加到我们的`pom.xml`:

```java
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-security</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-webflux</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.security</groupId>
    <artifactId>spring-security-oauth2-client</artifactId>
</dependency>
```

在 Maven Central 上可以获得[spring-boot-starter-security](https://web.archive.org/web/20220628061843/https://search.maven.org/search?q=a:spring-boot-starter-security%20AND%20g:org.springframework.boot)、[spring-boot-starter-web flux](https://web.archive.org/web/20220628061843/https://search.maven.org/search?q=a:spring-boot-starter-webflux)和[spring-security-oauth 2-client](https://web.archive.org/web/20220628061843/https://search.maven.org/search?q=a:spring-security-oauth2-client%20AND%20g:org.springframework.security)依赖项。

## 3。主控制器

接下来，我们将添加一个简单的控制器来在主页上显示用户名:

```java
@RestController
public class MainController {

    @GetMapping("/")
    public Mono<String> index(@AuthenticationPrincipal Mono<OAuth2User> oauth2User) {
       return oauth2User
        .map(OAuth2User::getName)
        .map(name -> String.format("Hi, %s", name));
    }
}
```

请注意，**我们将显示从 OAuth2 客户端`UserInfo`端点**获得的用户名。

## 4。使用谷歌登录

现在，我们将配置我们的应用程序来支持使用 Google 登录。

首先，我们需要在 [Google 开发者控制台](https://web.archive.org/web/20220628061843/https://console.developers.google.com/)创建一个新项目

现在，我们需要添加 OAuth2 凭证(创建凭证> OAuth 客户端 ID)。

接下来，我们将把它添加到“授权重定向 URIs”:

```java
http://localhost:8080/login/oauth2/code/google
```

然后，**我们需要配置我们的`application.yml`来使用客户端 ID 和秘密**:

```java
spring:
  security:
    oauth2:
      client:
        registration:
          google:
            client-id: YOUR_APP_CLIENT_ID
            client-secret: YOUR_APP_CLIENT_SECRET
```

因为在我们的道路上有`spring-security-oauth2-client`,我们的应用程序将是安全的。

用户将被重定向到使用 Google 登录，然后才能访问我们的主页。

## 5。使用授权提供商登录

我们还可以配置我们的应用程序从定制的授权服务器登录。

在下面的例子中，我们将使用来自[上一篇文章](/web/20220628061843/https://www.baeldung.com/rest-api-spring-oauth2-angularjs)的授权服务器。

这一次，我们需要配置更多属性，而不仅仅是 ClientID 和 Client Secret:

```java
spring:
  security:
    oauth2:
      client:
        registration:
          custom:
            client-id: fooClientIdPassword
            client-secret: secret
            scopes: read,foo
            authorization-grant-type: authorization_code
            redirect-uri-template: http://localhost:8080/login/oauth2/code/custom
        provider:
          custom:
            authorization-uri: http://localhost:8081/spring-security-oauth-server/oauth/authorize
            token-uri: http://localhost:8081/spring-security-oauth-server/oauth/token
            user-info-uri: http://localhost:8088/spring-security-oauth-resource/users/extra
            user-name-attribute: user_name
```

在这种情况下，我们还需要为 OAuth2 客户端指定`scope,` `grant type`和`redirect URI`。我们还将提供授权服务器的`authorization` 和 `token URI`。

最后，我们还需要配置`UserInfo`端点，以便能够获得用户认证细节。

## 6。安全配置

默认情况下，Spring Security 保护所有路径。因此，如果我们只有一个 OAuth 客户端，我们将被重定向到授权这个客户端并登录。

如果注册了多个 OAuth 客户端，那么将自动创建一个登录页面来选择登录方法。

如果我们愿意，我们可以更改这一点，并且**会提供详细的安全配置**:

```java
@EnableWebFluxSecurity
public class SecurityConfig {

    @Bean
    public SecurityWebFilterChain configure(ServerHttpSecurity http) throws Exception {
        return http.authorizeExchange()
          .pathMatchers("/about").permitAll()
          .anyExchange().authenticated()
          .and().oauth2Login()
          .and().build();
    }
}
```

在本例中，我们保护了除“/about”之外的所有路径。

## 7。`WebClient`

除了使用 OAuth2 认证用户，我们还可以做更多的事情。**我们可以使用`OAuth2AuthorizedClient`** 使用`WebClient`来访问 OAuth2 安全资源。

现在，让我们配置我们的`WebClient`:

```java
@Bean
public WebClient webClient(ReactiveClientRegistrationRepository clientRegistrationRepo, 
  ServerOAuth2AuthorizedClientRepository authorizedClientRepo) {
    ServerOAuth2AuthorizedClientExchangeFilterFunction filter = 
      new ServerOAuth2AuthorizedClientExchangeFilterFunction(clientRegistrationRepo, authorizedClientRepo);

    return WebClient.builder().filter(filter).build();
}
```

然后，我们可以检索一个 OAuth2 安全资源:

```java
@Autowired
private WebClient webClient;

@GetMapping("/foos/{id}")
public Mono<Foo> getFooResource(@RegisteredOAuth2AuthorizedClient("custom") 
  OAuth2AuthorizedClient client, @PathVariable final long id){
    return webClient
      .get()
      .uri("http://localhost:8088/spring-security-oauth-resource/foos/{id}", id)
      .attributes(oauth2AuthorizedClient(client))
      .retrieve()
      .bodyToMono(Foo.class); 
}
```

请注意，**我们使用`AccessToken`从** `**OAuth2AuthorizedClient**.`检索远程资源`Foo`

## 8。结论

在这篇简短的文章中，我们学习了如何配置我们的 WebFlux 应用程序来使用 OAuth2 登录支持，以及如何使用 WebClient 来访问 OAuth2 安全资源。

和往常一样，完整的源代码可以在 GitHub 上找到[。](https://web.archive.org/web/20220628061843/https://github.com/eugenp/tutorials/tree/master/spring-5-reactive-modules/spring-5-reactive-oauth)