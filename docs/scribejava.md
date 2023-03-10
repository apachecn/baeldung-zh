# ScribeJava 指南

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/scribejava>

## 1。简介

在本教程中，我们将看看 [ScribeJava](https://web.archive.org/web/20221208143837/https://github.com/scribejava/scribejava) 库。

ScribeJava 是一个简单的 OAuth Java 客户端，帮助管理 OAuth 流程。

该库的主要特性是支持所有主要的 1.0 和 2.0 OAuth APIs。此外，如果我们必须使用一个不受支持的 API，库提供了几个类来实现我们的 OAuth 的 API。

另一个重要的特性是可以选择哪个客户端使用。事实上，ScribeJava 支持几种 HTTP 客户端:

*   [异步 Http 客户端](/web/20221208143837/https://www.baeldung.com/async-http-client)
*   [OkHttp](/web/20221208143837/https://www.baeldung.com/guide-to-okhttp)
*   [Apache http components http client](https://web.archive.org/web/20221208143837/https://hc.apache.org/httpcomponents-client-5.1.x/)

此外，该库是线程安全的并且兼容 Java7，因此我们可以在遗留环境中使用它。

## 2。依赖性

**ScribeJava 被组织成一个核心和 API 模块**，后者包括一组外部 API (Google、GitHub、Twitter 等)和核心构件:

```java
<dependency>
    <groupId>com.github.scribejava</groupId>
    <artifactId>scribejava-apis</artifactId>
    <version>latest-version</version>
</dependency>
```

如果我们只需要核心类而不需要任何外部 API，我们必须只提取核心模块:

```java
<dependency>
    <groupId>com.github.scribejava</groupId>
    <artifactId>scribejava-core</artifactId>
    <version>latest-version</version>
</dependency>
```

最新版本可以在 [Maven 资源库](https://web.archive.org/web/20221208143837/https://search.maven.org/search?q=scribejava-core)找到。

## 3。`OAuthService`

**该库的主要部分是抽象类`OAuthService`** ，它包含了正确管理 OAuth 握手’所需的所有参数。

根据协议的版本，我们将分别为 [OAuth 1.0](https://web.archive.org/web/20221208143837/https://tools.ietf.org/html/rfc5849) 和 [OAuth 2.0](https://web.archive.org/web/20221208143837/https://tools.ietf.org/html/rfc6749) 使用`Oauth10Service`或`Oauth20Service`具体类。

为了构建`OAuthService`实现，库提供了一个`ServiceBuilder:`

```java
OAuthService service = new ServiceBuilder("api_key")
  .apiSecret("api_secret")
  .scope("scope")
  .callback("callback")
  .build(GoogleApi20.instance());
```

我们应该设置授权服务器提供的`api_key`和`api_secret`令牌。

此外，我们可以设置请求的`scope`和授权服务器在授权流程结束时应该将用户重定向到的`callback`。

请注意，根据协议的版本，并非所有参数都是强制性的。

最后，我们必须构建调用`build()`方法的`OAuthService`，并向其传递我们想要使用的 API 的实例。我们可以在 ScribeJava [GitHub](https://web.archive.org/web/20221208143837/https://github.com/scribejava/scribejava) 找到支持的 API 的完整列表。

### 3.1。HTTP 客户端

此外，**库允许我们选择使用哪个 HTTP 客户端:**

```java
ServiceBuilder builder = new ServiceBuilder("api_key")
  .httpClient(new OkHttpHttpClient());
```

当然，在之前的例子中，我们已经包括了所需的依赖项:

```java
<dependency>
    <groupId>com.github.scribejava</groupId>
    <artifactId>scribejava-httpclient-okhttp</artifactId>
    <version>latest-version</version>
</dependency>
```

最新版本可以在 [Maven 资源库](https://web.archive.org/web/20221208143837/https://mvnrepository.com/artifact/com.github.scribejava)找到。

### 3.2。调试模式

此外，**我们可以使用一种调试模式来帮助我们排除故障:**

```java
ServiceBuilder builder = new ServiceBuilder("api_key")
  .debug();
```

我们只需调用`debug()`方法。Debug 会向`System.out`输出一些相关信息。

另外，如果我们想使用不同的输出，还有另一个方法接受一个`OutputStream`来发送调试信息给:

```java
FileOutputStream debugFile = new FileOutputStream("debug");

ServiceBuilder builder = new ServiceBuilder("api_key")
  .debug()
  .debugStream(debugFile);
```

## 4。OAuth 1.0 流程

现在让我们关注如何处理 OAuth1 流。

在这个例子中，**我们将通过 Twitter APIs 获得一个`access token`,我们将使用它来发出一个请求。**

首先，我们必须使用 builder 构建`Oauth10Service`，正如我们前面看到的:

```java
OAuth10aService service = new ServiceBuilder("api_key")
  .apiSecret("api_secret")
  .build(TwitterApi.instance());
```

一旦我们有了`OAuth10Service, `,我们就可以得到一个`requestToken`,并用它来获得授权 URL:

```java
OAuth1RequestToken requestToken = service.getRequestToken();
String authUrl = service.getAuthorizationUrl(requestToken);
```

此时，需要将用户重定向到`authUrl`并获取页面提供的 *oauthVerifier* 。

因此，我们使用*oauthcverifier*来获得`accessToken`:

```java
OAuth1AccessToken accessToken = service.getAccessToken(requestToken,oauthVerifier);
```

最后，我们可以使用`OAuthRequest`对象创建一个请求，并使用`signRequest()`方法向其添加令牌:

```java
OAuthRequest request = new OAuthRequest(Verb.GET, 
    "https://api.twitter.com/1.1/account/verify_credentials.json");
service.signRequest(accessToken, request);

Response response = service.execute(request);
```

作为执行那个`request`的结果，我们得到一个`Response`对象。

## 5。OAuth 2.0 流程

OAuth 2.0 的流程与 OAuth 1.0 没有太大的不同。为了解释这些变化，**我们将使用 Google APIs 得到一个`access token`。**

同样，在 OAuth 1.0 流程中，我们必须构建`OAuthService`并获得`authUrl` `,`，但这次我们将使用一个`OAuth20Service`实例:

```java
OAuth20Service service = new ServiceBuilder("api_key")
  .apiSecret("api_secret")
  .scope("https://www.googleapis.com/auth/userinfo.email")
  .callback("http://localhost:8080/auth")
  .build(GoogleApi20.instance());

String authUrl = service.getAuthorizationUrl();
```

注意，在这种情况下，我们需要提供请求的`scope`和授权流程结束时联系我们的`callback`。

同样，我们必须将用户重定向到`authUrl`，并在回调的 url 中获取`code`参数:

```java
OAuth2AccessToken accessToken = service.getAccessToken(code);

OAuthRequest request = new OAuthRequest(Verb.GET, "https://www.googleapis.com/oauth2/v1/userinfo?alt=json");
service.signRequest(accessToken, request);

Response response = service.execute(request);
```

最后，为了得到`request`，我们用`getAccessToken()`方法得到了`accessToken `。

## 6。自定义 API

我们可能不得不使用 ScribeJava 不支持的 API。在这种情况下，**库允许我们实现自己的 API**。

我们唯一需要做的就是提供一个`DefaultApi10`或`DefaultApi20 `类的实现。

让我们想象一下，我们有一个 OAuth 2.0 授权服务器，带有密码授权。在这种情况下，我们可以实现`DefaultApi20`，这样我们就可以获得一个`access token`:

```java
public class MyApi extends DefaultApi20 {

    public MyApi() {}

    private static class InstanceHolder {
        private static final MyApi INSTANCE = new MyApi();
    }

    public static MyApi instance() {
        return InstanceHolder.INSTANCE;
    }

    @Override
    public String getAccessTokenEndpoint() {
        return "http://localhost:8080/oauth/token";
    }

    @Override
    protected String getAuthorizationBaseUrl() {
        return null;
    }
}
```

因此，我们可以像以前一样以类似的方式获得访问令牌:

```java
OAuth20Service service = new ServiceBuilder("baeldung_api_key")
  .apiSecret("baeldung_api_secret")
  .scope("read write")
  .build(MyApi.instance());

OAuth2AccessToken token = service.getAccessTokenPasswordGrant(username, password);

OAuthRequest request = new OAuthRequest(Verb.GET, "http://localhost:8080/me");
service.signRequest(token, request);
Response response = service.execute(request);
```

## 7。结论

在本文中，我们看了一下 ScribeJava 提供的最有用的类。

我们学习了如何用外部 API 处理 OAuth 1.0 和 OAuth 2.0 流。我们还学习了如何配置这个库，以便使用我们自己的 API。

像往常一样，本教程中显示的所有代码示例都可以从 GitHub 上的[处获得。](https://web.archive.org/web/20221208143837/https://github.com/eugenp/tutorials/tree/master/libraries-security)