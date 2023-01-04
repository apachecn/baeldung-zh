# Spring Security Kerberos 与 MiniKdc 的集成

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-security-kerberos-integration>

## 1.概观

在本教程中，我们将概述 Spring Security Kerberos。

我们将用 Java 编写一个 Kerberos 客户机，它授权自己访问我们的 Kerberos 化服务。我们将运行我们自己的嵌入式密钥分发中心来执行完整的端到端 Kerberos 身份验证。所有这些都不需要任何外部基础设施，这要归功于 Spring Security Kerberos。

## 2.Kerberos 及其优势

Kerberos 是一种网络认证协议，由 MIT 在 20 世纪 80 年代创建，特别适用于在网络上集中认证。

1987 年，麻省理工学院将它发布给开源社区，现在它仍在积极开发中。2005 年，它在[RFC 4120](https://web.archive.org/web/20220628123649/https://tools.ietf.org/html/rfc4120) 下被列为 IETF 标准。

通常，Kerberos 在企业环境中使用。在那里，它以这样一种方式保护环境，即**用户不必分别对每个服务进行认证**。这种架构解决方案被称为 **`Single Sign-on`** 。

简单来说，Kerberos 是一个票务系统。用户**认证一次**，**收到一张赠票票** (TGT)。**然后，网络基础设施向 TGT 交换服务票。**这些服务票允许用户与基础设施服务交互，只要 TGT 有效，通常是几个小时。

所以，用户只需登录一次就很棒了。但是还有一个安全好处:在这样的环境中，**用户的密码永远不会通过网络**发送。相反，Kerberos 将它作为一个因子来生成另一个密钥，该密钥将用于消息加密和解密。

另一个好处是**我们可以从一个中心位置管理用户，**比如说一个由 LDAP 支持的位置。因此，如果我们在我们的中央数据库中为一个给定的用户禁用一个帐户，那么我们将撤销他在我们的基础设施中的访问。因此，管理员不必在每个服务中分别撤销访问权。

[Spring 中的 SPNEGO/Kerberos 认证简介](/web/20220628123649/https://www.baeldung.com/spring-security-kerberos)提供了该技术的深入概述。

## 3.Kerberized 化环境

因此，让我们创建一个使用 Kerberos 协议进行身份验证的环境。该环境将由三个同时运行的独立应用程序组成。

首先，**我们将有一个密钥分发中心**，它将作为认证点。接下来，我们将编写一个客户端和一个服务应用程序，我们将把它们配置为使用 Kerberos 协议。

现在，运行 Kerberos 需要一些安装和配置。然而，我们将利用 [Spring Security Kerberos](https://web.archive.org/web/20220628123649/https://docs.spring.io/spring-security-kerberos/docs/1.0.1.RELEASE/reference/htmlsingle/) ，因此我们将在嵌入式模式下以编程方式运行密钥分发中心。此外，下面显示的`MiniKdc`在使用 Kerberized 化的基础设施进行集成测试的情况下非常有用。

### 3.1.运营密钥分发中心

首先，我们将推出我们的主要配送中心，它将为我们发放 TGTs:

```
String[] config = MiniKdcConfigBuilder.builder()
  .workDir(prepareWorkDir())
  .principals("client/localhost", "HTTP/localhost")
  .confDir("minikdc-krb5.conf")
  .keytabName("example.keytab")
  .build();

MiniKdc.main(config);
```

基本上，我们已经给了`MiniKdc` 一组主体和一个配置文件；此外，我们已经告诉`MiniKdc`如何称呼它生成的`[keytab](https://web.archive.org/web/20220628123649/https://web.mit.edu/kerberos/krb5-devel/doc/basic/keytab_def.html)` 。

`MiniKdc` 将生成一个 [`krb5.conf`](https://web.archive.org/web/20220628123649/https://web.mit.edu/kerberos/krb5-1.12/doc/admin/conf_files/krb5_conf.html) 文件，我们将把它提供给我们的客户端和服务应用程序。该文件包含在哪里可以找到我们的 KDC 的信息——给定领域的主机和端口。

`MiniKdc.main` 启动 KDC，应该会输出如下内容:

```
Standalone MiniKdc Running
---------------------------------------------------
  Realm           : EXAMPLE.COM
  Running at      : localhost:localhost
  krb5conf        : .\spring-security-sso\spring-security-sso-kerberos\krb-test-workdir\krb5.conf

  created keytab  : .\spring-security-sso\spring-security-sso-kerberos\krb-test-workdir\example.keytab
  with principals : [client/localhost, HTTP/localhost]
```

### 3.2.客户应用程序

我们的客户端将是一个 Spring Boot 应用程序，它使用一个`RestTemplate `来调用外部 REST API。

但是，我们打算用**代替**。它需要密钥表和客户端的主体:

```
@Configuration
public class KerberosConfig {

    @Value("${app.user-principal:client/localhost}")
    private String principal;

    @Value("${app.keytab-location}")
    private String keytabLocation;

    @Bean
    public RestTemplate restTemplate() {
        return new KerberosRestTemplate(keytabLocation, principal);
    }
}
```

就是这样！为我们协商 Kerberos 协议的客户端。

因此，让我们创建一个快速类，它将从一个托管在端点`app.access-url`的 Kerberized 化服务中查询一些数据:

```
@Service
class SampleClient {

    @Value("${app.access-url}")
    private String endpoint;

    private RestTemplate restTemplate;

    // constructor, getter, setter

    String getData() {
        return restTemplate.getForObject(endpoint, String.class);
    }
}
```

所以，现在让我们创建我们的服务应用程序，这样这个类就有东西可以调用了！

### 3.3.服务应用

我们将使用 Spring Security，用适当的特定于 Kerberos 的 beans 来配置它。

此外，请注意，服务将有其主体，并且也使用 keytab:

```
@Configuration
@EnableWebSecurity
public class WebSecurityConfig extends WebSecurityConfigurerAdapter {

    @Value("${app.service-principal:HTTP/localhost}")
    private String servicePrincipal;

    @Value("${app.keytab-location}")
    private String keytabLocation;

    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http
          .authorizeRequests()
            .antMatchers("/", "/home").permitAll()
            .anyRequest().authenticated()
            .and() 
          .exceptionHandling()
            .authenticationEntryPoint(spnegoEntryPoint())
            .and()
          .formLogin()
            .loginPage("/login").permitAll()
            .and()
          .logout().permitAll()
            .and()
          .addFilterBefore(spnegoAuthenticationProcessingFilter(authenticationManagerBean()),
            BasicAuthenticationFilter.class);
    }

    @Override
    protected void configure(AuthenticationManagerBuilder auth) throws Exception {
        auth
          .authenticationProvider(kerberosAuthenticationProvider())
          .authenticationProvider(kerberosServiceAuthenticationProvider());
    }

    @Bean
    public KerberosAuthenticationProvider kerberosAuthenticationProvider() {
        KerberosAuthenticationProvider provider = new KerberosAuthenticationProvider();
        // provider configuration
        return provider;
    }

    @Bean
    public SpnegoEntryPoint spnegoEntryPoint() {
        return new SpnegoEntryPoint("/login");
    }

    @Bean
    public SpnegoAuthenticationProcessingFilter spnegoAuthenticationProcessingFilter(
      AuthenticationManager authenticationManager) {
        SpnegoAuthenticationProcessingFilter filter = new SpnegoAuthenticationProcessingFilter();
        // filter configuration
        return filter;
    }

    @Bean
    public KerberosServiceAuthenticationProvider kerberosServiceAuthenticationProvider() {
        KerberosServiceAuthenticationProvider provider = new KerberosServiceAuthenticationProvider();
        // auth provider configuration  
        return provider;
    }

    @Bean
    public SunJaasKerberosTicketValidator sunJaasKerberosTicketValidator() {
        SunJaasKerberosTicketValidator ticketValidator = new SunJaasKerberosTicketValidator();
        // validator configuration
        return ticketValidator;
    }
}
```

[介绍文章](/web/20220628123649/https://www.baeldung.com/spring-security-kerberos)包含了上面所有的实现，所以为了简洁起见，我们在这里省略了所有的方法。

注意，我们已经为 [SPNEGO 认证](https://web.archive.org/web/20220628123649/https://tools.ietf.org/html/rfc4559)配置了 Spring 安全性。这样，我们将能够通过 HTTP 协议进行认证，尽管我们也可以用核心 Java 实现 [SPNEGO 认证。](https://web.archive.org/web/20220628123649/https://docs.oracle.com/en/java/javase/11/security/part-vi-http-spnego-authentication.html)

## 4.测试

现在，我们将运行一个集成测试来展示我们的客户端通过 Kerberos 协议成功地从外部服务器获取数据。为了运行这个测试，我们需要让我们的基础设施运行，所以`MiniKdc`和我们的服务应用程序都必须启动。

基本上，我们将使用来自客户端应用程序的`SampleClient`向我们的服务应用程序发出请求。让我们来测试一下:

```
@Autowired
private SampleClient sampleClient;

@Test
public void givenKerberizedRestTemplate_whenServiceCall_thenSuccess() {
    assertEquals("data from kerberized server", sampleClient.getData());
}
```

注意，我们也可以通过点击没有它的服务来证明`KerberizedRestTemplate`的重要性:

```
@Test
public void givenRestTemplate_whenServiceCall_thenFail() {
    sampleClient.setRestTemplate(new RestTemplate());
    assertThrows(RestClientException.class, sampleClient::getData);
}
```

顺便提一下，我们的第二个测试有可能重用已经存储在[凭证缓存](https://web.archive.org/web/20220628123649/https://web.mit.edu/kerberos/krb5-1.12/doc/basic/ccache_def.html) 中的票。由于在`HttpUrlConnection`中使用了自动 SPNEGO 协商，所以会发生这种情况。

结果，**数据可能实际返回，使我们的测试无效。**根据我们的需要，我们可以通过系统属性`http.use.global.creds=false.`禁用票证缓存

## 5.结论

在本教程中，**我们探讨了 Kerberos 的集中式用户管理**以及 Spring Security 如何支持 Kerberos 协议和 SPNEGO 认证机制。

我们用`MiniKdc`建立了一个嵌入式 KDC，还创建了一个非常简单的 Kerberos 客户端和服务器。这种设置便于探索，尤其是在我们创建集成测试来进行测试的时候。

现在，我们只是触及了表面。要更深入，查看 Kerberos [wiki 页面](https://web.archive.org/web/20220628123649/https://en.wikipedia.org/wiki/Kerberos_(protocol))或[其 RFC](https://web.archive.org/web/20220628123649/https://tools.ietf.org/html/rfc4120) 。另外，[官方文档页面](https://web.archive.org/web/20220628123649/https://web.mit.edu/kerberos/krb5-latest/doc/)将会很有用。除此之外，为了了解如何在核心 java 中完成这些事情，甲骨文教程后面的[详细展示了这一点。](https://web.archive.org/web/20220628123649/https://docs.oracle.com/en/java/javase/11/security/advanced-security-programming-java-se-authentication-secure-communication-and-single-sign1.html)

像往常一样，代码可以在我们的 [GitHub](https://web.archive.org/web/20220628123649/https://github.com/eugenp/tutorials/tree/master/spring-security-modules/spring-security-oauth2-sso/spring-security-sso-kerberos) 页面上找到。