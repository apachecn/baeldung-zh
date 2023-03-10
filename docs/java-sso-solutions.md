# Java 应用程序的单点登录解决方案

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-sso-solutions>

 ![](img/42f20b9df5295a924f5f4cd094e452e1.png)

It’s just plain hard to get true, **real-time visibility into a running auth flow.**

Parts of the process can be completely hidden from us; if the complete authorization process requires a redirect from a remote OAuth production server, then every debugging effort must go through the production server.

It’s practically unfeasible to debug this locally. There’s no way to reproduce the exact state and no way to inspect what is actually happening under the hood. Not ideal.

Knowing these types of challenges, we built Lightrun - a real-time production debugging tool - to allow you to understand complicated flows with code-level information. Add logs, take snapshots (virtual breakpoints), and instrument metrics without a remote debugger, without stopping the running service, and, most importantly - **in real-time and without side effects**.

**Learn more with this 5-minute tutorial** focused on debugging these kinds of scenarios using Lightrun:

[>> Debugging Authentication and Authorization Using Lightrun](/web/20220524055332/https://www.baeldung.com/lightrun-n-security)

## 1.概观

拥有多个应用程序的组织用户通常需要跨多个系统进行身份验证。因此，用户必须记住多个帐户和密码。单点登录( [SSO](/web/20220524055332/https://www.baeldung.com/cs/sso-guide) )技术是这个问题的解决方案。 **SSO 为一组系统提供单一登录凭证**。

在本教程中，我们将简要解释什么是 SSO，然后我们将看一下针对 Java 应用程序的七种不同的 SSO 解决方案。

## 2.单点登录

**可以使用两种协议中的任何一种来实现 SSO 解决方案:**

*   SAML 2.0
*   OpenID 连接

[SAML 2.0](https://web.archive.org/web/20220524055332/http://docs.oasis-open.org/security/saml/Post2.0/sstc-saml-tech-overview-2.0.html) (安全断言标记语言)简化了用户认证。它只允许用户在身份提供者处注册和认证，以访问多种服务。它是基于 XML 的。 [OpenID Connect](https://web.archive.org/web/20220524055332/https://openid.net/connect/) (OIDC)是 SAML 2.0 的继任者。而且，它是对用于认证的 [OAuth 2.0](https://web.archive.org/web/20220524055332/https://oauth.net/2/) 协议的扩展。OIDC 的配置比 SAML 2.0 简单。

## 3.钥匙锁

**[Keycloak](https://web.archive.org/web/20220524055332/https://www.keycloak.org/) 是一个开源的身份和访问管理(IAM)系统**。它提供了 SSO、用户联盟、细粒度授权、[社交登录](https://web.archive.org/web/20220524055332/https://en.wikipedia.org/wiki/Social_login)、[双因素认证(2FA)](https://web.archive.org/web/20220524055332/https://en.wikipedia.org/wiki/Multi-factor_authentication) 等特性。此外，它还支持 OpenID Connect、OAuth 2.0 和 SAML。它与第三方工具有很好的集成。例如，它与 [Spring Boot](/web/20220524055332/https://www.baeldung.com/spring-boot-keycloak) 应用程序集成得非常好。最新版本可以在[这里](https://web.archive.org/web/20220524055332/https://www.keycloak.org/downloads)找到。此外，它还为管理员和开发人员提供了一个友好的管理控制台来配置和管理 Keycloak。源代码可在 [GitHub](https://web.archive.org/web/20220524055332/https://github.com/keycloak/keycloak) 上获得。

## 4.WSO2 身份服务器

**[WSO2 身份服务器](https://web.archive.org/web/20220524055332/https://wso2.com/identity-server/)是由 [WSO2](https://web.archive.org/web/20220524055332/https://wso2.com/) 开发的开源 IAM 系统**。它提供了 SSO、2FA、身份联盟、社交登录等等。它还支持几乎所有流行的身份标准。此外，它还提供了一个管理控制台，并公开了用于与其他应用程序集成的 API。不过主要是用 Java 写的，源代码在 [GitHub](https://web.archive.org/web/20220524055332/https://github.com/wso2/product-is) 上有。

## 5\. Gluu

**[Gluu](https://web.archive.org/web/20220524055332/https://gluu.org/) 是一个开源的云原生 IAM 解决方案**，具有多种访问管理特性。它提供强认证、移动认证、2FA 和身份代理。此外，它还支持开放的 web 标准，如 OpenID Connect、SAML 2.0、FIDO 和用户管理的访问。它是用 [Python](https://web.archive.org/web/20220524055332/https://www.python.org/) 语言编写的。另外，自动化 Gluu 服务器部署和配置的脚本可以在 [GitHub](https://web.archive.org/web/20220524055332/https://github.com/GluuFederation/community-edition-setup) 上获得。

## 6.开启 CAS

**[Apereo CAS](https://web.archive.org/web/20220524055332/https://www.apereo.org/projects/cas) 是一个开源的企业级 SSO 系统**。此外，它还是中央认证服务(CAS)项目的一部分。与以前的解决方案类似，它支持 SAML、OAuth 2.0、OpenID Connect 等多种协议。此外，它可以与 uPortal，BlueSocket，TikiWiki，Mule，Liferay，Moodle 等集成。它建立在和春云之上。源代码可以在 [GitHub](https://web.archive.org/web/20220524055332/https://github.com/apereo/cas) 上获得。

## 7.Spring 安全 OAuth2

我们可以使用 [Spring Security OAuth](/web/20220524055332/https://www.baeldung.com/sso-spring-security-oauth2-legacy) 项目来实现 SSO 解决方案。它支持 OAuth 提供者和 OAuth 消费者。此外，我们可以用一个软令牌和 Spring 安全来实现 [2FA](/web/20220524055332/https://www.baeldung.com/spring-security-two-factor-authentication-with-soft-token) 功能。

## 8.OpenAM

**[OpenAM](https://web.archive.org/web/20220524055332/https://www.openidentityplatform.org/openam) 是一个开放访问管理解决方案，包括认证、授权、单点登录和身份提供者。**支持跨域单点登录(CDSSO)、SAML 2.0、OAuth 2.0、OpenID Connect。最新版本和源代码可以在[这里](https://web.archive.org/web/20220524055332/https://github.com/OpenIdentityPlatform/OpenAM/)找到。

## 9.奥特利亚

**[Authelia](https://web.archive.org/web/20220524055332/https://www.authelia.com/) 是一个开源的认证授权服务器，提供 SSO 和 2FA** 。它提供了几个基于硬件的 2FA，利用了 [FIDO2](https://web.archive.org/web/20220524055332/https://en.wikipedia.org/wiki/FIDO2_Project) [Webauthn](https://web.archive.org/web/20220524055332/https://en.wikipedia.org/wiki/WebAuthn) 兼容的安全密钥。此外，它支持由 Google Authenticator 等应用程序生成的基于时间的一次性密码。Authelia 服务器是用 [Go](https://web.archive.org/web/20220524055332/https://go.dev/) 语言编写的，它的所有源代码都可以在 [GitHub](https://web.archive.org/web/20220524055332/https://github.com/authelia/authelia) 上获得。

## 10.结论

如今，许多组织都在使用单点登录。在本文中，我们对 Java 生态系统中的 SSO 解决方案进行了高度概括。有些解决方案提供了完整的 IAM，而其他解决方案只提供了 SSO 服务器和身份验证方法。