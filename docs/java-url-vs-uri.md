# URL 和 URI 的区别

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-url-vs-uri>

## 1。概述

在这篇短文中，我们将看看 URIs 和 URL 之间的主要区别，并实现一些例子来突出这些区别。

## 2。URI 和网址

了解它们的定义后，它们之间的区别就很明显了:

*   **统一资源标识符(URI)**—允许完整识别任何抽象或物理资源的字符序列
*   **统一资源定位符(URL)**—URI 的一个子集，除了标识资源在哪里可用之外，还描述了访问资源的主要机制

**现在我们可以得出结论，每个 URL 都是一个 URI** ，但反过来就不正确了，我们稍后会看到。

### 2.1。语法

每个 URI，不管它是不是 URL，都遵循特定的形式:

```java
scheme:[//authority][/path][?query][#fragment]
```

其中每个部分描述如下:

*   `**scheme**`—对于 URL，是用于访问资源的协议的名称，对于其他 URIs，是指在该方案内分配标识符的规范
*   **`authority`**—由用户认证信息、主机和可选端口组成的可选部分
*   `**path**`——用于标识其`scheme`和`authority`范围内的资源
*   `**query**`—附加数据，与`path,`一起用于识别资源。对于 URL，这是查询字符串
*   `**fragment**`—资源特定部分的可选标识符

**为了容易识别某个特定的 URI 是否也是 URL，我们可以检查它的方案**。每个 URL 都必须以这些方案中的任何一个开始:`ftp`、`http`、`https,`、`gopher`、`mailto`、`news`、`nntp`、`telnet`、`wais`、`file`或`prospero`。如果不是以它开头，那就不是 URL。

现在我们知道了语法，让我们看一些例子。以下是 URIs 的列表，其中只有前三个是 URL:

```java
ftp://ftp.is.co.za/rfc/rfc1808.txt
https://tools.ietf.org/html/rfc3986
mailto:[[email protected]](/web/20221205131345/http://www.baeldung.com/cdn-cgi/l/email-protection)

tel:+1-816-555-1212
urn:oasis:names:docbook:dtd:xml:4.1
urn:isbn:1234567890
```

## 3。URI 和 URL Java API 的区别

在这一节中，我们将通过例子展示 Java 提供的`URI`和`URL`类之间的主要区别。

### 3.1。实例化

创建`URI`和`URL`实例非常相似，两个类都提供了几个接受其大部分内容的构造函数，然而，只有`URI`类有一个指定语法所有部分的构造函数:

```java
@Test
public void whenCreatingURIs_thenSameInfo() throws Exception {
    URI firstURI = new URI(
      "somescheme://theuser:[[email protected]](/web/20221205131345/http://www.baeldung.com/cdn-cgi/l/email-protection):80"
      + "/some/path?thequery#somefragment");

    URI secondURI = new URI(
      "somescheme", "theuser:thepassword", "someuthority", 80,
      "/some/path", "thequery", "somefragment");

    assertEquals(firstURI.getScheme(), secondURI.getScheme());
    assertEquals(firstURI.getPath(), secondURI.getPath());
}

@Test
public void whenCreatingURLs_thenSameInfo() throws Exception {
    URL firstURL = new URL(
      "http://theuser:[[email protected]](/web/20221205131345/http://www.baeldung.com/cdn-cgi/l/email-protection):80"
      + "/path/to/file?thequery#somefragment");
    URL secondURL = new URL("http", "somehost", 80, "/path/to/file");

    assertEquals(firstURL.getHost(), secondURL.getHost());
    assertEquals(firstURL.getPath(), secondURL.getPath());
}
```

`URI`类还提供了一个实用方法来创建一个不抛出检查异常的新实例:

```java
@Test
public void whenCreatingURI_thenCorrect() {
    URI uri = URI.create("urn:isbn:1234567890");

    assertNotNull(uri);
}
```

`URL`类没有提供这样的方法。

由于 URL 必须以前面提到的方案之一开头，因此尝试用不同的方案创建对象将导致异常:

```java
@Test(expected = MalformedURLException.class)
public void whenCreatingURLs_thenException() throws Exception {
    URL theURL = new URL("otherprotocol://somehost/path/to/file");

    assertNotNull(theURL);
}
```

两个类中都有其他构造函数，要发现它们，请参考 [URI](https://web.archive.org/web/20221205131345/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/net/URI.html) 和 [URL](https://web.archive.org/web/20221205131345/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/net/URL.html) 文档。

### 3.2。在 URI 和 URL 实例之间转换

URI 和 URL 之间的转换非常简单:

```java
@Test
public void givenObjects_whenConverting_thenCorrect()
  throws MalformedURLException, URISyntaxException {
    String aURIString = "http://somehost:80/path?thequery";
    URI uri = new URI(aURIString);
    URL url = new URL(aURIString);

    URL toURL = uri.toURL();
    URI toURI = url.toURI();

    assertNotNull(url);
    assertNotNull(uri);
    assertEquals(toURL.toString(), toURI.toString());
}
```

但是，尝试转换非 URL URI 会导致异常:

```java
@Test(expected = MalformedURLException.class)
public void givenURI_whenConvertingToURL_thenException()
  throws MalformedURLException, URISyntaxException {
    URI uri = new URI("somescheme://someauthority/path?thequery");

    URL url = uri.toURL();

    assertNotNull(url);
}
```

### 3.3。打开远程连接

因为 URL 是对远程资源的有效引用，所以 Java 提供了打开到该资源的连接并获取其内容的方法:

```java
@Test
public void givenURL_whenGettingContents_thenCorrect()
  throws MalformedURLException, IOException {
    URL url = new URL("http://courses.baeldung.com");

    String contents = IOUtils.toString(url.openStream());

    assertTrue(contents.contains("<!DOCTYPE html>"));
}
```

需要注意的是，`URL` equals()和 hashcode()函数的实现可能会触发`DNS`命名服务来解析 IP 地址。这是不一致的，会根据网络连接给出不同的结果，并且需要很长时间来运行。已知该实现与虚拟宿主不兼容，不应使用。我们建议使用`URI`来代替。

## 4T2。结论

在这篇简短的文章中，我们给出了几个例子来展示 Java 中的`URI`和`URL`之间的区别。

我们强调了创建两个对象的实例以及将一个对象转换为另一个对象时的差异。我们还展示了一个`URL`拥有打开指向资源的远程连接的方法。

和往常一样，本文的完整源代码可以在 Github 上找到[。](https://web.archive.org/web/20221205131345/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-networking)