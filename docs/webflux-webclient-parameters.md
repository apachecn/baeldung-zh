# 带参数的 Spring WebClient 请求

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/webflux-webclient-parameters>

## 1.概观

许多框架和项目正在引入**反应式编程和异步请求处理**。因此， [Spring 5](https://web.archive.org/web/20221205111957/https://baeldung.com/spring-5) 引入了一个反应式`[WebClient](https://web.archive.org/web/20221205111957/https://baeldung.com/spring-5-webclient)`实现作为 [WebFlux](https://web.archive.org/web/20221205111957/https://baeldung.com/spring-webflux) 框架的一部分。

在本教程中，我们将学习如何用`WebClient` 来**被动地消费 REST API 端点。**

## 2.REST API 端点

首先，让我们用下面的 GET 端点定义一个示例 [REST API](/web/20221205111957/https://www.baeldung.com/rest-api-spring-guide) 和**:**

*   `/products`–获取所有产品
*   `/products/{id}`–通过 ID 获取产品
*   `/products/{id}/attributes/{attributeId}`–通过 id 获取产品属性
*   `/products/?name={name}&deliveryDate;={deliveryDate}&color;={color}`–寻找产品
*   `/products/?tag[]={tag1}&tag;[]={tag2}`–通过标签获取产品
*   `/products/?category={category1}&category;={category2}`–按类别获取产品

这里我们定义了几个不同的 URIs。一会儿，我们将弄清楚如何用`WebClient`构建和发送每种类型的 URI。

请注意，通过标签和类别获取产品的 URIs 包含数组作为查询参数；然而，语法有所不同，因为在 URIs 中没有严格定义数组应该如何表示。这主要取决于服务器端的实现。因此，我们将涵盖这两种情况。

## 3.`WebClient` 设置

首先，我们需要创建一个`WebClient`的实例。对于本文，我们将使用一个[模拟对象](/web/20221205111957/https://www.baeldung.com/mockito-series)来验证是否请求了一个有效的 URI。

让我们定义客户机和相关的模拟对象:

```java
exchangeFunction = mock(ExchangeFunction.class);
ClientResponse mockResponse = mock(ClientResponse.class);
when(mockResponse.bodyToMono(String.class))
  .thenReturn(Mono.just("test"));

when(exchangeFunction.exchange(argumentCaptor.capture()))
  .thenReturn(Mono.just(mockResponse));

webClient = WebClient
  .builder()
  .baseUrl("https://example.com/api")
  .exchangeFunction(exchangeFunction)
  .build();
```

我们还将传递一个基本 URL，它将被添加到客户端发出的所有请求的前面。

最后，为了验证特定的 URI 已经被传递给底层的`ExchangeFunction`实例，我们将使用下面的帮助器方法:

```java
private void verifyCalledUrl(String relativeUrl) {
    ClientRequest request = argumentCaptor.getValue();
    assertEquals(String.format("%s%s", BASE_URL, relativeUrl), request.url().toString());

    verify(this.exchangeFunction).exchange(request);
    verifyNoMoreInteractions(this.exchangeFunction);
}
```

`WebClientBuilder`类的`uri()`方法提供了`UriBuilder`实例作为参数。通常，我们以下列方式进行 API 调用:

```java
webClient.get()
  .uri(uriBuilder -> uriBuilder
    //... building a URI
    .build())
  .retrieve()
  .bodyToMono(String.class)
  .block();
```

我们将在本指南中广泛使用`UriBuilder`来构建 URIs。值得注意的是，我们可以使用其他方法构建一个 URI，然后将生成的 URI 作为字符串传递。

## 4.URI 路径分量

**路径组件由一系列由斜杠(/ )** 分隔的路径段组成。首先，我们从一个简单的例子开始，其中 URI 没有任何可变段，`/products`:

```java
webClient.get()
  .uri("/products")
  .retrieve()
  .bodyToMono(String.class)
  .block();

verifyCalledUrl("/products");
```

对于这种情况，我们可以只传递一个`String`作为参数。

接下来，我们将采用`/products/{id}`端点并构建相应的 URI:

```java
webClient.get()
  .uri(uriBuilder - > uriBuilder
    .path("/products/{id}")
    .build(2))
  .retrieve()
  .bodyToMono(String.class)
  .block();

verifyCalledUrl("/products/2"); 
```

从上面的代码中，我们可以看到实际的段值被传递给了`build()`方法。

以类似的方式，我们可以为`/products/{id}/attributes/{attributeId}`端点创建一个具有多个路径段的 URI:

```java
webClient.get()
  .uri(uriBuilder - > uriBuilder
    .path("/products/{id}/attributes/{attributeId}")
    .build(2, 13))
  .retrieve()
  .bodyToMono(String.class)
  .block();

verifyCalledUrl("/products/2/attributes/13");
```

URI 可以根据需要拥有任意多的路径段，但最终的 URI 长度不得超过限制。最后，我们需要记住保持传递给`build()`方法的实际段值的正确顺序。

## 5.URI 查询参数

通常，查询参数是一个简单的键值对，如`title=Baeldung`。让我们看看如何建立这样的 URIs。

### 5.1.单值参数

我们将从单值参数开始，取`/products/?name={name}&deliveryDate;={deliveryDate}&color;={color}`端点。为了设置查询参数，我们将调用`UriBuilder`接口的`queryParam()`方法:

```java
webClient.get()
  .uri(uriBuilder - > uriBuilder
    .path("/products/")
    .queryParam("name", "AndroidPhone")
    .queryParam("color", "black")
    .queryParam("deliveryDate", "13/04/2019")
    .build())
  .retrieve()
  .bodyToMono(String.class)
  .block();

verifyCalledUrl("/products/?name=AndroidPhone&color;=black&deliveryDate;=13/04/2019"); 
```

这里我们添加了三个查询参数，并立即分配了实际值。相反，也可以留下占位符来代替精确值:

```java
webClient.get()
  .uri(uriBuilder - > uriBuilder
    .path("/products/")
    .queryParam("name", "{title}")
    .queryParam("color", "{authorId}")
    .queryParam("deliveryDate", "{date}")
    .build("AndroidPhone", "black", "13/04/2019"))
  .retrieve()
  .bodyToMono(String.class)
  .block();

verifyCalledUrl("/products/?name=AndroidPhone&color;=black&deliveryDate;=13%2F04%2F2019"); 
```

当在链中进一步传递构建器对象时，这可能特别有用。

注意，上面的两个代码片段有一个重要的区别。注意预期的 URIs，我们可以看到它们的**编码不同**。特别是，斜杠字符 **( / )** 在最后一个例子中被转义。

一般来说， [RFC3986](https://web.archive.org/web/20221205111957/https://www.ietf.org/rfc/rfc3986.txt) 不要求查询中斜线的编码；但是，一些服务器端应用程序可能需要这种转换。因此，我们将在本指南的后面看到如何改变这种行为。

### 5.2.数组参数

我们可能需要传递一组值，在查询字符串中传递数组没有严格的规则。因此，**查询字符串中的数组表示因项目而异，通常取决于底层框架**。我们将在本文中讨论最广泛使用的格式。

让我们从`/products/?tag[]={tag1}&tag;[]={tag2}`端点开始:

```java
webClient.get()
  .uri(uriBuilder - > uriBuilder
    .path("/products/")
    .queryParam("tag[]", "Snapdragon", "NFC")
    .build())
  .retrieve()
  .bodyToMono(String.class)
  .block();

verifyCalledUrl("/products/?tag%5B%5D=Snapdragon&tag;%5B%5D=NFC");
```

正如我们所看到的，最终的 URI 包含多个标记参数，后跟编码的方括号。`queryParam()`方法接受变量参数作为值，所以不需要多次调用该方法。

或者，我们可以**省略方括号，只传递多个具有相同键**，但不同值`/products/?category={category1}&category;={category2}`的查询参数:

```java
webClient.get()
  .uri(uriBuilder - > uriBuilder
    .path("/products/")
    .queryParam("category", "Phones", "Tablets")
    .build())
  .retrieve()
  .bodyToMono(String.class)
  .block();

verifyCalledUrl("/products/?category=Phones&category;=Tablets");
```

最后，还有一种更广泛使用的编码数组的方法，那就是传递逗号分隔的值。让我们把前面的例子转换成逗号分隔的值:

```java
webClient.get()
  .uri(uriBuilder - > uriBuilder
    .path("/products/")
    .queryParam("category", String.join(",", "Phones", "Tablets"))
    .build())
  .retrieve()
  .bodyToMono(String.class)
  .block();

verifyCalledUrl("/products/?category=Phones,Tablets");
```

我们只是使用`String`类的`join()`方法来创建一个逗号分隔的字符串。我们也可以使用应用程序期望的任何其他分隔符。

## 6.编码模式

还记得我们之前提到的 URL 编码吗？

如果默认行为不符合我们的要求，我们可以改变它。我们需要在构建一个`WebClient`实例的同时提供一个`UriBuilderFactory`实现。在这种情况下，我们将使用`DefaultUriBuilderFactory`类。为了设置编码，我们将调用`setEncodingMode()`方法。以下模式可用:

*   **TEMPLATE_AND_VALUES** :预编码 URI 模板，扩展时严格编码 URI 变量
*   **VALUES_ONLY** :不对 URI 模板进行编码，将 URI 变量展开到模板后严格编码
*   **URI _ 组件**:扩展 URI 变量后编码 URI 组件值
*   **无**:不应用编码

默认值为**模板 _ 和 _ 值**。让我们将模式设置为**URI _ 组件**:

```java
DefaultUriBuilderFactory factory = new DefaultUriBuilderFactory(BASE_URL);
factory.setEncodingMode(DefaultUriBuilderFactory.EncodingMode.URI_COMPONENT);
webClient = WebClient
  .builder()
  .uriBuilderFactory(factory)
  .baseUrl(BASE_URL)
  .exchangeFunction(exchangeFunction)
  .build();
```

因此，下面的断言将会成功:

```java
webClient.get()
  .uri(uriBuilder - > uriBuilder
    .path("/products/")
    .queryParam("name", "AndroidPhone")
    .queryParam("color", "black")
    .queryParam("deliveryDate", "13/04/2019")
    .build())
  .retrieve()
  .bodyToMono(String.class)
  .block();

verifyCalledUrl("/products/?name=AndroidPhone&color;=black&deliveryDate;=13/04/2019");
```

当然，我们可以提供一个完全定制的`UriBuilderFactory`实现来手动处理 URI 创建。

## 7.结论

在本文中，我们学习了如何使用`WebClient`和`DefaultUriBuilder.`构建不同类型的 URIs

一路上，我们讨论了各种类型和格式的查询参数。最后，我们通过更改 URL 构建器的默认编码模式来结束本文。

和往常一样，文章中的所有代码片段都可以在 GitHub 库上找到[。](https://web.archive.org/web/20221205111957/https://github.com/eugenp/tutorials/tree/master/spring-reactive-modules/spring-reactive)