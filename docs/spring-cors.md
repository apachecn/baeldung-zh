# 春天的 CORS

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-cors>

## 1。概述

在任何现代浏览器中，[跨源资源共享(CORS)](/web/20220926184159/https://www.baeldung.com/cs/cors-preflight-requests) 是随着通过 REST APIs 消费数据的 HTML5 和 JS 客户端的出现而出现的相关规范。

通常，提供 JS 的主机(例如 example.com)与提供数据的主机(例如 api.example.com)是不同的。在这种情况下，CORS 支持跨域通信。

Spring 为 CORS 提供了一流的[支持](https://web.archive.org/web/20220926184159/https://docs.spring.io/spring/docs/current/spring-framework-reference/html/cors.html)，提供了在任何 Spring 或 Spring Boot web 应用程序中配置它的简单而强大的方法。

## 延伸阅读:

## [用 CORS 预灯和弹簧安全装置固定 401s】](/web/20220926184159/https://www.baeldung.com/spring-security-cors-preflight)

Learn how to fix HTTP error status 401 for CORS preflight requests[Read more](/web/20220926184159/https://www.baeldung.com/spring-security-cors-preflight) →

## [Spring Webflux 和 CORS](/web/20220926184159/https://www.baeldung.com/spring-webflux-cors)

A quick and practical guide to working with CORS and Spring Webflux.[Read more](/web/20220926184159/https://www.baeldung.com/spring-webflux-cors) →

## 2。控制器方法 CORS 配置

**启用 CORS 很简单——只需添加注释`@CrossOrigin`。**

我们可以用几种不同的方法来实现这一点。

### 2.1。`@CrossOrigin`上一个`@RequestMapping-`带注释的处理程序方法

```java
@RestController
@RequestMapping("/account")
public class AccountController {

    @CrossOrigin
    @RequestMapping(method = RequestMethod.GET, path = "/{id}")
    public Account retrieve(@PathVariable Long id) {
        // ...
    }

    @RequestMapping(method = RequestMethod.DELETE, path = "/{id}")
    public void remove(@PathVariable Long id) {
        // ...
    }
}
```

在上面的例子中，我们只为`retrieve()`方法启用了 CORS。我们可以看到，我们没有为`@CrossOrigin`注释设置任何配置，所以它使用默认值:

*   所有起源都是允许的。
*   允许的 HTTP 方法是那些在`@RequestMapping`注释中指定的方法(对于这个例子是 GET)。
*   预检响应的缓存时间(`maxAge`)为 30 分钟。

### 2.2。`@CrossOrigin`控制器上的

```java
@CrossOrigin(origins = "http://example.com", maxAge = 3600)
@RestController
@RequestMapping("/account")
public class AccountController {

    @RequestMapping(method = RequestMethod.GET, path = "/{id}")
    public Account retrieve(@PathVariable Long id) {
        // ...
    }

    @RequestMapping(method = RequestMethod.DELETE, path = "/{id}")
    public void remove(@PathVariable Long id) {
        // ...
    }
}
```

这一次，我们在类的层面上增加了`@CrossOrigin`。因此，`retrieve()`和`remove()`方法都启用了它。我们可以通过指定注释属性之一的值来定制配置:`origins`、`methods`、`allowedHeaders`、`exposedHeaders`、`allowCredentials`或`maxAge`。

### 2.3。`@CrossOrigin`关于控制器和处理器的方法

```java
@CrossOrigin(maxAge = 3600)
@RestController
@RequestMapping("/account")
public class AccountController {

    @CrossOrigin("http://example.com")
    @RequestMapping(method = RequestMethod.GET, "/{id}")
    public Account retrieve(@PathVariable Long id) {
        // ...
    }

    @RequestMapping(method = RequestMethod.DELETE, path = "/{id}")
    public void remove(@PathVariable Long id) {
        // ...
    }
}
```

Spring 将组合两个注释的属性来创建一个合并的 CORS 配置。

这里，两种方法都有一个 3600 秒的`maxAge` ，方法`remove()`允许所有的起源，方法`retrieve()`只允许从`http://example.com`开始的起源。

## 3。全球 CORS 配置

作为细粒度基于注释的配置的替代方案，Spring 允许我们在控制器之外定义一个全局 CORS 配置。这类似于使用基于`Filter`的解决方案，但是可以在 Spring MVC 中声明，并与细粒度的`@CrossOrigin`配置相结合。

默认情况下，所有 origins 和 GET、HEAD 和 POST 方法都是允许的。

### 3.1。JavaConfig

```java
@Configuration
@EnableWebMvc
public class WebConfig implements WebMvcConfigurer {

    @Override
    public void addCorsMappings(CorsRegistry registry) {
        registry.addMapping("/**");
    }
}
```

上面的例子支持从应用程序中的任何起点到任何端点的 CORS 请求。

为了进一步锁定这一点，`registry.addMapping`方法返回了一个`CorsRegistration`对象，我们可以使用它进行额外的配置。还有一个`allowedOrigins` 方法，让我们指定一组允许的原点。如果我们需要在运行时从外部源加载这个数组，这可能是有用的。

此外，我们还可以使用`allowedMethods`、`allowedHeaders`、`exposedHeaders`、`maxAge`和`allowCredentials`来设置响应头和定制选项。

### 3.2。XML 名称空间

这个最小的 XML 配置使 CORS 能够在一个`/**`路径模式上使用与 JavaConfig 相同的默认属性:

```java
<mvc:cors>
    <mvc:mapping path="/**" />
</mvc:cors>
```

还可以用自定义属性声明几个 CORS 映射:

```java
<mvc:cors>

    <mvc:mapping path="/api/**"
        allowed-origins="http://domain1.com, http://domain2.com"
        allowed-methods="GET, PUT"
        allowed-headers="header1, header2, header3"
        exposed-headers="header1, header2" allow-credentials="false"
        max-age="123" />

    <mvc:mapping path="/resources/**"
        allowed-origins="http://domain1.com" />

</mvc:cors>
```

## 4.春天安全的 CORS

如果我们在项目中使用 Spring Security，我们必须采取额外的步骤来确保它与 CORS 配合良好。那是因为 CORS 需要先处理。否则，Spring Security 会在请求到达 Spring MVC 之前拒绝它。

幸运的是，Spring Security 提供了一个现成的解决方案:

```java
@EnableWebSecurity
public class WebSecurityConfig {

    @Bean
    public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
        http.cors().and()...
    }
}
```

[本文](/web/20220926184159/https://www.baeldung.com/spring-security-cors-preflight)更详细的解释。

## 5。工作原理

CORS 请求被自动发送到各个注册的`HandlerMappings`。它们使用一个 [`CorsProcessor`](https://web.archive.org/web/20220926184159/https://docs.spring.io/spring/docs/current/javadoc-api/org/springframework/web/cors/CorsProcessor.html) (默认为`DefaultCorsProcessor`)实现添加相关的 CORS 响应头(如`Access-Control-Allow-Origin`)来处理 CORS 的预检请求和拦截 CORS 的简单和实际请求。

[`CorsConfiguration`](https://web.archive.org/web/20220926184159/https://docs.spring.io/spring/docs/current/javadoc-api/org/springframework/web/cors/CorsConfiguration.html) 允许我们指定应该如何处理 CORS 请求，包括允许的来源、报头和方法等等。我们可以通过多种方式提供:

*   `[AbstractHandlerMapping#setCorsConfiguration()](https://web.archive.org/web/20220926184159/https://docs.spring.io/spring/docs/current/javadoc-api/org/springframework/web/servlet/handler/AbstractHandlerMapping.html#setCorsConfiguration-java.util.Map-)`允许我们用几个映射到路径模式的`CorsConfiguration`来指定一个`Map`，比如`/api/**`。
*   子类可以通过覆盖`AbstractHandlerMapping#getCorsConfiguration(Object, HttpServletRequest)`方法来提供自己的`CorsConfiguration`。
*   处理程序可以实现 [`CorsConfigurationSource`](https://web.archive.org/web/20220926184159/https://docs.spring.io/spring/docs/current/javadoc-api/org/springframework/web/cors/CorsConfigurationSource.html) 接口(就像现在的`ResourceHttpRequestHandler`一样)为每个请求提供一个`CorsConfiguration`。

## 6。结论

在本文中，我们展示了 Spring 如何支持在我们的应用程序中启用 CORS。

我们从控制器的配置开始。我们看到，我们只需要添加注释`@CrossOrigin`就可以将 CORS 添加到一个特定的方法或整个控制器中。

此外，我们还了解到，为了在控制器之外控制 CORS 配置，我们可以使用 JavaConfig 或 XML 在配置文件中顺利地执行这一操作。

GitHub 上的[提供了示例的完整源代码。](https://web.archive.org/web/20220926184159/https://github.com/eugenp/tutorials/tree/master/spring-boot-modules/spring-boot-runtime-2)