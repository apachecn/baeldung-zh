# 如何从 Apache HttpClient 响应中获取 Cookies

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-apache-httpclient-cookies>

## 1.概观

在这个简短的教程中，我们将看到如何从 Apache HttpClient 响应中获取 cookies。

首先，我们将展示如何发送带有`HttpClient`请求的定制 cookie。然后，我们再看如何从响应中获取。

请注意，这里给出的代码示例是基于 **HttpClient 4.3.x 和更高版本的，**所以它们不能在旧版本的 API 上工作。

## 2.在请求中发送 Cookies

在从响应中获取 cookie 之前，我们需要创建它并在请求中发送它:

```java
BasicCookieStore cookieStore = new BasicCookieStore();
BasicClientCookie cookie = new BasicClientCookie("custom_cookie", "test_value");
cookie.setDomain("baeldung.com");
cookie.setAttribute(ClientCookie.DOMAIN_ATTR, "true");
cookie.setPath("/");
cookieStore.addCookie(cookie);

HttpClientContext context = HttpClientContext.create();
context.setAttribute(HttpClientContext.COOKIE_STORE, cookieStore);

try (CloseableHttpClient httpClient = HttpClients.createDefault()) {
    try (CloseableHttpResponse response = httpClient.execute(new HttpGet("http://www.baeldung.com/"), context)) {
        //do something with the response
    }
}
```

首先，我们创建一个基本的 cookie store 和一个基本的 [cookie](/web/20220525131433/https://www.baeldung.com/httpclient-4-cookies) ，名称为`custom_cookie`，值为`test_value`。然后，我们创建一个`HttpClientContext`实例来保存 cookie 存储。最后，我们将创建的上下文作为参数传递给`execute()`方法。

## 3.访问 Cookies

现在我们已经在请求中发送了一个定制的 cookie，让我们看看如何从响应中读取它:

```java
HttpClientContext context = HttpClientContext.create();
context.setAttribute(HttpClientContext.COOKIE_STORE, createCustomCookieStore());

try (CloseableHttpClient httpClient = HttpClients.createDefault()) {
    try (CloseableHttpResponse response = httpClient.execute(new HttpGet(SAMPLE_URL), context)) {
        CookieStore cookieStore = context.getCookieStore();
        Cookie customCookie = cookieStore.getCookies()
          .stream()
          .peek(cookie -> log.info("cookie name:{}", cookie.getName()))
          .filter(cookie -> "custom_cookie".equals(cookie.getName()))
          .findFirst()
          .orElseThrow(IllegalStateException::new);

          assertEquals("test_value", customCookie.getValue());
    }
} 
```

为了从响应中获取自定义 cookie，**我们必须首先从上下文**中获取 cookie 存储。然后，我们使用`getCookies` 方法来获取 cookies 列表。然后我们可以利用 Java [流](/web/20220525131433/https://www.baeldung.com/java-streams)来遍历它并搜索我们的 cookie。此外，我们记录了商店中的所有 cookie 名称:

```java
[main] INFO  c.b.h.c.HttpClientGettingCookieValueTest - cookie name:__cfduid
[main] INFO  c.b.h.c.HttpClientGettingCookieValueTest - cookie name:custom_cookie
```

## 4.结论

在本文中，我们学习了如何从 Apache HttpClient 响应中获取 cookies。

和往常一样，代码可以在 GitHub 的[上获得。](https://web.archive.org/web/20220525131433/https://github.com/eugenp/tutorials/tree/master/apache-httpclient-2)