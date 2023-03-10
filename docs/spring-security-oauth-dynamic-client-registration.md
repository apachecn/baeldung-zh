# OAuth2.0 和动态客户端注册(使用 Spring Security OAuth 遗留堆栈)

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-security-oauth-dynamic-client-registration>

## 1。简介

在本教程中，我们将使用 OAuth2.0 准备一个动态客户端注册。OAuth2.0 是一个授权框架，支持对 HTTP 服务上的用户帐户进行有限的访问。OAuth2.0 客户端是希望访问用户帐户的应用程序。这个客户端可以是外部 web 应用程序、用户代理或者仅仅是本地客户端。

为了实现动态客户端注册，我们将凭证存储在数据库中，而不是硬编码的配置。我们要扩展的应用程序最初在 [Spring REST API + OAuth2 教程](/web/20220812064838/https://www.baeldung.com/rest-api-spring-oauth2-angular-legacy)中描述过。

**注**:本文使用的是 [Spring OAuth 遗留项目](https://web.archive.org/web/20220812064838/https://spring.io/projects/spring-security-oauth)。

## 2。Maven 依赖关系

我们将首先设置以下依赖关系集:

```java
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
</dependency>    
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-jdbc</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.security.oauth</groupId>
    <artifactId>spring-security-oauth2</artifactId>
</dependency>
```

注意，我们使用`spring-jdbc`是因为我们将使用一个 DB 来存储新注册的用户和密码。

## 3。OAuth2.0 服务器配置

首先，我们需要配置我们的 OAuth2.0 授权服务器。主要配置在以下类中:

```java
@Configuration
@PropertySource({ "classpath:persistence.properties" })
@EnableAuthorizationServer
public class OAuth2AuthorizationServerConfig
  extends AuthorizationServerConfigurerAdapter {

    // config
}
```

我们需要配置一些主要的东西；先说`ClientDetailsServiceConfigurer:`

```java
@Override
public void configure(final ClientDetailsServiceConfigurer clients) throws Exception {
    clients.jdbc(dataSource())

    // ...		
}
```

这将确保我们使用持久性来获取客户端信息。

当然，让我们建立这个标准数据源:

```java
@Bean
public DataSource dataSource() {
    DriverManagerDataSource dataSource = new DriverManagerDataSource();

    dataSource.setDriverClassName(env.getProperty("jdbc.driverClassName"));
    dataSource.setUrl(env.getProperty("jdbc.url"));
    dataSource.setUsername(env.getProperty("jdbc.user"));
    dataSource.setPassword(env.getProperty("jdbc.pass"));
    return dataSource;
}
```

因此，现在，我们的应用程序将使用数据库作为注册客户端的来源，而不是典型的硬编码在内存中的客户端。

## 4。DB 方案

现在让我们定义存储 OAuth 客户端的 SQL 结构:

```java
create table oauth_client_details (
    client_id VARCHAR(256) PRIMARY KEY,
    resource_ids VARCHAR(256),
    client_secret VARCHAR(256),
    scope VARCHAR(256),
    authorized_grant_types VARCHAR(256),
    web_server_redirect_uri VARCHAR(256),
    authorities VARCHAR(256),
    access_token_validity INTEGER,
    refresh_token_validity INTEGER,
    additional_information VARCHAR(4096),
    autoapprove VARCHAR(256)
);
```

我们应该关注的`oauth_client_details`中最重要的字段是:

*   `client_id` –存储新注册客户的 id
*   `client_secret`–存储客户的密码
*   `access_token_validity`–表示客户端是否仍然有效
*   `authorities`–指明特定客户允许的角色
*   `scope`–允许的动作，例如在脸书上写状态等。
*   *authorized_grant_types* ，它提供了用户如何登录特定客户端的信息(在我们的示例中，这是一个使用密码的表单登录)

请注意，每个客户端都与用户有一对多的关系，这自然意味着多个用户可以使用一个客户端。

## 5。让我们保留一些客户端

使用 SQL schema define，我们最终可以在系统中创建一些数据——并且基本上定义一个客户端。

我们将使用下面的`data.sql`脚本——Spring Boot 将默认运行该脚本——来初始化数据库:

```java
INSERT INTO oauth_client_details
	(client_id, client_secret, scope, authorized_grant_types,
	web_server_redirect_uri, authorities, access_token_validity,
	refresh_token_validity, additional_information, autoapprove)
VALUES
	("fooClientIdPassword", "secret", "foo,read,write,
	"password,authorization_code,refresh_token", null, null, 36000, 36000, null, true);
```

上一节介绍了`oauth_client_details`中最重要的字段。

## 6。测试

为了测试动态客户端注册，我们需要分别在 8081 和 8082 端口上运行`spring-security-oauth-server`和`spring-security-oauth-resource` 项目。

现在，我们终于可以编写几个现场测试了。

让我们假设，我们注册了一个名为`fooClientIdPassword`的客户端，它可以读取 foos。

首先，我们将尝试使用一个已经定义的客户机从 Auth 服务器获取一个访问令牌:

```java
@Test
public void givenDBUser_whenRevokeToken_thenAuthorized() {
    String accessToken = obtainAccessToken("fooClientIdPassword", "john", "123");

    assertNotNull(accessToken);
}
```

下面是获取访问令牌的逻辑:

```java
private String obtainAccessToken(String clientId, String username, String password) {
    Map<String, String> params = new HashMap<String, String>();
    params.put("grant_type", "password");
    params.put("client_id", clientId);
    params.put("username", username);
    params.put("password", password);
    Response response = RestAssured.given().auth().preemptive()
      .basic(clientId, "secret").and().with().params(params).when()
      .post("http://localhost:8081/spring-security-oauth-server/oauth/token");
    return response.jsonPath().getString("access_token");
}
```

## 7 .**。结论**

在本教程中，我们学习了如何向 OAuth2.0 框架动态注册无限数量的客户端。

本教程的完整实现可以在 GitHub 上找到[——这是一个基于 Maven 的项目，所以它应该很容易导入和运行。](https://web.archive.org/web/20220812064838/https://github.com/Baeldung/spring-security-oauth/tree/master/oauth-legacy "The Full Registration/Authentication Example Project on Github ")

请注意，为了进行测试，您需要将客户机添加到 DB 中，并且`.inMemory()`配置将不再有效。如果你想用旧的。`inMemory()` config，第二个文件包含硬编码客户端的配置。