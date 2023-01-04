# 使用 Spring Boot 在运行时启用和禁用端点

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-boot-enable-disable-endpoints-at-runtime>

## 1.概观

Spring Boot 应用程序中的端点是与其交互的机制。有时，比如在计划外的维护窗口期间，我们可能希望暂时限制应用程序与外界的交互。

在本教程中，**我们将学习使用一些流行的库在运行时启用和禁用 Spring Boot 应用程序**中的端点，例如 [Spring Cloud](/web/20221118043649/https://www.baeldung.com/spring-cloud-series) 、 [Spring Actuator](/web/20221118043649/https://www.baeldung.com/spring-boot-actuators) 和 [Apache 的 Commons 配置](https://web.archive.org/web/20221118043649/https://commons.apache.org/proper/commons-configuration/)。

## 2.设置

在这一节中，让我们集中精力设置我们的 Spring Boot 项目的关键方面。

### 2.1.Maven 依赖性

首先，我们需要我们的 Spring Boot 应用程序来**公开`/refresh`端点**，所以让我们在项目的`pom.xml` 文件中添加`[spring-boot-starter-actuator](https://web.archive.org/web/20221118043649/https://search.maven.org/search?q=g:%20org.springframework.boot%20AND%20a:spring-boot-starter-actuator)`依赖项:

```java
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-actuator</artifactId>
    <version>2.7.5</version>
</dependency>
```

接下来，因为我们稍后将需要`@RefreshScope`注释来重新加载环境中的属性源，所以让我们添加`[spring-cloud-starter](https://web.archive.org/web/20221118043649/https://search.maven.org/search?q=g:%20org.springframework.cloud%20AND%20a:spring-cloud-starter)`依赖项:

```java
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter</artifactId>
    <version>3.1.5</version>
</dependency>
```

此外，我们还必须**在项目的`pom.xml`** 文件的依赖管理部分添加[弹簧云](https://web.archive.org/web/20221118043649/https://search.maven.org/search?q=g:%20org.springframework.cloud%20AND%20a:spring-cloud-dependencies)的 [BOM](/web/20221118043649/https://www.baeldung.com/spring-maven-bom#2-what-is-maven-bom) ，以便 Maven 使用兼容版本的 `spring-cloud-starter`:

```java
<dependencyManagement>
    <dependencies>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-dependencies</artifactId>
            <version>2021.0.5</version>
            <type>pom</type>
            <scope>import</scope>
        </dependency>
    </dependencies>
</dependencyManagement>
```

最后，因为我们需要在运行时重新加载文件的能力，所以我们还要添加`[commons-configuration](https://web.archive.org/web/20221118043649/https://search.maven.org/search?q=g:%20commons-configuration%20AND%20a:commons-configuration)`依赖项:

```java
<dependency>
    <groupId>commons-configuration</groupId>
    <artifactId>commons-configuration</artifactId>
    <version>1.10</version>
</dependency>
```

### 2.2.配置

首先，让我们将配置添加到`application.properties`文件中，以便**在我们的应用程序中启用`/refresh`端点**:

```java
management.server.port=8081
management.endpoints.web.exposure.include=refresh
```

接下来，让我们定义一个可以用来重新加载属性的附加源:

```java
dynamic.endpoint.config.location=file:extra.properties
```

此外，让我们在`application.properties`文件中定义`spring.properties.refreshDelay`属性:

```java
spring.properties.refreshDelay=1
```

最后，让我们向`extra.properties`文件添加两个属性:

```java
endpoint.foo=false
endpoint.regex=.*
```

在后面的部分中，我们将理解这些附加属性的核心意义。

### 2.3.API 端点

首先，让我们定义一个在`/foo`路径可用的示例 **`GET` API:**

```java
@GetMapping("/foo")
public String fooHandler() {
    return "foo";
}
```

接下来，让我们再定义两个分别在`/bar1`和`/bar2`路径可用的**`GET`API:**

```java
@GetMapping("/bar1")
public String bar1Handler() {
    return "bar1";
}

@GetMapping("/bar2")
public String bar2Handler() {
    return "bar2";
}
```

在接下来的小节中，我们将学习如何切换单个端点，比如`/foo`。此外，我们还将看到如何切换一组端点，即`/bar1`和`/bar2`，它们可以通过一个简单的正则表达式识别。

### 2.4.配置`DynamicEndpointFilter`

为了在运行时切换一组端点，我们可以使用一个`[Filter](/web/20221118043649/https://www.baeldung.com/spring-boot-add-filter)`。通过使用`endpoint.regex` 模式匹配请求的端点，我们可以允许它成功，或者为不成功的匹配发送`503` HTTP 响应状态。

所以，让我们用**通过扩展** `**OncePerRequestFilter**`来定义`DynamicEndpointFilter`类:

```java
public class DynamicEndpointFilter extends OncePerRequestFilter {
    private Environment environment;

    // ...
}
```

此外，我们需要通过覆盖`doFilterInternal()`方法来添加模式匹配的逻辑:

```java
@Override
protected void doFilterInternal(HttpServletRequest request, HttpServletResponse response, 
  FilterChain filterChain) throws ServletException, IOException {
    String path = request.getRequestURI();
    String regex = this.environment.getProperty("endpoint.regex");
    Pattern pattern = Pattern.compile(regex);
    Matcher matcher = pattern.matcher(path);
    boolean matches = matcher.matches();

    if (!matches) {
        response.sendError(HttpStatus.SERVICE_UNAVAILABLE.value(), "Service is unavailable");
    } else {
        filterChain.doFilter(request,response);
    }
}
```

我们必须注意到，`endpoint.regex`属性的初始值是“`.*`，它允许所有请求通过这个`Filter.`

## 3.使用环境属性切换

在这一节中，我们将学习如何从`extra.properties`文件热重新加载环境属性。

### 3.1.重新加载配置

为此，让我们首先使用`FileChangedReloadingStrategy`为`PropertiesConfiguration`定义一个 bean:

```java
@Bean
@ConditionalOnProperty(name = "dynamic.endpoint.config.location", matchIfMissing = false)
public PropertiesConfiguration propertiesConfiguration(
  @Value("${dynamic.endpoint.config.location}") String path,
  @Value("${spring.properties.refreshDelay}") long refreshDelay) throws Exception {
    String filePath = path.substring("file:".length());
    PropertiesConfiguration configuration = new PropertiesConfiguration(
      new File(filePath).getCanonicalPath());
    FileChangedReloadingStrategy fileChangedReloadingStrategy = new FileChangedReloadingStrategy();
    fileChangedReloadingStrategy.setRefreshDelay(refreshDelay);
    configuration.setReloadingStrategy(fileChangedReloadingStrategy);
    return configuration;
}
```

我们必须注意到属性的**来源是使用`application.properties`文件中的`dynamic.endpoint.config.location`属性**得到的。此外，重载发生时有一个由`spring.properties.refreshDelay`属性定义的`1`秒的时间延迟。

接下来，我们需要在运行时读取特定于端点的属性。所以，让我们用属性获取器来定义`EnvironmentConfigBean`:

```java
@Component
public class EnvironmentConfigBean {

    private final Environment environment;

    public EnvironmentConfigBean(@Autowired Environment environment) {
        this.environment = environment;
    }

    public String getEndpointRegex() {
        return environment.getProperty("endpoint.regex");
    }

    public boolean isFooEndpointEnabled() {
        return Boolean.parseBoolean(environment.getProperty("endpoint.foo"));
    }

    public Environment getEnvironment() {
        return environment;
    }
} 
```

接下来，让我们用**创建一个`FilterRegistrationBean`来注册`DynamicEndpointFilter`和**:

```java
@Bean
@ConditionalOnBean(EnvironmentConfigBean.class)
public FilterRegistrationBean<DynamicEndpointFilter> dynamicEndpointFilterFilterRegistrationBean(
  EnvironmentConfigBean environmentConfigBean) {
    FilterRegistrationBean<DynamicEndpointFilter> registrationBean = new FilterRegistrationBean<>();
    registrationBean.setFilter(new DynamicEndpointFilter(environmentConfigBean.getEnvironment()));
    registrationBean.addUrlPatterns("*");
    return registrationBean;
}
```

### 3.2.确认

首先，让我们运行应用程序并访问`/bar1`或`/bar2`API:

```java
$ curl -iXGET http://localhost:9090/bar1
HTTP/1.1 200 
Content-Type: text/plain;charset=ISO-8859-1
Content-Length: 4
Date: Sat, 12 Nov 2022 12:46:32 GMT

bar1
```

正如所料，**我们得到了`200 OK` HTTP 响应，因为我们保留了`endpoint.regex`属性的初始值来启用所有端点**。

接下来，让我们通过更改`extra.properties`文件中的`endpoint.regex`属性来仅启用`/foo`端点:

```java
endpoint.regex=.*/foo
```

接下来，让我们看看是否能够访问`/bar1` API 端点:

```java
$ curl -iXGET http://localhost:9090/bar1
HTTP/1.1 503 
Content-Type: application/json
Transfer-Encoding: chunked
Date: Sat, 12 Nov 2022 12:56:12 GMT
Connection: close

{"timestamp":1668257772354,"status":503,"error":"Service Unavailable","message":"Service is unavailable","path":"/springbootapp/bar1"}
```

正如所料，`DynamicEndpointFilter`禁用了这个端点，并发送了一个带有 HTTP `503`状态代码的错误响应。

最后，我们还可以检查我们是否能够访问`/foo` API 端点:

```java
$ curl -iXGET http://localhost:9090/foo
HTTP/1.1 200 
Content-Type: text/plain;charset=ISO-8859-1
Content-Length: 3
Date: Sat, 12 Nov 2022 12:57:39 GMT

foo
```

完美！看起来我们做对了。

## 4.使用弹簧云和致动器进行切换

在本节中，我们将学习另一种方法，使用`@RefreshScope`注释和执行器`/refresh`端点在运行时切换 API 端点。

### 4.1.带`@RefreshScope`的端点配置

首先，我们需要**定义用于切换端点的配置 bean，并用`@RefreshScope`** 对其进行注释:

```java
@Component
@RefreshScope
public class EndpointRefreshConfigBean {

    private boolean foo;
    private String regex;

    public EndpointRefreshConfigBean(@Value("${endpoint.foo}") boolean foo, 
      @Value("${endpoint.regex}") String regex) {
        this.foo = foo;
        this.regex = regex;
    }
    // getters and setters
}
```

接下来，我们需要通过创建包装类，如 [`ReloadableProperties`](/web/20221118043649/https://www.baeldung.com/spring-reloading-properties#2-reloading-properties-instance) 和`[ReloadablePropertySource](/web/20221118043649/https://www.baeldung.com/spring-reloading-properties#1-reloading-environment-properties)`，使这些属性可被发现和可重载。

最后，让我们更新 API 处理程序，使用一个`EndpointRefreshConfigBean`实例来控制切换流:

```java
@GetMapping("/foo")
public ResponseEntity<String> fooHandler() {
    if (endpointRefreshConfigBean.isFoo()) {
        return ResponseEntity.status(200).body("foo");
    } else {
        return ResponseEntity.status(503).body("endpoint is unavailable");
    }
}
```

### 4.2.确认

首先，当`endpoint.foo`属性的值被设置为`true`时，让我们验证一下`/foo`端点:

```java
$ curl -isXGET http://localhost:9090/foo
HTTP/1.1 200
Content-Type: text/plain;charset=ISO-8859-1
Content-Length: 3
Date: Sat, 12 Nov 2022 15:28:52 GMT

foo
```

接下来，让我们**将`endpoint.foo`属性的值设置为`false`** `,`，并检查端点是否仍可访问:

```java
endpoint.foo=false
```

我们会注意到`/foo`端点仍然处于启用状态。这是因为我们需要通过调用`/refresh`端点来**重新加载属性源。所以，让我们做一次:**

```java
$ curl -Is --request POST 'http://localhost:8081/actuator/refresh'
HTTP/1.1 200
Content-Type: application/vnd.spring-boot.actuator.v3+json
Transfer-Encoding: chunked
Date: Sat, 12 Nov 2022 15:34:24 GMT 
```

最后，让我们尝试访问`/foo`端点:

```java
$ curl -isXGET http://localhost:9090/springbootapp/foo
HTTP/1.1 503
Content-Type: text/plain;charset=ISO-8859-1
Content-Length: 23
Date: Sat, 12 Nov 2022 15:35:26 GMT
Connection: close

endpoint is unavailable
```

我们可以看到，刷新后端点被禁用。

### 4.3.利弊

与直接从环境中获取属性相比，Spring Cloud 和 Actuator 方法有优点也有缺点。

首先，当我们依赖于`/refresh`端点时，我们拥有比基于时间的文件重载策略更好的控制。因此应用程序不会在后台进行不必要的 I/O 调用。然而，**在分布式系统的情况下，我们需要确保为所有节点**调用`/refresh`端点。

其次，管理带有`@RefreshScope`注释的配置 bean 需要我们明确定义`EndpointRefreshConfigBean`类中的成员变量，以映射到`extra.properties`文件中的属性。因此，无论何时我们添加或删除属性，这种方法**都会增加在配置 beans 中进行代码更改的开销。**

最后，我们还必须注意，脚本可以轻松解决第一个问题，第二个问题更具体地涉及我们如何利用属性。**如果我们使用基于正则表达式的 URL 模式和`Filter,`，那么我们可以用一个属性**控制多个端点，而不需要修改配置 bean 的代码。

## 5.结论

在这篇文章中，**我们探索了在运行时切换 API 端点的多种策略**在一个 Spring Boot 应用中。在这样做的时候，我们利用了一些核心概念，比如[属性的热重装](/web/20221118043649/https://www.baeldung.com/spring-reloading-properties)和`@RefreshScope`注释。

和往常一样，该教程的完整源代码可以在 GitHub 上的[处获得。](https://web.archive.org/web/20221118043649/https://github.com/eugenp/tutorials/tree/master/spring-boot-modules/spring-boot-mvc-5)