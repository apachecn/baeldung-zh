# 使用 Spring RestTemplate 上传多文件

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-rest-template-multipart-upload>

## 1。概述

这个快速教程主要讲述如何使用 Spring 的 RestTemplate 上传一个多部分文件。

我们将看到**单个文件和多个文件——使用 RestTemplate 上传** **。**

## 2.什么是 HTTP 多部分请求？

简单地说，一个基本的 HTTP POST 请求体以名称/值对的形式保存表单数据。

另一方面，HTTP 客户端可以构造 HTTP 多部分请求，向服务器发送文本或二进制文件；主要用于上传文件。

另一个常见的用例是发送带有附件的电子邮件。多部分文件请求将大文件分成较小的块，并使用边界标记来指示块的开始和结束。

点击了解更多关于多部分请求[的信息。](https://web.archive.org/web/20220707143824/https://www.w3.org/Protocols/rfc1341/7_2_Multipart.html)

## 3。Maven 依赖关系

对于客户端应用程序来说，这种依赖性就足够了:

```java
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-web</artifactId>
    <version>5.2.2.RELEASE</version>
</dependency>
```

## 4.**文件上传服务器**

文件服务器 API 公开了两个 REST 端点，分别用于上传单个和多个文件:

*   `POST /fileserver/singlefileupload/`
*   `POST /fileserver/multiplefileupload/`

## 5.上传单个文件

首先，让我们看看**单个文件的上传使用了`RestTemplate.`**

我们需要创建带有标题和正文的`HttpEntity`。将`content-type`标题值设置为`MediaType.MULTIPART_FORM_DATA`。当设置了这个头时，`RestTemplate`会自动将文件数据和一些元数据一起进行编组。

元数据包括文件名、文件大小和文件内容类型(例如`text/plain`):

```java
HttpHeaders headers = new HttpHeaders();
headers.setContentType(MediaType.MULTIPART_FORM_DATA);
```

接下来，构建请求体作为 [`LinkedMultiValueMap`](https://web.archive.org/web/20220707143824/https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/util/LinkedMultiValueMap.html) 类的实例。`LinkedMultiValueMap`包装了`LinkedHashMap` ，将每个键的多个值存储在一个`LinkedList`中。

在我们的示例中，`getTestFile( )`方法动态生成一个虚拟文件，并返回一个 [`FileSystemResource`](https://web.archive.org/web/20220707143824/https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/core/io/FileSystemResource.html) :

```java
MultiValueMap<String, Object> body
  = new LinkedMultiValueMap<>();
body.add("file", getTestFile());
```

最后，构建一个包装了 header 和 body 对象的`HttpEntity`实例，并使用一个`RestTemplate`来发布它。

注意，单个文件上传指向`/fileserver/singlefileupload/`端点。

最后，调用 [`restTemplate.postForEntity( )`](/web/20220707143824/https://www.baeldung.com/rest-template) 完成连接到给定 URL 并将文件发送到服务器的工作:

```java
HttpEntity<MultiValueMap<String, Object>> requestEntity
 = new HttpEntity<>(body, headers);

String serverUrl = "http://localhost:8082/spring-rest/fileserver/singlefileupload/";

RestTemplate restTemplate = new RestTemplate();
ResponseEntity<String> response = restTemplate
  .postForEntity(serverUrl, requestEntity, String.class);
```

## 6.上传多个文件

在多文件上传中，与单文件上传的唯一变化是构造请求体。

让我们创建多个文件，并用`MultiValueMap`中的同一个键将它们添加到**中。**

显然，请求 URL 应该指向多个文件上传的端点:

```java
MultiValueMap<String, Object> body
  = new LinkedMultiValueMap<>();
body.add("files", getTestFile());
body.add("files", getTestFile());
body.add("files", getTestFile());

HttpEntity<MultiValueMap<String, Object>> requestEntity
  = new HttpEntity<>(body, headers);

String serverUrl = "http://localhost:8082/spring-rest/fileserver/multiplefileupload/";

RestTemplate restTemplate = new RestTemplate();
ResponseEntity<String> response = restTemplate
  .postForEntity(serverUrl, requestEntity, String.class);
```

总是可以使用多文件上传来模拟单文件上传。

## 7。结论

总之，我们看到了一个使用弹簧`RestTemplate`进行`MultipartFile`传输的例子。

与往常一样，示例客户机和服务器源代码可以在 GitHub 上的[处获得。](https://web.archive.org/web/20220707143824/https://github.com/eugenp/tutorials/tree/master/spring-web-modules/spring-resttemplate-3)