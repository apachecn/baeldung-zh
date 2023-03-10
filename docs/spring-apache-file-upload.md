# Spring 和 Apache 文件上传

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-apache-file-upload>

## 1。概述

[**Apache Commons 文件上传库**](https://web.archive.org/web/20221128041557/https://commons.apache.org/proper/commons-fileupload/index.html) 帮助我们使用`multipart/form-data`内容类型通过 HTTP 协议上传大文件。

在这个快速教程中，我们将看看如何将其与 Spring 集成。

## 2。Maven 依赖关系

要使用这个库，我们需要`commons-fileupload`工件:

```java
<dependency>
    <groupId>commons-fileupload</groupId>
    <artifactId>commons-fileupload</artifactId>
    <version>1.4</version>
</dependency>
```

最新版本可以在 [Maven Central](https://web.archive.org/web/20221128041557/https://search.maven.org/classic/#search%7Cga%7C1%7Ca%3A%22commons-fileupload%22) 上找到。

## 3。一次全部转移

出于演示的目的，我们将创建一个使用文件有效负载处理请求的`Controller`:

```java
@PostMapping("/upload")
public String handleUpload(HttpServletRequest request) throws Exception {
    boolean isMultipart = ServletFileUpload.isMultipartContent(request);

    DiskFileItemFactory factory = new DiskFileItemFactory();
    factory.setRepository(
      new File(System.getProperty("java.io.tmpdir")));
    factory.setSizeThreshold(
      DiskFileItemFactory.DEFAULT_SIZE_THRESHOLD);
    factory.setFileCleaningTracker(null);

    ServletFileUpload upload = new ServletFileUpload(factory);

    List items = upload.parseRequest(request);

    Iterator iter = items.iterator();
    while (iter.hasNext()) {
        FileItem item = iter.next();

        if (!item.isFormField()) {
            try (
              InputStream uploadedStream = item.getInputStream();
              OutputStream out = new FileOutputStream("file.mov");) {

                IOUtils.copy(uploadedStream, out);
            }
        }
    }    
    return "success!";
} 
```

一开始，**我们需要使用库中的`ServletFileUpload` 类中的`isMultipartContent`方法来检查请求是否包含多部分内容**。

默认情况下， **Spring 有一个** `**MultipartResolver**` **特性，我们需要禁用它才能使用这个库。**否则，它会在请求到达我们的`Controller.`之前读取请求的内容

我们可以通过将此配置包含在我们的`application.properties`文件中来实现这一点:

```java
spring.http.multipart.enabled=false
```

现在，我们可以设置保存文件的目录、库决定写入磁盘的阈值以及请求结束后是否应该删除文件。

该库提供了一个`DiskFileItemFactory`类，由**负责配置文件保存和清理**。`setRepository` 方法设置目标目录，默认目录如示例所示。

接下来，`setSizeThreshold` 设置最大文件大小。

然后，我们有了`setFileCleaningTracker` 方法，当设置为 null 时，保持临时文件不变。默认情况下，它在请求完成后删除它们。

现在我们可以继续实际的文件处理。

首先，我们通过包含先前创建的工厂来创建我们的`ServletFileUpload` ；然后我们继续解析请求并生成一个列表`FileItem`,它是表单字段库的主要抽象。

现在，如果我们知道它不是一个普通的表单字段，那么我们继续提取`InputStream` 并从`IOUtils`调用有用的复制方法(更多选项你可以看看[这个【T4 教程】`.`](/web/20221128041557/https://www.baeldung.com/convert-input-stream-to-a-file)

现在，我们已经将文件存储在必要的文件夹中。这通常是处理这种情况的更方便的方法，因为它允许轻松访问文件，但时间/内存效率也不是最佳的。

在下一节中，我们将看看流式 API。

## 4。流式 API

流式 API 易于使用，这使得它成为处理大文件的一种很好的方式，只需不将其复制到临时位置即可:

```java
ServletFileUpload upload = new ServletFileUpload();
FileItemIterator iterStream = upload.getItemIterator(request);
while (iterStream.hasNext()) {
    FileItemStream item = iterStream.next();
    String name = item.getFieldName();
    InputStream stream = item.openStream();
    if (!item.isFormField()) {
        // Process the InputStream
    } else {
        String formFieldValue = Streams.asString(stream);
    }
} 
```

我们可以在前面的代码片段中看到，我们不再包含一个`DiskFileItemFactory`。这是因为，**使用流式 API 时，我们不需要它**。

接下来，为了处理字段，库提供了一个`FileItemIterator`，它不会读取任何东西，直到我们用`next`方法从请求中提取它们。

最后，我们可以看到如何获得其他表单字段的值。

## 5。结论

在本文中，我们回顾了如何使用 Apache Commons 文件上传库和 Spring 来上传和处理大文件。

和往常一样，完整的源代码可以在 GitHub 找到[。](https://web.archive.org/web/20221128041557/https://github.com/eugenp/tutorials/tree/master/spring-web-modules/spring-rest-simple)