# 春天云的速率限制网飞·祖尔

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-cloud-zuul-rate-limit>

## 1。简介

[春云网飞 Zuul](https://web.archive.org/web/20220703151955/https://github.com/spring-cloud/spring-cloud-netflix) 是一个开源网关，包装了[网飞 Zuul](https://web.archive.org/web/20220703151955/https://github.com/Netflix/zuul) 。它为 Spring Boot 应用程序添加了一些特定的功能。不幸的是，速率限制不是现成的。

在本教程中，我们将探索 [Spring Cloud Zuul RateLimit](https://web.archive.org/web/20220703151955/https://github.com/marcosbarbero/spring-cloud-zuul-ratelimit) ，它增加了对速率限制请求的支持。

## 2。Maven 配置

除了 Spring Cloud 网飞 Zuul 依赖项，我们需要将 [Spring Cloud Zuul RateLimit](https://web.archive.org/web/20220703151955/https://mvnrepository.com/artifact/com.marcosbarbero.cloud/spring-cloud-zuul-ratelimit) 添加到我们的应用程序的`pom.xml`:

```java
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-netflix-zuul</artifactId>
</dependency>
<dependency>
    <groupId>com.marcosbarbero.cloud</groupId>
    <artifactId>spring-cloud-zuul-ratelimit</artifactId>
    <version>2.2.0.RELEASE</version>
</dependency>
```

## 3。示例控制器

首先，让我们创建几个 REST 端点，我们将在其上应用速率限制。

下面是一个简单的 Spring 控制器类，有两个端点:

```java
@Controller
@RequestMapping("/greeting")
public class GreetingController {

    @GetMapping("/simple")
    public ResponseEntity<String> getSimple() {
        return ResponseEntity.ok("Hi!");
    }

    @GetMapping("/advanced")
    public ResponseEntity<String> getAdvanced() {
        return ResponseEntity.ok("Hello, how you doing?");
    }
}
```

正如我们所看到的，没有特定的代码来限制端点的速率。这是因为我们将在`application.yml`文件的 Zuul 属性中配置它。因此，保持我们的代码解耦。

## 4。Zuul 属性

其次，让我们在我们的`application.yml`文件中添加以下 Zuul 属性:

```java
zuul:
  routes:
    serviceSimple:
      path: /greeting/simple
      url: forward:/
    serviceAdvanced:
      path: /greeting/advanced
      url: forward:/
  ratelimit:
    enabled: true
    repository: JPA
    policy-list:
      serviceSimple:
        - limit: 5
          refresh-interval: 60
          type:
            - origin
      serviceAdvanced:
        - limit: 1
          refresh-interval: 2
          type:
            - origin
  strip-prefix: true
```

在`zuul.routes`下，我们提供端点细节。在`zuul.ratelimit.policy-list,`下，我们为端点提供速率限制配置。`limit`属性指定了端点在`refresh-interval`内被调用的次数。

如我们所见，我们为`serviceSimple`端点添加了每 60 秒 5 个请求的速率限制。相比之下，`serviceAdvanced`的速率限制为每 2 秒 1 个请求。

`type`配置指定了我们想要遵循的速率限制方法。以下是可能的值:

*   `origin`–基于用户原始请求的速率限制
*   `url`–基于下游服务请求路径的速率限制
*   `user`–基于认证用户名或“匿名”的速率限制
*   无值–作为每个服务的全局配置。要使用这种方法，只需不设置参数“类型”

## 5。测试速率限制

### 5.1。速率限制内的请求

接下来，让我们测试速率限制:

```java
@Test
public void whenRequestNotExceedingCapacity_thenReturnOkResponse() {
    ResponseEntity<String> response = restTemplate.getForEntity(SIMPLE_GREETING, String.class);
    assertEquals(OK, response.getStatusCode());

    HttpHeaders headers = response.getHeaders();
    String key = "rate-limit-application_serviceSimple_127.0.0.1";

    assertEquals("5", headers.getFirst(HEADER_LIMIT + key));
    assertEquals("4", headers.getFirst(HEADER_REMAINING + key));
    assertThat(
      parseInt(headers.getFirst(HEADER_RESET + key)),
      is(both(greaterThanOrEqualTo(0)).and(lessThanOrEqualTo(60000)))
    );
}
```

这里我们对端点`/greeting/simple`进行一次调用。请求成功，因为它在速率限制内。

另一个关键点是**对于每个响应，我们都得到返回头，为我们提供关于速率限制的进一步信息。**对于上述请求，我们将获得以下标题:

```java
X-RateLimit-Limit-rate-limit-application_serviceSimple_127.0.0.1: 5
X-RateLimit-Remaining-rate-limit-application_serviceSimple_127.0.0.1: 4
X-RateLimit-Reset-rate-limit-application_serviceSimple_127.0.0.1: 60000
```

换句话说:

*   `X-RateLimit-Limit-[key]:`为端点配置的`limit`
*   `X-RateLimit-Remaining-[key]:`呼叫端点的剩余尝试次数
*   `X-RateLimit-Reset-[key]:`为端点配置的`refresh-interval`的剩余毫秒数

此外，如果我们立即再次触发同一个端点，我们可能会得到:

```java
X-RateLimit-Limit-rate-limit-application_serviceSimple_127.0.0.1: 5
X-RateLimit-Remaining-rate-limit-application_serviceSimple_127.0.0.1: 3
X-RateLimit-Reset-rate-limit-application_serviceSimple_127.0.0.1: 57031
```

**注意减少的剩余尝试次数和剩余毫秒数。**

### 5.2。超过速率限制的请求

让我们看看当我们超过速率限制时会发生什么:

```java
@Test
public void whenRequestExceedingCapacity_thenReturnTooManyRequestsResponse() throws InterruptedException {
    ResponseEntity<String> response = this.restTemplate.getForEntity(ADVANCED_GREETING, String.class);
    assertEquals(OK, response.getStatusCode());

    for (int i = 0; i < 2; i++) {
        response = this.restTemplate.getForEntity(ADVANCED_GREETING, String.class);
    }

    assertEquals(TOO_MANY_REQUESTS, response.getStatusCode());

    HttpHeaders headers = response.getHeaders();
    String key = "rate-limit-application_serviceAdvanced_127.0.0.1";

    assertEquals("1", headers.getFirst(HEADER_LIMIT + key));
    assertEquals("0", headers.getFirst(HEADER_REMAINING + key));
    assertNotEquals("2000", headers.getFirst(HEADER_RESET + key));

    TimeUnit.SECONDS.sleep(2);

    response = this.restTemplate.getForEntity(ADVANCED_GREETING, String.class);
    assertEquals(OK, response.getStatusCode());
}
```

这里我们连续两次调用端点`/greeting/advanced`。由于我们已经将速率限制配置为每 2 秒一个请求，**第二次调用将失败**。结果，错误代码 **429 ( `Too Many Requests)`** )被返回给客户端。

以下是达到速率限制时返回的标题:

```java
X-RateLimit-Limit-rate-limit-application_serviceAdvanced_127.0.0.1: 1
X-RateLimit-Remaining-rate-limit-application_serviceAdvanced_127.0.0.1: 0
X-RateLimit-Reset-rate-limit-application_serviceAdvanced_127.0.0.1: 268
```

之后，我们睡两秒钟。这是为端点配置的`refresh-interval`。最后，我们再次触发端点并获得成功的响应。

## 6。 **自定义密钥生成器**

**我们可以使用定制的密钥生成器定制响应头中发送的密钥。**这很有用，因为应用程序可能需要控制`type`属性提供的选项之外的关键策略。

例如，这可以通过创建一个定制的`RateLimitKeyGenerator`实现来完成。我们可以添加更多的限定符或完全不同的东西:

```java
@Bean
public RateLimitKeyGenerator rateLimitKeyGenerator(RateLimitProperties properties, 
  RateLimitUtils rateLimitUtils) {
    return new DefaultRateLimitKeyGenerator(properties, rateLimitUtils) {
        @Override
        public String key(HttpServletRequest request, Route route, 
          RateLimitProperties.Policy policy) {
            return super.key(request, route, policy) + "_" + request.getMethod();
        }
    };
}
```

上面的代码将 REST 方法名附加到键上。例如:

```java
X-RateLimit-Limit-rate-limit-application_serviceSimple_127.0.0.1_GET: 5
```

另一个关键点是，`RateLimitKeyGenerator` bean 将由`spring-cloud-zuul-ratelimit`自动配置。

## 7。自定义错误处理

该框架支持速率限制数据存储的各种实现。例如，提供了 Spring 数据 JPA 和 Redis。默认情况下，使用`DefaultRateLimiterErrorHandler`类将**失败记录为错误**。

当我们需要以不同的方式处理错误时，我们可以定义一个定制的`RateLimiterErrorHandler` bean:

```java
@Bean
public RateLimiterErrorHandler rateLimitErrorHandler() {
    return new DefaultRateLimiterErrorHandler() {
        @Override
        public void handleSaveError(String key, Exception e) {
            // implementation
        }

        @Override
        public void handleFetchError(String key, Exception e) {
            // implementation
        }

        @Override
        public void handleError(String msg, Exception e) {
            // implementation
        }
    };
}
```

与`RateLimitKeyGenerator` bean 类似，`RateLimiterErrorHandler` bean 也将被自动配置。

## 8。结论

在本文中，我们看到了如何使用 Spring Cloud 网飞 Zuul 和 Spring Cloud Zuul RateLimit 对限额 API 进行评级。

一如既往，本文的完整代码可以在 GitHub 上找到[。](https://web.archive.org/web/20220703151955/https://github.com/eugenp/tutorials/tree/master/spring-cloud-modules/spring-cloud-zuul)