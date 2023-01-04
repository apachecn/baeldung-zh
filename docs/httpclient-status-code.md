# Apache http client–获取状态代码

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/httpclient-status-code>

## 1。概述

在这篇非常简短的教程中，我将展示如何使用 HttpClient 获取和验证 HTTP 响应的 StatusCode。

如果你想更深入地了解 HttpClient 并学习其他很酷的东西，你可以直接阅读 HttpClient 教程**[。](/web/20220917100509/https://www.baeldung.com/httpclient-guide "Cool basic and more advanced things you can do with the HttpClient 4")**

## 2。从 Http 响应中检索状态代码

在发送 Http 请求之后——我们得到了一个`org.apache.http.HttpResponse`的实例——它允许我们访问响应的状态行，并隐式地访问状态代码:

```java
response.getStatusLine().getStatusCode()
```

使用这个，我们可以**验证我们从服务器收到的代码确实是正确的**:

```java
@Test
public void givenGetRequestExecuted_whenAnalyzingTheResponse_thenCorrectStatusCode() 
  throws ClientProtocolException, IOException {
    HttpClient client = HttpClientBuilder.create().build();    
    HttpResponse response = client.execute(new HttpGet(SAMPLE_URL));
    int statusCode = response.getStatusLine().getStatusCode();
    assertThat(statusCode, equalTo(HttpStatus.SC_OK));
}
```

注意，我们使用的是**预定义的状态码**，也可以通过`org.apache.http.HttpStatus`从库中获得。

## 3。结论

这个非常简单的例子展示了如何使用 Apache HttpClient 检索和处理状态代码。

所有这些示例和代码片段的实现都可以在[我的 github 项目](https://web.archive.org/web/20220917100509/https://github.com/eugenp/tutorials/tree/master/httpclient-simple "Github Project exemplifying Live HttpClient 4.3 examples") 中找到——这是一个基于 Eclipse 的项目，所以它应该很容易导入和运行。