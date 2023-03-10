# Apache http client–发送自定义 Cookie

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/httpclient-cookies>

## 1。概述

本教程将关注如何使用 Apache HttpClient 发送定制 Cookie。

如果你想更深入地了解 HttpClient 并学习其他很酷的东西，你可以直接进入主教程[http cl](/web/20220916100042/https://www.baeldung.com/httpclient-guide "Cool basic and more advanced things you can do with the HttpClient 4")[客户端](/web/20220916100042/https://www.baeldung.com/httpclient-guide "Cool basic and more advanced things you can do with the HttpClient 4") [教程](/web/20220916100042/https://www.baeldung.com/httpclient-guide "Cool basic and more advanced things you can do with the HttpClient 4")。

## 2。在 HttpClient 上配置 Cookie 管理

### 2.1。4.3 之后的 HttpClient】

在较新的 HttpClient 4.3 中，我们将利用负责构建和配置客户端的 fluent builder API。

首先，我们需要创建一个 cookie 存储，并在存储中设置我们的示例 cookie:

```java
BasicCookieStore cookieStore = new BasicCookieStore();
BasicClientCookie cookie = new BasicClientCookie("JSESSIONID", "1234");
cookie.setDomain(".github.com");
cookie.setPath("/");
cookieStore.addCookie(cookie);
```

然后，**我们可以使用`setDefaultCookieStore()`方法**在 HttpClient 上建立这个 cookie 存储，并发送请求:

```java
@Test
public void whenSettingCookiesOnTheHttpClient_thenCookieSentCorrectly() 
  throws ClientProtocolException, IOException {
    BasicCookieStore cookieStore = new BasicCookieStore();
    BasicClientCookie cookie = new BasicClientCookie("JSESSIONID", "1234");
    cookie.setDomain(".github.com");
    cookie.setPath("/");
    cookieStore.addCookie(cookie);
    HttpClient client = HttpClientBuilder.create().setDefaultCookieStore(cookieStore).build();

    final HttpGet request = new HttpGet("http://www.github.com");

    response = client.execute(request);
    assertThat(response.getStatusLine().getStatusCode(), equalTo(200));
}
```

一个非常重要的元素是在 cookie 上设置的`domain`–**如果没有设置正确的域，客户端根本不会发送 cookie** ！

此外，根据您使用的确切版本，您可能还需要设置:

```java
cookie.setAttribute(ClientCookie.DOMAIN_ATTR, "true"); 
```

### 2.2。4.3 之前的 HttpClient】

对于较旧版本的 http client(4.3 之前)cookie 存储直接设置在`HttpClient`上:

```java
@Test
public void givenUsingDeprecatedApi_whenSettingCookiesOnTheHttpClient_thenCorrect() 
  throws ClientProtocolException, IOException {
    BasicCookieStore cookieStore = new BasicCookieStore();
    BasicClientCookie cookie = new BasicClientCookie("JSESSIONID", "1234");
    cookie.setDomain(".github.com");
    cookie.setPath("/");
    cookieStore.addCookie(cookie);
    DefaultHttpClient client = new DefaultHttpClient();
    client.setCookieStore(cookieStore);

    HttpGet request = new HttpGet("http://www.github.com");

    response = client.execute(request);
    assertThat(response.getStatusLine().getStatusCode(), equalTo(200));
}
```

除了构建客户端的方式之外，与前面的示例没有其他区别。

## 3。在请求上设置 Cookie

如果不能在整个 HttpClient 上设置 cookie，我们可以使用`HttpContext`类单独配置带有 cookie 的请求:

```java
@Test
public void whenSettingCookiesOnTheRequest_thenCookieSentCorrectly() 
  throws ClientProtocolException, IOException {
    BasicCookieStore cookieStore = new BasicCookieStore();
    BasicClientCookie cookie = new BasicClientCookie("JSESSIONID", "1234");
    cookie.setDomain(".github.com");
    cookie.setPath("/");
    cookieStore.addCookie(cookie);
    instance = HttpClientBuilder.create().build();

    HttpGet request = new HttpGet("http://www.github.com");

    HttpContext localContext = new BasicHttpContext();
    localContext.setAttribute(HttpClientContext.COOKIE_STORE, cookieStore);
    // localContext.setAttribute(ClientContext.COOKIE_STORE, cookieStore); // before 4.3
    response = instance.execute(request, localContext);

    assertThat(response.getStatusLine().getStatusCode(), equalTo(200));
}
```

## 4。在低级请求上设置 Cookie

在 HTTP 请求上设置 cookie 的一个低级替代方法是将其设置为原始头:

```java
@Test
public void whenSettingCookiesOnARequest_thenCorrect() 
  throws ClientProtocolException, IOException {
    instance = HttpClientBuilder.create().build();
    HttpGet request = new HttpGet("http://www.github.com");
    request.setHeader("Cookie", "JSESSIONID=1234");

    response = instance.execute(request);

    assertThat(response.getStatusLine().getStatusCode(), equalTo(200));
}
```

这当然比使用内置的 cookie 支持更容易出错。例如，请注意，在这种情况下，我们不再设置域，这是不正确的。

## 5。结论

本文展示了如何使用 HttpClient 发送自定义的、用户控制的 Cookie。

注意，这不同于让 HttpClient 处理服务器设置的 cookies。相反，它是在低层次上手动控制客户端。

所有这些例子和代码片段的实现都可以在[我的 github 项目](https://web.archive.org/web/20220916100042/https://github.com/eugenp/tutorials/tree/master/httpclient-simple "Github Project exemplifying Live HttpClient 4.3 examples")中找到。