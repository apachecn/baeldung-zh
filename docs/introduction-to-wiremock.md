# WireMock 简介

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/introduction-to-wiremock>

## 1。概述

WireMock 是一个用于存根和模仿 web 服务的库。它构造了一个 HTTP 服务器，我们可以像连接实际的 web 服务一样连接它。

当一个 [WireMock](https://web.archive.org/web/20220812052320/http://wiremock.org/) 服务器运行时，我们可以设置期望值，调用服务，然后验证它的行为。

## 2。Maven 依赖关系

为了利用 WireMock 库，我们需要在 POM 中包含这种依赖性:

```
<dependency>
    <groupId>com.github.tomakehurst</groupId>
    <artifactId>wiremock</artifactId>
    <version>1.58</version>
    <scope>test</scope>
</dependency>
```

## 3。以编程方式管理的服务器

本节将介绍如何手动配置 WireMock 服务器，也就是说，在没有 JUnit 自动配置支持的情况下。我们用一个非常简单的存根来演示用法。

### 3.1。服务器设置

首先，我们实例化一个 WireMock 服务器:

```
WireMockServer wireMockServer = new WireMockServer(String host, int port);
```

如果没有提供参数，服务器主机默认为`localhost`，服务器端口默认为`8080`。

然后，我们可以使用两种简单的方法启动和停止服务器:

```
wireMockServer.start();
```

并且:

```
wireMockServer.stop();
```

### 3.2。基本用法

我们将首先演示 WireMock 库的基本用法，其中提供了一个精确 URL 的存根，无需任何进一步的配置。

让我们创建一个服务器实例:

```
WireMockServer wireMockServer = new WireMockServer();
```

WireMock 服务器必须在客户端连接到它之前运行:

```
wireMockServer.start();
```

然后 web 服务被存根化:

```
configureFor("localhost", 8080);
stubFor(get(urlEqualTo("/baeldung")).willReturn(aResponse().withBody("Welcome to Baeldung!")));
```

本教程利用 Apache HttpClient API 来表示连接到服务器的客户机:

```
CloseableHttpClient httpClient = HttpClients.createDefault();
```

执行一个请求，然后返回一个响应:

```
HttpGet request = new HttpGet("http://localhost:8080/baeldung");
HttpResponse httpResponse = httpClient.execute(request);
```

我们将使用一个助手方法将`httpResponse`变量转换为`String`:

```
String responseString = convertResponseToString(httpResponse);
```

下面是转换助手方法的实现:

```
private String convertResponseToString(HttpResponse response) throws IOException {
    InputStream responseStream = response.getEntity().getContent();
    Scanner scanner = new Scanner(responseStream, "UTF-8");
    String responseString = scanner.useDelimiter("\\Z").next();
    scanner.close();
    return responseString;
}
```

下面的代码验证了服务器已经收到了对预期 URL 的请求，并且到达客户端的响应与发送的完全相同:

```
verify(getRequestedFor(urlEqualTo("/baeldung")));
assertEquals("Welcome to Baeldung!", stringResponse);
```

最后，我们应该停止 WireMock 服务器来释放系统资源:

```
wireMockServer.stop();
```

## 4。JUnit 托管服务器

与第 3 节相比，本节在 JUnit `Rule`的帮助下演示了 WireMock 服务器的使用。

### 4.1。服务器设置

我们可以通过使用`@Rule`注释将 WireMock 服务器集成到 JUnit 测试用例中。这允许 JUnit 管理生命周期，在每个测试方法之前启动服务器，在方法返回之后停止服务器。

与以编程方式管理的服务器类似，JUnit 管理的 WireMock 服务器可以创建为具有给定端口号的 Java 对象:

```
@Rule
public WireMockRule wireMockRule = new WireMockRule(int port);
```

如果没有提供参数，服务器端口将采用默认值`8080`。服务器主机，默认为`localhost`，其他配置可以使用`Options`界面指定。

### 4.2。URL 匹配

在设置了一个`WireMockRule`实例之后，下一步是配置一个存根。

在这一小节中，我们将使用正则表达式为服务端点提供一个 REST 存根:

```
stubFor(get(urlPathMatching("/baeldung/.*"))
  .willReturn(aResponse()
  .withStatus(200)
  .withHeader("Content-Type", "application/json")
  .withBody("\"testing-library\": \"WireMock\"")));
```

让我们继续创建一个 HTTP 客户端，执行请求并接收响应:

```
CloseableHttpClient httpClient = HttpClients.createDefault();
HttpGet request = new HttpGet("http://localhost:8080/baeldung/wiremock");
HttpResponse httpResponse = httpClient.execute(request);
String stringResponse = convertHttpResponseToString(httpResponse);
```

上面的代码片段利用了一个转换助手方法:

```
private String convertHttpResponseToString(HttpResponse httpResponse) throws IOException {
    InputStream inputStream = httpResponse.getEntity().getContent();
    return convertInputStreamToString(inputStream);
}
```

这又利用了另一个私有方法:

```
private String convertInputStreamToString(InputStream inputStream) {
    Scanner scanner = new Scanner(inputStream, "UTF-8");
    String string = scanner.useDelimiter("\\Z").next();
    scanner.close();
    return string;
}
```

下面的测试代码验证了存根的操作:

```
verify(getRequestedFor(urlEqualTo("/baeldung/wiremock")));
assertEquals(200, httpResponse.getStatusLine().getStatusCode());
assertEquals("application/json", httpResponse.getFirstHeader("Content-Type").getValue());
assertEquals("\"testing-library\": \"WireMock\"", stringResponse);
```

### 4.3。请求标题匹配

现在我们将演示如何用匹配的头来存根化 REST API。

让我们从存根配置开始:

```
stubFor(get(urlPathEqualTo("/baeldung/wiremock"))
  .withHeader("Accept", matching("text/.*"))
  .willReturn(aResponse()
  .withStatus(503)
  .withHeader("Content-Type", "text/html")
  .withBody("!!! Service Unavailable !!!")));
```

与前面的小节类似，我们使用 HttpClient API 说明 HTTP 交互，并借助相同的 helper 方法:

```
CloseableHttpClient httpClient = HttpClients.createDefault();
HttpGet request = new HttpGet("http://localhost:8080/baeldung/wiremock");
request.addHeader("Accept", "text/html");
HttpResponse httpResponse = httpClient.execute(request);
String stringResponse = convertHttpResponseToString(httpResponse);
```

以下验证和断言确认了我们之前创建的存根的功能:

```
verify(getRequestedFor(urlEqualTo("/baeldung/wiremock")));
assertEquals(503, httpResponse.getStatusLine().getStatusCode());
assertEquals("text/html", httpResponse.getFirstHeader("Content-Type").getValue());
assertEquals("!!! Service Unavailable !!!", stringResponse);
```

### 4.4。请求正文匹配

我们还可以使用 WireMock 库来存根化一个带有主体匹配的 REST API。

以下是这种存根的配置:

```
stubFor(post(urlEqualTo("/baeldung/wiremock"))
  .withHeader("Content-Type", equalTo("application/json"))
  .withRequestBody(containing("\"testing-library\": \"WireMock\""))
  .withRequestBody(containing("\"creator\": \"Tom Akehurst\""))
  .withRequestBody(containing("\"website\": \"wiremock.org\""))
  .willReturn(aResponse()
  .withStatus(200)));
```

现在是时候创建一个将被用作请求主体的`StringEntity`对象了:

```
InputStream jsonInputStream 
  = this.getClass().getClassLoader().getResourceAsStream("wiremock_intro.json");
String jsonString = convertInputStreamToString(jsonInputStream);
StringEntity entity = new StringEntity(jsonString);
```

上面的代码使用了之前定义的转换助手方法之一，`convertInputStreamToString`。

下面是类路径上的`wiremock_intro.json`文件的内容:

```
{
    "testing-library": "WireMock",
    "creator": "Tom Akehurst",
    "website": "wiremock.org"
}
```

我们可以配置和运行 HTTP 请求和响应:

```
CloseableHttpClient httpClient = HttpClients.createDefault();
HttpPost request = new HttpPost("http://localhost:8080/baeldung/wiremock");
request.addHeader("Content-Type", "application/json");
request.setEntity(entity);
HttpResponse response = httpClient.execute(request);
```

这是用于验证存根的测试代码:

```
verify(postRequestedFor(urlEqualTo("/baeldung/wiremock"))
  .withHeader("Content-Type", equalTo("application/json")));
assertEquals(200, response.getStatusLine().getStatusCode());
```

### 4.5。存根优先级

前面的小节处理了 HTTP 请求只匹配一个存根的情况。

如果一个请求有多个匹配项，情况就更复杂了。默认情况下，在这种情况下，最近添加的存根将优先。

但是，用户可以定制该行为，以便更好地控制 WireMock 存根。

我们将演示当一个到来的请求同时匹配两个不同的存根时，在设置和不设置优先级的情况下，WireMock 服务器的操作。

这两种情况都将使用下面的私有 helper 方法:

```
private HttpResponse generateClientAndReceiveResponseForPriorityTests() throws IOException {
    CloseableHttpClient httpClient = HttpClients.createDefault();
    HttpGet request = new HttpGet("http://localhost:8080/baeldung/wiremock");
    request.addHeader("Accept", "text/xml");
    return httpClient.execute(request);
}
```

首先，我们配置两个存根，不考虑优先级:

```
stubFor(get(urlPathMatching("/baeldung/.*"))
  .willReturn(aResponse()
  .withStatus(200)));
stubFor(get(urlPathEqualTo("/baeldung/wiremock"))
  .withHeader("Accept", matching("text/.*"))
  .willReturn(aResponse()
  .withStatus(503)));
```

接下来，我们创建一个 HTTP 客户机，并使用 helper 方法执行一个请求:

```
HttpResponse httpResponse = generateClientAndReceiveResponseForPriorityTests();
```

以下代码片段验证当请求与最后配置的存根匹配时，是否应用最后配置的存根，而不考虑之前定义的存根:

```
verify(getRequestedFor(urlEqualTo("/baeldung/wiremock")));
assertEquals(503, httpResponse.getStatusLine().getStatusCode());
```

让我们来看一下设置了优先级的存根，其中较低的数字代表较高的优先级:

```
stubFor(get(urlPathMatching("/baeldung/.*"))
  .atPriority(1)
  .willReturn(aResponse()
  .withStatus(200)));
stubFor(get(urlPathEqualTo("/baeldung/wiremock"))
  .atPriority(2)
  .withHeader("Accept", matching("text/.*"))
  .willReturn(aResponse()
  .withStatus(503)));
```

现在我们将创建和执行一个 HTTP 请求:

```
HttpResponse httpResponse = generateClientAndReceiveResponseForPriorityTests();
```

以下代码验证了优先级的效果，其中应用了第一个配置的存根而不是最后一个:

```
verify(getRequestedFor(urlEqualTo("/baeldung/wiremock")));
assertEquals(200, httpResponse.getStatusLine().getStatusCode());
```

## 5。结论

本文介绍了 WireMock 以及如何设置和配置这个库，以便使用各种技术测试 REST APIs，包括 URL、请求头和主体的匹配。

所有示例和代码片段的实现都可以在 GitHub 项目中找到。