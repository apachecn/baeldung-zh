# 使用 Bucket4j 限制 Spring API 的速率

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-bucket4j>

## 1.概观

在本教程中，**我们将关注如何使用 [Bucket4j](https://web.archive.org/web/20221208143832/https://github.com/vladimir-bukhtoyarov/bucket4j) 来限制一个 Spring REST API** 。

我们将探索 API 速率限制，了解 Bucket4j，然后在 Spring 应用程序中研究 REST APIs 的几种速率限制方法。

## 2.API 速率限制

速率限制是[限制访问 API](https://web.archive.org/web/20221208143832/https://cloud.google.com/solutions/rate-limiting-strategies-techniques)的一种策略。它限制了客户端在特定时间范围内可以进行的 API 调用的数量。这有助于防止 API 被过度使用，无论是无意的还是恶意的。

速率限制通常通过跟踪 IP 地址或更特定于业务的方式(如 API 密钥或访问令牌)应用于 API。作为 API 开发人员，当客户端达到极限时，我们有几种选择:

*   将请求排队，直到剩余的时间段过去
*   立即允许请求，但对此请求收取额外费用
*   拒绝请求(HTTP 429 请求太多)

## 3.Bucket4j 限速库

### 3.1.Bucket4j 是什么？

Bucket4j 是一个基于[令牌桶](https://web.archive.org/web/20221208143832/https://en.wikipedia.org/wiki/Token_bucket)算法的 Java 限速库。Bucket4j 是一个线程安全的库，可以用在独立的 JVM 应用程序中，也可以用在集群环境中。它还通过 [JCache (JSR107)](https://web.archive.org/web/20221208143832/https://www.jcp.org/en/jsr/detail?id=107) 规范支持内存或分布式缓存。

### 3.2.令牌桶算法

让我们在 API 速率限制的背景下直观地看一下算法。

假设我们有一个桶，其容量被定义为它可以容纳的令牌数。**每当消费者想要访问一个 API 端点时，它必须从 bucket** 中获取一个令牌。如果令牌可用，我们从桶中移除令牌并接受请求。相反，如果 bucket 没有任何令牌，我们会拒绝请求。

**由于请求正在消耗令牌，我们也以某个固定的速率补充它们**，这样我们就永远不会超过存储桶的容量。

让我们考虑一个 API，它的速率限制为每分钟 100 个请求。我们可以创建一个容量为 100 的桶，再填充速率为每分钟 100 个令牌。

如果我们收到 70 个请求，这少于给定分钟内的可用令牌，我们将在下一分钟开始时再添加 30 个令牌，以达到最大容量。另一方面，如果我们在 40 秒内用完所有的代币，我们将等待 20 秒来重新装满桶。

## 4.Bucket4j 入门

### 4.1.Maven 配置

让我们从将`[bucket4j](https://web.archive.org/web/20221208143832/https://search.maven.org/search?q=g:com.github.vladimir-bukhtoyarov%20AND%20a:%20bucket4j-core)` 依赖项添加到我们的`pom.xml`开始:

```java
<dependency>
    <groupId>com.github.vladimir-bukhtoyarov</groupId>
    <artifactId>bucket4j-core</artifactId>
    <version>4.10.0</version>
</dependency>
```

### 4.2.术语

在了解如何使用 Bucket4j 之前，我们将简要讨论一些核心类，以及它们如何表示令牌桶算法的正式模型中的不同元素。

[`Bucket`](https://web.archive.org/web/20221208143832/https://javadoc.io/doc/com.github.vladimir-bukhtoyarov/bucket4j-core/4.10.0/io/github/bucket4j/Bucket.html) 界面表示具有最大容量的令牌桶。它提供了[`tryConsume`](https://web.archive.org/web/20221208143832/https://javadoc.io/doc/com.github.vladimir-bukhtoyarov/bucket4j-core/4.10.0/io/github/bucket4j/Bucket.html#tryConsume-long-)[`tryConsumeAndReturnRemaining`](https://web.archive.org/web/20221208143832/https://javadoc.io/doc/com.github.vladimir-bukhtoyarov/bucket4j-core/4.10.0/io/github/bucket4j/Bucket.html#tryConsumeAndReturnRemaining-long-)等消耗令牌的方法。如果请求符合限制，这些方法返回消费结果为`true`，并且令牌已被消费。

[`Bandwidth`](https://web.archive.org/web/20221208143832/https://javadoc.io/doc/com.github.vladimir-bukhtoyarov/bucket4j-core/4.10.0/io/github/bucket4j/Bandwidth.html) 类是 bucket 的关键构造块，因为它定义了 bucket 的限制。我们使用`Bandwidth`来配置铲斗的容量和再装率。

[`Refill`](https://web.archive.org/web/20221208143832/https://javadoc.io/doc/com.github.vladimir-bukhtoyarov/bucket4j-core/4.10.0/io/github/bucket4j/Refill.html) 类用于定义令牌添加到桶中的固定速率。我们可以将速率配置为在给定时间段内添加的令牌数。例如，每秒 10 桶或每 5 分钟 200 个令牌，等等。

`Bucket`中的`tryConsumeAndReturnRemaining`方法返回 [`ConsumptionProbe`](https://web.archive.org/web/20221208143832/https://javadoc.io/doc/com.github.vladimir-bukhtoyarov/bucket4j-core/4.10.0/io/github/bucket4j/ConsumptionProbe.html) 。`ConsumptionProbe`包含桶的状态以及消费结果，例如剩余的令牌，或者直到所请求的令牌在桶中再次可用的剩余时间。

### 4.3.基本用法

让我们测试一些基本的限速模式。

对于每分钟 10 个请求的速率限制，我们将创建一个容量为 10 的存储桶，再填充速率为每分钟 10 个令牌:

```java
Refill refill = Refill.intervally(10, Duration.ofMinutes(1));
Bandwidth limit = Bandwidth.classic(10, refill);
Bucket bucket = Bucket4j.builder()
    .addLimit(limit)
    .build();

for (int i = 1; i <= 10; i++) {
    assertTrue(bucket.tryConsume(1));
}
assertFalse(bucket.tryConsume(1)); 
```

`Refill.intervally`在时间窗口开始时重新填充桶，在本例中，在一分钟开始时是 10 个令牌。

接下来，让我们看看实际使用的笔芯。

我们将设置每 2 秒 1 个令牌的充值速率，并且**限制我们的请求以遵守速率限制**:

```java
Bandwidth limit = Bandwidth.classic(1, Refill.intervally(1, Duration.ofSeconds(2)));
Bucket bucket = Bucket4j.builder()
    .addLimit(limit)
    .build();
assertTrue(bucket.tryConsume(1));     // first request
Executors.newScheduledThreadPool(1)   // schedule another request for 2 seconds later
    .schedule(() -> assertTrue(bucket.tryConsume(1)), 2, TimeUnit.SECONDS); 
```

假设我们的速率限制为每分钟 10 个请求。同时，**我们可能希望避免在前 5 秒耗尽所有令牌的峰值**。Bucket4j 允许我们在同一个桶上设置多个限制(`Bandwidth`)。让我们添加另一个限制，在 20 秒的时间窗口内只允许 5 个请求:

```java
Bucket bucket = Bucket4j.builder()
    .addLimit(Bandwidth.classic(10, Refill.intervally(10, Duration.ofMinutes(1))))
    .addLimit(Bandwidth.classic(5, Refill.intervally(5, Duration.ofSeconds(20))))
    .build();

for (int i = 1; i <= 5; i++) {
    assertTrue(bucket.tryConsume(1));
}
assertFalse(bucket.tryConsume(1)); 
```

## 5.使用 Bucket4j 限制 Spring API 的速率

让我们使用 Bucket4j 在 Spring REST API 中应用一个速率限制。

### 5.1.面积计算器 API

我们将实现一个简单但非常受欢迎的面积计算器 REST API。目前，它计算并返回给定尺寸的矩形的面积:

```java
@RestController
class AreaCalculationController {

    @PostMapping(value = "/api/v1/area/rectangle")
    public ResponseEntity<AreaV1> rectangle(@RequestBody RectangleDimensionsV1 dimensions) {
        return ResponseEntity.ok(new AreaV1("rectangle", dimensions.getLength() * dimensions.getWidth()));
    }
} 
```

让我们确保我们的 API 启动并运行:

```java
$ curl -X POST http://localhost:9001/api/v1/area/rectangle \
    -H "Content-Type: application/json" \
    -d '{ "length": 10, "width": 12 }'

{ "shape":"rectangle","area":120.0 } 
```

### 5.2.应用速率限制

现在我们将引入一个简单的速率限制，允许 API 每分钟 20 个请求。换句话说，如果在 1 分钟的时间窗口内已经收到 20 个请求，API 就会拒绝一个请求。

让我们修改我们的`Controller`来创建一个`Bucket`并添加限制`(Bandwidth):`

```java
@RestController
class AreaCalculationController {

    private final Bucket bucket;

    public AreaCalculationController() {
        Bandwidth limit = Bandwidth.classic(20, Refill.greedy(20, Duration.ofMinutes(1)));
        this.bucket = Bucket4j.builder()
            .addLimit(limit)
            .build();
    }
    //..
} 
```

在这个 API 中，我们可以通过使用方法`tryConsume`使用桶中的令牌来检查请求是否被允许。如果达到了限制，我们可以通过响应 HTTP 429 太多请求状态来拒绝请求:

```java
public ResponseEntity<AreaV1> rectangle(@RequestBody RectangleDimensionsV1 dimensions) {
    if (bucket.tryConsume(1)) {
        return ResponseEntity.ok(new AreaV1("rectangle", dimensions.getLength() * dimensions.getWidth()));
    }

    return ResponseEntity.status(HttpStatus.TOO_MANY_REQUESTS).build();
} 
```

```java
# 21st request within 1 minute
$ curl -v -X POST http://localhost:9001/api/v1/area/rectangle \
    -H "Content-Type: application/json" \
    -d '{ "length": 10, "width": 12 }'

< HTTP/1.1 429 
```

### 5.3.API 客户和定价计划

现在我们有了一个简单的速率限制，可以抑制 API 请求。接下来，我们将介绍更多以业务为中心的费率限制的定价计划。

定价计划帮助我们将 API 货币化。假设我们对 API 客户端有以下计划:

*   免费:每个 API 客户端每小时 20 个请求
*   基本:每个 API 客户端每小时 40 个请求
*   专业:每个 API 客户端每小时 100 个请求

每个 API 客户机获得一个唯一的 API 密钥，它们必须随每个请求一起发送。这有助于我们确定与 API 客户端相关的定价计划。

让我们为每个定价方案定义费率限制(`Bandwidth`):

```java
enum PricingPlan {
    FREE {
        Bandwidth getLimit() {
            return Bandwidth.classic(20, Refill.intervally(20, Duration.ofHours(1)));
        }
    },
    BASIC {
        Bandwidth getLimit() {
            return Bandwidth.classic(40, Refill.intervally(40, Duration.ofHours(1)));
        }
    },
    PROFESSIONAL {
        Bandwidth getLimit() {
            return Bandwidth.classic(100, Refill.intervally(100, Duration.ofHours(1)));
        }
    };
    //..
} 
```

然后，让我们添加一个方法，根据给定的 API 键来解析定价计划:

```java
enum PricingPlan {

    static PricingPlan resolvePlanFromApiKey(String apiKey) {
        if (apiKey == null || apiKey.isEmpty()) {
            return FREE;
        } else if (apiKey.startsWith("PX001-")) {
            return PROFESSIONAL;
        } else if (apiKey.startsWith("BX001-")) {
            return BASIC;
        }
        return FREE;
    }
    //..
} 
```

接下来，我们需要为每个 API 键存储`Bucket` ,并为速率限制检索`Bucket`:

```java
class PricingPlanService {

    private final Map<String, Bucket> cache = new ConcurrentHashMap<>();

    public Bucket resolveBucket(String apiKey) {
        return cache.computeIfAbsent(apiKey, this::newBucket);
    }

    private Bucket newBucket(String apiKey) {
        PricingPlan pricingPlan = PricingPlan.resolvePlanFromApiKey(apiKey);
        return Bucket4j.builder()
            .addLimit(pricingPlan.getLimit())
            .build();
    }
} 
```

现在我们在内存中存储了每个 API 键的桶。让我们修改我们的`Controller`来使用`PricingPlanService`:

```java
@RestController
class AreaCalculationController {

    private PricingPlanService pricingPlanService;

    public ResponseEntity<AreaV1> rectangle(@RequestHeader(value = "X-api-key") String apiKey,
        @RequestBody RectangleDimensionsV1 dimensions) {

        Bucket bucket = pricingPlanService.resolveBucket(apiKey);
        ConsumptionProbe probe = bucket.tryConsumeAndReturnRemaining(1);
        if (probe.isConsumed()) {
            return ResponseEntity.ok()
                .header("X-Rate-Limit-Remaining", Long.toString(probe.getRemainingTokens()))
                .body(new AreaV1("rectangle", dimensions.getLength() * dimensions.getWidth()));
        }

        long waitForRefill = probe.getNanosToWaitForRefill() / 1_000_000_000;
        return ResponseEntity.status(HttpStatus.TOO_MANY_REQUESTS)
            .header("X-Rate-Limit-Retry-After-Seconds", String.valueOf(waitForRefill))
            .build();
    }
} 
```

让我们来看看这些变化。API 客户端发送带有`X-api-key`请求头的 API 密钥。我们使用`PricingPlanService`来获取这个 API 键的桶，并通过使用桶中的令牌来检查请求是否被允许。

为了增强 API 的客户端体验，我们将使用以下附加响应头来发送有关速率限制的信息:

*   `X-Rate-Limit-Remaining`:当前时间窗口中剩余的令牌数
*   `X-Rate-Limit-Retry-After-Seconds`:铲斗重新装满前的剩余时间，以秒为单位

我们可以调用`ConsumptionProbe`方法`getRemainingTokens`和`getNanosToWaitForRefill` 来分别获得桶中剩余令牌的计数和下一次填充的剩余时间。如果我们能够成功消费令牌，那么`getNanosToWaitForRefill`方法返回 0。

让我们调用 API:

```java
## successful request
$ curl -v -X POST http://localhost:9001/api/v1/area/rectangle \
    -H "Content-Type: application/json" -H "X-api-key:FX001-99999" \
    -d '{ "length": 10, "width": 12 }'

< HTTP/1.1 200
< X-Rate-Limit-Remaining: 11
{"shape":"rectangle","area":120.0}

## rejected request
$ curl -v -X POST http://localhost:9001/api/v1/area/rectangle \
    -H "Content-Type: application/json" -H "X-api-key:FX001-99999" \
    -d '{ "length": 10, "width": 12 }'

< HTTP/1.1 429
< X-Rate-Limit-Retry-After-Seconds: 583 
```

### 5.4.使用 Spring MVC 拦截器

假设我们现在必须添加一个新的 API 端点，该端点根据给定的高度和底边计算并返回三角形的面积:

```java
@PostMapping(value = "/triangle")
public ResponseEntity<AreaV1> triangle(@RequestBody TriangleDimensionsV1 dimensions) {
    return ResponseEntity.ok(new AreaV1("triangle", 0.5d * dimensions.getHeight() * dimensions.getBase()));
} 
```

事实证明，我们还需要对我们的新终点进行速率限制。我们可以简单地从之前的端点复制并粘贴速率限制代码。或者，我们可以使用 **Spring MVC 的 [`HandlerInterceptor`](/web/20221208143832/https://www.baeldung.com/spring-mvc-handlerinterceptor) 将速率限制代码与业务代码**解耦。

让我们创建一个`RateLimitInterceptor`并在`preHandle`方法中实现速率限制代码:

```java
public class RateLimitInterceptor implements HandlerInterceptor {

    private PricingPlanService pricingPlanService;

    @Override
    public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) 
      throws Exception {
        String apiKey = request.getHeader("X-api-key");
        if (apiKey == null || apiKey.isEmpty()) {
            response.sendError(HttpStatus.BAD_REQUEST.value(), "Missing Header: X-api-key");
            return false;
        }

        Bucket tokenBucket = pricingPlanService.resolveBucket(apiKey);
        ConsumptionProbe probe = tokenBucket.tryConsumeAndReturnRemaining(1);
        if (probe.isConsumed()) {
            response.addHeader("X-Rate-Limit-Remaining", String.valueOf(probe.getRemainingTokens()));
            return true;
        } else {
            long waitForRefill = probe.getNanosToWaitForRefill() / 1_000_000_000;
            response.addHeader("X-Rate-Limit-Retry-After-Seconds", String.valueOf(waitForRefill));
            response.sendError(HttpStatus.TOO_MANY_REQUESTS.value(),
              "You have exhausted your API Request Quota"); 
            return false;
        }
    }
} 
```

最后，我们必须将拦截器添加到`InterceptorRegistry`:

```java
public class AppConfig implements WebMvcConfigurer {

    private RateLimitInterceptor interceptor;

    @Override
    public void addInterceptors(InterceptorRegistry registry) {
        registry.addInterceptor(interceptor)
            .addPathPatterns("/api/v1/area/**");
    }
} 
```

`RateLimitInterceptor`拦截对我们的面积计算 API 端点的每个请求。

让我们试试我们的新端点:

```java
## successful request
$ curl -v -X POST http://localhost:9001/api/v1/area/triangle \
    -H "Content-Type: application/json" -H "X-api-key:FX001-99999" \
    -d '{ "height": 15, "base": 8 }'

< HTTP/1.1 200
< X-Rate-Limit-Remaining: 9
{"shape":"triangle","area":60.0}

## rejected request
$ curl -v -X POST http://localhost:9001/api/v1/area/triangle \
    -H "Content-Type: application/json" -H "X-api-key:FX001-99999" \
    -d '{ "height": 15, "base": 8 }'

< HTTP/1.1 429
< X-Rate-Limit-Retry-After-Seconds: 299
{ "status": 429, "error": "Too Many Requests", "message": "You have exhausted your API Request Quota" } 
```

看起来我们结束了。我们可以不断添加端点，拦截器将对每个请求应用速率限制。

## 6.Bucket4j Spring Boot 起动机

让我们看看在 Spring 应用程序中使用 Bucket4j 的另一种方式。 [Bucket4j Spring Boot 启动器](https://web.archive.org/web/20221208143832/https://github.com/MarcGiffing/bucket4j-spring-boot-starter)为 Bucket4j 提供自动配置，帮助我们通过 Spring Boot 应用程序属性或配置实现 API 速率限制。

一旦我们将 Bucket4j starter 集成到我们的应用程序中，**我们将拥有一个完全声明式的 API 速率限制实现，而不需要任何应用程序代码**。

### 6.1.速率限制过滤器

在我们的例子中，我们使用请求头`X-api-key`的值作为识别和应用速率限制的键。

Bucket4j Spring Boot 启动器为定义我们的速率限制键提供了几个预定义的配置:

*   一个简单的速率限制过滤器，这是默认设置
*   按 IP 地址过滤
*   基于表达式的过滤器

基于表达式的过滤器使用 [Spring 表达式语言(SpEL)](/web/20221208143832/https://www.baeldung.com/spring-expression-language) 。SpEL 提供了对根对象的访问，比如可以用来在 IP 地址(`getRemoteAddr()`)、请求头(`getHeader(‘X-api-key')`)上构建过滤器表达式的`HttpServletRequest,`，等等。

该库还支持过滤器表达式中的定制类，这将在[文档](https://web.archive.org/web/20221208143832/https://github.com/MarcGiffing/bucket4j-spring-boot-starter#filter-types-bad-name-should-be-renamed-in-the-feature)中讨论。

### 6.2.Maven 配置

让我们从将`[bucket4j-spring-boot-starter](https://web.archive.org/web/20221208143832/https://search.maven.org/search?q=g:com.giffing.bucket4j.spring.boot.starter%20AND%20a:%20bucket4j-spring-boot-starter)`依赖项添加到我们的`pom.xml`开始:

```java
<dependency>
    <groupId>com.giffing.bucket4j.spring.boot.starter</groupId>
    <artifactId>bucket4j-spring-boot-starter</artifactId>
    <version>0.2.0</version>
</dependency> 
```

在我们早期的实现中，我们使用内存中的`Map`来存储每个 API 键(消费者)的`Bucket`。在这里，我们可以使用 Spring 的缓存抽象来配置一个内存存储，例如[咖啡因](/web/20221208143832/https://www.baeldung.com/java-caching-caffeine)或[番石榴](/web/20221208143832/https://www.baeldung.com/guava-cache)。

让我们添加缓存依赖项:

```java
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-cache</artifactId>
</dependency>
<dependency>
    <groupId>javax.cache</groupId>
    <artifactId>cache-api</artifactId>
</dependency>
<dependency>
    <groupId>com.github.ben-manes.caffeine</groupId>
    <artifactId>caffeine</artifactId>
    <version>2.8.2</version>
</dependency>
<dependency>
    <groupId>com.github.ben-manes.caffeine</groupId>
    <artifactId>jcache</artifactId>
    <version>2.8.2</version>
</dependency> 
```

注意:我们还添加了`[jcache](https://web.archive.org/web/20221208143832/https://search.maven.org/search?q=g:javax.cache%20AND%20a:cache-api)` 依赖项，以符合 Bucket4j 的缓存支持。

我们必须记住**通过向任何配置类**添加`@EnableCaching`注释来启用缓存特性。

### 6.3.应用程序配置

让我们配置我们的应用程序来使用 Bucket4j starter 库。首先，我们将[配置](https://web.archive.org/web/20221208143832/https://docs.spring.io/spring-boot/docs/2.1.6.RELEASE/reference/html/boot-features-caching.html#boot-features-caching-provider-caffeine)咖啡因缓存来存储 API 密钥和`Bucket`内存:

```java
spring:
  cache:
    cache-names:
    - rate-limit-buckets
    caffeine:
      spec: maximumSize=100000,expireAfterAccess=3600s 
```

接下来，让我们[配置](https://web.archive.org/web/20221208143832/https://github.com/MarcGiffing/bucket4j-spring-boot-starter#bucket4j_complete_properties) Bucket4j:

```java
bucket4j:
  enabled: true
  filters:
  - cache-name: rate-limit-buckets
    url: /api/v1/area.*
    strategy: first
    http-response-body: "{ \"status\": 429, \"error\": \"Too Many Requests\", \"message\": \"You have exhausted your API Request Quota\" }"
    rate-limits:
    - expression: "getHeader('X-api-key')"
      execute-condition: "getHeader('X-api-key').startsWith('PX001-')"
      bandwidths:
      - capacity: 100
        time: 1
        unit: hours
    - expression: "getHeader('X-api-key')"
      execute-condition: "getHeader('X-api-key').startsWith('BX001-')"
      bandwidths:
      - capacity: 40
        time: 1
        unit: hours
    - expression: "getHeader('X-api-key')"
      bandwidths:
      - capacity: 20
        time: 1
        unit: hours 
```

那么，我们刚刚配置了什么？

*   `bucket4j.enabled=true`–启用 Bucket4j 自动配置
*   `bucket4j.filters.cache-name`–从缓存中获取 API 键的`Bucket`
*   `bucket4j.filters.url`–表示应用速率限制的路径表达式
*   `bucket4j.filters.strategy=first`–停止在第一个匹配速率限制配置
*   `bucket4j.filters.rate-limits.expression`–使用 Spring 表达式语言(SpEL)检索密钥
*   `bucket4j.filters.rate-limits.execute-condition –` 决定是否使用 SpEL 执行速率限制
*   `bucket4j.filters.rate-limits.bandwidths –`定义 Bucket4j 速率限制参数

我们将`PricingPlanService`和`RateLimitInterceptor`替换为一系列速率限制配置，并按顺序进行评估。

让我们试一试:

```java
## successful request
$ curl -v -X POST http://localhost:9000/api/v1/area/triangle \
    -H "Content-Type: application/json" -H "X-api-key:FX001-99999" \
    -d '{ "height": 20, "base": 7 }'

< HTTP/1.1 200
< X-Rate-Limit-Remaining: 7
{"shape":"triangle","area":70.0}

## rejected request
$ curl -v -X POST http://localhost:9000/api/v1/area/triangle \
    -H "Content-Type: application/json" -H "X-api-key:FX001-99999" \
    -d '{ "height": 7, "base": 20 }'

< HTTP/1.1 429
< X-Rate-Limit-Retry-After-Seconds: 212
{ "status": 429, "error": "Too Many Requests", "message": "You have exhausted your API Request Quota" } 
```

## 7.**结论**

在本文中，我们展示了使用 Bucket4j 对 Spring APIs 进行速率限制的几种不同方法。要了解更多信息，请务必查看官方[文档](https://web.archive.org/web/20221208143832/https://github.com/vladimir-bukhtoyarov/bucket4j/blob/master/README.md)。

像往常一样，所有例子的源代码都可以在 GitHub 的[上找到。](https://web.archive.org/web/20221208143832/https://github.com/eugenp/tutorials/tree/master/spring-boot-modules/spring-boot-libraries)