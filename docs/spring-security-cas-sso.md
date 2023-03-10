# 具有 Spring 安全性的 CAS SSO

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-security-cas-sso>

## 1。概述

在本教程中，我们将了解 Apereo 中央认证服务(CAS ),并了解 Spring Boot 服务如何使用它进行认证。 [CAS](https://web.archive.org/web/20220707095557/https://apereo.github.io/cas/6.1.x/index.html) 是一个企业[单点登录(SSO)](/web/20220707095557/https://www.baeldung.com/cs/sso-guide) 解决方案，也是开源的。

What is SSO? When you log in to YouTube, Gmail and Maps with the same credentials, that's Single Sign-On. We're going to demonstrate this by setting up a CAS server and a Spring Boot app. The Spring Boot app will use CAS for authentication.

## 2。CAS 服务器设置

### 2.1。CAS 安装和依赖关系

服务器使用 Maven (Gradle) War 覆盖样式来简化设置和部署:

```java
git clone https://github.com/apereo/cas-overlay-template.git cas-server
```

这个命令将把`cas-overlay-template`克隆到`cas-server`目录中。

我们将涉及的一些方面包括 JSON 服务注册和 JDBC 数据库连接。因此，我们将把它们的模块添加到`build.gradle`文件的`dependencies`部分:

```java
compile "org.apereo.cas:cas-server-support-json-service-registry:${casServerVersion}"
compile "org.apereo.cas:cas-server-support-jdbc:${casServerVersion}"
```

让我们确保检查最新版本的 [casServer](https://web.archive.org/web/20220707095557/https://github.com/apereo/cas/releases) 。

### 2.2。CAS 服务器配置

在启动 CAS 服务器之前，我们需要添加一些基本配置。让我们从创建一个`cas-server/src/main/resources`文件夹开始，在这个文件夹中。随后在文件夹中也将创建`application.properties`:

```java
server.port=8443
spring.main.allow-bean-definition-overriding=true
server.ssl.key-store=classpath:/etc/cas/thekeystore
server.ssl.key-store-password=changeit
```

让我们继续创建上面配置中引用的密钥存储文件。首先，我们需要在`cas-server/src/main/resources`中创建文件夹`/etc/cas`和`/etc/cas/config`。

然后，我们需要将目录更改为`cas-server/src/main/resources/etc/cas` ，并运行命令生成`thekeystore`:

```java
keytool -genkey -keyalg RSA -alias thekeystore -keystore thekeystore -storepass changeit -validity 360 -keysize 2048
```

为了避免 SSL 握手错误，我们应该使用`localhost`作为名字和姓氏的值。我们也应该对组织名称和单位使用相同的名称。此外，我们需要将`thekeystore`导入到 JDK/JRE 中，我们将用它来运行我们的客户端应用程序:

```java
keytool -importkeystore -srckeystore thekeystore -destkeystore $JAVA11_HOME/jre/lib/security/cacerts
```

源和目标密钥库的密码是`**changeit**`。在 Unix 系统上，我们可能需要使用 admin ( `sudo`)权限运行这个命令。导入之后，我们应该重启所有正在运行的 Java 实例或者重启系统。

我们使用 JDK11 是因为它是 CAS 版本 6.1.x 所要求的。此外，我们定义了指向其主目录的环境变量$JAVA11_HOME。我们现在可以启动 CAS 服务器了:

```java
./gradlew run -Dorg.gradle.java.home=$JAVA11_HOME
```

**当应用程序启动时，我们会看到终端**上打印出“就绪”，服务器将在 [`https://localhost:8443`](https://web.archive.org/web/20220707095557/https://localhost:8443/) 可用。

### 2.3。CAS 服务器用户配置

我们还不能登录，因为我们还没有配置任何用户。CAS 有不同的[管理配置](https://web.archive.org/web/20220707095557/https://apereo.github.io/cas/6.1.x/configuration/Configuration-Server-Management.html)的方法，包括独立模式。让我们创建一个配置文件夹`cas-server/src/main/resources/etc/cas/config`，我们将在其中创建一个属性文件`cas.properties`。现在，我们可以在属性文件中定义一个静态用户:

```java
cas.authn.accept.users=casuser::Mellon
```

我们必须将配置文件夹的位置告知 CAS 服务器，以使设置生效。让我们更新`tasks.gradle`,这样我们可以从命令行将位置作为 JVM 参数传递:

```java
task run(group: "build", description: "Run the CAS web application in embedded container mode") {
    dependsOn 'build'
    doLast {
        def casRunArgs = new ArrayList<>(Arrays.asList(
          "-server -noverify -Xmx2048M -XX:+TieredCompilation -XX:TieredStopAtLevel=1".split(" ")))
        if (project.hasProperty('args')) {
            casRunArgs.addAll(project.args.split('\\s+'))
        }
        javaexec {
            main = "-jar"
            jvmArgs = casRunArgs
            args = ["build/libs/${casWebApplicationBinaryName}"]
            logger.info "Started ${commandLine}"
        }
    }
}
```

然后，我们保存文件并运行:

```java
./gradlew run
  -Dorg.gradle.java.home=$JAVA11_HOME
  -Pargs="-Dcas.standalone.configurationDirectory=/cas-server/src/main/resources/etc/cas/config"
```

**请注意`cas.standalone.configurationDirectory`的值是绝对路径**。我们现在可以去`https://localhost:8443`，用用户名`casuser`和密码`Mellon`登录。

## 3。CAS 客户端设置

我们将使用 [Spring Initializr](https://web.archive.org/web/20220707095557/https://start.spring.io/) 来生成一个 Spring Boot 客户端应用程序。它将有`Web`、`Security`、`Freemarker`和`DevTools` 依赖项。此外，我们还将为 [Spring Security CAS](https://web.archive.org/web/20220707095557/https://search.maven.org/classic/#search%7Cga%7C1%7Cg%3A%22org.springframework.security%22%20a%3A%22spring-security-cas%22) 模块添加对其`pom.xml`的依赖:

```java
<dependency>
    <groupId>org.springframework.security</groupId>
    <artifactId>spring-security-cas</artifactId>
    <versionId>5.3.0.RELEASE</versionId>
</dependency>
```

最后，让我们添加以下 Spring Boot 属性来配置应用程序:

```java
server.port=8900
spring.freemarker.suffix=.ftl
```

## 4。CAS 服务器服务注册

**客户端应用程序必须在任何认证之前向 CAS 服务器注册**。CAS 服务器支持使用 YAML、JSON、MongoDB 和 LDAP 客户端注册表。

在本教程中，我们将使用 JSON 服务注册方法。让我们创建另一个文件夹`cas-server/src/main/resources/etc/cas/services`。这个文件夹将存放服务注册 JSON 文件。

我们将创建一个 JSON 文件，其中包含我们的客户机应用程序的定义。文件名`casSecuredApp-8900.json,`遵循模式 s `erviceName-Id.json`:

```java
{
  "@class" : "org.apereo.cas.services.RegexRegisteredService",
  "serviceId" : "http://localhost:8900/login/cas",
  "name" : "casSecuredApp",
  "id" : 8900,
  "logoutType" : "BACK_CHANNEL",
  "logoutUrl" : "http://localhost:8900/exit/cas"
}
```

`serviceId`属性为客户端应用程序定义了一个 regex URL 模式。该模式应该与客户端应用程序的 URL 相匹配。

`id`属性应该是唯一的。换句话说，不应该有两个或更多的具有相同的`id`的服务注册到同一个 CAS 服务器。拥有重复的`id` 会导致配置的冲突和覆盖。

**We also configure the logout type to be `BACK_CHANNEL` and the URL to be [`http://localhost:8900/exit/cas`](https://web.archive.org/web/20220707095557/http://localhost:8900/exit/cas) so that we can do single logout later.**Before the CAS server can make use of our JSON configuration file, we have to enable the JSON registry in our `cas.properties`:

```java
cas.serviceRegistry.initFromJson=true
cas.serviceRegistry.json.location=classpath:/etc/cas/services
```

## 5。CAS 客户端单点登录配置

我们的下一步是配置 Spring Security 来与 CAS 服务器协同工作。我们还应该检查被称为 CAS 序列的[完整的交互流](https://web.archive.org/web/20220707095557/https://docs.spring.io/spring-security/site/docs/5.2.2.RELEASE/reference/html5/#cas-sequence)。

让我们将以下 bean 配置添加到 Spring Boot 应用程序的`CasSecuredApplication`类中:

```java
@Bean
public CasAuthenticationFilter casAuthenticationFilter(
  AuthenticationManager authenticationManager,
  ServiceProperties serviceProperties) throws Exception {
    CasAuthenticationFilter filter = new CasAuthenticationFilter();
    filter.setAuthenticationManager(authenticationManager);
    filter.setServiceProperties(serviceProperties);
    return filter;
}

@Bean
public ServiceProperties serviceProperties() {
    logger.info("service properties");
    ServiceProperties serviceProperties = new ServiceProperties();
    serviceProperties.setService("http://cas-client:8900/login/cas");
    serviceProperties.setSendRenew(false);
    return serviceProperties;
}

@Bean
public TicketValidator ticketValidator() {
    return new Cas30ServiceTicketValidator("https://localhost:8443");
}

@Bean
public CasAuthenticationProvider casAuthenticationProvider(
  TicketValidator ticketValidator,
  ServiceProperties serviceProperties) {
    CasAuthenticationProvider provider = new CasAuthenticationProvider();
    provider.setServiceProperties(serviceProperties);
    provider.setTicketValidator(ticketValidator);
    provider.setUserDetailsService(
      s -> new User("[[email protected]](/web/20220707095557/https://www.baeldung.com/cdn-cgi/l/email-protection)", "Mellon", true, true, true, true,
      AuthorityUtils.createAuthorityList("ROLE_ADMIN")));
    provider.setKey("CAS_PROVIDER_LOCALHOST_8900");
    return provider;
}
```

`ServiceProperties` bean 与`casSecuredApp-8900.json`中的`serviceId`具有相同的 URL。这很重要，因为它向 CAS 服务器标识该客户端。

将`ServiceProperties`的`sendRenew`属性设置为`false`。这意味着用户只需向服务器提供一次登录凭证。

`AuthenticationEntryPoint` bean 将处理认证异常。因此，它会将用户重定向到 CAS 服务器的登录 URL 进行身份验证。

总之，身份验证流程如下:

1.  用户试图访问安全页面，这将触发身份验证异常
2.  异常触发`AuthenticationEntryPoint`。作为响应，`AuthenticationEntryPoint`将把用户带到 CAS 服务器登录页面—[`https://localhost:8443/login`](https://web.archive.org/web/20220707095557/https://localhost:8443/login)
3.  身份验证成功后，服务器使用一个票证重定向回客户端
4.  `CasAuthenticationFilter`将接听重定向并呼叫`CasAuthenticationProvider`
5.  `CasAuthenticationProvider`将使用`TicketValidator`在 CAS 服务器上确认提交的票据
6.  如果票是有效的，用户将被重定向到所请求的安全 URL

最后，让我们配置`HttpSecurity`来保护`WebSecurityConfig`中的一些路由。在这个过程中，我们还将添加异常处理的身份验证入口点:

```java
@Override
protected void configure(HttpSecurity http) throws Exception {
    http.authorizeRequests().antMatchers( "/secured", "/login") 
      .authenticated() 
      .and().exceptionHandling() 
      .authenticationEntryPoint(authenticationEntryPoint());
}
```

## 6。CAS 客户端单点注销配置

到目前为止，我们已经处理了单点登录；现在让我们考虑 CAS 单点注销(SLO)。

使用 CAS 管理用户身份验证的应用程序可以从两个位置注销用户:

*   客户端应用程序可以在本地注销用户——这不会影响用户在使用同一 CAS 服务器的其他应用程序中的登录状态
*   客户端应用程序也可以从 CAS 服务器注销用户–这将导致用户从连接到同一 CAS 服务器的所有其他客户端应用程序注销。

我们将首先在客户端应用程序上设置注销，然后将其扩展到 CAS 服务器上的单点注销。

为了使幕后发生的事情更加明显，我们将创建一个`logout()`方法来处理本地注销。如果成功，它会将我们重定向到一个带有单点注销链接的页面:

```java
@GetMapping("/logout")
public String logout(
  HttpServletRequest request, 
  HttpServletResponse response, 
  SecurityContextLogoutHandler logoutHandler) {
    Authentication auth = SecurityContextHolder
      .getContext().getAuthentication();
    logoutHandler.logout(request, response, auth );
    new CookieClearingLogoutHandler(
      AbstractRememberMeServices.SPRING_SECURITY_REMEMBER_ME_COOKIE_KEY)
      .logout(request, response, auth);
    return "auth/logout";
}
```

在单次注销过程中，CAS 服务器将首先使用户的票证过期，然后向所有注册的客户端应用程序发送异步请求。收到此信号的每个客户端应用程序将执行本地注销。从而达到注销一次的目的，就会造成到处注销。

说到这里，让我们为我们的客户端应用程序添加一些 bean 配置。具体来说，在`CasSecuredApplicaiton`中:

```java
@Bean
public SecurityContextLogoutHandler securityContextLogoutHandler() {
    return new SecurityContextLogoutHandler();
}

@Bean
public LogoutFilter logoutFilter() {
    LogoutFilter logoutFilter = new LogoutFilter("https://localhost:8443/logout",
      securityContextLogoutHandler());
    logoutFilter.setFilterProcessesUrl("/logout/cas");
    return logoutFilter;
}

@Bean
public SingleSignOutFilter singleSignOutFilter() {
    SingleSignOutFilter singleSignOutFilter = new SingleSignOutFilter();
    singleSignOutFilter.setCasServerUrlPrefix("https://localhost:8443");
    singleSignOutFilter.setLogoutCallbackPath("/exit/cas");
    singleSignOutFilter.setIgnoreInitConfiguration(true);
    return singleSignOutFilter;
}
```

`logoutFilter`将拦截对`/logout/cas`的请求，并将应用程序重定向到 CAS 服务器。`SingleSignOutFilter`将拦截来自 CAS 服务器的请求，并执行本地注销。

## 7。将 CAS 服务器连接到数据库

我们可以配置 CAS 服务器从 MySQL 数据库中读取凭证。我们将使用运行在本地机器上的 MySQL 服务器的`test`数据库。来更新一下`cas-server/src/main/resources/etc/cas/config/cas.properties`:

```java
cas.authn.accept.users=

cas.authn.jdbc.query[0].sql=SELECT * FROM users WHERE email = ?
cas.authn.jdbc.query[0].url=
  jdbc:mysql://127.0.0.1:3306/test?
  useUnicode=true&useJDBCCompliantTimezoneShift;=true&useLegacyDatetimeCode;=false&serverTimezone;=UTC
cas.authn.jdbc.query[0].dialect=org.hibernate.dialect.MySQLDialect
cas.authn.jdbc.query[0].user=root
cas.authn.jdbc.query[0].password=root
cas.authn.jdbc.query[0].ddlAuto=none
cas.authn.jdbc.query[0].driverClass=com.mysql.cj.jdbc.Driver
cas.authn.jdbc.query[0].fieldPassword=password
cas.authn.jdbc.query[0].passwordEncoder.type=NONE
```

**我们将`cas.authn.accept.users`设置为空白。这将停用 CAS 服务器对静态用户存储库的使用。**

根据上面的 SQL，用户的凭证存储在`users`表中。`email`列代表用户的委托人(`username`)。

请务必查看[支持的数据库、可用驱动程序和方言的列表](https://web.archive.org/web/20220707095557/https://apereo.github.io/cas/6.1.x/installation/JDBC-Drivers.html)。我们还将密码编码器类型设置为`NONE`。其他的[加密机制](https://web.archive.org/web/20220707095557/https://apereo.github.io/cas/6.1.x/configuration/Configuration-Properties-Common.html#password-encoding)和它们特有的属性也是可用的。

请注意，CAS 服务器数据库中的主体必须与客户端应用程序的主体相同。

让我们将`CasAuthenticationProvider`更新为与 CAS 服务器相同的用户名:

```java
@Bean
public CasAuthenticationProvider casAuthenticationProvider() {
    CasAuthenticationProvider provider = new CasAuthenticationProvider();
    provider.setServiceProperties(serviceProperties());
    provider.setTicketValidator(ticketValidator());
    provider.setUserDetailsService(
      s -> new User("[[email protected]](/web/20220707095557/https://www.baeldung.com/cdn-cgi/l/email-protection)", "Mellon", true, true, true, true,
      AuthorityUtils.createAuthorityList("ROLE_ADMIN")));
    provider.setKey("CAS_PROVIDER_LOCALHOST_8900");
    return provider;
}
```

`CasAuthenticationProvider`不使用密码进行认证。尽管如此，它的用户名必须与 CAS 服务器的用户名相匹配，才能成功进行身份验证。CAS 服务器要求 MySQL 服务器在`localhost`的`3306`端口运行。用户名和密码应该是`root`。

再次重启 CAS 服务器和 Spring Boot 应用程序。然后使用新凭证进行身份验证。

## 8。结论

我们已经了解了如何将 CAS SSO 与 Spring Security 一起使用，以及许多相关的配置文件。CAS SSO 的许多其他方面都是可配置的。从主题和协议类型到认证策略。

这些和其他的都在[文档](https://web.archive.org/web/20220707095557/https://apereo.github.io/cas/6.1.x/configuration/Configuration-Properties-Common.html)中。在 GitHub 上可以找到 [CAS 服务器](https://web.archive.org/web/20220707095557/https://github.com/eugenp/tutorials/tree/master/cas/cas-server)和 [Spring Boot 应用](https://web.archive.org/web/20220707095557/https://github.com/eugenp/tutorials/tree/master/cas/cas-secured-app)的源代码。