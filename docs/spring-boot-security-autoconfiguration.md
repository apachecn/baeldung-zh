# Spring Boot 安全自动配置

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-boot-security-autoconfiguration>

## 1。概述

在本教程中，我们将看看 Spring Boot 固执己见的安全方法。

简而言之，我们将关注默认的安全配置，以及如何在需要时禁用或定制它。

## 延伸阅读:

## [Spring Security–安全无、过滤器无、访问许可全部](/web/20220701015845/https://www.baeldung.com/security-none-filters-none-access-permitAll)

The differences between access="permitAll", filters="none", security="none" in Spring Security.[Read more](/web/20220701015845/https://www.baeldung.com/security-none-filters-none-access-permitAll) →

## [春季安全表单登录](/web/20220701015845/https://www.baeldung.com/spring-security-login)

A Spring Login Example - How to Set Up a simple Login Form, a Basic Security XML Configuration and some more Advanced Configuration Techniques.[Read more](/web/20220701015845/https://www.baeldung.com/spring-security-login) →

## 2。默认安全设置

为了给我们的 Spring Boot 应用程序增加安全性，我们需要添加`security starter dependency`:

```java
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-security</artifactId>
</dependency>
```

这也将包括包含初始/默认安全配置的`SecurityAutoConfiguration` 类。

注意，我们没有在这里指定版本，假设项目已经使用 Boot 作为父项目。

**默认情况下，对应用程序启用认证。此外，内容协商用于确定应该使用 basic 还是 formLogin。**

有一些预定义的属性:

```java
spring.security.user.name
spring.security.user.password
```

如果我们不使用预定义属性`spring.security.user.password` 配置密码并启动应用程序，默认密码会随机生成并打印在控制台日志中:

```java
Using default security password: c8be15de-4488-4490-9dc6-fab3f91435c6
```

有关更多默认值，请参见 [Spring Boot 通用应用程序属性](https://web.archive.org/web/20220701015845/https://docs.spring.io/spring-boot/docs/current/reference/html/appendix-application-properties.html)参考页面的安全属性部分。

## 3。禁用自动配置

要放弃安全自动配置并添加我们自己的配置，我们需要排除`SecurityAutoConfiguration`类。

我们可以通过简单的排除来做到这一点:

```java
@SpringBootApplication(exclude = { SecurityAutoConfiguration.class })
public class SpringBootSecurityApplication {

    public static void main(String[] args) {
        SpringApplication.run(SpringBootSecurityApplication.class, args);
    }
} 
```

或者我们可以在`application.properties`文件中添加一些配置:

```java
spring.autoconfigure.exclude=org.springframework.boot.autoconfigure.security.SecurityAutoConfiguration
```

然而，也有一些特殊的情况，这种设置是不够的。

例如，几乎每个 Spring Boot 应用程序都是用类路径中的 Actuator 启动的。这导致了一些问题，因为另一个自动配置类需要我们刚刚排除的那个。因此，应用程序将无法启动。

为了解决这个问题，我们需要排除那个类；并且，具体到执行器的情况，我们还需要排除`ManagementWebSecurityAutoConfiguration`。

### 3.1。禁用 vs 超越安全自动配置

禁用自动配置和超越它有很大的区别。

禁用它就像从头开始添加 Spring 安全依赖项和整个设置一样。这在以下几种情况下很有用:

1.  将应用程序安全性与自定义安全提供程序集成
2.  将已有安全设置的遗留 Spring 应用程序迁移到 Spring Boot

但是大多数时候我们不需要完全禁用安全自动配置。

这是因为 Spring Boot 被配置为允许通过添加我们的新/自定义配置类来超越自动配置的安全性。这通常更容易，因为我们只是定制现有的安全设置来满足我们的需求。

## 4。配置 Spring Boot 安全性

如果我们选择了禁用安全自动配置，我们自然需要提供我们自己的配置。

正如我们之前所讨论的，这是默认的安全配置。然后我们通过修改属性文件来定制它。

例如，我们可以通过添加自己的密码来覆盖默认密码:

```java
spring.security.user.password=password
```

如果我们想要一个更灵活的配置，比如多个用户和角色，我们需要利用一个完整的`@Configuration`类:

```java
@Configuration
@EnableWebSecurity
public class BasicConfiguration extends WebSecurityConfigurerAdapter {

    @Override
    protected void configure(AuthenticationManagerBuilder auth) throws Exception {
    	PasswordEncoder encoder = 
          PasswordEncoderFactories.createDelegatingPasswordEncoder();
    	auth
          .inMemoryAuthentication()
          .withUser("user")
          .password(encoder.encode("password"))
          .roles("USER")
          .and()
          .withUser("admin")
          .password(encoder.encode("admin"))
          .roles("USER", "ADMIN");
    }

    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http
          .authorizeRequests()
          .anyRequest()
          .authenticated()
          .and()
          .httpBasic();
    }
}
```

**如果我们禁用默认安全配置，`@EnableWebSecurity`注释是至关重要的。**

如果缺少它，应用程序将无法启动。因此，只有当我们使用`WebSecurityConfigurerAdapter`覆盖默认行为时，注释才是可选的。

另外，请注意，在使用 Spring Boot 2 时，我们**需要使用`PasswordEncoder`来设置密码。**要了解更多细节，请参阅我们关于 Spring Security 5 中[默认密码编码器的指南。](/web/20220701015845/https://www.baeldung.com/spring-security-5-default-password-encoder)

现在，我们应该通过几个快速现场测试来验证我们的安全配置是否正确应用:

```java
@RunWith(SpringRunner.class)
@SpringBootTest(webEnvironment = RANDOM_PORT)
public class BasicConfigurationIntegrationTest {

    TestRestTemplate restTemplate;
    URL base;
    @LocalServerPort int port;

    @Before
    public void setUp() throws MalformedURLException {
        restTemplate = new TestRestTemplate("user", "password");
        base = new URL("http://localhost:" + port);
    }

    @Test
    public void whenLoggedUserRequestsHomePage_ThenSuccess()
     throws IllegalStateException, IOException {
        ResponseEntity<String> response =
          restTemplate.getForEntity(base.toString(), String.class);

        assertEquals(HttpStatus.OK, response.getStatusCode());
        assertTrue(response.getBody().contains("Baeldung"));
    }

    @Test
    public void whenUserWithWrongCredentials_thenUnauthorizedPage() 
      throws Exception {

        restTemplate = new TestRestTemplate("user", "wrongpassword");
        ResponseEntity<String> response =
          restTemplate.getForEntity(base.toString(), String.class);

        assertEquals(HttpStatus.UNAUTHORIZED, response.getStatusCode());
        assertTrue(response.getBody().contains("Unauthorized"));
    }
}
```

事实上，Spring Security 是 Spring Boot Security 的后台，因此任何可以用它完成的安全配置或它支持的任何集成也可以在 Spring Boot 中实现。

## 5。Spring Boot OAuth2 自动配置(使用传统堆栈)

Spring Boot 为 OAuth2 提供了专门的自动配置支持。

Spring Boot 1.x 附带的 [Spring Security OAuth](https://web.archive.org/web/20220701015845/https://docs.spring.io/spring-security-oauth2-boot/docs/current/reference/html5/) 支持在后来的引导版本中被移除，取代了与 [Spring Security 5](https://web.archive.org/web/20220701015845/https://docs.spring.io/spring-security/site/docs/5.2.12.RELEASE/reference/html/oauth2.html) 捆绑的一流 OAuth 支持。我们将在下一节看到如何使用它。

对于遗留堆栈(使用 Spring Security OAuth)，我们首先需要添加一个 Maven 依赖项来开始设置我们的应用程序:

```java
<dependency>
   <groupId>org.springframework.security.oauth</groupId>
   <artifactId>spring-security-oauth2</artifactId>
</dependency>
```

这个依赖项包括一组能够触发在`OAuth2AutoConfiguration`类中定义的自动配置机制的类。

现在，根据我们的应用范围，我们有多种选择来继续。

### 5.1。OAuth2 授权服务器自动配置

如果我们希望我们的应用程序成为 OAuth2 提供者，我们可以使用`@EnableAuthorizationServer`。

启动时，我们会在日志中注意到，自动配置类将为我们的授权服务器生成一个客户端 id 和一个客户端密码，当然还有一个用于基本身份验证的随机密码:

```java
Using default security password: a81cb256-f243-40c0-a585-81ce1b952a98
security.oauth2.client.client-id = 39d2835b-1f87-4a77-9798-e2975f36972e
security.oauth2.client.client-secret = f1463f8b-0791-46fe-9269-521b86c55b71
```

这些凭证可用于获取访问令牌:

```java
curl -X POST -u 39d2835b-1f87-4a77-9798-e2975f36972e:f1463f8b-0791-46fe-9269-521b86c55b71 \
 -d grant_type=client_credentials 
 -d username=user 
 -d password=a81cb256-f243-40c0-a585-81ce1b952a98 \
 -d scope=write  http://localhost:8080/oauth/token
```

我们的另一篇文章提供了关于这个主题的更多细节。

### 5.2。其他 Spring Boot OAuth2 自动配置设置

Spring Boot OAuth2 还介绍了其他一些使用案例:

1.  [资源服务器](/web/20220701015845/https://www.baeldung.com/rest-api-spring-oauth2-angular-legacy#resource)–`@EnableResourceServer`
2.  [客户端应用](/web/20220701015845/https://www.baeldung.com/sso-spring-security-oauth2-legacy)–`@EnableOAuth2Sso`或`@EnableOAuth2Client`

如果我们需要我们的应用程序是这些类型中的一种，我们只需要向应用程序属性添加一些配置，如链接所详述的。

所有 OAuth2 特定属性可在 [Spring Boot 通用应用属性](https://web.archive.org/web/20220701015845/https://docs.spring.io/spring-boot/docs/current/reference/html/appendix-application-properties.html)中找到。

## 6。Spring Boot OAuth2 自动配置(使用新堆栈)

要使用新的堆栈，我们需要根据我们想要配置的内容添加依赖项—授权服务器、资源服务器或客户端应用程序。

我们一个一个来看。

### 6.1。OAuth2 授权服务器支持

正如我们所看到的，Spring Security OAuth 堆栈提供了将授权服务器设置为 Spring 应用程序的可能性。但是该项目已经被弃用，Spring 目前还不支持自己的授权服务器。相反，建议使用现有的成熟提供商，如 Okta、Keycloak 和 ForgeRock。

然而，Spring Boot 让我们很容易配置这样的提供者。关于键盘锁配置的例子，我们可以参考[Spring Boot](/web/20220701015845/https://www.baeldung.com/spring-boot-keycloak)使用键盘锁的快速指南或者[嵌入 Spring Boot 应用](/web/20220701015845/https://www.baeldung.com/keycloak-embedded-in-spring-boot-app)的键盘锁。

### 6.2。 **OAuth2 资源服务器支持**

要包含对资源服务器的支持，我们需要添加以下依赖项:

```java
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-oauth2-resource-server</artifactId>    
</dependency>
```

要了解最新版本信息，请访问 [Maven Central](https://web.archive.org/web/20220701015845/https://search.maven.org/search?q=a:spring-boot-starter-oauth2-resource-server) 。

另外，在我们的安全配置中，我们需要包括 [`oauth2ResourceServer()`](https://web.archive.org/web/20220701015845/https://docs.spring.io/spring-security-oauth2-boot/docs/2.2.x-SNAPSHOT/reference/html/boot-features-security-oauth2-resource-server.html) DSL:

```java
@Configuration
public class JWTSecurityConfig extends WebSecurityConfigurerAdapter {

    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http
          ...
          .oauth2ResourceServer(oauth2 -> oauth2.jwt());
          ...
	}
}
```

我们的带有 Spring Security 5 的 OAuth 2.0 资源服务器对这个话题进行了深入的探讨。

### 6.3。 **OAuth2 客户端支持**

类似于我们如何配置资源服务器，客户机应用程序也需要它自己的依赖项和 DSL。

以下是 OAuth2 客户端支持的具体依赖关系:

```java
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-oauth2-client</artifactId>
</dependency> 
```

最新版本可以在 [Maven Central](https://web.archive.org/web/20220701015845/https://search.maven.org/search?q=a:spring-boot-starter-oauth2-client) 找到。

Spring Security 5 还通过其 [`oath2Login()`](/web/20220701015845/https://www.baeldung.com/spring-security-5-oauth2-login) DSL 提供一流的登录支持。

有关新堆栈中 SSO 支持的详细信息，请参考我们的文章[使用 Spring Security OAuth 的简单单点登录 2](/web/20220701015845/https://www.baeldung.com/sso-spring-security-oauth2) 。

## 7.Spring Boot 2 号保安队对 Spring Boot 1 号保安队

与 Spring Boot 1 相比，**“Spring Boot 2”大大简化了自动配置。**

在 Spring Boot 2 中，如果我们想要自己的安全配置，我们可以简单地添加一个自定义的`[WebSecurityConfigurerAdapter](/web/20220701015845/https://www.baeldung.com/java-config-spring-security).`这将禁用默认的自动配置，并启用我们的自定义安全配置。

Spring Boot 2 也使用了大多数 Spring Security 的默认设置。因此，**在 Spring Boot 协议 1 中，一些默认情况下不安全的终端现在默认情况下是安全的。**

这些端点包括静态资源，如/css/**、/js/*img/**、/webjars/**、/**/favicon.ico 和错误端点。如果我们需要允许对这些端点进行未经身份验证的访问，我们可以显式地进行配置。

为了简化与安全相关的配置， **Spring Boot 新协议删除了 Spring Boot 1 协议的以下属性**:

```java
security.basic.authorize-mode
security.basic.enabled
security.basic.path
security.basic.realm
security.enable-csrf
security.headers.cache
security.headers.content-security-policy
security.headers.content-security-policy-mode
security.headers.content-type
security.headers.frame
security.headers.hsts
security.headers.xss
security.ignored
security.require-ssl
security.sessions
```

## 8。结论

在本文中，我们主要关注 Spring Boot 提供的默认安全配置。我们看到了安全自动配置机制是如何被禁用或覆盖的。然后，我们看了如何应用新的安全配置。

OAuth2 的源代码可以在我们的 OAuth2 GitHub 存储库中找到，用于[遗留](https://web.archive.org/web/20220701015845/https://github.com/Baeldung/spring-security-oauth/tree/master/oauth-legacy)和[新](https://web.archive.org/web/20220701015845/https://github.com/Baeldung/spring-security-oauth/tree/master/oauth-rest)栈。其余的代码可以在 GitHub 的[中找到。](https://web.archive.org/web/20220701015845/https://github.com/eugenp/tutorials/tree/master/spring-boot-modules/spring-boot-security)