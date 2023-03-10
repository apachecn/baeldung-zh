# 用于反应式应用的 Spring Security 5

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-security-5-reactive>

## 1。简介

在本文中，我们将探索用于保护反应式应用程序的 [Spring Security 5](https://web.archive.org/web/20220707143840/https://spring.io/projects/spring-security) 框架的新特性。该释放与弹簧 5 和 Spring Boot 2 对齐。

在本文中，我们不会详细讨论反应式应用程序本身，这是 Spring 5 框架的一个新特性。请务必查看文章[反应堆堆芯介绍](/web/20220707143840/https://www.baeldung.com/reactor-core)了解更多详情。

## 2。Maven 设置

我们将使用 Spring Boot 启动器来引导我们的项目以及所有需要的依赖项。

基本设置需要父声明、web starter 和安全 starter 依赖项。我们还需要 Spring 安全测试框架:

```java
<parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>2.6.1</version>
    <relativePath/>
</parent>

<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-webflux</artifactId>
    </dependency>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-security</artifactId>
    </dependency>
    <dependency>
        <groupId>org.springframework.security</groupId>
        <artifactId>spring-security-test</artifactId>
        <scope>test</scope>
    </dependency>
</dependencies>
```

我们可以在 Maven Central 查看 Spring Boot 安全入门[的当前版本。](https://web.archive.org/web/20220707143840/https://search.maven.org/classic/#search%7Cga%7C1%7Cg%3A%22org.springframework.boot%22%20AND%20a%3A%22spring-boot-starter-security%22)

## 3。项目设置

### 3.1。启动反应式应用程序

我们不会使用标准的`@SpringBootApplication`配置，而是配置一个基于 Netty 的 web 服务器。Netty 是一个异步的基于 NIO 的框架，是反应式应用的良好基础。

`@EnableWebFlux`注释为应用程序启用了标准的 Spring Web 反应式配置:

```java
@ComponentScan(basePackages = {"com.baeldung.security"})
@EnableWebFlux
public class SpringSecurity5Application {

    public static void main(String[] args) {
        try (AnnotationConfigApplicationContext context 
         = new AnnotationConfigApplicationContext(
            SpringSecurity5Application.class)) {

            context.getBean(NettyContext.class).onClose().block();
        }
    }
```

这里，我们创建一个新的应用程序上下文，并通过调用 Netty 上下文上的`.onClose().block()`链来等待 Netty 关闭。

Netty 关闭后，将使用`try-with-resources`块自动关闭上下文。

我们还需要创建一个基于 Netty 的 HTTP 服务器，一个 HTTP 请求的处理程序，以及服务器和处理程序之间的适配器:

```java
@Bean
public NettyContext nettyContext(ApplicationContext context) {
    HttpHandler handler = WebHttpHandlerBuilder
      .applicationContext(context).build();
    ReactorHttpHandlerAdapter adapter 
      = new ReactorHttpHandlerAdapter(handler);
    HttpServer httpServer = HttpServer.create("localhost", 8080);
    return httpServer.newHandler(adapter).block();
}
```

### 3.2。Spring 安全配置类

对于我们的基本 Spring 安全配置，我们将创建一个配置类–`SecurityConfig`。

要在 Spring Security 5 中启用 WebFlux 支持，我们只需要指定`@EnableWebFluxSecurity`注释:

```java
@EnableWebFluxSecurity
public class SecurityConfig {
    // ...
}
```

**现在我们可以利用类`ServerHttpSecurity`来构建我们的安全配置。**

**这个类是 Spring 5 的新特性。**它类似于`HttpSecurity` builder，但是它只支持 WebFlux 应用程序。

`ServerHttpSecurity`已经预先配置了一些默认设置，所以我们可以完全跳过这个配置。但是对于初学者，我们将提供以下最低配置:

```java
@Bean
public SecurityWebFilterChain securityWebFilterChain(
  ServerHttpSecurity http) {
    return http.authorizeExchange()
      .anyExchange().authenticated()
      .and().build();
}
```

此外，我们需要一个用户详细信息服务。Spring Security 为我们提供了一个方便的模拟用户生成器和用户详细信息服务的内存实现:

```java
@Bean
public MapReactiveUserDetailsService userDetailsService() {
    UserDetails user = User
      .withUsername("user")
      .password(passwordEncoder().encode("password"))
      .roles("USER")
      .build();
    return new MapReactiveUserDetailsService(user);
}
```

既然我们处于被动领域，用户详细信息服务也应该是被动的。如果我们检查一下`ReactiveUserDetailsService`接口，**我们会看到它的`findByUsername`方法实际上返回了一个`Mono`发布者:**

```java
public interface ReactiveUserDetailsService {

    Mono<UserDetails> findByUsername(String username);
}
```

现在我们可以运行我们的应用程序并观察一个常规的 HTTP 基本身份验证表单。

## 4。样式化登录表单

Spring Security 5 中一个微小但引人注目的改进是使用 Bootstrap 4 CSS 框架的新风格登录表单。登录表单中的样式表链接到 CDN，所以我们只有在连接到互联网时才能看到改进。

要使用新的登录表单，让我们将相应的`formLogin()`构建器方法添加到`ServerHttpSecurity`构建器中:

```java
public SecurityWebFilterChain securityWebFilterChain(
  ServerHttpSecurity http) {
    return http.authorizeExchange()
      .anyExchange().authenticated()
      .and().formLogin()
      .and().build();
}
```

如果我们现在打开应用程序的主页，我们会看到它看起来比以前版本的 Spring Security 以来我们习惯的默认表单好得多:

[![](img/8829cf4254577a0c676e48963967a129.png)](/web/20220707143840/https://www.baeldung.com/wp-content/uploads/2017/11/2017-11-16_00-10-07.png)

请注意，这不是一个生产就绪的表单，但是它是我们应用程序的一个很好的引导。

如果我们现在登录，然后转到[http://localhost:8080/logout](https://web.archive.org/web/20220707143840/http://localhost:8080/logout)URL，我们将看到注销确认表单，它也是样式化的。

## 5。无功控制器安全

为了了解身份验证表单背后的东西，让我们实现一个简单的反应式控制器来问候用户:

```java
@RestController
public class GreetingController {

    @GetMapping("/")
    public Mono<String> greet(Mono<Principal> principal) {
        return principal
          .map(Principal::getName)
          .map(name -> String.format("Hello, %s", name));
    }

}
```

登录后，我们会看到问候语。让我们添加另一个只能由管理员访问的反应式处理程序:

```java
@GetMapping("/admin")
public Mono<String> greetAdmin(Mono<Principal> principal) {
    return principal
      .map(Principal::getName)
      .map(name -> String.format("Admin access: %s", name));
}
```

现在让我们在我们的用户详细信息服务中创建第二个角色为`ADMIN`的用户:

```java
UserDetails admin = User.withDefaultPasswordEncoder()
  .username("admin")
  .password("password")
  .roles("ADMIN")
  .build();
```

我们现在可以为 admin URL 添加一个匹配规则，要求用户拥有`ROLE_ADMIN`权限。

**注意，我们必须在`.anyExchange()`** 链调用之前放置匹配器。此调用适用于其他匹配器尚未覆盖的所有其他 URL:

```java
return http.authorizeExchange()
  .pathMatchers("/admin").hasAuthority("ROLE_ADMIN")
  .anyExchange().authenticated()
  .and().formLogin()
  .and().build();
```

如果我们现在用`user`或`admin`登录，我们会看到他们都遵守最初的问候，因为我们已经让所有经过身份验证的用户都可以访问它。

**但是只有`admin`的用户可以去[http://localhost:8080/admin](https://web.archive.org/web/20220707143840/http://localhost:8080/admin)网址，看到她的问候**。

## 6。反应式方法安全性

我们已经看到了如何保护 URL，但是方法呢？

要为反应式方法启用基于方法的安全性，我们只需向我们的`SecurityConfig`类添加`@EnableReactiveMethodSecurity`注释:

```java
@EnableWebFluxSecurity
@EnableReactiveMethodSecurity
public class SecurityConfig {
    // ...
}
```

现在让我们用以下内容创建一个反应式问候服务:

```java
@Service
public class GreetingService {

    public Mono<String> greet() {
        return Mono.just("Hello from service!");
    }
}
```

我们可以将它注入到控制器中，转到[http://localhost:8080/greeting service](https://web.archive.org/web/20220707143840/http://localhost:8080/greetService)并查看它实际工作的情况:

```java
@RestController
public class GreetingController {

    private GreetingService greetingService

    // constructor...

    @GetMapping("/greetingService")
    public Mono<String> greetingService() {
        return greetingService.greet();
    }

}
```

但是如果我们现在在具有`ADMIN`角色的服务方法上添加`@PreAuthorize`注释，那么普通用户将无法访问问候服务 URL:

```java
@Service
public class GreetingService {

    @PreAuthorize("hasRole('ADMIN')")
    public Mono<String> greet() {
        // ...
    }
}
```

## 7。在测试中嘲笑用户

让我们看看测试我们的反应式弹簧应用程序有多简单。

首先，我们将使用注入的应用程序上下文创建一个测试:

```java
@ContextConfiguration(classes = SpringSecurity5Application.class)
public class SecurityTest {

    @Autowired
    ApplicationContext context;

    // ...
}
```

现在我们将设置一个简单的反应式 web 测试客户端，这是 Spring 5 测试框架的一个特性:

```java
@Before
public void setup() {
    this.webTestClient = WebTestClient
      .bindToApplicationContext(this.context)
      .configureClient()
      .build();
}
```

这允许我们快速检查未授权用户是否从应用程序的主页重定向到登录页面:

```java
@Test
void whenNoCredentials_thenRedirectToLogin() {
    webTestClient.get()
      .uri("/")
      .exchange()
      .expectStatus().is3xxRedirection();
}
```

**如果我们现在将`@WithMockUser`注释添加到一个测试方法中，我们可以为这个方法提供一个经过验证的用户。**

该用户的登录名和密码分别为`user`和`password`，角色为`USER`。当然，这都可以用`@WithMockUser`注释参数进行配置。

现在，我们可以检查授权用户是否看到了问候语:

```java
@Test
@WithMockUser
void whenHasCredentials_thenSeesGreeting() {
    webTestClient.get()
      .uri("/")
      .exchange()
      .expectStatus().isOk()
      .expectBody(String.class).isEqualTo("Hello, user");
}
```

从 Spring Security 4 开始，`@WithMockUser`注释可用。然而，这也在 Spring Security 5 中进行了更新，以涵盖反应端点和方法。

## 8。结论

在本教程中，我们发现了即将发布的 Spring Security 5 的新特性，尤其是在反应式编程领域。

和往常一样，这篇文章的源代码可以在 GitHub 的[上找到。](https://web.archive.org/web/20220707143840/https://github.com/eugenp/tutorials/tree/master/spring-5-reactive-modules/spring-reactive)