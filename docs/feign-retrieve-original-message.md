# 从 Feign ErrorDecoder 检索原始消息

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/feign-retrieve-original-message>

## 1。概述

RESTful 服务失败的原因有很多。在本教程中，我们将看看如果集成的 REST 服务抛出错误，如何从 [Feign](/web/20220617075719/https://www.baeldung.com/spring-cloud-openfeign) 客户端检索原始消息。

## 2。假装客户端

Feign 是一个可插入的声明式 web 服务客户端，它使得编写 web 服务客户端更加容易。此外，为了假装注释，它还支持 [JAX-RS](/web/20220617075719/https://www.baeldung.com/jax-rs-spec-and-implementations) ，它支持**编码器和解码器，以提供更多的定制**。

## 3。从`ErrorDecoder` 检索消息

当错误发生时，**Feign 客户端隐藏原始消息，为了检索它，我们需要编写一个自定义的** `**[ErrorDecoder](https://web.archive.org/web/20220617075719/https://appdoc.app/artifact/com.netflix.feign/feign-core/8.11.0/feign/codec/ErrorDecoder.html)**` 。如果没有这样的定制，我们会得到下面的错误:

```java
feign.FeignException$NotFound: [404] during [POST] to [http://localhost:8080/upload-error-1] [UploadClient#fileUploadError(MultipartFile)]: [{"timestamp":"2022-02-18T13:25:22.083+00:00","status":404,"error":"Not Found","path":"/upload-error-1"}]
	at feign.FeignException.clientErrorStatus(FeignException.java:219) ~[feign-core-11.7.jar:na]
	at feign.FeignException.errorStatus(FeignException.java:194) ~[feign-core-11.7.jar:na] 
```

为了处理这个错误，我们将创建一个简单的`ExceptionMessage` Java bean 来表示错误消息:

```java
public class ExceptionMessage {
    private String timestamp;
    private int status;
    private String error;
    private String message;
    private String path;
    // standard getters and setters
}
```

让我们通过在定制的`ErrorDecoder`实现中提取原始消息来检索它:

```java
public class RetreiveMessageErrorDecoder implements ErrorDecoder {
    private ErrorDecoder errorDecoder = new Default();

    @Override
    public Exception decode(String methodKey, Response response) {
        ExceptionMessage message = null;
        try (InputStream bodyIs = response.body()
            .asInputStream()) {
            ObjectMapper mapper = new ObjectMapper();
            message = mapper.readValue(bodyIs, ExceptionMessage.class);
        } catch (IOException e) {
            return new Exception(e.getMessage());
        }
        switch (response.status()) {
        case 400:
            return new BadRequestException(message.getMessage() != null ? message.getMessage() : "Bad Request");
        case 404:
            return new NotFoundException(message.getMessage() != null ? message.getMessage() : "Not found");
        default:
            return errorDecoder.decode(methodKey, response);
        }
    }
} 
```

在我们的实现中，我们添加了基于可能错误的逻辑，因此我们可以定制它们来满足我们的需求。在我们的开关块的默认情况下，我们使用 [`Default`](https://web.archive.org/web/20220617075719/https://appdoc.app/artifact/com.netflix.feign/feign-core/8.11.0/feign/codec/ErrorDecoder.Default.html) 实现`ErrorDecoder.`

`Default`当状态不在 2xx 范围内时，实现对 HTTP 响应进行解码。当`throwable`是 [`retryable`](/web/20220617075719/https://www.baeldung.com/feign-retry) 时，它应该是`RetryableException,` 的一个子类型，我们应该尽可能引发特定于应用程序的异常。

为了配置我们定制的 `ErrorDecoder`，我们将把我们的实现作为 bean 添加到 Feign 配置中:

```java
@Bean
public ErrorDecoder errorDecoder() {
    return new RetreiveMessageErrorDecoder();
}
```

现在，让我们看看原始消息的异常:

```java
com.baeldung.cloud.openfeign.exception.NotFoundException: Page Not found
	at com.baeldung.cloud.openfeign.fileupload.config.RetreiveMessageErrorDecoder.decode(RetreiveMessageErrorDecoder.java:30) ~[classes/:na]
	at feign.AsyncResponseHandler.handleResponse(AsyncResponseHandler.java:96) ~[feign-core-11.7.jar:na] 
```

## 4。结论

在本文中，我们演示了如何定制`ErrorDecoder`，以便我们可以捕捉假装的错误来获取原始消息。

像往常一样，本教程中使用的所有代码示例都可以从 GitHub 上的[处获得。](https://web.archive.org/web/20220617075719/https://github.com/eugenp/tutorials/tree/master/spring-cloud-modules/spring-cloud-openfeign)