# Apache http client–遵循 POST 的重定向

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/httpclient-redirect-on-http-post>

## 1。概述

这个快速教程将展示如何配置 Apache HttpClient 来自动跟踪 POST 请求的重定向。

如果你想更深入地了解 HttpClient 并学习其他很酷的东西，你可以直接阅读 HttpClient 教程**[。](/web/20220822070118/https://www.baeldung.com/httpclient-guide "Cool basic and more advanced things you can do with the HttpClient 4")**

默认情况下，只自动跟踪导致重定向的 GET 请求。如果 POST 请求用`HTTP 301 Moved Permanently`或`302 Found`–**回答，重定向不会自动跟随**。

这由 [HTTP RFC 2616](https://web.archive.org/web/20220822070118/https://www.w3.org/Protocols/rfc2616/rfc2616-sec10.html#sec10.3 "Redirection 3xx") 指定:

> 如果 301 状态代码是为响应 GET 或 HEAD 之外的请求而接收的，则用户代理不能自动重定向请求，除非用户可以确认，因为这可能会改变发出请求的条件。

当然，在一些用例中，我们需要改变这种行为，放松严格的 HTTP 规范。

首先，让我们检查默认行为:

```java
@Test
public void givenPostRequest_whenConsumingUrlWhichRedirects_thenNotRedirected() 
  throws ClientProtocolException, IOException {
    HttpClient instance = HttpClientBuilder.create().build();
    HttpResponse response = instance.execute(new HttpPost("http://t.co/I5YYd9tddw"));
    assertThat(response.getStatusLine().getStatusCode(), equalTo(301));
}
```

正如你所看到的，**默认情况下重定向后面没有**，我们得到了`301 Status Code`。

## 2。在 HTTP POST 上重定向

### 2.1。对于 HttpClient 4.3 和之后的版本

在 HttpClient 4.3 中，为客户机的创建和配置引入了更高级别的 API:

```java
@Test
public void givenRedirectingPOST_whenConsumingUrlWhichRedirectsWithPOST_thenRedirected() 
  throws ClientProtocolException, IOException {
    HttpClient instance = 
      HttpClientBuilder.create().setRedirectStrategy(new LaxRedirectStrategy()).build();
    HttpResponse response = instance.execute(new HttpPost("http://t.co/I5YYd9tddw"));
    assertThat(response.getStatusLine().getStatusCode(), equalTo(200));
}
```

注意**`HttpClientBuilder`现在是流畅 API** 的起点，它允许以比以前更可读的方式对客户端进行完全配置。

### 2.2。适用于 httpclient 4.2

在以前版本的 HttpClient (4.2)中，我们可以直接在客户机上配置重定向策略:

```java
@SuppressWarnings("deprecation")
@Test
public void givenRedirectingPOST_whenConsumingUrlWhichRedirectsWithPOST_thenRedirected() 
  throws ClientProtocolException, IOException {
    DefaultHttpClient client = new DefaultHttpClient();
    client.setRedirectStrategy(new LaxRedirectStrategy());

    HttpResponse response = client.execute(new HttpPost("http://t.co/I5YYd9tddw"));
    assertThat(response.getStatusLine().getStatusCode(), equalTo(200));
}
```

注意，现在有了新的`LaxRedirectStrategy`，HTTP 限制放松了，**重定向也在 POST 之后**，导致了一个`200 OK`状态代码。

### 2.3。HttpClient 4.2 之前的版本

在 HttpClient 4.2 之前，`LaxRedirectStrategy`类并不存在，所以我们需要自己开发:

```java
@Test
public void givenRedirectingPOST_whenConsumingUrlWhichRedirectsWithPOST_thenRedirected() 
  throws ClientProtocolException, IOException {
    DefaultHttpClient client = new DefaultHttpClient();
    client.setRedirectStrategy(new DefaultRedirectStrategy() {
        /** Redirectable methods. */
        private String[] REDIRECT_METHODS = new String[] { 
            HttpGet.METHOD_NAME, HttpPost.METHOD_NAME, HttpHead.METHOD_NAME 
        };

        @Override
        protected boolean isRedirectable(String method) {
            for (String m : REDIRECT_METHODS) {
                if (m.equalsIgnoreCase(method)) {
                    return true;
                }
            }
            return false;
        }
    });

    HttpResponse response = client.execute(new HttpPost("http://t.co/I5YYd9tddw"));
    assertThat(response.getStatusLine().getStatusCode(), equalTo(200));
}
```

## 3。结论

这个快速指南演示了如何配置任何版本的 Apache HttpClient，使其也能遵循 HTTP POST 请求的重定向——放宽了严格的 HTTP 标准。

所有这些示例和代码片段的实现都可以在[我的 github 项目](https://web.archive.org/web/20220822070118/https://github.com/eugenp/tutorials/tree/master/apache-httpclient "Github Project exemplifying Live HttpClient 4.3 examples") 中找到——这是一个基于 Eclipse 的项目，所以它应该很容易导入和运行。