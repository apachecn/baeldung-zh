# 高级 Apache HttpClient 配置

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/httpclient-advanced-config>

## 1。概述

在本文中，我们将了解 Apache [`HttpClient`](https://web.archive.org/web/20220524035745/https://hc.apache.org/httpcomponents-client-5.1.x/examples.html) 库的高级用法。

我们将查看向 HTTP 请求添加自定义头的示例，并了解如何配置客户机通过代理服务器授权和发送请求。

我们将使用 Wiremock 来清除 HTTP 服务器。如果您想了解更多关于 Wiremock 的内容，请查看本文。

## 2。带有自定义标题`User-Agent`的 HTTP 请求

假设我们想要向 HTTP GET 请求添加一个定制的`User-Agent`头。`User-Agent` 报头包含一个特征字符串，该字符串允许网络协议对等体识别请求软件用户代理的应用类型、操作系统和软件供应商或软件版本。

在我们开始编写 HTTP 客户端之前，我们需要启动我们的嵌入式模拟服务器:

```
@Rule
public WireMockRule serviceMock = new WireMockRule(8089);
```

当我们创建一个`HttpGet` 实例时，我们可以简单地使用一个`setHeader()`方法来传递头部的名称和值。该标头将被添加到 HTTP 请求中:

```
String userAgent = "BaeldungAgent/1.0"; 
HttpClient httpClient = HttpClients.createDefault();

HttpGet httpGet = new HttpGet("http://localhost:8089/detail");
httpGet.setHeader(HttpHeaders.USER_AGENT, userAgent);

HttpResponse response = httpClient.execute(httpGet);

assertEquals(response.getStatusLine().getStatusCode(), 200);
```

我们添加了一个`User-Agent`头，并通过一个`execute()`方法发送请求。

当 GET 请求被发送给一个 URL `/detail` ，其标题`User-Agent` 的值等于“BaeldungAgent/1.0”时，那么`serviceMock`将返回 200 个 HTTP 响应代码:

```
serviceMock.stubFor(get(urlEqualTo("/detail"))
  .withHeader("User-Agent", equalTo(userAgent))
  .willReturn(aResponse().withStatus(200)));
```

## 3。在 POST 请求正文中发送数据

通常，当我们执行 HTTP POST 方法时，我们希望传递一个实体作为请求体。当创建一个`HttpPost` 对象的实例时，我们可以使用一个`setEntity()` 方法将主体添加到请求中:

```
String xmlBody = "<xml><id>1</id></xml>";
HttpClient httpClient = HttpClients.createDefault();
HttpPost httpPost = new HttpPost("http://localhost:8089/person");
httpPost.setHeader("Content-Type", "application/xml");

StringEntity xmlEntity = new StringEntity(xmlBody);
httpPost.setEntity(xmlEntity);

HttpResponse response = httpClient.execute(httpPost);

assertEquals(response.getStatusLine().getStatusCode(), 200);
```

我们正在创建一个主体为`XML`格式的`StringEntity` 实例。将`Content-Type` 头设置为“`application/xml`”以向服务器传递关于我们发送的内容类型的信息是很重要的。当`serviceMock` 收到带有 XML 主体的 POST 请求时，它用状态代码 200 OK 进行响应:

```
serviceMock.stubFor(post(urlEqualTo("/person"))
  .withHeader("Content-Type", equalTo("application/xml"))
  .withRequestBody(equalTo(xmlBody))
  .willReturn(aResponse().withStatus(200)));
```

## 4。通过代理服务器发送请求

通常，我们的 web 服务可以位于代理服务器之后，代理服务器执行一些额外的逻辑，缓存静态资源等。当我们创建 HTTP 客户端并向实际服务发送请求时，我们不想处理每一个 HTTP 请求。

为了测试这个场景，我们需要启动另一个嵌入式 web 服务器:

```
@Rule
public WireMockRule proxyMock = new WireMockRule(8090);
```

对于两个嵌入式服务器，第一个实际服务位于 8089 端口，一个代理服务器监听 8090 端口。

我们正在配置我们的`HttpClient`,通过创建一个将`HttpHost`实例代理作为参数的`DefaultProxyRoutePlanner`,通过代理发送所有请求:

```
HttpHost proxy = new HttpHost("localhost", 8090);
DefaultProxyRoutePlanner routePlanner = new DefaultProxyRoutePlanner(proxy);
HttpClient httpclient = HttpClients.custom()
  .setRoutePlanner(routePlanner)
  .build(); 
```

我们的代理服务器将所有请求重定向到监听 8090 端口的实际服务。在测试的最后，我们验证请求是通过代理发送到我们的实际服务的:

```
proxyMock.stubFor(get(urlMatching(".*"))
  .willReturn(aResponse().proxiedFrom("http://localhost:8089/")));

serviceMock.stubFor(get(urlEqualTo("/private"))
  .willReturn(aResponse().withStatus(200)));

assertEquals(response.getStatusLine().getStatusCode(), 200);
proxyMock.verify(getRequestedFor(urlEqualTo("/private")));
serviceMock.verify(getRequestedFor(urlEqualTo("/private")));
```

## 5。配置 HTTP 客户端通过代理授权

扩展前面的示例，在某些情况下，代理服务器用于执行授权。在这种配置中，代理可以授权所有请求，并将它们传递给隐藏在代理后面的服务器。

我们可以配置 HttpClient 通过代理发送每个请求，以及用于执行授权过程的`Authorization` 头。

假设我们有一个代理服务器，它只授权一个用户——“`username_admin`”`,` ，密码是“`secret_password`”*。*

我们需要用将通过代理授权的用户的凭证创建`BasicCredentialsProvider` 实例。为了让`HttpClient` 自动添加具有适当值的`Authorization` 头，我们需要创建一个提供凭证的`HttpClientContext` 和一个存储凭证的`BasicAuthCache` :

```
HttpHost proxy = new HttpHost("localhost", 8090);
DefaultProxyRoutePlanner routePlanner = new DefaultProxyRoutePlanner(proxy);

//Client credentials
CredentialsProvider credentialsProvider = new BasicCredentialsProvider();
credentialsProvider.setCredentials(new AuthScope(proxy), 
  new UsernamePasswordCredentials("username_admin", "secret_password"));

// Create AuthCache instance
AuthCache authCache = new BasicAuthCache();

BasicScheme basicAuth = new BasicScheme();
authCache.put(proxy, basicAuth);
HttpClientContext context = HttpClientContext.create();
context.setCredentialsProvider(credentialsProvider);
context.setAuthCache(authCache);

HttpClient httpclient = HttpClients.custom()
  .setRoutePlanner(routePlanner)
  .setDefaultCredentialsProvider(credentialsProvider)
  .build();
```

当我们设置我们的`HttpClient,` 时，向我们的服务发出请求将导致通过代理发送一个带有`Authorization` 头的请求来执行授权过程。它将在每个请求中自动设置。

让我们对服务执行一个实际的请求:

```
HttpGet httpGet = new HttpGet("http://localhost:8089/private");
HttpResponse response = httpclient.execute(httpGet, context);
```

用我们的配置验证`httpClient`上的`execute()` 方法，确认请求通过了带有`Authorization`头的代理:

```
proxyMock.stubFor(get(urlMatching("/private"))
  .willReturn(aResponse().proxiedFrom("http://localhost:8089/")));
serviceMock.stubFor(get(urlEqualTo("/private"))
  .willReturn(aResponse().withStatus(200)));

assertEquals(response.getStatusLine().getStatusCode(), 200);
proxyMock.verify(getRequestedFor(urlEqualTo("/private"))
  .withHeader("Authorization", containing("Basic")));
serviceMock.verify(getRequestedFor(urlEqualTo("/private")));
```

## 6。结论

本文展示了如何配置 Apache `HttpClient` 来执行高级 HTTP 调用。我们看到了如何通过代理服务器发送请求，以及如何通过代理进行授权。

所有这些示例和代码片段的实现都可以在 [GitHub 项目](https://web.archive.org/web/20220524035745/https://github.com/eugenp/tutorials/tree/master/apache-httpclient)中找到——这是一个 Maven 项目，因此它应该很容易导入和运行。