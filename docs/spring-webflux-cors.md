# 春网流量和 CORS

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-webflux-cors>

## 1.概观

在[之前的帖子](/web/20220701013321/https://www.baeldung.com/spring-cors)中，我们了解了跨源资源共享(CORS)规范以及如何在 Spring 中使用它。

在这个快速教程中，**我们将使用** [**Spring 的 5 WebFlux 框架**](https://web.archive.org/web/20220701013321/https://docs.spring.io/spring/docs/current/spring-framework-reference/web-reactive.html) 建立一个类似的 CORS 配置。

首先，我们将看看如何在基于注释的 API 上启用该机制。

然后，我们将分析如何在整个项目中将它作为一个全局配置来启用，或者通过使用一个特殊的`WebFilter`。

## 2.在注释元素上启用 CORS

**Spring 提供了`@CrossOrigin`注释来启用控制器类和/或处理程序方法上的 CORS 请求。**

### 2.1.在请求处理器方法上使用`@CrossOrigin`

让我们将这个注释添加到映射的请求方法中:

```java
@CrossOrigin
@PutMapping("/cors-enabled-endpoint")
public Mono<String> corsEnabledEndpoint() {
    // ...
}
```

我们将使用一个`WebTestClient`(正如我们在第 4 节中解释的。对[这篇文章](/web/20220701013321/https://www.baeldung.com/spring-5-functional-web)进行‘测试’，以分析我们从该端点获得的响应:

```java
ResponseSpec response = webTestClient.put()
  .uri("/cors-enabled-endpoint")
  .header("Origin", "http://any-origin.com")
  .exchange();

response.expectHeader()
  .valueEquals("Access-Control-Allow-Origin", "*");
```

此外，我们可以尝试一个飞行前请求，以确保 CORS 配置按预期工作:

```java
ResponseSpec response = webTestClient.options()
  .uri("/cors-enabled-endpoint")
  .header("Origin", "http://any-origin.com")
  .header("Access-Control-Request-Method", "PUT")
  .exchange();

response.expectHeader()
  .valueEquals("Access-Control-Allow-Origin", "*");
response.expectHeader()
  .valueEquals("Access-Control-Allow-Methods", "PUT");
response.expectHeader()
  .exists("Access-Control-Max-Age");
```

`@CrossOrigin`注释具有以下默认配置:

*   允许所有来源(这解释了响应头中的“*”值)
*   允许所有标题
*   允许 handler 方法映射的所有 HTTP 方法
*   凭据未启用
*   “最大年龄”值为 1800 秒(30 分钟)

但是，可以使用注释的参数覆盖这些值中的任何一个。

### 2.2.使用控制器上的`@CrossOrigin`

该注释在类级别也受支持，它将影响它的所有方法。

如果类级别的配置不适合我们所有的方法，我们可以注释这两个元素来获得想要的结果:

```java
@CrossOrigin(value = { "http://allowed-origin.com" },
  allowedHeaders = { "Baeldung-Allowed" },
  maxAge = 900
)
@RestController
public class CorsOnClassController {

    @PutMapping("/cors-enabled-endpoint")
    public Mono<String> corsEnabledEndpoint() {
        // ...
    }

    @CrossOrigin({ "http://another-allowed-origin.com" })
    @PutMapping("/endpoint-with-extra-origin-allowed")
    public Mono<String> corsEnabledWithExtraAllowedOrigin() {
        // ...
    }

    // ...
}
```

## 3.在全局配置中启用 CORS

**我们也可以通过覆盖`WebFluxConfigurer`实现的`addCorsMappings()`方法来定义一个全局 CORS 配置。**

此外，实现需要`@EnableWebFlux`注释来在普通的 Spring 应用程序中导入 Spring WebFlux 配置。如果我们使用 Spring Boot，那么我们只需要这个注释来覆盖自动配置:

```java
@Configuration
@EnableWebFlux
public class CorsGlobalConfiguration implements WebFluxConfigurer {

    @Override
    public void addCorsMappings(CorsRegistry corsRegistry) {
        corsRegistry.addMapping("/**")
          .allowedOrigins("http://allowed-origin.com")
          .allowedMethods("PUT")
          .maxAge(3600);
    }
}
```

因此，我们正在为该特定路径模式启用跨来源请求处理。

默认配置类似于`@CrossOrigin`配置，但是只允许使用`GET`、`HEAD`和`POST`方法。

我们还可以将这种配置与本地配置相结合:

*   对于多值属性，生成的 CORS 配置将是每个规范的相加
*   另一方面，对于单值值，局部值将优先于全局值

但是，使用这种方法对于功能端点来说是无效的。

## 4.使用`WebFilter`启用 CORS

**在功能端点上启用 CORS 的最佳方式是使用`WebFilter`。**

正如我们在这篇文章中看到的[，我们可以使用`WebFilter`来修改请求和响应，同时保持端点的实现不变。](/web/20220701013321/https://www.baeldung.com/spring-webflux-filters)

**Spring 提供了内置的`CorsWebFilter`来轻松处理跨原点配置:**

```java
@Bean
CorsWebFilter corsWebFilter() {
    CorsConfiguration corsConfig = new CorsConfiguration();
    corsConfig.setAllowedOrigins(Arrays.asList("http://allowed-origin.com"));
    corsConfig.setMaxAge(8000L);
    corsConfig.addAllowedMethod("PUT");
    corsConfig.addAllowedHeader("Baeldung-Allowed");

    UrlBasedCorsConfigurationSource source =
      new UrlBasedCorsConfigurationSource();
    source.registerCorsConfiguration("/**", corsConfig);

    return new CorsWebFilter(source);
}
```

这对于带注释的处理程序也是有效的，但是它不能与更细粒度的`@CrossOrigin`配置相结合。

我们必须记住,`CorsConfiguration`没有默认配置。

因此，除非我们指定所有相关的属性，否则 CORS 的实现将会受到很大的限制。

**设置默认值的一个简单方法是在对象上使用`applyPermitDefaultValues()`方法。**

## 5.结论

总之，我们通过非常简短的例子了解了如何在我们基于 webflux 的服务上启用 CORS。

我们看到了不同的方法，因此我们现在要做的就是分析哪种方法最适合我们的要求。

我们可以在 GitHub repo 中找到大量的例子，以及测试案例，我们在这些案例中分析了关于这个主题的大多数边缘案例。