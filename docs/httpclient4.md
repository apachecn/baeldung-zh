# Apache HttpClient 4 食谱

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/httpclient4>

## 1。概述

这本食谱展示了如何在各种例子和用例中使用 Apache HttpClient。

重点是在 **HttpClient 4.3.x 和更高版本的**，所以一些例子可能无法与老版本的 API 一起工作。

食谱的格式是以实例为中心的，实用的——不需要多余的细节和解释。

如果你想更深入地了解 HttpClient 并学习其他很酷的东西，你可以直接阅读 HttpClient 教程**[。](/web/20220824045322/https://www.baeldung.com/httpclient-guide "Cool basic and more advanced things you can do with the HttpClient 4")**

## 2。食谱

**创建 http 客户端**

```java
CloseableHttpClient client = HttpClientBuilder.create().build();
```

**发送基本获取请求**

```java
instance.execute(new HttpGet("http://www.google.com"));
```

**获取 HTTP 响应的状态码**

```java
CloseableHttpResponse response = instance.execute(new HttpGet("http://www.google.com"));
assertThat(response.getStatusLine().getStatusCode(), equalTo(200));
```

**获取响应的媒体类型**

```java
CloseableHttpResponse response = instance.execute(new HttpGet("http://www.google.com"));
String contentMimeType = ContentType.getOrDefault(response.getEntity()).getMimeType();
assertThat(contentMimeType, equalTo(ContentType.TEXT_HTML.getMimeType()));
```

**得到回应的正文**

```java
CloseableHttpResponse response = instance.execute(new HttpGet("http://www.google.com"));
String bodyAsString = EntityUtils.toString(response.getEntity());
assertThat(bodyAsString, notNullValue());
```

**配置请求超时**

```java
@Test(expected = SocketTimeoutException.class)
public void givenLowTimeout_whenExecutingRequestWithTimeout_thenException() 
    throws ClientProtocolException, IOException {
    RequestConfig requestConfig = RequestConfig.custom()
      .setConnectionRequestTimeout(1000).setConnectTimeout(1000).setSocketTimeout(1000).build();
    HttpGet request = new HttpGet(SAMPLE_URL);
    request.setConfig(requestConfig);
    instance.execute(request);
}
```

**在整个客户端上配置超时**

```java
RequestConfig requestConfig = RequestConfig.custom().
    setConnectionRequestTimeout(1000).setConnectTimeout(1000).setSocketTimeout(1000).build();
HttpClientBuilder builder = HttpClientBuilder.create().setDefaultRequestConfig(requestConfig);
```

**发帖子请求**

```java
instance.execute(new HttpPost(SAMPLE_URL));
```

**给请求添加参数**

```java
List<NameValuePair> params = new ArrayList<NameValuePair>();
params.add(new BasicNameValuePair("key1", "value1"));
params.add(new BasicNameValuePair("key2", "value2"));
request.setEntity(new UrlEncodedFormEntity(params, Consts.UTF_8));
```

**配置如何处理 HTTP 请求的重定向**

```java
CloseableHttpClient instance = HttpClientBuilder.create().disableRedirectHandling().build();
CloseableHttpResponse response = instance.execute(new HttpGet("http://t.co/I5YYd9tddw"));
assertThat(response.getStatusLine().getStatusCode(), equalTo(301));
```

**为请求配置报头**

```java
HttpGet request = new HttpGet(SAMPLE_URL);
request.addHeader(HttpHeaders.ACCEPT, "application/xml");
response = instance.execute(request);
```

**从响应中获取报头**

```java
CloseableHttpResponse response = instance.execute(new HttpGet(SAMPLE_URL));
Header[] headers = response.getHeaders(HttpHeaders.CONTENT_TYPE);
assertThat(headers, not(emptyArray()));
```

**关闭/释放资源**

```java
response = instance.execute(new HttpGet(SAMPLE_URL));
try {
    HttpEntity entity = response.getEntity();
    if (entity != null) {
        InputStream instream = entity.getContent();
        instream.close();
    }
} finally {
    response.close();
}
```

## 3。深入 HttpClient

如果使用正确，HttpClient 库是一个非常强大的工具——如果您想开始**探索客户端能做什么**——请查看一些教程:

*   [http client–获取状态代码](/web/20220824045322/https://www.baeldung.com/httpclient-status-code "Get the Status Code of an Http Request")

*   [http client–设置自定义标题](/web/20220824045322/https://www.baeldung.com/httpclient-custom-http-header "Set a Custom Http Header on a Request")

您还可以通过探索[整个系列](/web/20220824045322/https://www.baeldung.com/httpclient-guide "The Full HttpClient 4 Series")来**更深入地挖掘 HttpClient** 。

## 4。结论

这种格式与我通常的文章结构有点不同——我正在发布一些我的内部开发食谱——关于一个给定的主题——关于[谷歌番石榴(](/web/20220824045322/https://www.baeldung.com/guava-collections "Guava Collections Cookbook")、[哈姆克雷斯特](/web/20220824045322/https://www.baeldung.com/hamcrest-collections-arrays "My Hamcrest Collections cookbook")和[摩奇托](/web/20220824045322/https://www.baeldung.com/mockito-verify "Mockito Verify Cookbook")——现在是 HttpClient。我的目标是让这些信息在网上随时可用——并且每当我遇到一个新的有用的例子时，就把它添加进去。

所有这些例子和代码片段**的实现可以在 GitHubT3 上的[中找到。](https://web.archive.org/web/20220824045322/https://github.com/eugenp/tutorials/tree/master/apache-httpclient "Github Project exemplifying Live HttpClient 4.3 examples")**

这是一个基于 Maven 的项目，因此应该很容易导入和运行。