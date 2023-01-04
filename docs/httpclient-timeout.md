# Apache HttpClient 超时

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/httpclient-timeout>

## 1。概述

本教程将展示如何用 Apache `HttpClient 4` 配置超时。

如果你想更深入地了解和学习其他很酷的东西，你可以使用 http client——直接进入 **[主`HttpClient`教程](/web/20220625174301/https://www.baeldung.com/httpclient-guide "Cool basic and more advanced things you can do with the HttpClient 4")** 。

## 延伸阅读:

## [Apache HttpClient 连接管理](/web/20220625174301/https://www.baeldung.com/httpclient-connection-management)

How to open, manage and close connections with the Apache HttpClient 4.[Read more](/web/20220625174301/https://www.baeldung.com/httpclient-connection-management) →

## [Apache http client–遵循帖子的重定向](/web/20220625174301/https://www.baeldung.com/httpclient-redirect-on-http-post)

How to enable POST Redirect with Apache HttpClient.[Read more](/web/20220625174301/https://www.baeldung.com/httpclient-redirect-on-http-post) →

## [高级 Apache HttpClient 配置](/web/20220625174301/https://www.baeldung.com/httpclient-advanced-config)

HttpClient configurations for advanced use cases.[Read more](/web/20220625174301/https://www.baeldung.com/httpclient-advanced-config) →

## 2。T3`HttpClient`4.3 前配置超时

### 2.1。原始`String`参数

在 4.3 版本出来之前，`HttpClient`附带了许多配置参数，所有这些都可以以通用的、类似地图的方式设置。

配置有 **3 个超时参数:**

```java
DefaultHttpClient httpClient = new DefaultHttpClient();

int timeout = 5; // seconds
HttpParams httpParams = httpClient.getParams();
httpParams.setParameter(
  CoreConnectionPNames.CONNECTION_TIMEOUT, timeout * 1000);
httpParams.setParameter(
  CoreConnectionPNames.SO_TIMEOUT, timeout * 1000);
httpParams.setParameter(
  ClientPNames.CONN_MANAGER_TIMEOUT, new Long(timeout * 1000));
```

### 2.2。API

这些参数中更重要的参数(即前两个参数)也可以通过更类型安全的 API 来设置:

```java
DefaultHttpClient httpClient = new DefaultHttpClient();

int timeout = 5; // seconds
HttpParams httpParams = httpClient.getParams();
HttpConnectionParams.setConnectionTimeout(
  httpParams, timeout * 1000); // http.connection.timeout
HttpConnectionParams.setSoTimeout(
  httpParams, timeout * 1000); // http.socket.timeout
```

第三个参数在`HttpConnectionParams`中没有自定义设置器，仍然需要通过`setParameter`方法手动设置。

## 3。使用新的 4.3 版本配置超时。建造者

4.3 中引入的 fluent builder API 为**提供了在高级别设置超时的正确方法**:

```java
int timeout = 5;
RequestConfig config = RequestConfig.custom()
  .setConnectTimeout(timeout * 1000)
  .setConnectionRequestTimeout(timeout * 1000)
  .setSocketTimeout(timeout * 1000).build();
CloseableHttpClient client = 
  HttpClientBuilder.create().setDefaultRequestConfig(config).build();
```

这是以类型安全和可读的方式配置所有三种超时的推荐方式。

## 4。解释的超时属性

现在，让我们解释一下这些不同类型的超时意味着什么:

*   **连接超时**(`http.connection.timeout`)——与远程主机建立连接的时间
*   **套接字超时**(`http.socket.timeout`)——建立连接后等待数据的时间；两个数据包之间不活动的最长时间
*   **连接管理器超时**(`http.connection-manager.timeout`)–等待来自连接管理器/池的连接的时间

前两个参数——连接和套接字超时——是最重要的。然而，在高负载场景中，为获取连接设置一个超时是非常重要的，这就是为什么第三个参数不应该被忽略。

## 5。使用`HttpClient`

配置完成后，我们现在可以使用客户机来执行 HTTP 请求:

```java
HttpGet getMethod = new HttpGet("http://host:8080/path");
HttpResponse response = httpClient.execute(getMethod);
System.out.println(
  "HTTP Status of response: " + response.getStatusLine().getStatusCode());
```

使用之前定义的客户端，**与主机的连接将在 5 秒后超时。**同样，如果连接建立但没有接收到数据，超时也将是 **5 秒额外的**。

注意，连接超时将导致抛出一个`org.apache.http.conn.ConnectTimeoutException`，而套接字超时将导致一个`java.net.SocketTimeoutException`。

## 6。硬超时

虽然在建立 HTTP 连接和不接收数据时设置超时非常有用，但有时我们需要为整个请求设置一个**硬超时。**

例如，下载一个潜在的大文件就属于这一类。在这种情况下，连接可能会成功建立，数据可能会持续传入，但我们仍然需要确保操作不会超过某个特定的时间阈值。

`HttpClient`没有任何允许我们为请求设置总超时的配置；然而，它确实为请求提供了**中止功能，因此我们可以利用该机制来实现一个简单的超时机制:**

```java
HttpGet getMethod = new HttpGet(
  "http://localhost:8080/httpclient-simple/api/bars/1");

int hardTimeout = 5; // seconds
TimerTask task = new TimerTask() {
    @Override
    public void run() {
        if (getMethod != null) {
            getMethod.abort();
        }
    }
};
new Timer(true).schedule(task, hardTimeout * 1000);

HttpResponse response = httpClient.execute(getMethod);
System.out.println(
  "HTTP Status of response: " + response.getStatusLine().getStatusCode());
```

我们利用`java.util.Timer`和`java.util.TimerTask`来设置一个**简单的延迟任务，该任务在 5 秒钟的硬超时后中止 HTTP GET 请求**。

## 7。超时和 DNS 循环——需要注意的事项

一些较大的域将使用 DNS 循环配置是很常见的——本质上是让**同一个域映射到多个 IP 地址**。这为针对这样的域的超时引入了新的挑战，仅仅是因为 HttpClient 将尝试连接到超时的域的方式:

*   HttpClient 获取到该域的 IP 路由列表
*   它尝试**第一个**——超时(使用我们配置的超时)
*   它尝试第二个——也超时了
*   诸如此类…

因此，如您所见–**整体操作不会在我们预期的时间**超时。相反，当所有可能的路线都超时时，它将超时。此外，这对于客户端来说是完全透明的(除非您在调试级别配置了日志)。

这里有一个简单的例子，您可以运行并复制这个问题:

```java
int timeout = 3;
RequestConfig config = RequestConfig.custom().
  setConnectTimeout(timeout * 1000).
  setConnectionRequestTimeout(timeout * 1000).
  setSocketTimeout(timeout * 1000).build();
CloseableHttpClient client = HttpClientBuilder.create()
  .setDefaultRequestConfig(config).build();

HttpGet request = new HttpGet("http://www.google.com:81");
response = client.execute(request);
```

您将注意到带有调试日志级别的重试逻辑:

```java
DEBUG o.a.h.i.c.HttpClientConnectionOperator - Connecting to www.google.com/173.194.34.212:81
DEBUG o.a.h.i.c.HttpClientConnectionOperator - 
 Connect to www.google.com/173.194.34.212:81 timed out. Connection will be retried using another IP address

DEBUG o.a.h.i.c.HttpClientConnectionOperator - Connecting to www.google.com/173.194.34.208:81
DEBUG o.a.h.i.c.HttpClientConnectionOperator - 
 Connect to www.google.com/173.194.34.208:81 timed out. Connection will be retried using another IP address

DEBUG o.a.h.i.c.HttpClientConnectionOperator - Connecting to www.google.com/173.194.34.209:81
DEBUG o.a.h.i.c.HttpClientConnectionOperator - 
 Connect to www.google.com/173.194.34.209:81 timed out. Connection will be retried using another IP address
//...
```

## 8。结论

本教程讨论了如何为`HttpClient`配置各种类型的超时。它还展示了一个简单的正在进行的 HTTP 连接的硬超时机制。

这些例子的实现可以在[GitHub 项目](https://web.archive.org/web/20220625174301/https://github.com/eugenp/tutorials/tree/master/httpclient-simple "HttpClient example project - configure timeout")中找到。