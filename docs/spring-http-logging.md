# spring——记录传入请求

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-http-logging>

## 1。简介

在这个快速教程中，我们将演示使用 Spring 的日志过滤器记录传入请求的基础知识。如果我们刚刚开始学习日志，我们可以看看这篇[日志介绍文章](/web/20221208143917/https://www.baeldung.com/java-logging-intro)，以及 [SLF4J 文章](/web/20221208143917/https://www.baeldung.com/slf4j-with-log4j2-logback)。

## 2。Maven 依赖关系

日志依赖将与 intro 文章中的相同；我们将简单地在这里添加弹簧:

```java
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-core</artifactId>
    <version>5.2.2.RELEASE</version>   
</dependency>
```

在这里可以找到[弹簧芯](https://web.archive.org/web/20221208143917/https://search.maven.org/classic/#search%7Cgav%7C1%7Cg%3A%22org.springframework%22%20AND%20a%3A%22spring-core%22)的最新版本。

## 3。基本网络控制器

首先，我们将定义一个在示例中使用的控制器:

```java
@RestController
public class TaxiFareController {

    @GetMapping("/taxifare/get/")
    public RateCard getTaxiFare() {
        return new RateCard();
    }

    @PostMapping("/taxifare/calculate/")
    public String calculateTaxiFare(
      @RequestBody @Valid TaxiRide taxiRide) {

        // return the calculated fare
    }
}
```

## 4。自定义请求记录

Spring 提供了一种机制来配置用户定义的拦截器，以便在 web 请求之前和之后执行操作。

在 Spring 请求拦截器中，一个值得注意的接口是 `HandlerInterceptor`，我们可以通过实现以下方法使用它来记录传入的请求:

1.  `preHandle() –` 我们在实际控制器服务方法之前执行这个方法
2.  在控制器准备好发送响应后，我们执行这个方法

此外，Spring 以`HandlerInterceptorAdaptor` 类的形式提供了`HandlerInterceptor` 接口的默认实现，用户可以对其进行扩展。

让我们通过扩展`HandlerInterceptorAdaptor `来创建自己的拦截器，如下所示:

```java
@Component
public class TaxiFareRequestInterceptor 
  extends HandlerInterceptorAdapter {

    @Override
    public boolean preHandle(
      HttpServletRequest request, 
      HttpServletResponse response, 
      Object handler) {
        return true;
    }

    @Override
    public void afterCompletion(
      HttpServletRequest request, 
      HttpServletResponse response, 
      Object handler, 
      Exception ex) {
        //
    }
}
```

最后，我们将在 MVC 生命周期中配置`TaxiRideRequestInterceptor` 来捕获映射到在`TaxiFareController` 类中定义的路径`/taxifare`的控制器方法调用的预处理和后处理:

```java
@Configuration
public class TaxiFareMVCConfig implements WebMvcConfigurer {

    @Autowired
    private TaxiFareRequestInterceptor taxiFareRequestInterceptor;

    @Override
    public void addInterceptors(InterceptorRegistry registry) {
        registry.addInterceptor(taxiFareRequestInterceptor)
          .addPathPatterns("/taxifare/*/");
    }
}
```

总之，`WebMvcConfigurer`通过调用`addInterceptors()` 方法将`TaxiFareRequestInterceptor` 添加到 spring MVC 生命周期中。

最大的挑战是获取请求和响应有效负载的副本以进行日志记录，同时仍然将请求的有效负载留给 servlet 来处理:

> 读取请求的主要问题是，一旦第一次读取输入流，它就被标记为已使用，不能再次读取。

应用程序在读取请求流后将引发异常:

```java
{
  "timestamp": 1500645243383,
  "status": 400,
  "error": "Bad Request",
  "exception": "org.springframework.http.converter
    .HttpMessageNotReadableException",
  "message": "Could not read document: Stream closed; 
    nested exception is java.io.IOException: Stream closed",
  "path": "/rest-log/taxifare/calculate/"
}
```

**为了克服这个问题**，我们可以利用缓存来存储请求流并将其用于日志记录。

Spring 提供了一些有用的类，比如[ContentCachingRequestWrapper](https://web.archive.org/web/20221208143917/https://docs.spring.io/spring/docs/current/javadoc-api/org/springframework/web/util/ContentCachingRequestWrapper.html)和[ContentCachingResponseWrapper](https://web.archive.org/web/20221208143917/https://docs.spring.io/spring/docs/current/javadoc-api/org/springframework/web/util/ContentCachingResponseWrapper.html)，这些类可以用于缓存请求数据以用于日志记录。

让我们调整`TaxiRideRequestInterceptor` 类的`preHandle()` ,使用`ContentCachingRequestWrapper`类缓存请求对象:

```java
@Override
public boolean preHandle(HttpServletRequest request, 
  HttpServletResponse response, Object handler) {

    HttpServletRequest requestCacheWrapperObject
      = new ContentCachingRequestWrapper(request);
    requestCacheWrapperObject.getParameterMap();
    // Read inputStream from requestCacheWrapperObject and log it
    return true;
}
```

正如我们所看到的，我们使用`ContentCachingRequestWrapper`类缓存请求对象，我们可以使用它来读取日志记录的有效负载数据，而不会干扰实际的请求对象:

```java
requestCacheWrapperObject.getContentAsByteArray();
```

**限制**

*   `ContentCachingRequestWrapper`类仅支持以下内容:

```java
Content-Type:application/x-www-form-urlencoded
Method-Type:POST
```

*   我们必须调用下面的方法来确保请求数据在使用之前缓存在`ContentCachingRequestWrapper`中:

```java
requestCacheWrapperObject.getParameterMap();
```

## 5。Spring 内置请求日志

Spring 提供了一个内置的解决方案来记录负载。我们可以通过使用 configuration 插入到 Spring 应用程序中来使用现成的过滤器。

`AbstractRequestLoggingFilter`是一个过滤器，提供日志记录的基本功能。子类应该覆盖`beforeRequest()`和`afterRequest()`方法来执行请求的实际日志记录。

Spring 框架提供了三个具体的实现类，我们可以用它们来记录传入的请求。这三个类别是:

*   `CommonsRequestLoggingFilter`
*   `Log4jNestedDiagnosticContextFilter`(已弃用)
*   `ServletContextRequestLoggingFilter`

现在让我们转到`CommonsRequestLoggingFilter,` 并配置它来捕获日志记录的传入请求。

### 5.1。配置 Spring Boot 应用程序

我们可以通过添加 bean 定义来配置 Spring Boot 应用程序，以启用请求日志记录:

```java
@Configuration
public class RequestLoggingFilterConfig {

    @Bean
    public CommonsRequestLoggingFilter logFilter() {
        CommonsRequestLoggingFilter filter
          = new CommonsRequestLoggingFilter();
        filter.setIncludeQueryString(true);
        filter.setIncludePayload(true);
        filter.setMaxPayloadLength(10000);
        filter.setIncludeHeaders(false);
        filter.setAfterMessagePrefix("REQUEST DATA : ");
        return filter;
    }
}
```

这个日志过滤器还要求我们将日志级别设置为 DEBUG。我们可以通过在`logback.xml`中添加以下元素来启用调试模式:

```java
<logger name="org.springframework.web.filter.CommonsRequestLoggingFilter">
    <level value="DEBUG" />
</logger>
```

启用调试级日志的另一种方式是在`application.properties`中添加以下内容:

```java
logging.level.org.springframework.web.filter.CommonsRequestLoggingFilter=
  DEBUG
```

### 5.2。配置传统 Web 应用程序

在标准的 Spring web 应用程序中，我们可以通过 XML 配置或 Java 配置来设置`Filter`。因此，让我们使用传统的基于 Java 的配置来设置`CommonsRequestLoggingFilter`。

我们知道，`CommonsRequestLoggingFilter`的`includePayload`属性默认设置为 false。在使用 Java 配置注入容器之前，我们需要一个自定义类来覆盖属性的值以启用`includePayload` :

```java
public class CustomeRequestLoggingFilter 
  extends CommonsRequestLoggingFilter {

    public CustomeRequestLoggingFilter() {
        super.setIncludeQueryString(true);
        super.setIncludePayload(true);
        super.setMaxPayloadLength(10000);
    }
}
```

然后我们需要使用[基于 Java 的 web 初始化器](/web/20221208143917/https://www.baeldung.com/spring-xml-vs-java-config)来注入`CustomeRequestLoggingFilter`:

```java
public class CustomWebAppInitializer implements 
  WebApplicationInitializer {
    public void onStartup(ServletContext container) {

        AnnotationConfigWebApplicationContext context 
          = new AnnotationConfigWebApplicationContext();
	context.setConfigLocation("com.baeldung");
	container.addListener(new ContextLoaderListener(context));

	ServletRegistration.Dynamic dispatcher 
          = container.addServlet("dispatcher", 
          new DispatcherServlet(context));
	dispatcher.setLoadOnStartup(1);
	dispatcher.addMapping("/");	

	container.addFilter("customRequestLoggingFilter", 
          CustomeRequestLoggingFilter.class)
          .addMappingForServletNames(null, false, "dispatcher");
    }
}
```

## 6。行动范例

最后，我们可以将 Spring Boot 与上下文关联起来，以查看传入请求的日志记录是否如预期那样工作:

```java
@Test
public void givenRequest_whenFetchTaxiFareRateCard_thanOK() {
    TestRestTemplate testRestTemplate = new TestRestTemplate();
    TaxiRide taxiRide = new TaxiRide(true, 10l);
    String fare = testRestTemplate.postForObject(
      URL + "calculate/", 
      taxiRide, String.class);

    assertThat(fare, equalTo("200"));
}
```

## 7。结论

在本文中，我们学习了如何使用拦截器实现基本的 web 请求日志记录。我们还探讨了该解决方案的局限性和挑战。

然后我们讨论了内置的 filter 类，它提供了简单易用的日志记录机制。

与往常一样，GitHub 上的[提供了示例和代码片段的实现。](https://web.archive.org/web/20221208143917/https://github.com/eugenp/tutorials/tree/master/spring-boot-modules/spring-boot-runtime)