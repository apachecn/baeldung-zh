# 使用 Apache HttpClient 扩展缩短的 URL

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/apache-httpclient-expand-url>

## 1。概述

在本文中，我们将展示如何使用`HttpClient`**扩展 URL。**

一个简单的例子是，当原始 URL 被缩短了一次(T2)，这是通过像 T0 这样的服务。

一个更复杂的例子是，当 URL 被不同的服务多次缩短时，需要多次传递才能到达原始的完整 URL。

如果你想更深入地了解 HttpClient 并学习其他很酷的东西，你可以直接阅读 HttpClient 教程**[。](/web/20220813064331/https://www.baeldung.com/httpclient-guide "Cool basic and more advanced things you can do with the HttpClient 4")**

## 2。展开 URL 一次

让我们从简单的开始，扩展一个只通过一次 shorten URL 服务的 URL。

我们首先需要的是一个 HTTP 客户端，它不会自动跟随重定向:

```java
CloseableHttpClient client = 
  HttpClientBuilder.create().disableRedirectHandling().build();
```

这是必要的，因为我们需要手动拦截重定向响应并从中提取信息。

我们首先向缩短的 URL 发送一个请求——我们得到的响应将是一个`301 Moved Permanently`。

然后，我们需要**提取`Location`头**指向下一个，在本例中是最后一个 URL:

```java
public String expandSingleLevel(String url) throws IOException {
    HttpHead request = null;
    try {
        request = new HttpHead(url);
        HttpResponse httpResponse = client.execute(request);

        int statusCode = httpResponse.getStatusLine().getStatusCode();
        if (statusCode != 301 && statusCode != 302) {
            return url;
        }
        Header[] headers = httpResponse.getHeaders(HttpHeaders.LOCATION);
        Preconditions.checkState(headers.length == 1);
        String newUrl = headers[0].getValue();
        return newUrl;
    } catch (IllegalArgumentException uriEx) {
        return url;
    } finally {
        if (request != null) {
            request.releaseConnection();
        }
    }
}
```

最后，用一个“未缩短”的 URL 进行一个简单的现场测试:

```java
@Test
public final void givenShortenedOnce_whenUrlIsExpanded_thenCorrectResult() throws IOException {
    final String expectedResult = "https://www.baeldung.com/rest-versioning";
    final String actualResult = expandSingleLevel("http://bit.ly/3LScTri");
    assertThat(actualResult, equalTo(expectedResult));
}
```

## 3。处理多个 URL 级别

短网址的问题是它们可能被完全不同的服务缩短了很多倍。扩展这样的 URL 将需要多次传递才能到达原始 URL。

我们将应用之前定义的`expandSingleLevel`原语操作来简单地**遍历所有中间 URL，并到达最终目标**:

```java
public String expand(String urlArg) throws IOException {
    String originalUrl = urlArg;
    String newUrl = expandSingleLevel(originalUrl);
    while (!originalUrl.equals(newUrl)) {
        originalUrl = newUrl;
        newUrl = expandSingleLevel(originalUrl);
    }
    return newUrl;
}
```

现在，有了扩展多级 URL 的新机制，让我们定义一个测试并开始工作:

```java
@Test
public final void givenShortenedMultiple_whenUrlIsExpanded_thenCorrectResult() throws IOException {
    final String expectedResult = "https://www.baeldung.com/rest-versioning";
    final String actualResult = expand("http://t.co/e4rDDbnzmk");
    assertThat(actualResult, equalTo(expectedResult));
}
```

这一次，短 URL——`http://t.co/e4rDDbnzmk`——实际上被缩短了两次——一次通过`bit.ly`，第二次通过`t.co`服务——被正确地扩展为原始 URL。

## 4。检测重定向循环

最后，有些 URL 无法展开，因为它们形成了重定向循环。这种类型的问题会被`HttpClient`检测到，但是因为我们关闭了重定向的自动跟踪，它就不再检测到了。

URL 扩展机制的最后一步是检测重定向循环，并在发生这种循环时快速失败。

为了使其有效，我们需要一些来自我们之前定义的`expandSingleLevel`方法的额外信息——主要是，我们还需要返回响应的状态代码和 URL。

由于 java 不支持多个返回值，我们将**将信息包装在`org.apache.commons.lang3.tuple.Pair`对象**中——该方法的新签名现在将是:

```java
public Pair<Integer, String> expandSingleLevelSafe(String url) throws IOException {
```

最后，让我们在主扩展机制中包含重定向循环检测:

```java
public String expandSafe(String urlArg) throws IOException {
    String originalUrl = urlArg;
    String newUrl = expandSingleLevelSafe(originalUrl).getRight();
    List<String> alreadyVisited = Lists.newArrayList(originalUrl, newUrl);
    while (!originalUrl.equals(newUrl)) {
        originalUrl = newUrl;
        Pair<Integer, String> statusAndUrl = expandSingleLevelSafe(originalUrl);
        newUrl = statusAndUrl.getRight();
        boolean isRedirect = statusAndUrl.getLeft() == 301 || statusAndUrl.getLeft() == 302;
        if (isRedirect && alreadyVisited.contains(newUrl)) {
            throw new IllegalStateException("Likely a redirect loop");
        }
        alreadyVisited.add(newUrl);
    }
    return newUrl;
}
```

就是这样——`expandSafe`机制能够通过任意数量的 URL 缩短服务来扩展 URL，同时在重定向循环上正确地快速失败。

## 5。结论

本教程讨论了如何使用 Apache `HttpClient`在 java 中**扩展短 URL。**

我们从一个 URL 只缩短一次的简单用例开始，然后实现了一个更通用的机制，能够处理多级重定向并检测过程中的重定向循环。

GitHub 上的[提供了这些例子的实现。](https://web.archive.org/web/20220813064331/https://github.com/eugenp/tutorials/tree/master/apache-httpclient-2 "URL expansion mechanism - all in an example test")