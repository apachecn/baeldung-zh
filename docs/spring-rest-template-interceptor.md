# 使用 Spring RestTemplate 拦截器

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-rest-template-interceptor>

## 1。概述

在本教程中，我们将学习如何实现一个 Spring [`RestTemplate`](/web/20220926202551/https://www.baeldung.com/rest-template) 拦截器。

我们将看一个例子，在这个例子中，我们将创建一个拦截器，为响应添加一个定制的头。

## 2.拦截器使用场景

除了头修改之外，`RestTemplate`拦截器有用的其他一些用例有:

*   请求和响应日志记录
*   使用可配置的回退策略重试请求
*   基于某些请求参数的请求拒绝
*   更改请求 URL 地址

## 3。创建拦截器

在大多数编程范例中，拦截器是一个必不可少的部分，它使程序员能够通过拦截来控制执行。 Spring 框架还支持各种不同用途的拦截器。

Spring `RestTemplate`允许我们添加实现 [`ClientHttpRequestInterceptor`](https://web.archive.org/web/20220926202551/https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/http/client/ClientHttpRequestInterceptor.html) 接口的拦截器。这个接口的`intercept(HttpRequest, byte[], ClientHttpRequestExecution)`方法将截取给定的请求，并通过给我们提供对`request`、`body`和`execution`对象的访问来返回响应。

我们将使用`ClientHttpRequestExecution`参数进行实际执行，并将请求传递给后续的流程链。

**作为第一步，让我们创建一个实现`ClientHttpRequestInterceptor`接口的拦截器类:**

```java
public class RestTemplateHeaderModifierInterceptor
  implements ClientHttpRequestInterceptor {

    @Override
    public ClientHttpResponse intercept(
      HttpRequest request, 
      byte[] body, 
      ClientHttpRequestExecution execution) throws IOException {

        ClientHttpResponse response = execution.execute(request, body);
        response.getHeaders().add("Foo", "bar");
        return response;
    }
}
```

**我们的拦截器将为每个传入的请求**调用，一旦执行完成并返回，它将为每个响应添加一个自定义头`Foo` 。

因为`intercept()`方法包含了`request`和`body`作为参数，所以也可以对请求做任何修改，甚至基于某些条件拒绝请求的执行。

## 4。`RestTemplate`设置

现在我们已经创建了我们的拦截器，让我们创建`RestTemplate` bean 并将我们的拦截器添加到其中:

```java
@Configuration
public class RestClientConfig {

    @Bean
    public RestTemplate restTemplate() {
        RestTemplate restTemplate = new RestTemplate();

        List<ClientHttpRequestInterceptor> interceptors
          = restTemplate.getInterceptors();
        if (CollectionUtils.isEmpty(interceptors)) {
            interceptors = new ArrayList<>();
        }
        interceptors.add(new RestTemplateHeaderModifierInterceptor());
        restTemplate.setInterceptors(interceptors);
        return restTemplate;
    }
}
```

在某些情况下，可能已经有拦截器添加到了`RestTemplate`对象中。因此，为了确保一切按预期运行，我们的代码将只在拦截器列表为空时初始化它。

正如我们的代码所示，我们使用默认的构造函数来创建`RestTemplate`对象，但是在一些场景中，我们需要读取请求/响应流两次。

例如，如果我们希望我们的拦截器充当请求/响应记录器，那么我们需要读取它两次——第一次由拦截器读取，第二次由客户机读取。

默认实现只允许我们读取响应流一次。为了迎合这种特定的场景，Spring 提供了一个名为`BufferingClientHttpRequestFactory.` 的特殊类，顾名思义，这个类将在 JVM 内存中缓冲请求/响应以供多次使用。

下面是如何使用`BufferingClientHttpRequestFactory`初始化`RestTemplate`对象以启用请求/响应流缓存:

```java
RestTemplate restTemplate 
  = new RestTemplate(
    new BufferingClientHttpRequestFactory(
      new SimpleClientHttpRequestFactory()
    )
  );
```

## 5。测试我们的示例

下面是测试我们的`RestTemplate`拦截器的 JUnit 测试用例:

```java
public class RestTemplateItegrationTest {

    @Autowired
    RestTemplate restTemplate;

    @Test
    public void givenRestTemplate_whenRequested_thenLogAndModifyResponse() {
        LoginForm loginForm = new LoginForm("username", "password");
        HttpEntity<LoginForm> requestEntity
          = new HttpEntity<LoginForm>(loginForm);
        HttpHeaders headers = new HttpHeaders();
        headers.setContentType(MediaType.APPLICATION_JSON);

        ResponseEntity<String> responseEntity
          = restTemplate.postForEntity(
            "http://httpbin.org/post", requestEntity, String.class
          );

        Assertions.assertEquals(responseEntity.getStatusCode(), HttpStatus.OK);
        Assertions.assertEquals(responseEntity.getHeaders()
                .get("Foo")
                .get(0), "bar");
    }
}
```

这里，我们使用免费托管的 HTTP 请求和响应服务[http://httpbin.org](https://web.archive.org/web/20220926202551/https://httpbin.org/)**来发布我们的数据。这个测试服务将返回我们的请求体和一些元数据。**

## 6。结论

本教程是关于如何设置一个拦截器并将其添加到`RestTemplate`对象中。这种拦截器也可以用于过滤、监视和控制传入的请求。

`RestTemplate`拦截器的一个常见用例是头部修改——我们已经在本文中详细说明了这一点。

和往常一样，你可以在 [Github 项目](https://web.archive.org/web/20220926202551/https://github.com/eugenp/tutorials/tree/master/spring-web-modules/spring-resttemplate)中找到示例代码。这是一个基于 Maven 的项目，因此应该很容易导入和运行。