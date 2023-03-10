# 春季课程指南

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-session>

## 1。概述

`Spring Session` 的简单目标是将会话管理从存储在服务器中的 HTTP 会话的限制中解放出来。

该解决方案使得在云中的服务之间共享会话数据变得容易，而不局限于单个容器(例如 Tomcat)。此外，它支持在同一浏览器中的多个会话，并在标题中发送会话。

在本文中，我们将使用`Spring Session` 来管理 web 应用程序中的认证信息。虽然`Spring Session`可以使用 JDBC、Gemfire 或 MongoDB 持久化数据，但我们将使用`Redis`。

关于`Redis`的介绍，请看[这篇](/web/20220920100541/https://www.baeldung.com/spring-data-redis-tutorial)的文章。

## 2。一个简单的项目

让我们首先创建一个简单的`Spring Boot`项目，作为我们稍后的会话示例的基础:

```java
<parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>2.7.2</version>
    <relativePath/>
</parent>

<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-security</artifactId>
    </dependency>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-test</artifactId>
        <scope>test</scope>
    </dependency>
</dependencies>
```

我们的应用程序使用`Spring Boot`运行，父 pom 为每个条目提供版本。每个依赖的最新版本可以在这里找到:[spring-boot-starter-security](https://web.archive.org/web/20220920100541/https://search.maven.org/classic/#search%7Cgav%7C1%7Cg%3A%22org.springframework.boot%22%20AND%20a%3A%22spring-boot-starter-security%22)， [spring-boot-starter-web](https://web.archive.org/web/20220920100541/https://search.maven.org/classic/#search%7Cgav%7C1%7Cg%3A%22org.springframework.boot%22%20AND%20a%3A%22spring-boot-starter-web%22) ， [spring-boot-starter-test](https://web.archive.org/web/20220920100541/https://search.maven.org/classic/#search%7Cgav%7C1%7Cg%3A%22org.springframework.boot%22%20AND%20a%3A%22spring-boot-starter-test%22) 。

让我们在`application.properties`中为我们的 Redis 服务器添加一些配置属性:

```java
spring.redis.host=localhost
spring.redis.port=6379
```

## 3。Spring Boot 配置

对于 Spring Boot，**添加以下依赖关系**就足够了，自动配置会处理剩下的事情:

```java
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-redis</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.session</groupId>
    <artifactId>spring-session-data-redis</artifactId>
</dependency>
```

我们在这里使用引导父节点`pom`来设置版本，因此这些版本保证可以与我们的其他依赖项一起工作。每个依赖的最新版本可以在这里找到:[spring-boot-starter-data-redis](https://web.archive.org/web/20220920100541/https://search.maven.org/classic/#search%7Cgav%7C1%7Cg%3A%22org.springframework.boot%22%20AND%20a%3A%22spring-boot-starter-data-redis%22)， [spring-session](https://web.archive.org/web/20220920100541/https://search.maven.org/classic/#search%7Cgav%7C1%7Cg%3A%22org.springframework.session%22%20AND%20a%3A%22spring-session%22) 。

## 4。标准弹簧配置(无启动)

让我们也看看没有 Spring Boot 的集成和配置`spring-session`-只有普通弹簧。

### 4.1。依赖性

首先，如果我们将`spring-session`添加到一个标准的 Spring 项目中，我们需要显式地定义:

```java
<dependency>
    <groupId>org.springframework.session</groupId>
    <artifactId>spring-session</artifactId>
    <version>1.2.2.RELEASE</version>
</dependency>
<dependency>
    <groupId>org.springframework.data</groupId>
    <artifactId>spring-data-redis</artifactId>
    <version>1.5.0.RELEASE</version>
</dependency>
```

这些模块的最新版本可以在这里找到: [spring-session](https://web.archive.org/web/20220920100541/https://search.maven.org/classic/#search%7Cgav%7C1%7Cg%3A%22org.springframework.session%22%20AND%20a%3A%22spring-session%22) ， [spring-data-redis](https://web.archive.org/web/20220920100541/https://search.maven.org/classic/#search%7Cgav%7C1%7Cg%3A%22org.springframework.data%22%20AND%20a%3A%22spring-data-redis%22) 。

### 4.2。春季会议配置

现在让我们为`Spring Session`添加一个配置类:

```java
@Configuration
@EnableRedisHttpSession
public class SessionConfig extends AbstractHttpSessionApplicationInitializer {
    @Bean
    public JedisConnectionFactory connectionFactory() {
        return new JedisConnectionFactory();
    }
}
```

`@EnableRedisHttpSession`和`AbstractHttpSessionApplicationInitializer`的扩展将在我们所有的安全基础设施前创建并连接一个过滤器，以寻找活动会话，并根据存储在`Redis`中的值填充安全上下文。

现在让我们用一个控制器和安全配置来完成这个应用程序。

## 5。应用程序配置

导航到我们的主应用程序文件并添加一个控制器:

```java
@RestController
public class SessionController {
    @RequestMapping("/")
    public String helloAdmin() {
        return "hello admin";
    }
}
```

这将为我们提供一个测试端点。

接下来，添加我们的安全配置类:

```java
@Configuration
@EnableWebSecurity
public class SecurityConfig {

    @Bean
    public InMemoryUserDetailsManager userDetailsService(PasswordEncoder passwordEncoder) {
        UserDetails user = User.withUsername("admin")
            .password(passwordEncoder.encode("password"))
            .roles("ADMIN")
            .build();
        return new InMemoryUserDetailsManager(user);
    }

    @Bean
    public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
        http.httpBasic()
            .and()
            .authorizeRequests()
            .antMatchers("/")
            .hasRole("ADMIN")
            .anyRequest()
            .authenticated();
        return http.build();
    }

    @Bean 
    public PasswordEncoder passwordEncoder() { 
        return new BCryptPasswordEncoder(); 
    } 
}
```

这通过基本身份验证保护了我们的端点，并设置了一个用户进行测试。

## 6。测试

最后，让我们测试所有的东西——我们将在这里定义一个简单的测试，它允许我们做两件事:

*   使用实时 web 应用程序
*   说话再说一遍

让我们先来设置一下:

```java
public class SessionControllerTest {

    private Jedis jedis;
    private TestRestTemplate testRestTemplate;
    private TestRestTemplate testRestTemplateWithAuth;
    private String testUrl = "http://localhost:8080/";

    @Before
    public void clearRedisData() {
        testRestTemplate = new TestRestTemplate();
        testRestTemplateWithAuth = new TestRestTemplate("admin", "password", null);

        jedis = new Jedis("localhost", 6379);
        jedis.flushAll();
    }
}
```

注意我们是如何设置这两个客户机的 HTTP 客户机和 Redis 客户机。当然，此时服务器(和 Redis)应该已经启动并运行——这样我们就可以通过这些测试与它们进行通信。

让我们从测试`Redis`为空开始:

```java
@Test
public void testRedisIsEmpty() {
    Set<String> result = jedis.keys("*");
    assertEquals(0, result.size());
}
```

现在测试我们的安全性是否为未经验证的请求返回 401:

```java
@Test
public void testUnauthenticatedCantAccess() {
    ResponseEntity<String> result = testRestTemplate.getForEntity(testUrl, String.class);
    assertEquals(HttpStatus.UNAUTHORIZED, result.getStatusCode());
}
```

接下来，我们测试`Spring Session`正在管理我们的认证令牌:

```java
@Test
public void testRedisControlsSession() {
    ResponseEntity<String> result = testRestTemplateWithAuth.getForEntity(testUrl, String.class);
    assertEquals("hello admin", result.getBody()); //login worked

    Set<String> redisResult = jedis.keys("*");
    assertTrue(redisResult.size() > 0); //redis is populated with session data

    String sessionCookie = result.getHeaders().get("Set-Cookie").get(0).split(";")[0];
    HttpHeaders headers = new HttpHeaders();
    headers.add("Cookie", sessionCookie);
    HttpEntity<String> httpEntity = new HttpEntity<>(headers);

    result = testRestTemplate.exchange(testUrl, HttpMethod.GET, httpEntity, String.class);
    assertEquals("hello admin", result.getBody()); //access with session works worked

    jedis.flushAll(); //clear all keys in redis

    result = testRestTemplate.exchange(testUrl, HttpMethod.GET, httpEntity, String.class);
    assertEquals(HttpStatus.UNAUTHORIZED, result.getStatusCode());
    //access denied after sessions are removed in redis
}
```

首先，我们的测试使用管理员身份验证凭证确认我们的请求是成功的。

然后，我们从响应头中提取会话值，并在第二个请求中将其用作身份验证。我们对此进行验证，然后清除`Redis`中的所有数据。

最后，我们使用会话 cookie 发出另一个请求，并确认我们已注销。这证实了`Spring Session`正在管理我们的会话。

## 7。结论

`Spring Session`是管理 HTTP 会话的强大工具。随着我们的会话存储简化为一个配置类和一些 Maven 依赖项，我们现在可以将多个应用程序连接到同一个`Redis`实例并共享认证信息。

像往常一样，所有的例子都可以在 Github 上找到[。](https://web.archive.org/web/20220920100541/https://github.com/eugenp/tutorials/tree/master/spring-web-modules/spring-session)