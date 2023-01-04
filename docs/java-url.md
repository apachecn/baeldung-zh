# Java URL 指南

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-url>

## 1。概述

在本文中，我们将探讨 Java 网络编程的底层操作。我们将更深入地了解 URL。

URL 是对网络上资源的引用或地址。简单地说，通过网络通信的 Java 代码可以使用`java.net.URL`类来表示资源的地址。

Java 平台附带内置的网络支持，打包在`java.net`包中:

```
import java.net.*;
```

## 2。创建一个网址

让我们首先创建一个`java.net.URL`对象，方法是使用它的构造函数并传入一个表示人类可读的资源地址的字符串:

```
URL url = new URL("/a-guide-to-java-sockets");
```

我们刚刚创建了一个绝对 URL 对象。该地址包含到达所需资源所需的所有部分。

我们还可以创建**一个相对 URL**；假设我们有代表 Baeldung 主页的 URL 对象:

```
URL home = new URL("http://baeldung.com");
```

接下来，让我们创建一个指向我们已经知道的资源的新 URL 我们将使用另一个构造函数，它接受一个现有的 URL 和一个相对于该 URL 的资源名:

```
URL url = new URL(home, "a-guide-to-java-sockets");
```

我们现在已经创建了一个相对于`home`的新 URL 对象`url`；所以相对 URL 只在基本 URL 的上下文中有效。

我们可以在一个测试中看到这一点:

```
@Test
public void givenBaseUrl_whenCreatesRelativeUrl_thenCorrect() {
    URL baseUrl = new URL("http://baeldung.com");
    URL relativeUrl = new URL(baseUrl, "a-guide-to-java-sockets");

    assertEquals("http://baeldung.com/a-guide-to-java-sockets", 
      relativeUrl.toString());
}
```

然而，如果检测到相对 URL 在其组成部分中是绝对的，则`baseURL`被忽略:

```
@Test
public void givenAbsoluteUrl_whenIgnoresBaseUrl_thenCorrect() {
    URL baseUrl = new URL("http://baeldung.com");
    URL relativeUrl = new URL(
      baseUrl, "/a-guide-to-java-sockets");

    assertEquals("http://baeldung.com/a-guide-to-java-sockets", 
      relativeUrl.toString());
}
```

最后，我们可以通过调用另一个接受 URL 字符串组成部分的构造函数来创建一个 URL。在介绍完 URL 组件之后，我们将在下一节中介绍这一点。

## 3。URL 组件

URL 由几个部分组成——我们将在这一部分探讨。

让我们首先来看一下协议标识符和资源之间的分隔——这两个部分由一个冒号后跟两个正斜杠分隔，即`://.`

如果我们有一个像`http://baeldung.com`这样的 URL，那么分隔符之前的部分`http,`是协议标识符，而后面的部分是资源名`baeldung.com`。

让我们来看看`URL`对象公开的 API。

### 3.1。协议

为了检索**协议**，我们使用`getProtocol()`方法:

```
@Test
public void givenUrl_whenCanIdentifyProtocol_thenCorrect(){
    URL url = new URL("http://baeldung.com");

    assertEquals("http", url.getProtocol());
}
```

### 3.2。端口

为了得到**端口**，我们使用`getPort()`方法:

```
@Test
public void givenUrl_whenGetsDefaultPort_thenCorrect(){
    URL url = new URL("http://baeldung.com");

    assertEquals(-1, url.getPort());
    assertEquals(80, url.getDefaultPort());
}
```

请注意，该方法检索显式定义的端口。如果没有显式定义端口，它将返回-1。

因为 HTTP 通信默认使用端口 80，所以没有定义端口。

这里有一个例子，我们有一个明确定义的端口:

```
@Test
public void givenUrl_whenGetsPort_thenCorrect(){
    URL url = new URL("http://baeldung.com:8090");

    assertEquals(8090, url.getPort());
}
```

### 3.3。主持人

**主机**是资源名称的一部分，从`://`分隔符之后开始，以域名扩展名结束，在我们的例子中是`.com`。

我们调用`getHost()`方法来检索主机名:

```
@Test
public void givenUrl_whenCanGetHost_thenCorrect(){
    URL url = new URL("http://baeldung.com");

    assertEquals("baeldung.com", url.getHost());
}
```

### 3.4。文件名

URL 中主机名后面的内容被称为资源的**文件名。它可以包含路径和查询参数，也可以只包含文件名:**

```
@Test
public void givenUrl_whenCanGetFileName_thenCorrect1() {
    URL url = new URL("http://baeldung.com/guidelines.txt");

    assertEquals("/guidelines.txt", url.getFile());
}
```

假设 Baeldung 在 URL `/articles?topic=java&version;=8`下有 java 8 的文章。主机名后面的所有内容都是文件名:

```
@Test
public void givenUrl_whenCanGetFileName_thenCorrect2() {
    URL url = new URL("http://baeldung.com/articles?topic=java&version;=8");

    assertEquals("/articles?topic=java&version;=8", url.getFile());
}
```

### 3.5。路径参数

我们还可以只检查**路径**参数，在我们的例子中是`/articles`:

```
@Test
public void givenUrl_whenCanGetPathParams_thenCorrect() {
    URL url = new URL("http://baeldung.com/articles?topic=java&version;=8");

    assertEquals("/articles", url.getPath());
}
```

### 3.6。查询参数

同样，我们可以检查**的查询参数**，也就是`topic=java&version;=8`:

```
@Test
public void givenUrl_whenCanGetQueryParams_thenCorrect() {
    URL url = new URL("http://baeldung.com/articles?topic=java<em>&version=8</em>");

    assertEquals("topic=java<em>&version=8</em>", url.getQuery());
}
```

## 4。创建包含组件部分的 URL

既然我们已经看到了不同的 URL 组件以及它们在形成完整的资源地址中的位置，我们可以看看通过传入组件部分来创建 URL 对象的另一种方法。

第一个构造函数分别接受协议、主机名和文件名:

```
@Test
public void givenUrlComponents_whenConstructsCompleteUrl_thenCorrect() {
    String protocol = "http";
    String host = "baeldung.com";
    String file = "/guidelines.txt";
    URL url = new URL(protocol, host, file);

    assertEquals("http://baeldung.com/guidelines.txt", url.toString());
}
```

请记住 filename 在此上下文中的含义，下面的测试应该会使它更清楚:

```
@Test
public void givenUrlComponents_whenConstructsCompleteUrl_thenCorrect2() {
    String protocol = "http";
    String host = "baeldung.com";
    String file = "/articles?topic=java&version;=8";
    URL url = new URL(protocol, host, file);

    assertEquals("http://baeldung.com/articles?topic=java&version;=8", url.toString());
}
```

第二个构造函数分别接受协议、主机名、端口号和文件名:

```
@Test
public void givenUrlComponentsWithPort_whenConstructsCompleteUrl_
  thenCorrect() {
    String protocol = "http";
    String host = "baeldung.com";
    int port = 9000;
    String file = "/guidelines.txt";
    URL url = new URL(protocol, host, port, file);

    assertEquals(
      "http://baeldung.com:9000/guidelines.txt", url.toString());
}
```

## 5。结论

在本教程中，我们介绍了`URL`类，并展示了如何在 Java 中使用它以编程方式访问网络资源。

与往常一样，本文的完整源代码和所有代码片段都可以在 [GitHub 项目](https://web.archive.org/web/20220817160131/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-networking)中找到。