# 检查 Java 中是否存在 URL

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-check-url-exists>

## 1。概述

在本教程中，我们将通过一个 Java 示例来看看如何使用`GET`和`HEAD` [HTTP 方法](/web/20221205154733/https://www.baeldung.com/java-http-request)来检查 URL 是否存在。

## 2。网址存在

在编程中可能会出现这样的情况:在访问某个资源之前，我们必须知道它是否存在于给定的 URL 中，或者我们甚至需要检查 URL 以了解资源的健康状况。

我们通过查看资源的响应代码来决定资源在 URL 上的存在。通常**我们会寻找一个`200`，这意味着“ok”并且请求已经成功。**

## 3。使用获取请求

首先，要发出一个`GET`请求，我们可以创建一个`java.net.URL`的实例，并将我们想要访问的 URL 作为构造函数参数传递。之后，我们只需打开连接并获得响应代码:

```java
URL url = new URL("http://www.example.com");
HttpURLConnection huc = (HttpURLConnection) url.openConnection();

int responseCode = huc.getResponseCode();

Assert.assertEquals(HttpURLConnection.HTTP_OK, responseCode);
```

当在 URL 上找不到资源时，我们得到一个`404` 响应代码:

```java
URL url = new URL("http://www.example.com/xyz"); 
HttpURLConnection huc = (HttpURLConnection) url.openConnection();

int responseCode = huc.getResponseCode();

Assert.assertEquals(HttpURLConnection.HTTP_NOT_FOUND, responseCode);
```

由于**在`HttpURLConnection`中的默认 HTTP 方法是`GET`** ，所以我们没有在本节的例子中设置请求方法。我们将在下一节看到如何覆盖默认方法。

## 4。使用头部请求

**HEAD 也是一个 HTTP 请求方法，除了不返回响应体之外，它与 GET 完全相同。**

如果使用 GET 方法请求相同的资源，它将获得响应代码以及我们将收到的响应头。

要创建 HEAD 请求，我们只需在获得响应代码之前将请求方法设置为 HEAD:

```java
URL url = new URL("http://www.example.com");
HttpURLConnection huc = (HttpURLConnection) url.openConnection();
huc.setRequestMethod("HEAD");

int responseCode = huc.getResponseCode();

Assert.assertEquals(HttpURLConnection.HTTP_OK, responseCode);
```

同样，当在 URL 上找不到资源时:

```java
URL url = new URL("http://www.example.com/xyz");
HttpURLConnection huc = (HttpURLConnection) url.openConnection();
huc.setRequestMethod("HEAD");

int responseCode = huc.getResponseCode();

Assert.assertEquals(HttpURLConnection.HTTP_NOT_FOUND, responseCode);
```

**通过使用 HEAD 方法，从而不下载响应体，我们减少了响应时间和带宽，并提高了性能**。

尽管大多数现代服务器都支持 HEAD 方法，但是一些自主开发的或遗留的服务器可能会拒绝 HEAD 方法，并出现无效的方法类型错误。所以，我们应该谨慎使用头法。

## 5。跟随重定向

最后，当寻找 URL 存在时，不跟随重定向可能是个好主意。但这也取决于我们寻找 URL 的原因。

当 URL 被移动时，服务器可以使用 3xx 响应代码将请求重定向到新的 URL。**默认是跟随重定向**。我们可以根据需要选择跟随或忽略重定向。

为此，我们可以覆盖所有`HttpURLConnection`的默认值`followRedirects`:

```java
URL url = new URL("http://www.example.com");
HttpURLConnection.setFollowRedirects(false);
HttpURLConnection huc = (HttpURLConnection) url.openConnection();

int responseCode = huc.getResponseCode();

Assert.assertEquals(HttpURLConnection.HTTP_OK, responseCode);
```

或者，我们可以通过使用`setInstanceFollowRedirects()`方法禁用单个连接的以下重定向:

```java
URL url = new URL("http://www.example.com");
HttpURLConnection huc = (HttpURLConnection) url.openConnection();
huc.setInstanceFollowRedirects(false);

int responseCode = huc.getResponseCode();

Assert.assertEquals(HttpURLConnection.HTTP_OK, responseCode);
```

## 6。结论

在本文中，我们研究了如何检查响应代码以发现 URL 的可用性。此外，我们还研究了如何使用 HEAD 方法来节省带宽并获得更快的响应。

本教程中使用的代码示例可以在我们的 [GitHub 项目](https://web.archive.org/web/20221205154733/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-networking-2)中找到。