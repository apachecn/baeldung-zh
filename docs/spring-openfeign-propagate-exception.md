# 用 OpenFeign 和 Spring 传播异常

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-openfeign-propagate-exception>

## 1.概观

我们预计微服务之间的 HTTP API 调用偶尔会遇到错误。

在 Spring Boot 用[open feign](/web/20220906000116/https://www.baeldung.com/spring-cloud-openfeign)，默认错误处理程序将下游错误，如`Not Found`，作为`Internal Server Error`进行传播。这很少是传达错误的最佳方式 。然而，Spring 和 OpenFeign 都允许我们提供自己的错误处理。

在本文中，我们将了解默认异常传播是如何工作的。我们还将学习如何提供我们自己的错误。

## 2.默认异常传播策略

使用注释和配置属性，Feign client 使微服务之间的交互变得简单明了且高度可配置。然而，API 调用可能会由于任何随机的技术原因、错误的用户请求或编码错误而失败。

幸运的是， **佯而春有明智的默认实现** 进行错误处理。

### 2.1.Feign 中的默认异常传播

佯用 `ErrorDecoder` 。 `Default ` 类为其错误处理。这样，每当 Feign 接收到任何非 2xx 的状态代码时，它就把它传递给 `ErrorDecoder's decode ` 方法。如果 HTTP 响应有 `a Retry-After` 标头，则 `decode ` 方法返回一个 [`RetryableException`](/web/20220906000116/https://www.baeldung.com/feign-retry#:~:text=Feign%20provides%20a%20sensible%20default,retry%20up%20to%20provided%20maximum.) ，否则返回一个`FeignException`。重试时，如果请求在默认的重试次数后失败，那么将返回`FeignException`。

**`decode` 方法将 HTTP 方法密钥和响应**存储在`FeignException`中。

### 2.2.Spring Rest 控制器中的默认异常传播

每当`RestController`接收到任何未处理的异常`,` 时，它向客户端返回 500 内部服务器错误响应。

此外，Spring 提供了一个结构良好的错误响应，包括时间戳、HTTP 状态代码、错误和路径等细节:

```java
{
    "timestamp": "2022-07-08T08:07:51.120+00:00",
    "status": 500,
    "error": "Internal Server Error",
    "path": "/myapp1/product/Test123"
}
```

让我们用一个例子来深入探讨一下这个问题。

## 3.示例应用程序

假设我们需要构建一个简单的微服务，从另一个外部服务返回产品信息。

首先，让我们用几个属性对`Product`类建模:

```java
public class Product {
    private String id;
    private String productName;
    private double price;
}
```

然后，让我们用`Get` `Product`端点实现`ProductController` :

```java
@RestController("product_controller")
@RequestMapping(value ="myapp1")
public class ProductController {

    private ProductClient productClient;

    @Autowired
    public ProductController(ProductClient productClient) {
        this.productClient = productClient;
    }

    @GetMapping("/product/{id}")
    public Product getProduct(@PathVariable String id) {
        return productClient.getProduct(id);
    }
}
```

接下来，我们来看看如何将假扮的 [`Logger`](/web/20220906000116/https://www.baeldung.com/java-feign-logging) 注册为`Bean` :

```java
public class FeignConfig {

    @Bean
    Logger.Level feignLoggerLevel() {
        return Logger.Level.FULL;
    }
}
```

最后，让我们实现`ProductClient` 来与外部 API 接口:

```java
@FeignClient(name = "product-client", url="http://localhost:8081/product/", configuration = FeignConfig.class)
public interface ProductClient {
    @RequestMapping(value = "{id}", method = RequestMethod.GET")
    Product getProduct(@PathVariable(value = "id") String id);
}
```

现在让我们使用上面的例子来研究默认的错误传播。

## 4.默认异常传播

### 4.1.使用 WireMock 服务器

为了进行实验，我们需要使用一个模拟框架来模拟我们调用的服务。

首先，让我们把 [`WireMockServer`](/web/20220906000116/https://www.baeldung.com/introduction-to-wiremock) 包括在内，美文依赖:

```java
<dependency>
    <groupId>com.github.tomakehurst</groupId>
    <artifactId>wiremock-jre8</artifactId>
    <version>2.33.2</version>
    <scope>test</scope>
</dependency>
```

然后，让我们配置并启动`WireMockServer`:

```java
WireMockServer wireMockServer = new WireMockServer(8081);
configureFor("localhost", 8081);
wireMockServer.start();
```

`WireMockServer`与假扮客户端配置使用的 `host` 和 `port` 同时启动。

### 4.2.假装客户端中的默认异常传播

Feign 的默认错误处理程序`ErrorDecoder.Default`，总是抛出一个`FeignException`。

让我们用`WireMock.stubFor`模拟`getProduct` 方法，让它看起来不可用:

```java
String productId = "test";
stubFor(get(urlEqualTo("/product/" + productId))
  .willReturn(aResponse()
  .withStatus(HttpStatus.SERVICE_UNAVAILABLE.value())));

assertThrows(FeignException.class, () -> productClient.getProduct(productId));
```

在上面的测试用例中，`ProductClient`在遇到来自下游服务的 503 错误时抛出`FeignException`。

接下来，让我们尝试相同的实验，但使用 404 Not Found 响应:

```java
String productId = "test";
stubFor(get(urlEqualTo("/product/" + productId))
  .willReturn(aResponse()
  .withStatus(HttpStatus.NOT_FOUND.value())));

assertThrows(FeignException.class, () -> productClient.getProduct(productId));
```

同样，我们得到了一个通用的`FeignException`。在这种情况下，也许用户请求了一些错误的东西，我们的 Spring 应用程序需要知道这是一个错误的用户请求，以便它能够以不同的方式处理。

我们应该注意到，`FeignException`确实有一个包含 HTTP 状态代码的`status `属性，但是一个`try` / `catch`策略基于它们的类型而不是它们的属性来路由异常。

### 4.3.Spring Rest 控制器中的默认异常传播

现在让我们看看 `FeignException` 如何传播回请求者。

当 `ProductController ` 从 `ProductClient`获得 `FeignException ` 时，将其传递给框架提供的默认错误处理实现。

让我们断言产品服务何时不可用:

```java
String productId = "test";
stubFor(WireMock.get(urlEqualTo("/product/" + productId))
  .willReturn(aResponse()
  .withStatus(HttpStatus.SERVICE_UNAVAILABLE.value())));

mockMvc.perform(get("/myapp1/product/" + productId))
  .andExpect(status().is(HttpStatus.INTERNAL_SERVER_ERROR.value()));
```

在这里，我们可以看到我们得到了弹簧`INTERNAL_SERVER_ERROR`。这种默认行为并不总是最好的，因为不同的服务错误可能需要不同的结果。

## 5.用`ErrorDecoder`在 Feign 中传播自定义异常

我们不应该总是返回默认的 `FeignException` ，而是应该根据 HTTP 状态码返回一些特定于应用程序的异常。

让我们在自定义的`ErrorDecoder`实现中覆盖`decode`方法:

```java
public class CustomErrorDecoder implements ErrorDecoder {

    @Override
    public Exception decode(String methodKey, Response response) {
        switch (response.status()){
            case 400:
                return new BadRequestException();
            case 404:
                return new ProductNotFoundException("Product not found");
            case 503:
                return new ProductServiceNotAvailableException("Product Api is unavailable");
            default:
                return new Exception("Exception while getting product details");
        }
    }
} 
```

在我们的自定义`decode`方法中，我们返回了不同的异常和一些特定于应用程序的异常，为实际问题提供了更多的上下文。我们还可以在特定于应用程序的异常消息中包含更多细节。

我们要注意的是， **t** **he `decode`方法返回的是 `FeignException` 而不是抛出的** 。

现在，让我们在 `FeignConfig as a ` 弹簧 `Bean` : 中配置 `CustomErrorDecoder`

```java
@Bean
public ErrorDecoder errorDecoder() {
   return new CustomErrorDecoder();
}
```

或者，`CustomErrorDecoder` 可以直接在`ProductClient`中配置:

```java
@FeignClient(name = "product-client-2", url = "http://localhost:8081/product/", 
   configuration = { FeignConfig.class, CustomErrorDecoder.class })
```

然后，让我们检查`CustomErrorDecoder` 是否返回 `ProductServiceNotAvailableException`:

```java
String productId = "test";
stubFor(get(urlEqualTo("/product/" + productId))
  .willReturn(aResponse()
  .withStatus(HttpStatus.SERVICE_UNAVAILABLE.value())));

assertThrows(ProductServiceNotAvailableException.class, 
  () -> productClient.getProduct(productId));
```

再次，让我们写一个测试用例来断言 `ProductNotFoundException` 当产品不存在的时候:

```java
String productId = "test";
stubFor(get(urlEqualTo("/product/" + productId))
  .willReturn(aResponse()
  .withStatus(HttpStatus.NOT_FOUND.value())));

assertThrows(ProductNotFoundException.class, 
  () -> productClient.getProduct(productId));
```

虽然我们现在提供了来自 Feign client 的各种异常，但是当 Spring 捕获所有异常时，它仍然会产生一个通用的内部服务器错误。因为这不是我们想要的，所以让我们看看如何改进。

## 6.在 Spring **Rest 控制器**中传播自定义异常

正如我们已经看到的，默认的 Spring Boot 错误处理程序提供了一个通用的错误响应。API 消费者可能需要相关错误响应的详细信息。理想情况下，错误响应应该能够解释问题并有助于调试。

我们可以用很多方式覆盖`RestController`中的默认异常处理程序。

我们将通过 [`RestControllerAdvice`](/web/20220906000116/https://www.baeldung.com/exception-handling-for-rest-with-spring) 注释来研究一种处理错误的方法。

### 6.1.使用`@RestControllerAdvice`

@ `RestControllerAdvice`注释允许我们**将多个异常合并到一个单一的全局错误处理组件中。**

让我们想象一个场景，其中 `ProductController ` 需要基于下游错误返回不同的自定义错误响应。

首先，让我们创建`ErrorResponse`类来定制错误响应:

```java
public class ErrorResponse {

    @JsonFormat(shape = JsonFormat.Shape.STRING, pattern = "dd-MM-yyyy hh:mm:ss")
    private Date timestamp;

    @JsonProperty(value = "code")
    private int code;

    @JsonProperty(value = "status")
    private String status;

    @JsonProperty(value = "message")
    private String message;

    @JsonProperty(value = "details")
    private String details;
}
```

现在，让我们子类化*ResponseEntityExceptionHandler*，并在错误处理程序中包含`@ExceptionHandler`注释:

```java
@RestControllerAdvice
public class ProductExceptionHandler extends ResponseEntityExceptionHandler {

    @ExceptionHandler({ProductServiceNotAvailableException.class})
    public ResponseEntity<ErrorResponse> handleProductServiceNotAvailableException(ProductServiceNotAvailableException exception, WebRequest request) {
        return new ResponseEntity<>(new ErrorResponse(
          HttpStatus.INTERNAL_SERVER_ERROR,
          exception.getMessage(),
          request.getDescription(false)),
          HttpStatus.INTERNAL_SERVER_ERROR);
    }

    @ExceptionHandler({ProductNotFoundException.class})
    public ResponseEntity<ErrorResponse> handleProductNotFoundException(ProductNotFoundException exception, WebRequest request) {
        return new ResponseEntity<>(new ErrorResponse(
          HttpStatus.NOT_FOUND,
          exception.getMessage(),
          request.getDescription(false)),
          HttpStatus.NOT_FOUND);
    }
} 
```

在上面的代码中， `ProductServiceNotAvailableException` 作为一个`INTERNAL_SERVER_ERROR` 响应返回给客户端。相反，像 `ProductNotFoundException` 这样的特定于用户的错误会得到不同的处理，并作为`NOT_FOUND`响应返回。

### 6.2.测试弹簧支架控制器

当产品服务不可用时，让我们测试一下`ProductController`:

```java
String productId = "test";
stubFor(WireMock.get(urlEqualTo("/product/" + productId))
  .willReturn(aResponse()
  .withStatus(HttpStatus.SERVICE_UNAVAILABLE.value())));

MvcResult result = mockMvc.perform(get("/myapp2/product/" + productId))
  .andExpect(status().isInternalServerError()).andReturn();

ErrorResponse errorResponse = objectMapper.readValue(result.getResponse().getContentAsString(), ErrorResponse.class);
assertEquals(500, errorResponse.getCode());
assertEquals("Product Api is unavailable", errorResponse.getMessage()); 
```

同样，让我们测试相同的`ProductController`，但出现产品未找到错误:

```java
String productId = "test";
stubFor(WireMock.get(urlEqualTo("/product/" + productId))
  .willReturn(aResponse()
  .withStatus(HttpStatus.NOT_FOUND.value())));

MvcResult result = mockMvc.perform(get("/myapp2/product/" + productId))
  .andExpect(status().isNotFound()).andReturn();

ErrorResponse errorResponse = objectMapper.readValue(result.getResponse().getContentAsString(), ErrorResponse.class);
assertEquals(404, errorResponse.getCode());
assertEquals("Product not found", errorResponse.getMessage()); 
```

以上测试显示了 `ProductController` 如何根据下游错误返回不同的错误响应。

如果我们没有实现我们的 `CustomErrorDecoder` ，那么 **`RestControllerAdvice`被要求处理默认的`FeignException`** **作为回退**以具有通用错误响应。

## 7.结论

在本文中，我们已经探索了默认错误处理是如何在 Feign and Spring 中实现的。

此外，我们已经看到了如何用`CustomErrorDecoder `在 Feign client 中定制这个，用`RestControllerAdvice`在 Rest 控制器中定制。

和往常一样，所有这些代码示例都可以在 GitHub 上找到[。](https://web.archive.org/web/20220906000116/https://github.com/eugenp/tutorials/tree/master/spring-cloud-modules/spring-cloud-openfeign)