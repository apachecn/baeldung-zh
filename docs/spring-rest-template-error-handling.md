# Spring RestTemplate 错误处理

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-rest-template-error-handling>

## 1.概观

在这个简短的教程中，我们将讨论如何在一个`RestTemplate`实例中实现和注入`ResponseErrorHandler`接口，以优雅地处理远程 API 返回的 HTTP 错误。

## 2.默认错误处理

默认情况下，`RestTemplate`将在 HTTP 错误的情况下抛出这些异常之一:

1.  `HttpClientErrorException`–对于 HTTP 状态 4xx
2.  `HttpServerErrorException –` 在 HTTP 状态 5xx 的情况下
3.  `UnknownHttpStatusCodeException –` 在未知 HTTP 状态的情况下

所有这些异常都是`RestClientResponseException`的扩展。

**显然，添加自定义错误处理的最简单策略是将调用包装在`try/catch`块中。**然后我们可以按照我们认为合适的方式处理被捕获的异常。

然而，**随着远程 API 或调用数量的增加，这个简单的策略不能很好地扩展**。如果我们能够为所有的远程调用实现一个可重用的错误处理程序，那么效率会更高。

## 3.实现一个`ResponseErrorHandler`

实现`ResponseErrorHandler` 的类将从响应中读取 HTTP 状态，并且:

1.  抛出对我们的应用程序有意义的异常
2.  只需忽略 HTTP 状态，让响应流不间断地继续

我们需要将`ResponseErrorHandler`实现注入到`RestTemplate`实例中。

因此，我们可以使用`RestTemplateBuilder`来构建模板，并替换响应流中的`DefaultResponseErrorHandler`。

所以让我们先实现我们的`RestTemplateResponseErrorHandler:`

```java
@Component
public class RestTemplateResponseErrorHandler 
  implements ResponseErrorHandler {

    @Override
    public boolean hasError(ClientHttpResponse httpResponse) 
      throws IOException {

        return (
          httpResponse.getStatusCode().series() == CLIENT_ERROR 
          || httpResponse.getStatusCode().series() == SERVER_ERROR);
    }

    @Override
    public void handleError(ClientHttpResponse httpResponse) 
      throws IOException {

        if (httpResponse.getStatusCode()
          .series() == HttpStatus.Series.SERVER_ERROR) {
            // handle SERVER_ERROR
        } else if (httpResponse.getStatusCode()
          .series() == HttpStatus.Series.CLIENT_ERROR) {
            // handle CLIENT_ERROR
            if (httpResponse.getStatusCode() == HttpStatus.NOT_FOUND) {
                throw new NotFoundException();
            }
        }
    }
}
```

然后我们可以构建`RestTemplate`实例，使用`RestTemplateBuilder` 来引入我们的 `RestTemplateResponseErrorHandler`

```java
@Service
public class BarConsumerService {

    private RestTemplate restTemplate;

    @Autowired
    public BarConsumerService(RestTemplateBuilder restTemplateBuilder) {
        RestTemplate restTemplate = restTemplateBuilder
          .errorHandler(new RestTemplateResponseErrorHandler())
          .build();
    }

    public Bar fetchBarById(String barId) {
        return restTemplate.getForObject("/bars/4242", Bar.class);
    }

}
```

## 4.测试我们的实现

最后，我们将通过模拟服务器并返回一个`NOT_FOUND`状态来测试这个处理程序:

```java
@ExtendWith(SpringExtension.class)
@ContextConfiguration(classes = { NotFoundException.class, Bar.class })
@RestClientTest
public class RestTemplateResponseErrorHandlerIntegrationTest {

    @Autowired 
    private MockRestServiceServer server;

    @Autowired 
    private RestTemplateBuilder builder;

    @Test
    public void  givenRemoteApiCall_when404Error_thenThrowNotFound() {
        Assertions.assertNotNull(this.builder);
        Assertions.assertNotNull(this.server);

        RestTemplate restTemplate = this.builder
          .errorHandler(new RestTemplateResponseErrorHandler())
          .build();

        this.server
          .expect(ExpectedCount.once(), requestTo("/bars/4242"))
          .andExpect(method(HttpMethod.GET))
          .andRespond(withStatus(HttpStatus.NOT_FOUND));

        Assertions.assertThrows(NotFoundException.class, () -> {
            Bar response = restTemplate.getForObject("/bars/4242", Bar.class);
        });
    }
}
```

## 5.结论

在本文中，我们提出了一个解决方案来实现和测试一个定制的错误处理程序，用于将 HTTP 错误转换成有意义的异常。

和往常一样，本文中的代码可以从 Github 上的[处获得。](https://web.archive.org/web/20220707143824/https://github.com/eugenp/tutorials/tree/master/spring-web-modules/spring-resttemplate)