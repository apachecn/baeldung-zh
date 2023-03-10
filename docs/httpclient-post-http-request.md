# 使用 Apache HttpClient 发布

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/httpclient-post-http-request>

## 1。概述

在本教程中，**我们将使用`HttpClient` 4** 发布，首先使用授权，然后使用流畅的 HttpClient API。

最后，我们将讨论如何使用 HttpClient 上传文件。

## 延伸阅读:

## [高级 Apache HttpClient 配置](/web/20220625165746/https://www.baeldung.com/httpclient-advanced-config)

高级用例的 HttpClient 配置。[阅读更多](/web/20220625165746/https://www.baeldung.com/httpclient-advanced-config)→

## [Apache HttpClient–发送自定义 Cookie](/web/20220625165746/https://www.baeldung.com/httpclient-cookies)

如何用 Apache http client 发送自定义 Cookie。[阅读更多](/web/20220625165746/https://www.baeldung.com/httpclient-cookies)→

## [Apache HttpClient with SSL](/web/20220625165746/https://www.baeldung.com/httpclient-ssl)

如何配置 http client with SSL 的例子。[阅读更多](/web/20220625165746/https://www.baeldung.com/httpclient-ssl) →

## 2。基本岗位

首先，我们来看一个简单的例子，使用`HttpClient`发送一个 POST 请求。

我们将使用两个参数“T0”和“T1”进行发布:

```java
@Test
public void whenSendPostRequestUsingHttpClient_thenCorrect() 
  throws ClientProtocolException, IOException {
    CloseableHttpClient client = HttpClients.createDefault();
    HttpPost httpPost = new HttpPost("http://www.example.com");

    List<NameValuePair> params = new ArrayList<NameValuePair>();
    params.add(new BasicNameValuePair("username", "John"));
    params.add(new BasicNameValuePair("password", "pass"));
    httpPost.setEntity(new UrlEncodedFormEntity(params));

    CloseableHttpResponse response = client.execute(httpPost);
    assertThat(response.getStatusLine().getStatusCode(), equalTo(200));
    client.close();
}
```

**注意我们如何使用`NameValuePair`中的`List`在 POST 请求中包含参数。**

## 3。授权发布

接下来，让我们看看如何使用`HttpClient`使用认证凭证进行 POST。

在下面的例子中，我们将通过添加一个授权头向一个使用基本身份验证保护的 URL 发送一个 POST 请求:

```java
@Test
public void whenSendPostRequestWithAuthorizationUsingHttpClient_thenCorrect()
  throws ClientProtocolException, IOException, AuthenticationException {
    CloseableHttpClient client = HttpClients.createDefault();
    HttpPost httpPost = new HttpPost("http://www.example.com");

    httpPost.setEntity(new StringEntity("test post"));
    UsernamePasswordCredentials creds
      = new UsernamePasswordCredentials("John", "pass");
    httpPost.addHeader(new BasicScheme().authenticate(creds, httpPost, null));

    CloseableHttpResponse response = client.execute(httpPost);
    assertThat(response.getStatusLine().getStatusCode(), equalTo(200));
    client.close();
}
```

## 4。用 JSON 发布

现在让我们看看如何使用`HttpClient`发送带有 JSON 主体的 POST 请求。

在下面的例子中，我们将把一些`person`信息(`id, name`)作为 JSON 发送:

```java
@Test
public void whenPostJsonUsingHttpClient_thenCorrect() 
  throws ClientProtocolException, IOException {
    CloseableHttpClient client = HttpClients.createDefault();
    HttpPost httpPost = new HttpPost("http://www.example.com");

    String json = "{"id":1,"name":"John"}";
    StringEntity entity = new StringEntity(json);
    httpPost.setEntity(entity);
    httpPost.setHeader("Accept", "application/json");
    httpPost.setHeader("Content-type", "application/json");

    CloseableHttpResponse response = client.execute(httpPost);
    assertThat(response.getStatusLine().getStatusCode(), equalTo(200));
    client.close();
}
```

**注意我们如何使用`StringEntity`来设置请求的主体。**

**我们还将`ContentType`头设置为`application/json`** ，以向服务器提供关于我们发送的内容表示的必要信息。

## 5。`HttpClient Fluent API`贴着

接下来，我们用`HttpClient` Fluent API 来发帖吧。

我们将发送一个带有两个参数的请求，“T0”和“T1”:

```java
@Test
public void whenPostFormUsingHttpClientFluentAPI_thenCorrect() 
  throws ClientProtocolException, IOException {
    HttpResponse response = Request.Post("http://www.example.com").bodyForm(
      Form.form().add("username", "John").add("password", "pass").build())
      .execute().returnResponse();

    assertThat(response.getStatusLine().getStatusCode(), equalTo(200));
}
```

## 6。发布多部分请求

现在让我们发布一个多部分请求。

**我们将使用`MultipartEntityBuilder` :** 发布一个`File`，用户名和密码

```java
@Test
public void whenSendMultipartRequestUsingHttpClient_thenCorrect() 
  throws ClientProtocolException, IOException {
    CloseableHttpClient client = HttpClients.createDefault();
    HttpPost httpPost = new HttpPost("http://www.example.com");

    MultipartEntityBuilder builder = MultipartEntityBuilder.create();
    builder.addTextBody("username", "John");
    builder.addTextBody("password", "pass");
    builder.addBinaryBody(
      "file", new File("test.txt"), ContentType.APPLICATION_OCTET_STREAM, "file.ext");

    HttpEntity multipart = builder.build();
    httpPost.setEntity(multipart);

    CloseableHttpResponse response = client.execute(httpPost);
    assertThat(response.getStatusLine().getStatusCode(), equalTo(200));
    client.close();
}
```

## 7。使用`HttpClient` 上传一个`File`

接下来，让我们看看如何使用`HttpClient.` 上传一个`File`

**我们将使用`MultipartEntityBuilder` :** 上传“`test.txt`”文件

```java
@Test
public void whenUploadFileUsingHttpClient_thenCorrect() 
  throws ClientProtocolException, IOException {
    CloseableHttpClient client = HttpClients.createDefault();
    HttpPost httpPost = new HttpPost("http://www.example.com");

    MultipartEntityBuilder builder = MultipartEntityBuilder.create();
    builder.addBinaryBody(
      "file", new File("test.txt"), ContentType.APPLICATION_OCTET_STREAM, "file.ext");
    HttpEntity multipart = builder.build();
    httpPost.setEntity(multipart);

    CloseableHttpResponse response = client.execute(httpPost);
    assertThat(response.getStatusLine().getStatusCode(), equalTo(200));
    client.close();
}
```

## 8。获取`File`上传进度

最后，让我们看看如何使用`HttpClient`来获取`File`上传的进度。

在下面的例子中，我们将扩展`HttpEntityWrapper`来获得上传过程的可见性。

首先，上传方法如下:

```java
@Test
public void whenGetUploadFileProgressUsingHttpClient_thenCorrect()
  throws ClientProtocolException, IOException {
    CloseableHttpClient client = HttpClients.createDefault();
    HttpPost httpPost = new HttpPost("http://www.example.com");

    MultipartEntityBuilder builder = MultipartEntityBuilder.create();
    builder.addBinaryBody(
      "file", new File("test.txt"), ContentType.APPLICATION_OCTET_STREAM, "file.ext");
    HttpEntity multipart = builder.build();

    ProgressEntityWrapper.ProgressListener pListener = 
      percentage -> assertFalse(Float.compare(percentage, 100) > 0);
    httpPost.setEntity(new ProgressEntityWrapper(multipart, pListener));

    CloseableHttpResponse response = client.execute(httpPost);
    assertThat(response.getStatusLine().getStatusCode(), equalTo(200));
    client.close();
}
```

**我们还将添加界面`ProgressListener`，使我们能够观察上传进度:**

```java
public static interface ProgressListener {
    void progress(float percentage);
}
```

下面是我们的`HttpEntityWrapper,` `ProgressEntityWrapper`的加长版:

```java
public class ProgressEntityWrapper extends HttpEntityWrapper {
    private ProgressListener listener;

    public ProgressEntityWrapper(HttpEntity entity, ProgressListener listener) {
        super(entity);
        this.listener = listener;
    }

    @Override
    public void writeTo(OutputStream outstream) throws IOException {
        super.writeTo(new CountingOutputStream(outstream, listener, getContentLength()));
    }
} 
```

而这里是`FilterOutputStream,``CountingOutputStream`的加长版:

```java
public static class CountingOutputStream extends FilterOutputStream {
    private ProgressListener listener;
    private long transferred;
    private long totalBytes;

    public CountingOutputStream(
      OutputStream out, ProgressListener listener, long totalBytes) {
        super(out);
        this.listener = listener;
        transferred = 0;
        this.totalBytes = totalBytes;
    }

    @Override
    public void write(byte[] b, int off, int len) throws IOException {
        out.write(b, off, len);
        transferred += len;
        listener.progress(getCurrentProgress());
    }

    @Override
    public void write(int b) throws IOException {
        out.write(b);
        transferred++;
        listener.progress(getCurrentProgress());
    }

    private float getCurrentProgress() {
        return ((float) transferred / totalBytes) * 100;
    }
}
```

请注意:

*   当将`FilterOutputStream` 扩展为"`CountingOutputStream,”` 时，我们覆盖了`write()` 方法来计算写入(传输)的字节数
*   当将`HttpEntityWrapper` 扩展到`ProgressEntityWrapper,”` 时，我们覆盖了 `writeTo()` 方法来使用我们的 `“CountingOutputStream”`

## 9。结论

在本文中，我们举例说明了用`Apache HttpClient 4`发送 POST HTTP 请求的最常见方式。

我们学习了如何通过授权发送 POST 请求，如何使用`HttpClient` fluent API 发布，以及如何上传文件并跟踪其进度。

所有这些例子和代码片段的实现都可以在 github 项目中找到。