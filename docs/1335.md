# 使用 WebClient 上传文件

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-webclient-upload-file>

## 1.概观

我们的应用程序经常需要通过 HTTP 请求来处理文件上传。从 Spring 5 开始，我们现在可以对这些请求做出反应。
增加的对[反应式编程](/web/20220529024647/https://www.baeldung.com/java-reactive-systems)的支持允许我们以`non-blocking`的方式工作，使用少量线程和[背压](/web/20220529024647/https://www.baeldung.com/spring-webflux-backpressure)。

在本文中，我们将使用`[WebClient](/web/20220529024647/https://www.baeldung.com/spring-5-webclient)`——一个非阻塞、反应式 HTTP 客户端——来演示如何上传文件。`WebClient`是名为`Project Reactor`的反应式编程库的一部分。我们将介绍使用`BodyInserter`上传文件的两种不同方法。

## 2.上传带有`WebClient`的文件

为了使用`WebClient`，我们需要向我们的项目添加`spring-boot-starter-webflux`依赖项:

```
<dependency>
    <groupId>org.springframework.boot</groupId>. 
    <artifactId>spring-boot-starter-webflux</artifactId>
</dependency>
```

### 2.1.从资源上传文件

首先，我们要声明我们的 URL:

```
URI url = UriComponentsBuilder.fromHttpUrl(EXTERNAL_UPLOAD_URL).build().toUri(); 
```

假设在这个例子中，我们想要上传一个 PDF。**我们将用`MediaType.APPLICATION_PDF`作为我们的`ContentType`** 。
我们的上传端点返回一个`HttpStatus. `，因为我们只期望一个结果，所以我们将它包装在一个`Mono`中:

```
Mono<HttpStatus> httpStatusMono = webClient.post()
    .uri(url)
    .contentType(MediaType.APPLICATION_PDF)
    .body(BodyInserters.fromResource(resource))
    .exchangeToMono(response -> {
        if (response.statusCode().equals(HttpStatus.OK)) {
            return response.bodyToMono(HttpStatus.class).thenReturn(response.statusCode());
        } else {
            throw new ServiceException("Error uploading file");
        }
     });
```

使用这个方法的方法也可以返回一个`Mono`，我们可以继续，直到我们真正需要访问结果。一旦我们准备好了，我们就可以在`Mono`对象上调用`block()`方法。

`fromResource()`方法使用传递的资源的`InputStream`来写入输出消息。

### 2.2.从多部分资源上传文件

**如果我们的外部上传端点接受多部分表单数据，我们可以使用** **`MultiPartBodyBuilder`** 来处理这些部分:

```
MultipartBodyBuilder builder = new MultipartBodyBuilder();
builder.part("file", multipartFile.getResource());
```

在这里，我们可以根据自己的需求添加各种部件。图中的值可以是一个`Object`或一个 `HttpEntity.`

当我们调用`WebClient`时，我们使用`BodyInsterter.fromMultipartData`并构建对象:

```
.body(BodyInserters.fromMultipartData(builder.build()))
```

我们将内容类型更新为`MediaType.MULTIPART_FORM_DATA`以反映这些变化。

让我们看一下整个通话:

```
Mono<HttpStatus> httpStatusMono = webClient.post()
    .uri(url)
    .contentType(MediaType.MULTIPART_FORM_DATA)
    .body(BodyInserters.fromMultipartData(builder.build()))
    .exchangeToMono(response -> {
        if (response.statusCode().equals(HttpStatus.OK)) {
            return response.bodyToMono(HttpStatus.class).thenReturn(response.statusCode());
        } else {
            throw new ServiceException("Error uploading file");
        }
      });
```

## 3.结论

在本教程中，我们展示了使用`BodyInserter` s 上传带有`WebClient`的文件的两种方法。与往常一样，代码可以在 GitHub 上的[处获得。](https://web.archive.org/web/20220529024647/https://github.com/eugenp/tutorials/tree/master/spring-5-reactive-client)