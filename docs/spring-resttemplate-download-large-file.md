# 通过 Spring RestTemplate 下载一个大文件

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-resttemplate-download-large-file>

## 1.概观

在本教程中，我们将展示如何用`RestTemplate`下载大文件的不同技巧。

## 2.`RestTemplate`

`[RestTemplate](/web/20221129010752/https://www.baeldung.com/rest-template)`是 Spring 3 中引入的一个阻塞式同步 HTTP 客户端。根据 [Spring 文档](https://web.archive.org/web/20221129010752/https://docs.spring.io/spring/docs/current/javadoc-api/org/springframework/web/client/RestTemplate.html)，它将在未来被弃用，因为他们在版本 5 中引入了`[WebClient](/web/20221129010752/https://www.baeldung.com/spring-5-webclient)` 作为反应式非阻塞 HTTP 客户端。

## 3.陷阱

通常，当我们下载一个文件时，我们将它存储在我们的文件系统中或者作为字节数组加载到内存中。但是当它是一个大文件时，内存加载可能会导致一个`[OutOfMemoryError](/web/20221129010752/https://www.baeldung.com/java-gc-overhead-limit-exceeded)`。因此，我们必须在读取响应块时将数据存储在一个文件中。

让我们先来看看几种不起作用的方法:

首先，如果我们返回一个`Resource`作为我们的返回类型会发生什么:

```java
Resource download() {
    return new ClassPathResource(locationForLargeFile);
}
```

这不起作用的原因是`ResourceHttpMesssageConverter ` **会将整个响应体加载到`ByteArrayInputStream`** 中，这仍然增加了我们想要避免的内存压力。

第二，如果我们返回一个`InputStreamResource `并配置`ResourceHttpMessageConverter#supportsReadStreaming`会怎么样？嗯，这也不起作用，因为当我们调用`InputStreamResource.getInputStream()`时，我们得到了一个`socket closed”` 错误！这是因为`“execute`在退出之前关闭了响应输入流。

那么我们能做些什么来解决这个问题呢？实际上，这里也有两件事:

*   **编写一个自定义 [`HttpMessageConverter`](/web/20221129010752/https://www.baeldung.com/spring-httpmessageconverter-rest)** ，支持`File`作为返回类型
*   使用`RestTemplate.execute`和**自定义`ResponseExtractor`** 将输入流存储在`File`中

在本教程中，我们将使用第二种解决方案，因为它更灵活，也更省力。

## 4.不带简历下载

让我们实现一个`ResponseExtractor` 来将主体写到一个临时文件`:`

```java
File file = restTemplate.execute(FILE_URL, HttpMethod.GET, null, clientHttpResponse -> {
    File ret = File.createTempFile("download", "tmp");
    StreamUtils.copy(clientHttpResponse.getBody(), new FileOutputStream(ret));
    return ret;
});

Assert.assertNotNull(file);
Assertions
  .assertThat(file.length())
  .isEqualTo(contentLength);
```

这里我们使用了`StreamUtils.copy` 来复制 `FileOutputStream,`中的响应输入流，但是其他的[技术和库](/web/20221129010752/https://www.baeldung.com/convert-input-stream-to-a-file)也是可用的。

## 5.暂停和继续下载

因为我们要下载一个大文件，所以在出于某种原因暂停后考虑下载是合理的。

所以首先让我们检查下载 URL 是否支持 resume:

```java
HttpHeaders headers = restTemplate.headForHeaders(FILE_URL);

Assertions
  .assertThat(headers.get("Accept-Ranges"))
  .contains("bytes");
Assertions
  .assertThat(headers.getContentLength())
  .isGreaterThan(0);
```

然后我们可以实现一个`RequestCallback`来设置“Range”头并继续下载:

```java
restTemplate.execute(
  FILE_URL,
  HttpMethod.GET,
  clientHttpRequest -> clientHttpRequest.getHeaders().set(
    "Range",
    String.format("bytes=%d-%d", file.length(), contentLength)),
    clientHttpResponse -> {
        StreamUtils.copy(clientHttpResponse.getBody(), new FileOutputStream(file, true));
    return file;
});

Assertions
  .assertThat(file.length())
  .isLessThanOrEqualTo(contentLength);
```

如果我们不知道确切的内容长度，我们可以使用`String.format`设置`Range`头值:

```java
String.format("bytes=%d-", file.length())
```

## 6.结论

我们已经讨论了下载大文件时可能出现的问题。我们在使用`RestTemplate`时也提出了解决方案。最后，我们展示了如何实现可恢复的下载。

和往常一样，代码可以在我们的 [GitHub](https://web.archive.org/web/20221129010752/https://github.com/eugenp/tutorials/tree/master/spring-web-modules/spring-resttemplate-3) 中找到。