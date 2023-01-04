# 向 Apache HttpClient 请求添加参数

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/apache-httpclient-parameters>

## 1.介绍

HttpClient 是 Apache HttpComponents 项目的一部分，该项目提供了一个专注于 HTTP 和相关协议的低级 Java 组件工具集。HttpClient 最本质的功能就是执行 HTTP 方法。

在这个简短的教程中，我们将讨论向`HttpClient`请求添加参数。我们将学习如何将`UriBuilder`与字符串名称-值对以及`NameValuePair`一起使用。类似地，我们将看到如何使用`UrlEncodedFormEntity`传递参数。

## 2.使用`UriBuilder`向`HttpClient`请求添加参数

`UriBuilder`帮助我们通过构建器模式轻松创建 URIs 和添加参数。我们可以使用`String`名称-值对**添加参数，或者为此目的利用`NameValuePair`的类**。

在本例中，最终的 URL 应该如下所示:

```java
https://example.com?param1=value1&param2;=value2
```

让我们看看如何使用`String`名称-值对:

```java
public CloseableHttpResponse sendHttpRequest() {
    HttpGet httpGet = new HttpGet("https://example.com");
    URI uri = new URIBuilder(httpGet.getURI())
      .addParameter("param1", "value1")
      .addParameter("param2", "value2")
      .build();
   ((HttpRequestBase) httpGet).setURI(uri);
    CloseableHttpResponse response = client.execute(httpGet);
    client.close();
}
```

此外，我们可以使用`HttpClient`请求的`NameValuePair`列表:

```java
public CloseableHttpResponse sendHttpRequest() {
    List nameValuePairs = new ArrayList();
    nameValuePairs.add(new BasicNameValuePair("param1", "value1"));
    nameValuePairs.add(new BasicNameValuePair("param2", "value2"));
    HttpGet httpGet = new HttpGet("https://example.com");
    URI uri = new URIBuilder(httpGet.getURI())
      .addParameters(nameValuePairs)
      .build();
   ((HttpRequestBase) httpGet).setURI(uri);
    CloseableHttpResponse response = client.execute(httpGet);
    client.close();
}
```

类似地，`UriBuilder`可以用来给其他 HttpClient 请求方法添加参数。

## 3.使用`UrlEncodedFormEntity`向`HttpClient`请求添加参数

另一种方法是利用`UrlEncodedFormEntity`:

```java
public CloseableHttpResponse sendHttpRequest() {
    List nameValuePairs = new ArrayList();
    nameValuePairs.add(new BasicNameValuePair("param1", "value1"));
    nameValuePairs.add(new BasicNameValuePair("param2", "value2"));
    HttpPost httpPost = new HttpPost("https://example.com");
    httpPost.setEntity(new UrlEncodedFormEntity(nameValuePairs, StandardCharsets.UTF_8));
    CloseableHttpResponse response = client.execute(httpPost);
    client.close();
}
```

注意 **`UrlEncodedFormEntity`不能用于 GET 请求**，因为 GET 请求没有包含实体的主体。

## 4.结论

在这个例子中，我们展示了如何向 HttpClient 请求添加参数。此外，所有这些示例和代码片段的实现都可以在 [GitHub](https://web.archive.org/web/20220520160738/https://github.com/eugenp/tutorials/tree/master/httpclient-simple) 上找到。