# 使用 RestTemplateBuilder 配置 RestTemplate

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-rest-template-builder>

## 1。简介

在这个快速教程中，我们将看看**如何配置一个 Spring `RestTemplate ` bean。**

让我们从讨论三种主要的配置类型开始:

*   使用默认`RestTemplateBuilder`
*   使用`RestTemplateCustomizer`
*   创造我们自己的`RestTemplateBuilder`

为了能够容易地测试这一点，请按照指南[如何建立一个简单的 Spring Boot 应用程序](/web/20220707143824/https://www.baeldung.com/spring-boot-start)。

## 2。配置使用默认的`RestTemplateBuilder`

为了这样配置一个`RestTemplate`，我们需要**将 Spring Boot** 提供的默认`RestTemplateBuilder` bean 注入到我们的类中:

```java
private RestTemplate restTemplate;

@Autowired
public HelloController(RestTemplateBuilder builder) {
    this.restTemplate = builder.build();
}
```

用这种方法创建的`RestTemplate ` bean 的**范围被限制在我们构建它的类**中。

## 3。配置使用`RestTemplateCustomizer`

通过这种方法，我们可以**创建应用范围的附加定制。**

这是一个稍微复杂一点的方法。为此，我们需要**创建一个实现`RestTemplateCustomizer,`** 的类，并将其定义为一个 bean:

```java
public class CustomRestTemplateCustomizer implements RestTemplateCustomizer {
    @Override
    public void customize(RestTemplate restTemplate) {
        restTemplate.getInterceptors().add(new CustomClientHttpRequestInterceptor());
    }
}
```

`CustomClientHttpRequestInterceptor `拦截器正在对请求进行基本的日志记录:

```java
public class CustomClientHttpRequestInterceptor implements ClientHttpRequestInterceptor {
    private static Logger LOGGER = LoggerFactory
      .getLogger(CustomClientHttpRequestInterceptor.class);

    @Override
    public ClientHttpResponse intercept(
      HttpRequest request, byte[] body, 
      ClientHttpRequestExecution execution) throws IOException {

        logRequestDetails(request);
        return execution.execute(request, body);
    }
    private void logRequestDetails(HttpRequest request) {
        LOGGER.info("Headers: {}", request.getHeaders());
        LOGGER.info("Request Method: {}", request.getMethod());
        LOGGER.info("Request URI: {}", request.getURI());
    }
}
```

现在，我们将`CustomRestTemplateCustomizer `定义为配置类或 Spring Boot 应用程序类中的 bean:

```java
@Bean
public CustomRestTemplateCustomizer customRestTemplateCustomizer() {
    return new CustomRestTemplateCustomizer();
}
```

有了这个配置，我们将在应用程序中使用的每个`RestTemplate`**都将在其上设置定制的拦截器。**

## 4。通过创建我们自己的`RestTemplateBuilder`配置

这是定制一个`RestTemplate.`的最极端的方法，它**禁用了`RestTemplateBuilder` `,`的默认自动配置，所以我们需要自己定义:**

```java
@Bean
@DependsOn(value = {"customRestTemplateCustomizer"})
public RestTemplateBuilder restTemplateBuilder() {
    return new RestTemplateBuilder(customRestTemplateCustomizer());
}
```

在这之后，我们可以**将定制构建器**注入到我们的类中，就像我们对默认的`RestTemplateBuilder`所做的那样，并像往常一样创建一个`RestTemplate` :

```java
private RestTemplate restTemplate;

@Autowired
public HelloController(RestTemplateBuilder builder) {
    this.restTemplate = builder.build();
}
```

## 5。结论

我们已经看到了如何用默认的`RestTemplateBuilder`配置一个`RestTemplate`，构建我们自己的`RestTemplateBuilder,`或者使用一个`RestTemplateCustomizer` bean `.`

和往常一样，这个例子的完整代码库可以在我们的 [GitHub 库](https://web.archive.org/web/20220707143824/https://github.com/eugenp/tutorials/tree/master/spring-web-modules/spring-resttemplate)中找到。