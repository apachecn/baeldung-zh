# Spring 中 SPNEGO/Kerberos 认证简介

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-security-kerberos>

## 1.概观

在本教程中，我们将了解 Kerberos 身份验证协议的基础。我们还将讨论 Kerberos 中对 [SPNEGO](https://web.archive.org/web/20221025183651/https://tools.ietf.org/html/rfc4178) 的需求。

最后，我们将看到如何利用 Spring Security Kerberos 的扩展来创建支持 Kerberos 和 SPNEGO 的应用程序。

在我们继续之前，值得注意的是，本教程将为这一领域的外行介绍许多新术语。因此，我们将花一些时间在前面覆盖地面。

## 2.了解 Kerberos

**[Kerberos](https://web.archive.org/web/20221025183651/https://web.mit.edu/kerberos/) 是八十年代初由麻省理工学院(MIT)开发的网络认证协议**。正如你可能意识到的，这是比较古老的，并且经受住了时间的考验。Windows Server 广泛支持 Kerberos 作为身份验证机制，甚至将它作为默认的身份验证选项。

从技术上来说， **Kerberos 是一种基于票证的认证协议**，它允许计算机网络中的节点相互识别自己。

### 2.1.Kerberos 的简单用例

让我们拟定一个假设的情况来证明这一点。

假设一个用户，通过他机器上的邮件客户端，需要从同一个网络上另一台机器上的邮件服务器获取他的邮件。这里显然需要认证。邮件客户端和邮件服务器必须能够相互识别和信任，才能安全地进行通信。

Kerberos 如何在这方面帮助我们呢？ **Kerberos 引入了第三方，称为密钥分发中心(KDC)** ，它与网络中的每个节点都有相互信任。让我们看看这在我们的例子中是如何工作的:

[![Kerberos Protocol](img/0f8df931e2969ecf5555d4008a70a701.png)](/web/20221025183651/https://www.baeldung.com/wp-content/uploads/2019/04/Kerberos-Protocol.jpg)

### 2.2.Kerberos 协议的关键方面

虽然这听起来有点深奥，但在保护不安全网络上的通信时，这是非常简单和有创造性的。这里提出的一些问题，在到处都是 TLS 的时代，是相当理所当然的！

虽然这里不可能详细讨论 Kerberos 协议，但是让我们来看一些突出的方面:

*   假设节点(客户端和服务器)和 KDC 之间的信任存在于同一领域中
*   **密码从不通过网络交换**
*   客户端和服务器之间的信任是基于这样一个事实，即它们可以使用仅与 KDC 共享的密钥来解密消息
*   **客户端和服务器之间的信任是相互的**
*   客户端可以缓存票证以供重复使用，直到过期，从而提供单点登录体验
*   验证者消息基于时间戳，因此只适合一次性使用
*   这里的三方必须有相对同步的时间

虽然这只是触及了这个漂亮的身份验证协议的表面，但它足以让我们继续我们的教程。

## 3.了解 SPNEGO

SPNEGO 代表[简单且受保护的 GSS-API 协商机制](https://web.archive.org/web/20221025183651/https://tools.ietf.org/html/rfc4178)。很好听的名字！让我们先来看看 GSS-API 代表什么。[通用安全服务应用程序接口](https://web.archive.org/web/20221025183651/https://tools.ietf.org/html/rfc2743) (GSS-API)只不过是一个 IETF 标准，用于客户端和服务器以安全且独立于供应商的方式进行通信。

SPNEGO 是 GSS API 的一部分，用于客户端和服务器协商选择安全机制来使用，例如 Kerberos 或 NTLM。

## 4.为什么 Kerberos 需要`SPNEGO`？

正如我们在上一节中看到的，Kerberos 是一种纯粹的网络身份验证协议，主要在传输层(TCP/UDP)中运行。虽然这对于许多用例来说是好的，但是这不能满足现代 web 的需求。如果我们有一个运行在更高抽象上的应用程序，比如 HTTP，就不可能直接使用 Kerberos。

这就是 SPNEGO 帮助我们的地方。在 web 应用程序的情况下，通信主要发生在像 Chrome 这样的 web 浏览器和像 Tomcat 这样通过 HTTP 托管 web 应用程序的 web 服务器之间。如果启用，他们可以**通过 SPNEGO 协商 Kerberos 作为安全机制，并通过 HTTP** 交换作为 SPNEGO 令牌的票证。

那么这如何改变我们之前提到的场景呢？让我们用 web 浏览器替换简单的邮件客户端，用 web 应用程序替换邮件服务器:

[![Kerberos with SPNEGO](img/dd322559c00e23553823f3c02d46cced.png)](/web/20221025183651/https://www.baeldung.com/wp-content/uploads/2019/04/Kerberos-with-SPNEGO.jpg)

因此，与之前的图表相比，这里没有太多变化，只是客户端和服务器之间的通信现在通过 HTTP 显式地进行。让我们更好地理解这一点:

*   客户机根据 KDC 进行身份验证，并缓存 TGT
*   客户机上的 Web 浏览器被配置为使用 SPNEGO 和 Kerberos
*   web 应用程序也被配置为支持 SPNEGO 和 Kerberos
*   Web 应用程序向试图访问受保护资源的 web 浏览器抛出“协商”挑战
*   **服务票被包装为 SPNEGO 令牌，并作为 HTTP 报头进行交换**

## 5.要求

在开发支持 Kerberos 身份验证模式的 web 应用程序之前，我们必须收集一些基本的设置。让我们快速浏览一下这些任务。

### 5.1.建立 KDC

设置用于生产的 Kerberos 环境超出了本教程的范围。不幸的是，这不是一项微不足道的任务，也很脆弱。有几个选项可以用来实现 Kerberos，包括开源版本和商业版本:

*   MIT 使得 Kerberos v5 的[实现可用于多种操作系统](https://web.archive.org/web/20221025183651/http://web.mit.edu/Kerberos/dist/)
*   Apache Kerby 是 Apache Directory 的扩展，它提供了一个 Java Kerberos 绑定
*   微软的 Windows Server 支持由活动目录提供本地支持的 Kerberos v5
*   Heimdel 有一个 Kerberos v5 的[实现](https://web.archive.org/web/20221025183651/https://www.h5l.org/)

KDC 和相关基础设施的实际设置取决于提供商，应遵循其各自的文档。然而， [Apache Kerby 可以在 Docker 容器](https://web.archive.org/web/20221025183651/https://coheigea.blogspot.com/2018/06/running-apache-kerby-kdc-in-docker.html)中运行，这使得它与平台无关。

### 5.2.在 KDC 设置用户

我们需要在 KDC 建立两个用户——或者他们称之为委托人。为此，我们可以使用“kadmin”命令行工具。假设我们已经在 KDC 数据库中创建了一个名为“baeldung.com”的领域，并使用一个具有管理员权限的用户登录到“kadmin”。

我们将创建我们的第一个用户，我们希望通过 web 浏览器对其进行身份验证，使用:

```java
$ kadmin: addprinc -randkey kchandrakant -pw password
Principal "[[email protected]](/web/20221025183651/https://www.baeldung.com/cdn-cgi/l/email-protection)" created.
```

我们还需要向 KDC 注册我们的 web 应用程序:

```java
$ kadmin: addprinc -randkey HTTP/[[email protected]](/web/20221025183651/https://www.baeldung.com/cdn-cgi/l/email-protection) -pw password
Principal "HTTP/[[email protected]](/web/20221025183651/https://www.baeldung.com/cdn-cgi/l/email-protection)" created.
```

请注意这里命名主体的约定，因为这必须与可以从 web 浏览器访问应用程序的域相匹配。当出现“协商”挑战时，web **浏览器自动尝试使用此约定**创建服务主体名称(SPN)。

我们还需要将其导出为 keytab 文件，以便 web 应用程序可以使用它:

```java
$ kadmin: ktadd -k baeldung.keytab HTTP/[[email protected]](/web/20221025183651/https://www.baeldung.com/cdn-cgi/l/email-protection)
```

这应该会给我们一个名为“baeldung.keytab”的文件。

### 5.3.浏览器配置

我们需要启用 web 浏览器来访问 web 应用程序上的受保护资源，以实现“协商”身份验证方案。幸运的是，大多数像 Chrome 这样的现代 web 浏览器默认支持“协商”作为一种身份验证方案。

此外，我们可以配置浏览器以提供“集成认证”。在这种模式下，当出现“协商”质询时，浏览器会尝试利用主机中缓存的凭据，该凭据已经登录到 KDC 主体。然而，我们不会在这里使用这种模式来保持事情的明确性。

### 5.4.域配置

可以理解的是，我们可能没有实际的域来测试我们的 web 应用程序。但遗憾的是，我们不能使用 localhost 或 127.0.0.1 或任何其他带有 Kerberos 身份验证的 IP 地址。但是，对此有一个简单的解决方案，它包括在“hosts”文件中设置如下条目:

```java
demo.kerberos.bealdung.com 127.0.0.1
```

## 6.春天来拯救我们了！

最后，我们已经清楚了基本原理，是时候测试这个理论了。但是，创建一个支持 SPNEGO 和 Kerberos 的 web 应用程序不会很麻烦吗？如果我们用弹簧就不会。作为 Spring 安全的一部分，Spring 有一个 Kerberos 扩展，它无缝地支持 SPNEGO 和 Kerberos。

我们所要做的几乎只是在 Spring Security 中进行配置，使 SPNEGO 支持 Kerberos。这里我们将使用 Java 风格的配置，但是 XML 配置也可以很容易地设置。我们可以扩展`WebSecurityConfigurerAdapter`类来配置我们需要的一切。

### 6.1.Maven 依赖性

我们首先要设置的是依赖关系:

```java
<dependency>
    <groupId>org.springframework.security.kerberos</groupId>
    <artifactId>spring-security-kerberos-web</artifactId>
    <version>${kerberos.extension.version}</version>
</dependency>
<dependency>
    <groupId>org.springframework.security.kerberos</groupId>
    <artifactId>spring-security-kerberos-client</artifactId>
    <version>${kerberos.extension.version}</version>
</dependency>
```

这些依赖项可以从 [Maven Central](https://web.archive.org/web/20221025183651/https://search.maven.org/search?q=g:org.springframework.security.kerberos) 下载。

### 6.2.SPNEGO 配置

首先，SPNEGO 作为`HTTPSecurity`中的`Filter`集成到 Spring Security 中:

```java
@Override
protected void configure(HttpSecurity http) throws Exception {
    http.authorizeRequests()
      .anyRequest()
      .authenticated()
    .and()
      .addFilterBefore(
        spnegoAuthenticationProcessingFilter(authenticationManagerBean()),
        BasicAuthenticationFilter.class);
}
```

这仅显示了配置 SPNEGO `Filter`所需的部分，并不是完整的`HTTPSecurity`配置，应根据应用安全要求进行配置。

接下来，我们需要提供 SPNEGO `Filter`作为`Bean`:

```java
@Bean
public SpnegoAuthenticationProcessingFilter spnegoAuthenticationProcessingFilter(
  AuthenticationManager authenticationManager) {
    SpnegoAuthenticationProcessingFilter filter = new SpnegoAuthenticationProcessingFilter();
    filter.setAuthenticationManager(authenticationManager);
    return filter;
}
```

### 6.3.Kerberos 配置

此外，我们可以通过在 Spring Security 中添加`AuthenticationProvider`到`AuthenticationManagerBuilder`来配置 Kerberos:

```java
@Override
protected void configure(AuthenticationManagerBuilder auth) throws Exception {
    auth
      .authenticationProvider(kerberosAuthenticationProvider())
      .authenticationProvider(kerberosServiceAuthenticationProvider());
}
```

我们首先要提供一个`KerberosAuthenticationProvider`作为`Bean`。这是`AuthenticationProvider`的一个实现，这里我们将`SunJaasKerberosClient`设置为`KerberosClient`:

```java
@Bean
public KerberosAuthenticationProvider kerberosAuthenticationProvider() {
    KerberosAuthenticationProvider provider = new KerberosAuthenticationProvider();
    SunJaasKerberosClient client = new SunJaasKerberosClient();
    provider.setKerberosClient(client);
    provider.setUserDetailsService(userDetailsService());
    return provider;
}
```

接下来，我们还必须提供一个`KerberosServiceAuthenticationProvider`作为一个`Bean`。这是验证 Kerberos 服务票证或 SPNEGO 令牌的类:

```java
@Bean
public KerberosServiceAuthenticationProvider kerberosServiceAuthenticationProvider() {
    KerberosServiceAuthenticationProvider provider = new KerberosServiceAuthenticationProvider();
    provider.setTicketValidator(sunJaasKerberosTicketValidator());
    provider.setUserDetailsService(userDetailsService());
    return provider;
}
```

最后，我们需要提供一个`SunJaasKerberosTicketValidator`作为一个`Bean`。这是一个`KerberosTicketValidator`的实现，使用的是孙 JAAS 的登录模块:

```java
@Bean
public SunJaasKerberosTicketValidator sunJaasKerberosTicketValidator() {
    SunJaasKerberosTicketValidator ticketValidator = new SunJaasKerberosTicketValidator();
    ticketValidator.setServicePrincipal("HTTP/[[email protected]](/web/20221025183651/https://www.baeldung.com/cdn-cgi/l/email-protection)");
    ticketValidator.setKeyTabLocation(new FileSystemResource("baeldung.keytab"));
    return ticketValidator;
}
```

### 6.4.用户详细信息

我们已经在前面的`AuthenticationProvider`中看到了对`UserDetailsService`的引用，那么我们为什么还需要它呢？正如我们所了解的，Kerberos 是一种纯粹的基于票证的身份验证机制。

因此，虽然它能够识别用户，但它不提供与用户相关的其他细节，如他们的授权。我们需要一个有效的`UserDetailsService`提供给我们的`AuthenticationProvider`来填补这个空白。

### 6.5.运行应用程序

这正是我们为 SPNEGO 和 Kerberos 设置一个启用了 Spring 安全的 web 应用程序所需要的。当我们启动 web 应用程序并访问其中的任何页面时，web 浏览器应该提示输入用户名和密码，准备一个带有服务票的 SPNEGO 令牌，并将其发送给应用程序。

应用程序应该能够使用 keytab 文件中的凭证对其进行处理，并以成功的身份验证作为响应。

然而，正如我们前面所看到的，设置一个工作的 Kerberos 环境是复杂且相当脆弱的。如果事情没有按预期进行，那么重新检查所有步骤是值得的。像域名不匹配这样的简单错误可能会导致失败，并显示一些不是特别有用的错误消息。

## 7.SPNEGO 和 Kerberos 的实际使用

既然我们已经看到了 Kerberos 身份验证是如何工作的，以及如何在 web 应用程序中使用 SPNEGO 和 Kerberos，我们可能会质疑它的必要性。虽然在企业网络中将它用作 SSO 机制是完全有意义的，但是我们为什么要在 web 应用程序中使用它呢？

首先，即使过了这么多年，Kerberos 仍然在企业应用程序中非常活跃地使用，尤其是基于 Windows 的应用程序。如果一个组织有几个内部和外部的 web 应用程序，那么**扩展相同的 SSO 基础设施来覆盖它们是有意义的**。这使得组织的管理员和用户更容易通过不同的应用程序获得无缝体验。

## 8.结论

总之，在本教程中，我们了解了 Kerberos 身份验证协议的基础。我们还讨论了作为 GSS API 一部分的 SPNEGO，以及我们如何使用它在基于 HTTP 的 web 应用程序中促进基于 Kerberos 的身份验证。此外，我们试图利用 Spring Security 对 SPNEGO 和 Kerberos 的内置支持来构建一个小型 web 应用程序。

本教程只是提供了一个强大的、经过时间考验的认证机制的快速预览。有相当丰富的信息可供我们了解更多，甚至可能欣赏更多！

和往常一样，代码可以在 GitHub 上找到[。](https://web.archive.org/web/20221025183651/https://github.com/eugenp/tutorials/tree/master/spring-security-modules/spring-security-oauth2-sso/spring-security-sso-kerberos)