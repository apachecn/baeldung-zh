# 如何给 Jsoup 添加代理支持？

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-jsoup-proxy>

## 1.概观

在本教程中，我们将看看**如何给[j 组](/web/20221208143841/https://www.baeldung.com/java-with-jsoup)添加代理支持。**

## 2.使用代理的常见原因

有两个主要原因，我们可能希望在 Jsoup 中使用代理服务器。

### 2.1.组织代理背后的用法

对于组织来说，让代理控制互联网访问是很常见的。如果我们试图**通过代理本地网络访问 Jsoup，我们将得到一个异常**:

```
java.net.SocketTimeoutException: connect timed out
```

当我们看到这个错误时，我们需要在试图访问网络之外的任何 URL 之前**为 Jsoup 设置一个代理。**

### 2.2.防止 IP 阻止

Jsoup 使用代理的另一个常见原因是防止网站阻止 IP 地址。

换句话说，使用一个代理(或多个旋转代理)允许我们更可靠地解析 HTML，减少了由于我们的 IP 地址被阻塞或禁止而导致代码停止工作的机会。

## 3.设置

当使用 Maven 时，我们需要将 Jsoup 依赖关系添加到我们的`pom.xml`中:

```
<dependency>
    <groupId>org.jsoup</groupId>
    <artifactId>jsoup</artifactId>
    <version>1.13.1</version>
</dependency>
```

在 Gradle 中，我们必须在`build.gradle`中声明我们的依赖关系:

```
compile 'org.jsoup:jsoup:1.13.1'
```

## 4.通过主机和端口属性添加代理支持

向 Jsoup 添加代理支持非常简单。**我们需要做的就是在构建`Connection`对象时调用`[proxy(String, int)](https://web.archive.org/web/20221208143841/https://jsoup.org/apidocs/org/jsoup/Connection.html#proxy(java.lang.String,int))`方法**:

```
Jsoup.connect("https://spring.io/blog")
  .proxy("127.0.0.1", 1080)
  .get();
```

在这里，我们设置用于该请求的 HTTP 代理，第一个参数表示代理主机名，第二个参数表示代理端口。

## 5.通过`Proxy`对象添加代理支持

或者，使用`Proxy`类将代理添加到 Jsoup，我们**调用`Connection`对象的`[proxy(java.net.Proxy)](https://web.archive.org/web/20221208143841/https://jsoup.org/apidocs/org/jsoup/Connection.html#proxy(java.net.Proxy))`方法:**

```
Proxy proxy = new Proxy(Proxy.Type.HTTP, 
  new InetSocketAddress("127.0.0.1", 1080));

Jsoup.connect("https://spring.io/blog")
  .proxy(proxy)
  .get();
```

该方法采用一个由代理类型(通常是 HTTP 类型)和一个 [`InetSocketAddress`](https://web.archive.org/web/20221208143841/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/net/InetSocketAddress.html) (一个分别包装代理主机名和端口的类)组成的`Proxy`对象。

## 6.结论

在本教程中，我们探索了向 Jsoup 添加代理支持的两种不同方式。

首先，我们学习了如何使用带有主机和端口属性的 Jsoup 方法来完成这项工作。其次，我们学习了如何使用一个`Proxy`对象作为参数来获得相同的结果。

和往常一样，代码样本在 GitHub 上可用 [。](https://web.archive.org/web/20221208143841/https://github.com/eugenp/tutorials/tree/master/jsoup)