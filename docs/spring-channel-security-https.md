# Spring HTTP/HTTPS 通道安全性

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-channel-security-https>

## 1。概述

本教程展示了如何使用 Spring 的通道安全特性，使用 HTTPS 来保护你的应用程序的登录页面。

使用 HTTPS 进行身份验证对于保护传输中的敏感数据的完整性至关重要。

这篇文章建立在 Spring 安全登录教程的基础上，增加了一个额外的安全层。我们强调通过编码的 HTTPS 通道为登录页面提供服务来保护身份验证数据所需的步骤。

## 2。无通道安全的初始设置

让我们从前述文章中解释的安全配置开始。

该网络应用程序允许用户访问:

1.  `/anonymous.html`未经认证，
2.  `/login.html`，以及
3.  成功登录后的其他页面(`/homepage.html`)。

访问由以下配置控制:

```java
@Override
protected void configure(HttpSecurity http) throws Exception {
    http.authorizeRequests() 
      .antMatchers("/anonymous*")
      .anonymous();

    http.authorizeRequests()
      .antMatchers("/login*")
      .permitAll();

    http.authorizeRequests()
      .anyRequest()
      .authenticated(); 
```

或者通过 XML:

```java
<http use-expressions="true">
    <intercept-url pattern="/anonymous*" access="isAnonymous()"/>
    <intercept-url pattern="/login*" access="permitAll"/>
    <intercept-url pattern="/**" access="isAuthenticated()"/>
</http>
```

此时，登录页面位于:

```java
http://localhost:8080/spring-security-login/login.html
```

用户可以通过 HTTP 验证自己，但是这是不安全的，因为密码将以明文形式发送。

## 3。HTTPS 服务器配置

为了只通过 HTTPS `**–**` **发送登录页面，你的网络服务器必须能够提供 HTTPS 页面**。这需要启用 [SSL/TLS](https://web.archive.org/web/20220926195606/https://en.wikipedia.org/wiki/Transport_Layer_Security) 支持。

请注意，您可以使用有效的证书，或者出于测试目的，您可以生成自己的证书。

假设我们正在使用 Tomcat 并滚动我们自己的证书。我们首先需要创建一个带有自签名证书的`keystore`。

可以在终端中发出以下命令来生成密钥库:

```java
keytool -genkey -alias tomcat -keyalg RSA -storepass changeit -keypass changeit -dname 'CN=tomcat'
```

这将在您的主文件夹中为您的用户配置文件创建一个私有 a 密钥和一个自签名证书。

下一步是编辑`conf/server.xml`,使其看起来像这样:

```java
<Connector port="8080" protocol="HTTP/1.1"
   connectionTimeout="20000"
   redirectPort="8443" />

<Connector port="8443" protocol="org.apache.coyote.http11.Http11NioProtocol"
   maxThreads="150" SSLEnabled="true" scheme="https" secure="true"
   clientAuth="false" sslProtocol="TLS"
   keystoreFile="${user.home}/.keystore" keystorePass="changeit" />
```

第二个 SSL/TLS `<Connector>`标记通常在配置文件中被注释掉，所以只需要取消注释并添加密钥库信息。更多信息可在 [Tomcat 的相关文档](https://web.archive.org/web/20220926195606/https://tomcat.apache.org/tomcat-8.0-doc/ssl-howto.html)中获得。

HTTPS 配置就绪后，登录页面现在也可以在以下 URL 下提供:

```java
https://localhost:8443/spring-security-login/login.html
```

除了 Tomcat 之外，Web 服务器需要不同但可能相似的配置。

## 4。配置通道安全性

此时，我们能够在 HTTP 和 HTTPS 下提供登录页面。本节说明如何强制使用 HTTPS。

**要在登录页面**要求 HTTPS，请通过添加以下内容来修改您的安全配置:

```java
http.requiresChannel()
  .antMatchers("/login*").requiresSecure();
```

或者将`requires-channel=”https”`属性添加到 XML 配置中:

```java
<intercept-url pattern="/login*" access="permitAll" requires-channel="https"/>
```

此后，用户只能通过 HTTPS 登录。所有相关链接，例如转发到`/homepage.html`将继承原始请求的协议，并将在 HTTPS 下提供服务。

当在一个 web 应用程序中混合使用 HTTP 和 HTTPS 请求时，需要注意其他方面，并且需要进一步配置。

## 5。混合 HTTP 和 HTTPS

从安全的角度来看，为 HTTPS 的一切提供服务是一个很好的实践，也是一个坚实的目标。

但是，如果不能选择只使用 HTTPS，我们可以通过在配置中添加以下内容来配置 Spring 使用 HTTP:

```java
http.requiresChannel()
  .anyRequest().requiresInsecure();
```

或者在 XML 中添加`requires-channel=”http”`属性:

```java
<intercept‐url pattern="/**" access="isAuthenticated()" requires‐channel="http"/>
```

这指示 Spring 对所有没有明确配置为使用 HTTPS 的请求使用 HTTP，但同时也破坏了原始的登录机制。以下部分解释了根本原因。

### 5.1。HTTPS 上的自定义登录处理 URL

原始安全教程中的安全配置包含以下内容:

```java
<form-login login-processing-url="/perform_login"/>
```

**如果不强制`/perform_login`使用 HTTPS，它的 HTTP 变体会发生重定向，丢失与原始请求一起发送的登录信息**。

为了克服这个问题，我们需要配置 Spring 来使用 HTTPS 来处理 URL:

```java
http.requiresChannel()
  .antMatchers("/login*", "/perform_login");
```

注意传递给`antMatchers`方法的额外参数`/perform_login`。

XML 配置中的等效内容需要向配置中添加一个新的`<` `intercept-url>`元素:

```java
<intercept-url pattern="/perform_login" requires-channel="https"/>
```

如果您自己的应用程序使用默认的`login-processing-url`(即`/login`)，您不需要明确地配置它，因为`/login*`模式已经包含了这一点。

配置就绪后，用户可以登录，但不能访问认证页面，例如 HTTP 协议下的`/homepage.html`，因为 [Spring 的会话固定保护特性](https://web.archive.org/web/20220926195606/https://docs.spring.io/spring-security/site/docs/current/api/org/springframework/security/web/authentication/session/SessionFixationProtectionStrategy.html)。

### 5.2。禁用`session-fixation-protection`

[会话固定](https://web.archive.org/web/20220926195606/https://en.wikipedia.org/wiki/Session_fixation)是在 HTTP 和 HTTPS 之间切换时无法避免的问题。

默认情况下，Spring 会在成功登录后创建一个新的`session-id`。当用户加载 HTTPS 登录页面时，用户的`session-id` cookie 会被标记为`secure.`登录后，上下文会切换到 HTTP，[cookie 会丢失](https://web.archive.org/web/20220926195606/https://docs.spring.io/spring-security/site/faq/faq.html#faq-tomcat-https-session)，因为 HTTP 是不安全的。

为了避免这种 **s** **设置`[session-fixation-protection](https://web.archive.org/web/20220926195606/https://docs.spring.io/spring-security/site/docs/5.2.12.RELEASE/reference/html/jc-authentication.html#ns-session-fixation)` 到`none`需要**。

```java
http.sessionManagement()
  .sessionFixation()
  .none();
```

或者通过 XML:

```java
<session-management session-fixation-protection="none"/> 
```

**禁用会话固定保护可能会带来安全隐患**，因此，如果您担心基于会话固定的攻击，您需要权衡利弊。

## 6。测试

在应用了所有这些配置更改之后，在不登录的情况下访问`/anonymous.html`(使用`http://` 或 https://)将通过 HTTP 把你带到页面。

像`/homepage.html`一样直接打开其他页面应该会让你通过 HTTPS 转到登录页面，登录后你会使用 HTTP 转回到`/homepage.html`。

## 7。结论

在本教程中，我们已经了解了如何配置一个通过 HTTP 通信的 Spring web 应用程序，除了登录机制。然而**新的现代网络应用程序应该几乎总是只使用 HTTPS**作为它们的通信协议。降低安全级别或关闭安全功能(如`session-fixation-protection`)从来都不是一个好主意。

本教程基于 GitHub 上的[代码库。通过将`https` 列为活动的](https://web.archive.org/web/20220926195606/https://github.com/eugenp/tutorials/tree/master/spring-security-modules/spring-security-web-login) [Spring profile](/web/20220926195606/https://www.baeldung.com/spring-profiles) ，可以启用通道安全配置。