# 如何使用 Spring RestTemplate 压缩请求

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-resttemplate-compressing-requests>

## 1。简介

在这个简短的教程中，我们将看看如何发送包含压缩数据的 HTTP 请求。

此外，我们将了解如何配置 Spring web 应用程序，以便它处理压缩请求。

## 2。发送压缩请求

首先，让我们创建一个压缩字节数组的方法。这将很快派上用场:

```java
public static byte[] compress(byte[] body) throws IOException {
    ByteArrayOutputStream baos = new ByteArrayOutputStream();
    try (GZIPOutputStream gzipOutputStream = new GZIPOutputStream(baos)) {
        gzipOutputStream.write(body);
    }
    return baos.toByteArray();
}
```

接下来，我们需要实现一个`ClientHttpRequestInterceptor`来修改请求。注意，我们将发送适当的 HTTP 压缩头，并调用我们的主体压缩方法:

```java
public ClientHttpResponse intercept(HttpRequest req, byte[] body, ClientHttpRequestExecution exec)
  throws IOException {
    HttpHeaders httpHeaders = req.getHeaders();
    httpHeaders.add(HttpHeaders.CONTENT_ENCODING, "gzip");
    httpHeaders.add(HttpHeaders.ACCEPT_ENCODING, "gzip");
    return exec.execute(req, compress(body));
} 
```

我们的拦截器接收出站请求体，并使用 GZIP 格式对其进行压缩。在这个例子中，我们使用 Java 的标准`GZIPOutputStream`来为我们完成这项工作。

此外，我们必须为数据编码添加适当的头。这让目的地端点知道它正在处理 GZIP 压缩数据。

最后，我们将拦截器添加到我们的`RestTemplate` 定义中:

```java
@Bean
public RestTemplate getRestTemplate() {
    RestTemplate restTemplate = new RestTemplate();
    restTemplate.getInterceptors().add(new CompressingClientHttpRequestInterceptor());
    return restTemplate;
} 
```

## 3。处理压缩请求

默认情况下，大多数 web 服务器不理解包含压缩数据的请求。Spring web 应用程序也不例外。因此，我们需要配置它们来处理这样的请求。

目前，只有 Jetty 和 Undertow web 服务器处理带有 GZIP 格式数据的请求体。请参阅我们关于 [Spring Boot 应用程序配置](/web/20220630140112/https://www.baeldung.com/spring-boot-application-configuration)的文章，以设置 Jetty 或 Undertow web 服务器。

### 3.1.Jetty Web 服务器

在这个例子中，我们通过添加一个 Jetty `GzipHandler`来定制一个 Jetty web 服务器。这个 Jetty 处理程序是为压缩响应和解压缩请求而构建的。

然而，将它添加到 Jetty web 服务器是不够的。我们需要将`inflateBufferSize`设置为大于零的值来启用解压缩:

```java
@Bean
public JettyServletWebServerFactory jettyServletWebServerFactory() {
    JettyServletWebServerFactory factory = new JettyServletWebServerFactory();
    factory.addServerCustomizers(server -> {
        GzipHandler gzipHandler = new GzipHandler();
        gzipHandler.setInflateBufferSize(1);
        gzipHandler.setHandler(server.getHandler());

        HandlerCollection handlerCollection = new HandlerCollection(gzipHandler);
        server.setHandler(handlerCollection);
    });
    return factory;
}
```

### 3.2.水下网络服务器

同样，我们可以定制一个 Undertow web 服务器来为我们自动解压请求。在这种情况下，我们需要添加一个自定义的`RequestEncodingHandler`。

我们配置编码处理程序来处理来自请求的 GZIP 源数据:

```java
@Bean
public UndertowServletWebServerFactory undertowServletWebServerFactory() {
    UndertowServletWebServerFactory factory = new UndertowServletWebServerFactory();
    factory.addDeploymentInfoCustomizers((deploymentInfo) -> {
        deploymentInfo.addInitialHandlerChainWrapper(handler -> new RequestEncodingHandler(handler)
          .addEncoding("gzip", GzipStreamSourceConduit.WRAPPER));
    });
    return factory;
}
```

## 4。结论

这就是我们要让压缩请求工作所需要做的一切！

在本教程中，我们介绍了如何为压缩请求内容的`RestTemplate`创建一个拦截器。此外，我们还研究了如何在 Spring web 应用程序中自动解压缩这些请求。

值得注意的是，我们应该**只将压缩内容发送给能够处理这种请求的 web 服务器**。

GitHub 上的[是 Jetty web 服务器的一个完整的工作示例。](https://web.archive.org/web/20220630140112/https://github.com/eugenp/tutorials/tree/master/spring-web-modules/spring-resttemplate-2)