# 使用 Spring Security OAuth2 的简单单点登录(传统堆栈)

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/sso-spring-security-oauth2-legacy>

## 1。概述

在本教程中，我们将讨论如何使用 Spring Security OAuth 和 Spring Boot 实现单点登录。

我们将使用三个独立的应用程序:

*   授权服务器——这是中央认证机制
*   两个客户端应用程序:使用 SSO 的应用程序

非常简单地说，当用户试图访问客户端应用程序中的安全页面时，他们将首先通过身份验证服务器重定向到身份验证。

我们将使用 OAuth2 中的`Authorization Code` 授权类型来驱动身份验证的委托。

**注**:本文使用的是 [Spring OAuth 遗留项目](https://web.archive.org/web/20220925074421/https://spring.io/projects/spring-security-oauth)。关于本文使用新的 Spring Security 5 堆栈的版本，请看我们的文章[使用 Spring Security OAuth2 的简单单点登录](/web/20220925074421/https://www.baeldung.com/sso-spring-security-oauth2)。

## 2。客户端应用

让我们从我们的客户端应用程序开始；当然，我们将使用 Spring Boot 来最小化配置:

### 2.1。Maven 依赖关系

首先，在我们的`pom.xml`中，我们需要以下依赖项:

```java
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-security</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.security.oauth.boot</groupId>
    <artifactId>spring-security-oauth2-autoconfigure</artifactId>
    <version>2.0.1.RELEASE</version>
</dependency>
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-thymeleaf</artifactId>
</dependency>
<dependency>
    <groupId>org.thymeleaf.extras</groupId>
    <artifactId>thymeleaf-extras-springsecurity4</artifactId>
</dependency>
```

### 2.2。安全配置

接下来，最重要的部分，我们的客户端应用程序的安全配置:

```java
@Configuration
@EnableOAuth2Sso
public class UiSecurityConfig extends WebSecurityConfigurerAdapter {

    @Override
    public void configure(HttpSecurity http) throws Exception {
        http.antMatcher("/**")
          .authorizeRequests()
          .antMatchers("/", "/login**")
          .permitAll()
          .anyRequest()
          .authenticated();
    }
}
```

这个配置的核心部分当然是我们用来启用单点登录的 [`@EnableOAuth2Sso`](https://web.archive.org/web/20220925074421/https://docs.spring.io/spring-boot/docs/1.5.7.RELEASE/api/org/springframework/boot/autoconfigure/security/oauth2/client/EnableOAuth2Sso.html) 注释。

注意，我们需要扩展`WebSecurityConfigurerAdapter`——没有它，所有的路径都将受到保护——因此当用户试图访问任何页面时，他们将被重定向登录。在我们的例子中，索引和登录页面是唯一不需要认证就可以访问的页面。

最后，我们还定义了一个`RequestContextListener` bean 来处理请求范围。

而`application.yml`:

```java
server:
    port: 8082
    servlet:
        context-path: /ui
    session:
      cookie:
        name: UISESSION
security:
  basic:
    enabled: false
  oauth2:
    client:
      clientId: SampleClientId
      clientSecret: secret
      accessTokenUri: http://localhost:8081/auth/oauth/token
      userAuthorizationUri: http://localhost:8081/auth/oauth/authorize
    resource:
      userInfoUri: http://localhost:8081/auth/user/me
spring:
  thymeleaf:
    cache: false
```

一些简短的注释:

*   我们禁用了默认的基本身份验证
*   `accessTokenUri`是获取访问令牌的 URI
*   `userAuthorizationUri`是用户将被重定向到的授权 URI
*   `userInfoUri`用户端点的 URI，用于获取当前用户的详细信息

还要注意，在我们的例子中，我们部署了授权服务器，当然我们也可以使用其他第三方提供商，比如脸书或 GitHub。

### 2.3。前端

现在，让我们看看我们的客户端应用程序的前端配置。我们不会在这里集中讨论这个问题，主要是因为我们已经在网站上介绍过了[。](/web/20220925074421/https://www.baeldung.com/spring-thymeleaf-3)

我们的客户端应用程序有一个非常简单的前端；下面是`index.html`:

```java
<h1>Spring Security SSO</h1>
<a href="securedPage">Login</a>
```

而`securedPage.html`:

```java
<h1>Secured Page</h1>
Welcome, <span th:text="${#authentication.name}">Name</span>
```

页面需要对用户进行身份验证。如果一个未经认证的用户试图访问`securedPage.html`，他们将首先被重定向到登录页面。

## 3。认证服务器

现在让我们在这里讨论我们的授权服务器。

### 3.1。Maven 依赖关系

首先，我们需要在我们的`pom.xml`中定义依赖关系:

```java
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.security.oauth</groupId>
    <artifactId>spring-security-oauth2</artifactId>
    <version>2.3.3.RELEASE</version>
</dependency>
```

### 3.2。OAuth 配置

理解这一点很重要，在这里我们将授权服务器和资源服务器作为一个可部署的单元一起运行。

让我们从资源服务器的配置开始——它也是我们的主要启动应用程序:

```java
@SpringBootApplication
@EnableResourceServer
public class AuthorizationServerApplication extends SpringBootServletInitializer {
    public static void main(String[] args) {
        SpringApplication.run(AuthorizationServerApplication.class, args);
    }
}
```

然后，我们将配置我们的授权服务器:

```java
@Configuration
@EnableAuthorizationServer
public class AuthServerConfig extends AuthorizationServerConfigurerAdapter {

    @Autowired
    private BCryptPasswordEncoder passwordEncoder;

    @Override
    public void configure(
      AuthorizationServerSecurityConfigurer oauthServer) throws Exception {
        oauthServer.tokenKeyAccess("permitAll()")
          .checkTokenAccess("isAuthenticated()");
    }

    @Override
    public void configure(ClientDetailsServiceConfigurer clients) throws Exception {
        clients.inMemory()
          .withClient("SampleClientId")
          .secret(passwordEncoder.encode("secret"))
          .authorizedGrantTypes("authorization_code")
          .scopes("user_info")
          .autoApprove(true) 
          .redirectUris(
            "http://localhost:8082/ui/login","http://localhost:8083/ui2/login"); 
    }
}
```

请注意，我们只是使用`authorization_code`授权类型来启用一个简单的客户端。

此外，请注意`autoApprove`是如何设置为 true 的，这样我们就不会被重定向和提升为手动批准任何范围。

### 3.3。安全配置

首先，我们将禁用默认的基本认证，通过我们的`application.properties`:

```java
server.port=8081
server.servlet.context-path=/auth
```

现在，让我们转到配置，定义一个简单的表单登录机制:

```java
@Configuration
@Order(1)
public class SecurityConfig extends WebSecurityConfigurerAdapter {

    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http.requestMatchers()
          .antMatchers("/login", "/oauth/authorize")
          .and()
          .authorizeRequests()
          .anyRequest().authenticated()
          .and()
          .formLogin().permitAll();
    }

    @Override
    protected void configure(AuthenticationManagerBuilder auth) throws Exception {
        auth.inMemoryAuthentication()
            .withUser("john")
            .password(passwordEncoder().encode("123"))
            .roles("USER");
    }

    @Bean 
    public BCryptPasswordEncoder passwordEncoder(){ 
        return new BCryptPasswordEncoder(); 
    }
}
```

注意，我们使用了简单的内存认证，但是我们可以简单地用自定义的`userDetailsService`来代替它。

### 3.4。用户端点

最后，我们将创建之前在配置中使用的用户端点:

```java
@RestController
public class UserController {
    @GetMapping("/user/me")
    public Principal user(Principal principal) {
        return principal;
    }
}
```

自然，这将返回带有 JSON 表示的用户数据。

## 4。结论

在这个快速教程中，我们重点介绍了使用 Spring Security Oauth2 和 Spring Boot 实现单点登录。

和往常一样，完整的源代码可以在 GitHub 上找到[。](https://web.archive.org/web/20220925074421/https://github.com/eugenp/tutorials/tree/master/spring-security-modules/spring-security-oauth2-sso)