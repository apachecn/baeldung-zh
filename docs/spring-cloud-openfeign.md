# 春云 OpenFeign 简介

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-cloud-openfeign>

## 1。概述

在本教程中，我们将描述[Spring Cloud open feign](https://web.archive.org/web/20220815030846/https://spring.io/projects/spring-cloud-openfeign)——一个用于 Spring Boot 应用的声明式 REST 客户端。

利用可插入的注释支持，包括佯注释和 JAX-RS 注释，佯使得编写 web 服务客户端变得更加容易。

另外， [Spring Cloud](/web/20220815030846/https://www.baeldung.com/spring-cloud-series) 增加了对 [Spring MVC 注解](/web/20220815030846/https://www.baeldung.com/spring-mvc-annotations)的支持，并支持使用与 Spring Web 中相同的 [`HttpMessageConverters`](/web/20220815030846/https://www.baeldung.com/spring-httpmessageconverter-rest) 。

使用 Feign 的一个好处是，除了接口定义之外，我们不需要编写任何调用服务的代码。

## 2。依赖性

首先，我们将创建一个 Spring Boot web 项目，并将`spring-cloud-starter-openfeign`依赖项添加到我们的`pom.xml`文件中:

```java
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-openfeign</artifactId>
</dependency>
```

此外，我们需要添加 `spring-cloud-dependencies`:

```java
 <dependencyManagement>
     <dependencies>
         <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-dependencies</artifactId>
            <version>${spring-cloud.version}</version>
            <type>pom</type>
            <scope>import</scope>
        </dependency>
    </dependencies>
</dependencyManagement>
```

我们可以在 Maven Central 上找到`[spring-cloud-starter-openfeign](https://web.archive.org/web/20220815030846/https://search.maven.org/search?q=g:org.springframework.cloud%20AND%20a:spring-cloud-starter-openfeign)`和 [`spring-cloud-dependencies`](https://web.archive.org/web/20220815030846/https://search.maven.org/search?q=g:org.springframework.cloud%20AND%20a:spring-cloud-dependencies) 的最新版本。

## 3。假装客户端

接下来，我们需要将`@EnableFeignClients` 添加到我们的主类中:

```java
@SpringBootApplication
@EnableFeignClients
public class ExampleApplication {

    public static void main(String[] args) {
        SpringApplication.run(ExampleApplication.class, args);
    }
}
```

有了这个注释，我们就可以对声明为假客户端的接口进行组件扫描。

然后**我们使用`@FeignClient` 注释**声明一个虚拟客户端:

```java
@FeignClient(value = "jplaceholder", url = "https://jsonplaceholder.typicode.com/")
public interface JSONPlaceHolderClient {

    @RequestMapping(method = RequestMethod.GET, value = "/posts")
    List<Post> getPosts();

    @RequestMapping(method = RequestMethod.GET, value = "/posts/{postId}", produces = "application/json")
    Post getPostById(@PathVariable("postId") Long postId);
}
```

在这个例子中，我们已经配置了一个客户端来读取[JSONPlaceholder API](https://web.archive.org/web/20220815030846/https://jsonplaceholder.typicode.com/)。

在`@FeignClient` 注释中传递的*值*参数是一个强制的、任意的客户端名称，而使用`url`参数，我们指定 API 基本 URL。

此外，由于这个接口是一个虚拟客户端，我们可以使用 Spring Web 注释来声明我们想要访问的 API。

## 4。配置

现在，非常重要的一点是要理解**每个虚拟客户端都是由一组可定制的组件组成的。**

Spring Cloud 使用`FeignClientsConfiguration`类按需为每个命名的客户端创建一个新的缺省集，我们可以按照下一节中的解释对其进行定制。

上面的类包含这些 beans:

*   解码器—`ResponseEntityDecoder`，包装`SpringDecoder`，用于解码`Response`
*   编码器-`SpringEncoder`用于对`RequestBody`进行编码。
*   logger–`Slf4jLogger` 是 Feign 使用的默认记录器。
*   合同-`SpringMvcContract`，提供注释处理
*   feign-Builder–`HystrixFeign.Builder`用于构建组件。
*   客户端—`LoadBalancerFeignClient`或默认假装客户端

### 4.1。自定义 Beans 配置

**如果我们想要定制这些 beans 中的一个或多个**，我们可以使用一个`@Configuration`类覆盖它们，然后将它添加到`FeignClient`注释中:

```java
@FeignClient(value = "jplaceholder",
  url = "https://jsonplaceholder.typicode.com/",
  configuration = MyClientConfiguration.class)
```

```java
@Configuration
public class MyClientConfiguration {

    @Bean
    public OkHttpClient client() {
        return new OkHttpClient();
    }
}
```

在这个例子中，我们告诉 Feign 使用 [`OkHttpClient`](/web/20220815030846/https://www.baeldung.com/guide-to-okhttp) 而不是默认的来支持 HTTP/2。

Feign 支持针对不同用例的多个客户端，包括`ApacheHttpClient`，它随请求发送更多的头，例如`Content-Length`，这是一些服务器所期望的。

为了使用这些客户端，我们不要忘记将所需的依赖项添加到我们的`pom.xml`文件中:

```java
<dependency>
    <groupId>io.github.openfeign</groupId>
    <artifactId>feign-okhttp</artifactId>
</dependency>

<dependency>
    <groupId>io.github.openfeign</groupId>
    <artifactId>feign-httpclient</artifactId>
</dependency>
```

我们可以在 Maven Central 上找到[`feign-okhttp`](https://web.archive.org/web/20220815030846/https://search.maven.org/search?q=g:io.github.openfeign%20AND%20a:feign-okhttp)[`feign-httpclient`](https://web.archive.org/web/20220815030846/https://search.maven.org/search?q=g:io.github.openfeign%20AND%20a:feign-httpclient)的最新版本。

### 4.2。使用属性的配置

不使用`@Configuration`、**类，我们可以使用应用程序属性来配置假装客户端**，如这个`application.yaml`示例所示:

```java
feign:
  client:
    config:
      default:
        connectTimeout: 5000
        readTimeout: 5000
        loggerLevel: basic
```

使用这种配置，我们将应用程序中每个声明的客户端的超时设置为 5 秒，日志级别设置为`basic`。

最后，我们可以用`default`作为客户机名来创建配置，以配置所有的`@FeignClient`对象，或者我们可以为配置声明一个假的客户机名:

```java
feign:
  client:
    config:
      jplaceholder:
```

如果我们有`@Configuration` bean 和配置属性，配置属性将覆盖`@Configuration`值。

## 5。截击机

添加拦截器是 Feign 提供的另一个有用的特性。

拦截器可以为每个 HTTP 请求/响应执行各种隐式任务，从身份验证到日志记录。

在这一节中，我们将实现自己的拦截器，并使用 Spring Cloud OpenFeign 提供的开箱即用的拦截器。两者都将**为每个请求添加一个基本认证头。**

### 5.1.实施`RequestInterceptor`

让我们实现我们的定制请求拦截器:

```java
@Bean
public RequestInterceptor requestInterceptor() {
  return requestTemplate -> {
      requestTemplate.header("user", username);
      requestTemplate.header("password", password);
      requestTemplate.header("Accept", ContentType.APPLICATION_JSON.getMimeType());
  };
}
```

此外，要将拦截器添加到请求链中，我们只需要将这个 bean 添加到我们的`@Configuration`类中，或者，正如我们之前看到的，在属性文件中声明它:

```java
feign:
  client:
    config:
      default:
        requestInterceptors:
          com.baeldung.cloud.openfeign.JSONPlaceHolderInterceptor
```

### 5.2.使用`BasicAuthRequestInterceptor`

或者，我们可以使用 Spring Cloud OpenFeign 提供的`BasicAuthRequestInterceptor`类:

```java
@Bean
public BasicAuthRequestInterceptor basicAuthRequestInterceptor() {
    return new BasicAuthRequestInterceptor("username", "password");
}
```

就这么简单。现在，所有的请求都将包含基本的身份验证头。

## 6。Hystrix 支架

Feign 支持 [Hystrix](/web/20220815030846/https://www.baeldung.com/spring-cloud-netflix-hystrix) ，所以如果我们启用了它，**我们就可以实现回退模式。**

使用回退模式，当远程服务调用失败时，服务使用者不会生成异常，而是会执行替代代码路径，尝试通过另一种方式执行操作。

为了实现这个目标，我们需要通过在属性文件中添加`feign.hystrix.enabled=true`来启用 Hystrix。

这允许我们实现当服务失败时调用的回退方法:

```java
@Component
public class JSONPlaceHolderFallback implements JSONPlaceHolderClient {

    @Override
    public List<Post> getPosts() {
        return Collections.emptyList();
    }

    @Override
    public Post getPostById(Long postId) {
        return null;
    }
}
```

为了让 Feign 知道已经提供了回退方法，我们还需要在`@FeignClient`注释中设置回退类:

```java
@FeignClient(value = "jplaceholder",
  url = "https://jsonplaceholder.typicode.com/",
  fallback = JSONPlaceHolderFallback.class)
public interface JSONPlaceHolderClient {
    // APIs
}
```

## 7 .**。记录日志**

对于每个 Feign 客户端，默认情况下会创建一个记录器。

要启用日志记录，我们应该使用客户端接口的包名在`application.propertie` s 文件中声明它:

```java
logging.level.com.baeldung.cloud.openfeign.client: DEBUG
```

或者，如果我们希望只为包中的一个特定客户端启用日志记录，我们可以使用完整的类名:

```java
logging.level.com.baeldung.cloud.openfeign.client.JSONPlaceHolderClient: DEBUG
```

**注意，假造日志仅响应于`DEBUG`级别。**

我们可以为每个客户端配置的 `Logger.Level`表示要记录多少内容:

```java
@Configuration
public class ClientConfiguration {

    @Bean
    Logger.Level feignLoggerLevel() {
        return Logger.Level.BASIC;
    }
}
```

有四种日志记录级别可供选择:

*   `NONE`–无日志记录，这是默认设置
*   `BASIC`–仅记录请求方法、URL 和响应状态
*   `HEADERS`–记录基本信息以及请求和响应标题
*   `FULL`–记录请求和响应的正文、标题和元数据

## 8。错误处理

Feign 的默认错误处理程序`ErrorDecoder.default`，总是抛出一个`FeignException`。

这种行为并不总是最有用的。所以，**要自定义抛出的异常，我们可以使用`CustomErrorDecoder`** :

```java
public class CustomErrorDecoder implements ErrorDecoder {
    @Override
    public Exception decode(String methodKey, Response response) {

        switch (response.status()){
            case 400:
                return new BadRequestException();
            case 404:
                return new NotFoundException();
            default:
                return new Exception("Generic error");
        }
    }
}
```

然后，正如我们之前所做的，我们必须通过向`@Configuration`类添加一个 bean 来替换默认的`ErrorDecoder`:

```java
@Configuration
public class ClientConfiguration {

    @Bean
    public ErrorDecoder errorDecoder() {
        return new CustomErrorDecoder();
    }
}
```

## 9。结论

在本文中，我们讨论了 Spring Cloud OpenFeign 及其在一个简单示例应用程序中的实现。

我们还看到了如何配置一个客户机，为我们的请求添加拦截器，以及使用`Hystrix`和`ErrorDecoder`处理错误。

像往常一样，本教程中显示的所有代码示例都可以从 GitHub 上的[处获得。](https://web.archive.org/web/20220815030846/https://github.com/eugenp/tutorials/tree/master/spring-cloud-modules/spring-cloud-openfeign)