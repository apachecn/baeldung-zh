# 春季安全-攻击 OAuth

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-security-oauth-attack-redirect>

## 1.介绍

OAuth 是委托授权的行业标准框架。在创建构成标准的各种流的过程中，投入了大量的思考和精力。即便如此，它也不是没有弱点。

在这一系列文章中，我们将从理论的角度讨论针对 OAuth 的攻击，并描述保护我们的应用程序的各种选项。

## 2.授权码授权

[授权代码授权](https://web.archive.org/web/20220627184928/https://oauth.net/2/grant-types/authorization-code/)流是大多数实现委托授权的应用程序使用的默认流。

在流程开始之前，客户端必须已经向授权服务器进行了预注册，在这个过程中，它还必须提供一个重定向 URL——也就是说，**一个 URL，授权服务器可以在这个 URL 上使用授权代码回调客户端。**

让我们仔细看看它是如何工作的，以及其中一些术语的含义。

在授权代码授权流程中，客户端(请求委托授权的应用程序)将资源所有者(用户)重定向到授权服务器(例如，[使用 Google](https://web.archive.org/web/20220627184928/https://developers.google.com/identity/sign-in/web/sign-in) 登录)。登录后，授权服务器**用一个授权码重定向回客户端。**

接下来，客户端调用授权服务器上的端点，通过提供授权代码来请求访问令牌。此时，流程结束，客户端可以使用令牌访问受授权服务器保护的资源。

现在，**OAuth 2.0 框架允许这些客户端成为`public`** ，比如说在客户端不能安全保存客户端秘密的情况下。让我们来看看一些可能针对公共客户端的重定向攻击。

## 3.重定向攻击

### 3.1.攻击前提条件

重定向攻击依赖于这样一个事实:OAuth 标准没有完全描述这个重定向 URL 必须被指定的程度。这是设计好的。

这允许 OAuth 协议的一些实现允许部分重定向 URL。

例如，如果我们在授权服务器上注册一个客户端 ID 和一个客户端重定向 URL，并使用以下基于通配符的匹配:

`*.cloudapp.net`

这对以下情况有效:

`app.cloudapp.net`

也是为了:

`evil.cloudapp.net`

我们特意选择了`cloudapp.net`域，因为这是一个我们可以托管 OAuth 支持的应用程序的真实位置。该域名是微软 Windows Azure 平台的一部分，允许任何开发者在其下托管一个子域来测试应用。这本身不是问题，但它是更大的利用的重要部分。

这个漏洞的第二部分是一个授权服务器，它允许回调 URL 上的通配符匹配。

最后，为了实现这个漏洞，应用程序开发人员需要向授权服务器注册，以接受主域下的任何 URL，格式为`*.cloudapp.net`。

### 3.2.袭击

当这些条件满足时，攻击者就需要欺骗用户从他控制的子域中启动一个页面，例如，通过向用户发送[一封看起来可信的电子邮件](https://web.archive.org/web/20220627184928/https://www.vadesecure.com/en/5-common-phishing-techniques/)，要求他对 OAuth 保护的帐户采取一些行动。通常情况下，这看起来类似于`https://evil.cloudapp.net/login`。当用户打开此链接并选择登录时，他将被重定向到授权服务器，并带有一个授权请求:

```java
GET /authorize?response_type=code&client_id={apps-client-id}&state={state}&redirect_uri=https%3A%2F%2Fevil.cloudapp.net%2Fcb HTTP/1.1
```

虽然这可能看起来很典型，但这个 URL 是恶意的。看，在这种情况下，授权服务器**接收到一个篡改过的 URL，带有`app's`客户端 ID** ，以及一个指向 *evil 的*应用的重定向 URL。

然后，授权服务器将验证该 URL，它是指定主域下的一个子域。因为授权服务器认为请求来自一个有效的来源，所以它将对用户进行身份验证，然后像往常一样请求同意。

完成后，它将重定向回`evil.cloudapp.net`子域，将授权代码交给攻击者。

由于攻击者现在有了授权码，他需要做的只是用授权码调用授权服务器的令牌端点来接收令牌，这允许他访问资源所有者的受保护资源。

## 4.Spring OAuth 授权服务器漏洞评估

让我们来看看一个简单的 Spring OAuth 授权服务器配置:

```java
@Configuration
public class AuthorizationServerConfig extends AuthorizationServerConfigurerAdapter {    
    @Override
    public void configure(ClientDetailsServiceConfigurer clients) throws Exception {
        clients.inMemory()
          .withClient("apricot-client-id")
          .authorizedGrantTypes("authorization_code")
          .scopes("scope1", "scope2")
          .redirectUris("https://app.cloudapp.net/oauth");
    }
    // ...
}
```

我们可以看到授权服务器正在配置一个 id 为`“apricot-client-id”`的新客户端。没有客户端机密，所以这是一个公共客户端。

**我们的安全耳朵应该在这次**上竖起来，因为我们现在有了三个条件中的两个——邪恶的人可以注册子域`and`我们使用的是公共客户端。

但是，请注意，我们在这里也配置了**重定向 URL，并且它是绝对的**。我们可以通过这样做来减少漏洞。

### 4.1.严格的

默认情况下，Spring OAuth 在重定向 URL 匹配方面允许一定程度的灵活性。

例如，`DefaultRedirectResolver `支持子域匹配。

让我们只使用我们需要的东西。如果我们可以精确匹配重定向 URL，我们应该做:

```java
@Configuration
public class AuthorizationServerConfig extends AuthorizationServerConfigurerAdapter {    
    //...

    @Override
    public void configure(AuthorizationServerEndpointsConfigurer endpoints) {
        endpoints.redirectResolver(new ExactMatchRedirectResolver());
    }
}
```

在这种情况下，我们已经切换到使用`ExactMatchRedirectResolver`来重定向 URL。这个解析器进行精确的字符串匹配，而不以任何方式解析重定向 URL。这使得它的行为更加安全可靠。

### 4.2.仁慈的

我们可以在[Spring Security OAuth source](https://web.archive.org/web/20220627184928/https://github.com/spring-projects/spring-security-oauth/blob/7bfe08d8f95b2fec035de484068f7907851b27d0/spring-security-oauth2/src/main/java/org/springframework/security/oauth2/provider/endpoint/DefaultRedirectResolver.java)中找到处理重定向 URL 匹配的默认代码:

```java
/**
Whether the requested redirect URI "matches" the specified redirect URI. For a URL, this implementation tests if
the user requested redirect starts with the registered redirect, so it would have the same host and root path if
it is an HTTP URL. The port, userinfo, query params also matched. Request redirect uri path can include
additional parameters which are ignored for the match
<p>
For other (non-URL) cases, such as for some implicit clients, the redirect_uri must be an exact match.
@param requestedRedirect The requested redirect URI.
@param redirectUri The registered redirect URI.
@return Whether the requested redirect URI "matches" the specified redirect URI.
*/
protected boolean redirectMatches(String requestedRedirect, String redirectUri) {
   UriComponents requestedRedirectUri = UriComponentsBuilder.fromUriString(requestedRedirect).build();
   UriComponents registeredRedirectUri = UriComponentsBuilder.fromUriString(redirectUri).build();
   boolean schemeMatch = isEqual(registeredRedirectUri.getScheme(), requestedRedirectUri.getScheme());
   boolean userInfoMatch = isEqual(registeredRedirectUri.getUserInfo(), requestedRedirectUri.getUserInfo());
   boolean hostMatch = hostMatches(registeredRedirectUri.getHost(), requestedRedirectUri.getHost());
   boolean portMatch = matchPorts ? registeredRedirectUri.getPort() == requestedRedirectUri.getPort() : true;
   boolean pathMatch = isEqual(registeredRedirectUri.getPath(),
     StringUtils.cleanPath(requestedRedirectUri.getPath()));
   boolean queryParamMatch = matchQueryParams(registeredRedirectUri.getQueryParams(),
     requestedRedirectUri.getQueryParams());

   return schemeMatch && userInfoMatch && hostMatch && portMatch && pathMatch && queryParamMatch;
}
```

我们可以看到，URL 匹配是通过将传入的重定向 URL 解析成其组成部分来完成的。这是非常复杂的，因为它有几个特性，比如端口、子域和查询参数是否应该匹配。选择允许子域匹配是需要再三考虑的事情。

当然，这种灵活性是存在的，如果我们需要的话——让我们小心使用它。

## 5.隐式流重定向攻击

**需要明确的是，[隐流](https://web.archive.org/web/20220627184928/https://oauth.net/2/grant-types/implicit/)不推荐。**最好使用授权码授权流，由 [PKCE](https://web.archive.org/web/20220627184928/https://tools.ietf.org/html/rfc7636) 提供额外的安全性。也就是说，让我们看看重定向攻击是如何通过隐式流表现出来的。

针对隐式流的重定向攻击将遵循与我们上面看到的相同的基本轮廓。主要区别在于攻击者立即获得令牌，因为没有授权码交换步骤。

和以前一样，重定向 URL 的绝对匹配也将减轻这类攻击。

此外，我们可以发现隐式流包含另一个相关的漏洞。**攻击者可以使用客户端作为开放重定向器，并让它重新附加片段**。

攻击像以前一样开始，攻击者让用户访问攻击者控制下的页面，例如`https://evil.cloudapp.net/info`。该页面被精心设计为像以前一样启动授权请求。但是，它现在包含一个重定向 URL:

```java
GET /authorize?response_type=token&client;_id=ABCD&state;=xyz&redirect;_uri=https%3A%2F%2Fapp.cloudapp.net%2Fcb%26redirect_to
%253Dhttps%253A%252F%252Fevil.cloudapp.net%252Fcb HTTP/1.1 
```

`redirect_to https://evil.cloudapp.net`正在设置授权端点，将令牌重定向到攻击者控制下的域。授权服务器现在将首先重定向到实际的应用程序站点:

```java
Location: https://app.cloudapp.net/cb?redirect_to%3Dhttps%3A%2F%2Fevil.cloudapp.net%2Fcb#access_token=LdKgJIfEWR34aslkf&... 
```

当该请求到达开放重定向器时，它将提取重定向 URL `evil.cloudapp.net`，然后重定向到攻击者的站点:

```java
https://evil.cloudapp.net/cb#access_token=LdKgJIfEWR34aslkf&... 
```

绝对 URL 匹配也会减轻这种攻击。

## 6.摘要

在本文中，我们讨论了一类基于重定向 URL 的针对 OAuth 协议的攻击。

虽然这有潜在的严重后果，但在授权服务器上使用绝对 URL 匹配可以减轻这类攻击。