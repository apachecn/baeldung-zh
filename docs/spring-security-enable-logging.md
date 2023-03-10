# 为 Spring 安全启用日志记录

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-security-enable-logging>

## 1.概观

使用 [弹簧安全](/web/20221208143830/https://www.baeldung.com/security-spring) 时，我们可能需要登录到比默认级别更高的级别。例如，我们可能需要检查用户的角色或端点如何受到保护。或者，我们可能还需要更多关于认证或授权的信息，例如，了解用户无法访问端点的原因。

在这个简短的教程中，我们将看到如何修改 Spring 安全日志级别。

## 2.配置 Spring 安全日志记录

与任何 Spring 或 Java 应用程序一样，我们可以使用一个[日志库](/web/20221208143830/https://www.baeldung.com/logback)，并为 Spring 安全模块定义一个日志级别。

通常，我们可以在配置文件中编写如下内容:

```java
<logger name="org.springframework.security" level="DEBUG" /> 
```

**然而，如果我们正在运行一个 [Spring Boot](/web/20221208143830/https://www.baeldung.com/spring-boot) 应用**，**我们可以在我们的`application.properties`** 文件中进行配置:

```java
logging.level.org.springframework.security=DEBUG
```

同样，我们可以使用`yaml`语法:

```java
logging:
  level:
    org:
      springframework:
        security: DEBUG
```

这样，**我们可以查看关于[认证](/web/20221208143830/https://www.baeldung.com/tag/authentication/)或[过滤链](https://web.archive.org/web/20221208143830/https://docs.spring.io/spring-security/reference/servlet/architecture.html#servlet-filterchainproxy)** 的日志。而且，我们甚至可以使用`trace`级别进行更深入的调试。

此外， **Spring Security 提供了记录关于请求和应用过滤器的特定信息的可能性**:

```java
@EnableWebSecurity
public class SecurityConfig {

    @Value("${spring.websecurity.debug:false}")
    boolean webSecurityDebug;

    @Bean
    public WebSecurityCustomizer webSecurityCustomizer() {
        return (web) -> web.debug(webSecurityDebug);
    }
    // ...
}
```

## 3.日志样本

最后，为了测试我们的应用程序，让我们定义一个简单的控制器:

```java
@Controller
public class LoggingController {

    @GetMapping("/logging")
    public ResponseEntity<String> logging() {
        return new ResponseEntity<>("logging/baeldung", HttpStatus.OK);
    }

}
```

如果我们点击了 `/logging` 端点，我们可以查看我们的日志:

```java
2022-02-10 21:30:32.104 DEBUG 5489 --- [nio-8080-exec-1] o.s.s.w.a.i.FilterSecurityInterceptor    : Authorized filter invocation [GET /logging] with attributes [permitAll]
2022-02-10 21:30:32.105 DEBUG 5489 --- [nio-8080-exec-1] o.s.security.web.FilterChainProxy        : Secured GET /logging
2022-02-10 21:30:32.141 DEBUG 5489 --- [nio-8080-exec-1] w.c.HttpSessionSecurityContextRepository : Did not store anonymous SecurityContext
2022-02-10 21:30:32.146 DEBUG 5489 --- [nio-8080-exec-1] s.s.w.c.SecurityContextPersistenceFilter : Cleared SecurityContextHolder to complete request
```

```java
Request received for GET '/logging':

[[email protected]](/web/20221208143830/https://www.baeldung.com/cdn-cgi/l/email-protection)

servletPath:/logging
pathInfo:null
headers: 
host: localhost:8080
connection: keep-alive
sec-ch-ua: " Not A;Brand";v="99", "Chromium";v="98", "Google Chrome";v="98"
sec-ch-ua-mobile: ?0
sec-ch-ua-platform: "Linux"
upgrade-insecure-requests: 1
user-agent: Mozilla/5.0 (X11; Linux x86_64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/98.0.4758.80 Safari/537.36
accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.9
sec-fetch-site: none
sec-fetch-mode: navigate
sec-fetch-user: ?1
sec-fetch-dest: document
accept-encoding: gzip, deflate, br
accept-language: en,it;q=0.9,en-US;q=0.8
cookie: PGADMIN_LANGUAGE=en; NX-ANTI-CSRF-TOKEN=0.7130543323088452; _ga=GA1.1.1440105797.1623675414; NXSESSIONID=bec8cae2-30e2-4ad4-9333-cba1af5dc95c; JSESSIONID=1C7CD365F521609AD887B3D6C2BE26CC

Security filter chain: [
  WebAsyncManagerIntegrationFilter
  SecurityContextPersistenceFilter
  HeaderWriterFilter
  CsrfFilter
  LogoutFilter
  RequestCacheAwareFilter
  SecurityContextHolderAwareRequestFilter
  AnonymousAuthenticationFilter
  SessionManagementFilter
  ExceptionTranslationFilter
  FilterSecurityInterceptor
]
```

## 4.结论

在本文中，我们研究了一些选项来为 Spring 安全性启用不同的日志记录级别。

我们已经看到了如何为 Spring 安全模块使用`debug`级别。此外，我们还看到了如何记录单个请求的特定信息。

和往常一样，这些例子的代码可以在 GitHub 的[上找到。](https://web.archive.org/web/20221208143830/https://github.com/eugenp/tutorials/tree/master/spring-security-modules/spring-security-web-boot-3)