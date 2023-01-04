# 使用 HttpUrlConnection 进行身份验证

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-http-url-connection>

## 1.概观

在本教程中，我们将探索如何使用`HttpUrlConnection`类认证 [HTTP 请求](/web/20221205124452/https://www.baeldung.com/java-http-request)。

## 2.HTTP 认证

在 web 应用程序中，服务器可能要求客户端对自己进行身份验证。不遵守通常会导致服务器返回 HTTP 401(未授权)状态代码。

有多种[认证方案](https://web.archive.org/web/20221205124452/https://www.iana.org/assignments/http-authschemes/http-authschemes.xhtml)，它们提供的安全强度不同。然而，实现工作也各不相同。

让我们看看其中的三个:

*   是一个方案，我们将在下一节详细讨论
*   `digest`对用户凭证和服务器指定的随机数应用哈希算法
*   `bearer` 利用访问令牌作为 [OAuth 2.0](/web/20221205124452/https://www.baeldung.com/spring-security-5-oauth2-login) 的一部分

## 3.基本认证

基本认证允许客户端通过`Authorization`头使用**编码的用户名和密码**对自己进行认证:

```
GET / HTTP/1.1
Authorization: Basic dXNlcjpwYXNzd29yZA==
```

要创建编码的用户名和密码字符串，我们只需对用户名进行 Base64 编码，后跟一个冒号，然后是密码:

```
basic(user, pass) = base64-encode(user + ":" + pass)
```

不过，请记住来自 [RFC 7617](https://web.archive.org/web/20221205124452/https://tools.ietf.org/html/rfc7617) 的一些警告:

> 除非与某些外部安全系统(如 TLS)结合使用，否则这种方案不被视为安全的用户身份验证方法

当然，这是因为用户名和密码在每个请求中以明文形式通过网络传输。

## 4.验证连接

好了，有了这个背景，让我们开始配置`HttpUrlConnection` 使用 HTTP Basic。

类`HttpUrlConnection`可以发送请求，但是首先，我们必须从一个 URL 对象获得它的一个实例:

```
HttpURLConnection connection = (HttpURLConnection) url.openConnection();
```

一个连接提供了许多配置它的方法，比如`setRequestMethod` 和`setRequestProperty.`

虽然听起来很奇怪，但这正是我们想要的。

一旦我们使用“:”连接了用户名和密码，我们就可以使用`java.util.Base64`类对凭证进行编码:

```
String auth = user + ":" + password;
byte[] encodedAuth = Base64.encodeBase64(auth.getBytes(StandardCharsets.UTF_8));
```

然后，我们从文字“Basic”创建头值，后跟编码的凭证:

```
String authHeaderValue = "Basic " + new String(encodedAuth);
```

接下来，我们调用方法`setRequestProperty(key, value)`来认证请求。如前所述，**我们必须使用`“Authorization”`作为我们的标题，使用`“Basic ” + encoded credentials` 作为我们的值:**

```
connection.setRequestProperty("Authorization", authHeaderValue);
```

最后，我们需要实际发送 HTTP 请求，例如通过调用`getResponseCode()`。结果，我们从服务器得到一个 HTTP 响应代码:

```
int responseCode = connection.getResponseCode();
```

2xx 系列中的任何内容都意味着我们的请求(包括身份验证部分)是正确的！

## 5.Java `Authenticator`

上面提到的基本 auth 实现需要为每个请求设置授权头。相反，抽象类`java.net.Authenticator`允许**为所有连接**设置全局认证。

我们首先需要扩展类。然后，我们调用静态方法`Authenticator.setDefault()` 来注册我们的验证器的实例:

```
Authenticator.setDefault(new BasicAuthenticator());
```

我们的基本 auth 类只是覆盖了基类的`getPasswordAuthentication()`非抽象方法:

```
private final class BasicAuthenticator extends Authenticator {
protected PasswordAuthentication getPasswordAuthentication() {
return new PasswordAuthentication(user, password.toCharArray());
}
}
```

Authenticator 类利用我们的身份验证者的凭证来自动完成服务器所需的身份验证方案。

## 6.结论

在这个简短的教程中，我们看到了如何对通过`HttpUrlConnection`发送的请求应用基本认证。

和往常一样，代码示例可以在 GitHub 上找到[。](https://web.archive.org/web/20221205124452/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-networking-2)