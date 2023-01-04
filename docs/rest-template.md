# RestTemplate 指南

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/rest-template>

## 1。概述

在本教程中，我们将展示 Spring REST 客户端— `RestTemplate` —可以使用的广泛操作，并且使用得很好。

对于所有例子的 API 端，我们将从[到](https://web.archive.org/web/20221129014436/https://github.com/eugenp/tutorials/tree/master/spring-web-modules/spring-resttemplate)运行 RESTful 服务。

## 延伸阅读:

## [使用 RestTemplate 的基本认证](/web/20221129014436/https://www.baeldung.com/how-to-use-resttemplate-with-basic-authentication-in-spring)

How to do Basic Authentication with the Spring RestTemplate.[Read more](/web/20221129014436/https://www.baeldung.com/how-to-use-resttemplate-with-basic-authentication-in-spring) →

## [带摘要认证的 RestTemplate】](/web/20221129014436/https://www.baeldung.com/resttemplate-digest-authentication)

How to set up Digest Authentication for the Spring RestTemplate using HttpClient 4.[Read more](/web/20221129014436/https://www.baeldung.com/resttemplate-digest-authentication) →

## [探索 Spring Boot TestRestTemplate](/web/20221129014436/https://www.baeldung.com/spring-boot-testresttemplate)

Learn how to use the new TestRestTemplate in Spring Boot to test a simple API.[Read more](/web/20221129014436/https://www.baeldung.com/spring-boot-testresttemplate) →

## 2.弃用通知

从 Spring Framework 5 开始，除了 WebFlux 栈，Spring 还引入了一个名为`[WebClient](/web/20221129014436/https://www.baeldung.com/spring-5-webclient)`的新 HTTP 客户端。

`WebClient`是一个现代的、替代`[RestTemplate](/web/20221129014436/https://www.baeldung.com/spring-webclient-resttemplate)`的 HTTP 客户端。它不仅提供了传统的同步 API，还支持高效的非阻塞和异步方法。

也就是说，如果我们正在开发新的应用程序或者迁移旧的应用程序，**使用`WebClient`是个好主意。今后，`RestTemplate`将在未来版本中被弃用。**

## 3。使用 GET 检索资源

### 3.1。获取普通 JSON

让我们从简单的开始谈论 GET 请求，用**一个使用`getForEntity()` API** 的简单例子:

```java
RestTemplate restTemplate = new RestTemplate();
String fooResourceUrl
  = "http://localhost:8080/spring-rest/foos";
ResponseEntity<String> response
  = restTemplate.getForEntity(fooResourceUrl + "/1", String.class);
Assertions.assertEquals(response.getStatusCode(), HttpStatus.OK); 
```

**注意，我们拥有对 HTTP 响应**的完全访问权限，因此我们可以做一些事情，比如检查状态代码以确保操作成功，或者处理响应的实际主体:

```java
ObjectMapper mapper = new ObjectMapper();
JsonNode root = mapper.readTree(response.getBody());
JsonNode name = root.path("name");
Assertions.assertNotNull(name.asText());
```

我们在这里将响应体作为标准字符串使用，并使用 Jackson(以及 Jackson 提供的 JSON 节点结构)来验证一些细节。

### 3.2。检索 POJO 而不是 JSON

我们还可以将回应直接映射到资源 DTO:

```java
public class Foo implements Serializable {
    private long id;

    private String name;
    // standard getters and setters
}
```

现在我们可以简单地在模板中使用`getForObject` API:

```java
Foo foo = restTemplate
  .getForObject(fooResourceUrl + "/1", Foo.class);
Assertions.assertNotNull(foo.getName());
Assertions.assertEquals(foo.getId(), 1L); 
```

## 4。使用 HEAD 检索标题

现在，让我们先快速浏览一下 HEAD 的用法，然后再讨论更常见的方法。

我们将在这里使用`headForHeaders()` API:

```java
HttpHeaders httpHeaders = restTemplate.headForHeaders(fooResourceUrl);
Assertions.assertTrue(httpHeaders.getContentType().includes(MediaType.APPLICATION_JSON));
```

## 5。使用 POST 创建一个资源

**为了在 API 中创建新的资源，我们可以很好地利用`postForLocation()`、`postForObject()`或 `postForEntity()`API。**

第一个返回新创建的资源的 URI，而第二个返回资源本身。

### 5.1。`postForObject()`API

```java
RestTemplate restTemplate = new RestTemplate();

HttpEntity<Foo> request = new HttpEntity<>(new Foo("bar"));
Foo foo = restTemplate.postForObject(fooResourceUrl, request, Foo.class);
Assertions.assertNotNull(foo);
Assertions.assertEquals(foo.getName(), "bar");
```

### 5.2。`postForLocation()` API

类似地，让我们看看这个操作，它不是返回完整的资源，而是返回新创建的资源的`Location`:

```java
HttpEntity<Foo> request = new HttpEntity<>(new Foo("bar"));
URI location = restTemplate
  .postForLocation(fooResourceUrl, request);
Assertions.assertNotNull(location);
```

### 5.3。`exchange()`API

让我们来看看如何用更通用的`exchange` API 写一篇文章:

```java
RestTemplate restTemplate = new RestTemplate();
HttpEntity<Foo> request = new HttpEntity<>(new Foo("bar"));
ResponseEntity<Foo> response = restTemplate
  .exchange(fooResourceUrl, HttpMethod.POST, request, Foo.class);

Assertions.assertEquals(response.getStatusCode(), HttpStatus.CREATED);

Foo foo = response.getBody();

Assertions.assertNotNull(foo);
Assertions.assertEquals(foo.getName(), "bar"); 
```

### 5.4。提交表单数据

接下来，让我们看看如何使用 POST 方法提交表单。

**首先，我们需要将`Content-Type`头设置为`application/x-www-form-urlencoded.`**

这确保了可以向服务器发送一个大的查询字符串，其中包含由`&`分隔的名称/值对:

```java
HttpHeaders headers = new HttpHeaders();
headers.setContentType(MediaType.APPLICATION_FORM_URLENCODED);
```

我们可以将表单变量包装成一个`[LinkedMultiValueMap](https://web.archive.org/web/20221129014436/https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/util/LinkedMultiValueMap.html)`:

```java
MultiValueMap<String, String> map= new LinkedMultiValueMap<>();
map.add("id", "1");
```

接下来，**我们使用一个 [`HttpEntity`](https://web.archive.org/web/20221129014436/https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/http/HttpEntity.html) 实例**构建请求:

```java
HttpEntity<MultiValueMap<String, String>> request = new HttpEntity<>(map, headers);
```

最后，我们可以通过在端点上调用 [`restTemplate.postForEntity()`](https://web.archive.org/web/20221129014436/https://docs.spring.io/spring/docs/current/javadoc-api/org/springframework/web/client/RestTemplate.html#postForEntity-java.net.URI-java.lang.Object-java.lang.Class-) 来连接到 REST 服务:`/` foos `/form`

```java
ResponseEntity<String> response = restTemplate.postForEntity(
  fooResourceUrl+"/form", request , String.class);
Assertions.assertEquals(response.getStatusCode(), HttpStatus.CREATED); 
```

## 6。使用选项获得允许的操作

接下来，我们将快速了解如何使用 OPTIONS 请求，并探索使用此类请求在特定 URI 上允许的操作；API 是`optionsForAllow`:

```java
Set<HttpMethod> optionsForAllow = restTemplate.optionsForAllow(fooResourceUrl);
HttpMethod[] supportedMethods
  = {HttpMethod.GET, HttpMethod.POST, HttpMethod.PUT, HttpMethod.DELETE};
Assertions.assertTrue(optionsForAllow.containsAll(Arrays.asList(supportedMethods))); 
```

## 7。使用 PUT 更新资源

接下来，我们将开始查看 PUT，更具体地说是这个操作的`exchange()` API，因为`template.put` API 非常简单。

### 7.1。简单的`PUT`跟跟`**exchange()**`

我们将从一个简单的针对 API 的 PUT 操作开始——请记住，该操作不是将主体返回给客户端:

```java
Foo updatedInstance = new Foo("newName");
updatedInstance.setId(createResponse.getBody().getId());
String resourceUrl = 
  fooResourceUrl + '/' + createResponse.getBody().getId();
HttpEntity<Foo> requestUpdate = new HttpEntity<>(updatedInstance, headers);
template.exchange(resourceUrl, HttpMethod.PUT, requestUpdate, Void.class);
```

### 7.2。放入`exchange()`和请求回调

接下来，我们将使用请求回调来发出 PUT。

让我们确保准备好回调，在回调中我们可以设置我们需要的所有头和请求体:

```java
RequestCallback requestCallback(final Foo updatedInstance) {
    return clientHttpRequest -> {
        ObjectMapper mapper = new ObjectMapper();
        mapper.writeValue(clientHttpRequest.getBody(), updatedInstance);
        clientHttpRequest.getHeaders().add(
          HttpHeaders.CONTENT_TYPE, MediaType.APPLICATION_JSON_VALUE);
        clientHttpRequest.getHeaders().add(
          HttpHeaders.AUTHORIZATION, "Basic " + getBase64EncodedLogPass());
    };
}
```

接下来，我们用 POST 请求创建资源:

```java
ResponseEntity<Foo> response = restTemplate
  .exchange(fooResourceUrl, HttpMethod.POST, request, Foo.class);
Assertions.assertEquals(response.getStatusCode(), HttpStatus.CREATED); 
```

然后我们更新资源:

```java
Foo updatedInstance = new Foo("newName");
updatedInstance.setId(response.getBody().getId());
String resourceUrl =fooResourceUrl + '/' + response.getBody().getId();
restTemplate.execute(
  resourceUrl, 
  HttpMethod.PUT, 
  requestCallback(updatedInstance), 
  clientHttpResponse -> null);
```

## 8。使用删除来删除资源

要删除现有资源，我们将快速使用`delete()` API:

```java
String entityUrl = fooResourceUrl + "/" + existingResource.getId();
restTemplate.delete(entityUrl); 
```

## 9。配置超时

我们可以通过简单地使用`ClientHttpRequestFactory`来配置`RestTemplate`超时:

```java
RestTemplate restTemplate = new RestTemplate(getClientHttpRequestFactory());

private ClientHttpRequestFactory getClientHttpRequestFactory() {
    int timeout = 5000;
    HttpComponentsClientHttpRequestFactory clientHttpRequestFactory
      = new HttpComponentsClientHttpRequestFactory();
    clientHttpRequestFactory.setConnectTimeout(timeout);
    return clientHttpRequestFactory;
}
```

我们可以使用`HttpClient`获得更多配置选项:

```java
private ClientHttpRequestFactory getClientHttpRequestFactory() {
    int timeout = 5000;
    RequestConfig config = RequestConfig.custom()
      .setConnectTimeout(timeout)
      .setConnectionRequestTimeout(timeout)
      .setSocketTimeout(timeout)
      .build();
    CloseableHttpClient client = HttpClientBuilder
      .create()
      .setDefaultRequestConfig(config)
      .build();
    return new HttpComponentsClientHttpRequestFactory(client);
}
```

## 10。结论

在本文中，我们回顾了主要的 HTTP 动词，使用`RestTemplate`来编排使用所有这些动词的请求。

如果您想深入了解如何使用模板进行身份验证，请查看我们关于使用 RestTemplate 进行基本身份验证的文章。

所有这些例子和代码片段的实现都可以在 GitHub 上找到[。](https://web.archive.org/web/20221129014436/https://github.com/eugenp/tutorials/tree/master/spring-web-modules/spring-resttemplate "The code samples for the async http client on github")