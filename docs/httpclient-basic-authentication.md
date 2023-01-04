# Apache HttpClient 基本身份验证

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/httpclient-basic-authentication>

## 1。概述

本教程将说明如何在 Apache HttpClient 上配置基本认证。

如果你想更深入地了解 HttpClient 并学习其他很酷的东西，你可以直接阅读 HttpClient 教程**[。](/web/20220628101144/https://www.baeldung.com/httpclient-guide "Cool basic and more advanced things you can do with the HttpClient 4")**

## 延伸阅读:

## [Apache HttpAsyncClient 教程](/web/20220628101144/https://www.baeldung.com/httpasyncclient-tutorial)

HttpAsyncClient Tutorial - send a basic GET request, use the multi-threaded client, set up the client with SSL as well as with a proxy, and finally - do authentication.[Read more](/web/20220628101144/https://www.baeldung.com/httpasyncclient-tutorial) →

## [高级 Apache HttpClient 配置](/web/20220628101144/https://www.baeldung.com/httpclient-advanced-config)

HttpClient configurations for advanced use cases.[Read more](/web/20220628101144/https://www.baeldung.com/httpclient-advanced-config) →

## 2。使用 API 进行基本认证

让我们从在 HttpClient 上配置基本认证的标准方式**开始——通过`CredentialsProvider`T2:**

```java
CredentialsProvider provider = new BasicCredentialsProvider();
UsernamePasswordCredentials credentials
 = new UsernamePasswordCredentials("user1", "user1Pass");
provider.setCredentials(AuthScope.ANY, credentials);

HttpClient client = HttpClientBuilder.create()
  .setDefaultCredentialsProvider(provider)
  .build();

HttpResponse response = client.execute(
  new HttpGet(URL_SECURED_BY_BASIC_AUTHENTICATION));
int statusCode = response.getStatusLine()
  .getStatusCode();

assertThat(statusCode, equalTo(HttpStatus.SC_OK));
```

正如我们所看到的，创建一个带有凭证提供者的客户机并不困难。

现在，为了理解`HttpClient`在幕后实际会做什么，我们需要查看日志:

```java
# ... request is sent with no credentials
[main] DEBUG ... - Authentication required
[main] DEBUG ... - localhost:8080 requested authentication
[main] DEBUG ... - Authentication schemes in the order of preference: 
  [negotiate, Kerberos, NTLM, Digest, Basic]
[main] DEBUG ... - Challenge for negotiate authentication scheme not available
[main] DEBUG ... - Challenge for Kerberos authentication scheme not available
[main] DEBUG ... - Challenge for NTLM authentication scheme not available
[main] DEBUG ... - Challenge for Digest authentication scheme not available
[main] DEBUG ... - Selected authentication options: [BASIC]
# ... the request is sent again - with credentials
```

整个**客户端-服务器通信现在是清晰的**:

*   客户端发送没有凭据的 HTTP 请求
*   服务器发回一个询问
*   客户端协商并确定正确的身份验证方案
*   客户端向**发送第二个请求**，这一次带有凭证

## 3。抢先基本认证

开箱即用，`HttpClient`不做抢先认证。相反，这必须是客户做出的明确决定。

首先，**我们需要创建`HttpContext`，用预先选择的正确类型的身份验证方案的身份验证缓存**预先填充它。这意味着不再需要上一个示例中的协商—**已经选择了基本认证**:

```java
HttpHost targetHost = new HttpHost("localhost", 8082, "http");
CredentialsProvider credsProvider = new BasicCredentialsProvider();
credsProvider.setCredentials(AuthScope.ANY, 
  new UsernamePasswordCredentials(DEFAULT_USER, DEFAULT_PASS));

AuthCache authCache = new BasicAuthCache();
authCache.put(targetHost, new BasicScheme());

// Add AuthCache to the execution context
HttpClientContext context = HttpClientContext.create();
context.setCredentialsProvider(credsProvider);
context.setAuthCache(authCache);
```

现在，我们可以使用具有新上下文的客户端，**发送预认证请求**:

```java
HttpClient client = HttpClientBuilder.create().build();
response = client.execute(
  new HttpGet(URL_SECURED_BY_BASIC_AUTHENTICATION), context);

int statusCode = response.getStatusLine().getStatusCode();
assertThat(statusCode, equalTo(HttpStatus.SC_OK));
```

让我们看看日志:

```java
[main] DEBUG ... - Re-using cached 'basic' auth scheme for http://localhost:8082
[main] DEBUG ... - Executing request GET /spring-security-rest-basic-auth/api/foos/1 HTTP/1.1
[main] DEBUG ... >> GET /spring-security-rest-basic-auth/api/foos/1 HTTP/1.1
[main] DEBUG ... >> Host: localhost:8082
[main] DEBUG ... >> Authorization: Basic dXNlcjE6dXNlcjFQYXNz
[main] DEBUG ... << HTTP/1.1 200 OK
[main] DEBUG ... - Authentication succeeded
```

一切正常:

*   预先选择“基本认证”方案
*   该请求与`Authorization`报头一起发送
*   服务器以`200 OK`作为响应
*   认证成功

## 4。带有原始 HTTP 头的基本验证

抢先基本认证基本上意味着预先发送`Authorization`报头。

因此，**我们可以控制这个头文件并手工构建它，而不是通过前面相当复杂的例子来设置它**:

```java
HttpGet request = new HttpGet(URL_SECURED_BY_BASIC_AUTHENTICATION);
String auth = DEFAULT_USER + ":" + DEFAULT_PASS;
byte[] encodedAuth = Base64.encodeBase64(
  auth.getBytes(StandardCharsets.ISO_8859_1));
String authHeader = "Basic " + new String(encodedAuth);
request.setHeader(HttpHeaders.AUTHORIZATION, authHeader);

HttpClient client = HttpClientBuilder.create().build();
HttpResponse response = client.execute(request);

int statusCode = response.getStatusLine().getStatusCode();
assertThat(statusCode, equalTo(HttpStatus.SC_OK));
```

让我们确保它正常工作:

```java
[main] DEBUG ... - Auth cache not set in the context
[main] DEBUG ... - Opening connection {}->http://localhost:8080
[main] DEBUG ... - Connecting to localhost/127.0.0.1:8080
[main] DEBUG ... - Executing request GET /spring-security-rest-basic-auth/api/foos/1 HTTP/1.1
[main] DEBUG ... - Proxy auth state: UNCHALLENGED
[main] DEBUG ... - http-outgoing-0 >> GET /spring-security-rest-basic-auth/api/foos/1 HTTP/1.1
[main] DEBUG ... - http-outgoing-0 >> Authorization: Basic dXNlcjE6dXNlcjFQYXNz
[main] DEBUG ... - http-outgoing-0 << HTTP/1.1 200 OK
```

因此，即使没有身份验证缓存，**基本身份验证仍然可以正常工作，我们会收到`200 OK.`**

## 5。结论

本文展示了在 Apache HttpClient 中设置和使用基本身份验证的各种方法。

和往常一样，本文中的代码可以从 Github 上的[处获得。这是一个基于 Maven 的项目，因此应该很容易导入和运行。](https://web.archive.org/web/20220628101144/https://github.com/eugenp/tutorials/tree/master/httpclient-simple)