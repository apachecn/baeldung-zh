# 使用 Apache HttpClient 进行多部分上传

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/httpclient-multipart-upload>

## 1。概述

在本教程中，我们将演示如何使用 HttpClient 进行**多部分上传操作。**

我们将使用`http://echo.200please.com`作为测试服务器，因为它是公共的，可以接受大多数类型的内容。

如果你想更深入地了解 **和学习其他很酷的东西，你可以使用 HttpClient**——直接进入 [主 http client 教程](/web/20220831070027/https://www.baeldung.com/httpclient-guide "Cool basic and more advanced things you can do with the HttpClient 4") 。

## 2。使用`AddPart`方法

让我们从查看向 Http 实体添加部分的`MultipartEntityBuilder`对象开始，该实体将通过 POST 操作上传。

这是一个通用方法，将部件添加到代表表单的`HttpEntity`中。

例 2.1。–**上传包含两个文本部分和一个文件的表单**

```java
File file = new File(textFileName);
HttpPost post = new HttpPost("http://echo.200please.com");
FileBody fileBody = new FileBody(file, ContentType.DEFAULT_BINARY);
StringBody stringBody1 = new StringBody("Message 1", ContentType.MULTIPART_FORM_DATA);
StringBody stringBody2 = new StringBody("Message 2", ContentType.MULTIPART_FORM_DATA);
// 
MultipartEntityBuilder builder = MultipartEntityBuilder.create();
builder.setMode(HttpMultipartMode.BROWSER_COMPATIBLE);
builder.addPart("upfile", fileBody);
builder.addPart("text1", stringBody1);
builder.addPart("text2", stringBody2);
HttpEntity entity = builder.build();
//
post.setEntity(entity);
HttpResponse response = client.execute(post);
```

注意，我们还通过指定服务器要使用的`ContentType`值来实例化`File`对象。

另外，请注意，`addPart`方法有两个参数，就像表单的`key/value`对一样。只有当服务器端实际需要并使用参数名时，这些才是相关的——否则，它们会被忽略。

## 3。使用`addBinaryBody`和`addTextBody`方法

创建多部分实体更直接的方法是使用 `addBinaryBody`和`AddTextBody`方法。这些方法用于上传文本、文件、字符数组和`InputStream`对象。让我们用简单的例子来说明。

例 3.1。–**上传文本和文本文件部分**

```java
HttpPost post = new HttpPost("http://echo.200please.com");
File file = new File(textFileName);
String message = "This is a multipart post";
MultipartEntityBuilder builder = MultipartEntityBuilder.create();
builder.setMode(HttpMultipartMode.BROWSER_COMPATIBLE);
builder.addBinaryBody("upfile", file, ContentType.DEFAULT_BINARY, textFileName);
builder.addTextBody("text", message, ContentType.DEFAULT_BINARY);
// 
HttpEntity entity = builder.build();
post.setEntity(entity);
HttpResponse response = client.execute(post);
```

注意这里不需要`FileBody`和`StringBody`对象。

同样重要的是，大多数服务器不检查文本体的`ContentType`，所以`addTextBody`方法可能会省略 `ContentType`值。

`addBinaryBody` API 接受一个`ContentType` ——但是它 `is also` 可能仅仅从一个二进制体和保存文件的表单参数的名称来创建实体。如前所述，如果没有指定`ContentType`值，一些服务器将无法识别该文件。

接下来，我们将添加一个 zip 文件作为`InputStream,`，而图像文件将作为`File`对象添加:

例 3.2。–**上传一个Zip 文件、一个图像文件和一个文本部分**

```java
HttpPost post = new HttpPost("http://echo.200please.com");
InputStream inputStream = new FileInputStream(zipFileName);
File file = new File(imageFileName);
String message = "This is a multipart post";
MultipartEntityBuilder builder = MultipartEntityBuilder.create();         
builder.setMode(HttpMultipartMode.BROWSER_COMPATIBLE);
builder.addBinaryBody
  ("upfile", file, ContentType.DEFAULT_BINARY, imageFileName);
builder.addBinaryBody
  ("upstream", inputStream, ContentType.create("application/zip"), zipFileName);
builder.addTextBody("text", message, ContentType.TEXT_PLAIN);
// 
HttpEntity entity = builder.build();
post.setEntity(entity);
HttpResponse response = client.execute(post);
```

请注意，`ContentType`值可以动态创建，就像上面例子中 zip 文件的情况一样。

最后，不是所有的服务器都承认 `InputStream`部分。我们在第一行代码中实例化的服务器识别 `InputStream` s。

现在让我们看看另一个例子，其中`addBinaryBody`直接使用字节数组:

例 3.3。–**上传字节数组和文本**

```java
HttpPost post = new HttpPost("http://echo.200please.com");
String message = "This is a multipart post";
byte[] bytes = "binary code".getBytes(); 
// 
MultipartEntityBuilder builder = MultipartEntityBuilder.create();
builder.setMode(HttpMultipartMode.BROWSER_COMPATIBLE);
builder.addBinaryBody("upfile", bytes, ContentType.DEFAULT_BINARY, textFileName);
builder.addTextBody("text", message, ContentType.TEXT_PLAIN);
// 
HttpEntity entity = builder.build();
post.setEntity(entity);
HttpResponse response = client.execute(post);
```

注意`ContentType`–它现在指定二进制数据。

## 4。结论

本文将`MultipartEntityBuilder`展示为一个灵活的对象，它提供了多种 API 选择来创建多部分表单。

这些例子还展示了如何使用`HttpClient`来上传类似于表单实体的`a HttpEntity`。

所有这些例子和代码片段**的实现可以在我们的 [GitHub 项目](https://web.archive.org/web/20220831070027/https://github.com/eugenp/tutorials/tree/master/apache-httpclient "Github Project with live test showing multipart upload with the HttpClient")** 中找到——这是一个基于 Eclipse 的项目，所以它应该很容易导入和运行。