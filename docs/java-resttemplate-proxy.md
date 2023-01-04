# 使用 RestTemplate 的代理

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-resttemplate-proxy>

## 1.概观

在这个简短的教程中，我们将看看如何使用 [`RestTemplate`](/web/20220707143820/https://www.baeldung.com/rest-template) 向[代理](/web/20220707143820/https://www.baeldung.com/java-connect-via-proxy-server)发送请求。

## 2.属国

首先，`RestTemplateCustomizer`使用`HttpClient`类连接到代理。

要使用该类，我们需要将 [Apache 的`httpcore`依赖项](https://web.archive.org/web/20220707143820/https://search.maven.org/classic/#search%7Cga%7C1%7Cg%3A%22org.apache.httpcomponents%22%20AND%20a%3A%22httpcore%22)添加到我们的 Maven `pom.xml` 文件中:

```java
<dependency>
    <groupId>org.apache.httpcomponents</groupId>
    <artifactId>httpcore</artifactId>
    <version>4.4.13</version>
</dependency>
```

或者到我们的 Gradle `build.gradle`文件:

```java
compile 'org.apache.httpcomponents:httpcore:4.4.13'
```

## 3.使用`SimpleClientHttpRequestFactory`

使用`RestTemplate`向代理发送请求非常简单。我们需要做的就是在构建`RestTemplate` 对象之前**从`SimpleClientHttpRequestFactory`调用 [`setProxy(java.net.Proxy)`](https://web.archive.org/web/20220707143820/https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/http/client/SimpleClientHttpRequestFactory.html#setProxy-java.net.Proxy-) 。**

首先，我们从配置`SimpleClientHttpRequestFactory`开始:

```java
Proxy proxy = new Proxy(Type.HTTP, new InetSocketAddress(PROXY_SERVER_HOST, PROXY_SERVER_PORT));
SimpleClientHttpRequestFactory requestFactory = new SimpleClientHttpRequestFactory();
requestFactory.setProxy(proxy);
```

然后，我们继续将请求工厂实例传递给`RestTemplate` 构造函数:

```java
RestTemplate restTemplate = new RestTemplate(requestFactory);
```

最后，一旦我们构建了`RestTemplate`，我们就可以用它来发出代理请求:

```java
ResponseEntity<String> responseEntity = restTemplate.getForEntity("http://httpbin.org/get", String.class);

assertThat(responseEntity.getStatusCode(), is(equalTo(HttpStatus.OK)));
```

## 4.使用`RestTemplateCustomizer`

另一种方法是用一个 **`RestTemplateCustomizer`和`RestTemplateBuilder`来构建一个定制的`RestTemplate`** 。

让我们开始定义一个`ProxyCustomizer`:

```java
class ProxyCustomizer implements RestTemplateCustomizer {

    @Override
    public void customize(RestTemplate restTemplate) {
        HttpHost proxy = new HttpHost(PROXY_SERVER_HOST, PROXY_SERVER_PORT);
        HttpClient httpClient = HttpClientBuilder.create()
            .setRoutePlanner(new DefaultProxyRoutePlanner(proxy) {
                @Override
                public HttpHost determineProxy(HttpHost target, HttpRequest request, HttpContext context) throws HttpException {
                    return super.determineProxy(target, request, context);
                }
            })
            .build();
        restTemplate.setRequestFactory(new HttpComponentsClientHttpRequestFactory(httpClient));
    }
}
```

之后，我们构建我们定制的 `RestTemplate`:

```java
RestTemplate restTemplate = new RestTemplateBuilder(new ProxyCustomizer()).build();
```

最后，我们使用`RestTemplate`发出首先通过代理的请求:

```java
ResponseEntity<String> responseEntity = restTemplate.getForEntity("http://httpbin.org/get", String.class);
assertThat(responseEntity.getStatusCode(), is(equalTo(HttpStatus.OK)));
```

## 5.结论

在这个简短的教程中，我们探索了使用`RestTemplate`向代理发送请求的两种不同方式。

首先，我们学习了如何通过使用`SimpleClientHttpRequestFactory.` 构建的`RestTemplate`发送请求，然后我们学习了如何使用`RestTemplateCustomizer`做同样的事情，这是根据[文档](https://web.archive.org/web/20220707143820/https://docs.spring.io/spring-boot/docs/current/reference/html/howto.html#howto-http-clients-proxy-configuration)推荐的方法。

与往常一样，代码示例可以在 GitHub 上的[处获得。](https://web.archive.org/web/20220707143820/https://github.com/eugenp/tutorials/tree/master/spring-web-modules/spring-resttemplate-2)