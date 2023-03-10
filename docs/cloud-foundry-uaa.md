# 云铸造 UAA 快速使用指南

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/cloud-foundry-uaa>

## 1。概述

**Cloud Foundry 用户账户和认证(CF UAA)是一项身份管理和授权服务。**更准确地说，它是一个 OAuth 2.0 提供者，允许认证和向客户端应用程序发放令牌。

在本教程中，我们将涵盖设置一个 CF UAA 服务器的基础知识。然后我们将看看如何使用它来保护资源服务器应用程序。

但在此之前，让我们澄清一下 UAA 在 [OAuth 2.0](https://web.archive.org/web/20220911130933/https://auth0.com/docs/protocols/oauth2) 授权框架中的角色。

## 2。云代工厂 UAA 和 OAuth 2.0

让我们从理解 UAA 与 OAuth 2.0 规范的关系开始。

OAuth 2.0 规范定义了四个可以相互连接的参与者:资源所有者、资源服务器、客户端和授权服务器。

作为 OAuth 2.0 提供商，UAA 扮演了`authorization server.` 的角色，这意味着**它的主要目标是为`client`应用程序颁发访问令牌，并为`resource server`和**验证这些令牌

为了允许这些参与者进行交互，我们需要首先设置一个 UAA 服务器，然后再实现两个应用程序:一个作为客户机，另一个作为资源服务器。

我们将使用客户端的 [`authorization_code`授权](https://web.archive.org/web/20220911130933/https://tools.ietf.org/html/rfc6749#section-1.3.1)流。我们将对资源服务器使用不记名令牌授权。为了更安全有效的握手，我们将使用签名的 jwt 作为我们的[访问令牌](https://web.archive.org/web/20220911130933/https://tools.ietf.org/html/rfc6749#section-1.4)。

## 3。设置 UAA 服务器

首先，**我们将安装 UAA 并用一些演示数据填充它。**

安装完成后，我们将注册一个名为`webappclient.`的客户端应用程序，然后我们将创建一个名为`appuser`的用户，他有两个角色，`resource.read`和`resource.write`。

### 3.1。安装

UAA 是一个 Java web 应用程序，可以在任何兼容的 servlet 容器中运行。在本教程中，我们将[使用 Tomcat](https://web.archive.org/web/20220911130933/https://tomcat.apache.org/whichversion.html) 。

让我们继续**下载[UAA 战争](https://web.archive.org/web/20220911130933/https://search.maven.org/search?q=g:org.cloudfoundry.identity%20AND%20a:cloudfoundry-identity-uaa)并存放到我们的 Tomcat** 部署中:

```java
wget -O $CATALINA_HOME/webapps/uaa.war \
  https://search.maven.org/remotecontent?filepath=org/cloudfoundry/identity/cloudfoundry-identity-uaa/4.27.0/cloudfoundry-identity-uaa-4.27.0.war
```

但是，在启动它之前，我们需要配置它的数据源和 JWS 密钥对。

### 3.2.所需配置

**默认情况下，UAA 从其类路径上的`uaa.yml`读取配置。**但是，由于我们刚刚下载了`war`文件，我们最好告诉 UAA 一个文件系统上的自定义位置。

我们可以通过**设置`UAA_CONFIG_PATH`属性:**来做到这一点

```java
export UAA_CONFIG_PATH=~/.uaa
```

或者，我们可以设置`CLOUD_FOUNDRY_CONFIG_PATH.` ，或者，我们可以用`UAA_CONFIG_URL.`指定一个远程位置

然后，我们可以将 [UAA 所需的配置](https://web.archive.org/web/20220911130933/https://raw.githubusercontent.com/cloudfoundry/uaa/4.27.0/uaa/src/main/resources/required_configuration.yml)复制到我们的配置路径中:

```java
wget -qO- https://raw.githubusercontent.com/cloudfoundry/uaa/4.27.0/uaa/src/main/resources/required_configuration.yml \
  > $UAA_CONFIG_PATH/uaa.yml
```

请注意，我们删除了最后三行，因为我们稍后将替换它们。

### 3.3.配置数据源

因此，让我们配置数据源，**UAA 将在其中存储关于客户端的信息。**

出于本教程的目的，我们将使用 HSQLDB:

```java
export SPRING_PROFILES="default,hsqldb"
```

当然，由于这是一个 Spring Boot 应用程序，我们也可以在`uaa.yml `中将它指定为`spring.profiles`属性。

### 3.4.配置 JWS 密钥对

因为我们使用 JWT，UAA 需要一个私钥来签署 UAA 颁发的每个 JWT。

OpenSSL 使这变得简单:

```java
openssl genrsa -out signingkey.pem 2048
openssl rsa -in signingkey.pem -pubout -out verificationkey.pem
```

授权服务器将使用私钥进行 JWT，我们的客户端和资源服务器将使用公钥进行签名。

我们将它们导出到`JWT_TOKEN_SIGNING_KEY` 和`JWT_TOKEN_VERIFICATION_KEY`:

```java
export JWT_TOKEN_SIGNING_KEY=$(cat signingkey.pem)
export JWT_TOKEN_VERIFICATION_KEY=$(cat verificationkey.pem) 
```

同样，我们可以通过`jwt.token.signing-key`和`jwt.token.verification-key`属性在`uaa.yml`中指定这些。

### 3.5.启动 UAA

最后，让我们开始吧:

```java
$CATALINA_HOME/bin/catalina.sh run
```

此时，我们应该在 [`http://localhost:8080/uaa`](https://web.archive.org/web/20220911130933/http://localhost:8080/uaa) 有一个可用的 UAA 服务器。

如果我们去 [`http://localhost:8080/uaa/info`](https://web.archive.org/web/20220911130933/http://localhost:8080/uaa/info) ，那么我们会看到一些基本的启动信息

### 3.6。安装 UAA 命令行客户端

CF UAA 命令行客户端是管理 UAA 的主要工具，但是要使用它，我们需要[先安装 Ruby】:](https://web.archive.org/web/20220911130933/https://rubygems.org/)

```java
sudo apt install rubygems
gem install cf-uaac
```

然后，我们可以配置`uaac`指向我们正在运行的 UAA 实例:

```java
uaac target http://localhost:8080/uaa
```

注意，如果我们不想使用命令行客户端，我们当然可以使用 UAA 的 HTTP 客户端。

### 3.7。使用 UAAC 填充客户端和用户

现在我们已经安装了 *uaac* ，让我们用一些演示数据填充 UAA。**至少，我们需要:一个`client`，一个`user`，以及`resource.read`和`resource.write` 组。**

因此，要进行任何管理，我们都需要对自己进行身份验证。我们将选择 UAA 附带的默认管理员，**，该管理员有权创建其他客户端、用户和组:**

```java
uaac token client get admin -s adminsecret
```

(当然，**我们肯定需要在发货前通过 [oauth-clients.xml](https://web.archive.org/web/20220911130933/https://github.com/cloudfoundry/uaa/blob/master/uaa/src/main/webapp/WEB-INF/spring/oauth-clients.xml) 文件更改这个账户**！)

基本上，我们可以把这个命令理解为:“给我一个使用`client` 凭证的`token,` ，凭证的 client_id 是`admin`，ecret 是`adminsecret`。

如果一切顺利，我们将看到一条成功消息:

```java
Successfully fetched token via client credentials grant.
```

令牌存储在`uaac`的状态中。

**现在，作为`admin`操作，我们可以在` client add:`** 注册一个名为`webappclient`的客户端

```java
uaac client add webappclient -s webappclientsecret \ 
--name WebAppClient \ 
--scope resource.read,resource.write,openid,profile,email,address,phone \ 
--authorized_grant_types authorization_code,refresh_token,client_credentials,password \ 
--authorities uaa.resource \ 
--redirect_uri http://localhost:8081/login/oauth2/code/uaa
```

**同样，我们可以用`user add:`** 注册一个名为`appuser` 的用户

```java
uaac user add appuser -p appusersecret --emails [[email protected]](/web/20220911130933/https://www.baeldung.com/cdn-cgi/l/email-protection)
```

接下来，我们将添加两个组——`resource.read`和`resource.write`——使用` group add:`

```java
uaac group add resource.read
uaac group add resource.write
```

最后，我们将用` member add:`将这些组分配给`appuser`

```java
uaac member add resource.read appuser
uaac member add resource.write appuser
```

唷！到目前为止，我们所做的是:

*   已安装和配置的 UAA
*   已安装`uaac`
*   添加了演示客户端、用户和组

所以，让我们记住这些信息，然后跳到下一步。

## 4。OAuth 2.0 客户端

在本节中，**我们将使用 Spring Boot 创建一个 OAuth 2.0 客户端应用程序**。

### 4.1。应用程序设置

让我们从访问 [Spring Initializr](https://web.archive.org/web/20220911130933/https://start.spring.io/) 并生成一个 Spring Boot web 应用程序开始。我们只选择`Web` 和`OAuth2 Client`组件:

```java
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-oauth2-client</artifactId>
</dependency>
```

在这个例子中，我们使用了 Spring Boot 的版本 2.1.3 。

接下来，**我们需要注册我们的客户，`webapp`** `**client**.`

很简单，我们需要给应用程序赋予`client-id,` `client-secret,` 和 UAA 的`issuer-uri`。我们还将指定该客户端希望用户授予它的 OAuth 2.0 范围:

```java
#registration
spring.security.oauth2.client.registration.uaa.client-id=webappclient
spring.security.oauth2.client.registration.uaa.client-secret=webappclientsecret
spring.security.oauth2.client.registration.uaa.scope=resource.read,resource.write,openid,profile

#provider
spring.security.oauth2.client.provider.uaa.issuer-uri=http://localhost:8080/uaa/oauth/token
```

关于这些属性的更多信息，我们可以查看一下关于[注册](https://web.archive.org/web/20220911130933/https://docs.spring.io/spring-boot/docs/current/api/org/springframework/boot/autoconfigure/security/oauth2/client/OAuth2ClientProperties.Registration.html)和[提供者](https://web.archive.org/web/20220911130933/https://docs.spring.io/spring-boot/docs/current/api/org/springframework/boot/autoconfigure/security/oauth2/client/OAuth2ClientProperties.Provider.html)bean 的 Java 文档。

由于我们已经将端口 8080 用于 UAA，让我们在 8081 上运行:

```java
server.port=8081
```

### 4.2。登录

现在，如果我们访问`/login`路径，我们应该有一个所有注册客户端的列表。在我们的案例中，我们只有一个注册客户:

[![uaa-1](img/6905bc282fa55f3f7bddf8c20883361e.png)](/web/20220911130933/https://www.baeldung.com/wp-content/uploads/2019/04/uaa-1.png) 
**点击链接会将我们重定向到 UAA 登录页面:**

[![uaa1](img/ac05eaae90c4ce9dc3b216f0cf9d0c39.png)](/web/20220911130933/https://www.baeldung.com/wp-content/uploads/2019/04/uaa1.png)

在这里，我们用`appuser/appusersecret`登录。

**提交表单应该会将我们重定向到一个批准表单，用户可以在其中授权或拒绝访问我们的客户端:**

[![uaa2](img/fbfe5c40cdae373bb77da9ad872aa59e.png)](/web/20220911130933/https://www.baeldung.com/wp-content/uploads/2019/04/uaa2.png)

然后，用户可以授予她想要的特权。出于我们的目的，**我们将选择一切`except resource:write.`**

无论用户检查什么，都将是结果访问令牌中的范围。

为了证明这一点，我们可以复制索引路径中显示的令牌， [`http://localhost:8081`](https://web.archive.org/web/20220911130933/http://localhost:8081/) ，并使用 [JWT 调试器](https://web.archive.org/web/20220911130933/https://jwt.io/#debugger-io)对其进行解码。**我们应该会看到我们在批准页面上检查的范围:**

```java
{
  "jti": "f228d8d7486942089ff7b892c796d3ac",
  "sub": "0e6101d8-d14b-49c5-8c33-fc12d8d1cc7d",
  "scope": [
    "resource.read",
    "openid",
    "profile"
  ],
  "client_id": "webappclient"
  // more claims
}
```

一旦我们的客户端应用程序收到这个令牌，它就可以对用户进行身份验证，然后他们就可以访问这个应用程序了。

现在，**一个不显示任何数据的应用程序不是很有用，所以我们的下一步将是建立一个资源服务器**——它拥有用户的数据——并将客户端连接到它。

完整的资源服务器将有两个受保护的 API:一个需要`resource.read` 范围，另一个需要`resource.write.`

我们将看到的是**客户端，使用我们授予的作用域，将能够调用`read` API，但不能调用`write.`**

## 5。资源服务器

**资源服务器托管用户的受保护资源。**

它通过`Authorization`头认证客户端，并与授权服务器协商——在我们的例子中，就是 UAA。

### 5.1。应用程序设置

为了创建我们的资源服务器，我们将再次使用 [Spring Initializr](https://web.archive.org/web/20220911130933/https://start.spring.io/) 来生成一个 Spring Boot web 应用程序。这一次，我们将选择`Web`和`OAuth2 Resource Server`组件:

```java
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-oauth2-resource-server</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
</dependency>
```

与客户端应用程序一样，我们使用的是 Spring Boot 的版本 2.1.3。

**下一步是在`application.properties`文件中指出运行 CF UAA 的位置:**

```java
spring.security.oauth2.resourceserver.jwt.issuer-uri=http://localhost:8080/uaa/oauth/token
```

当然，我们也在这里选择一个新的端口。8082 将工作良好:

```java
server.port=8082
```

就是这样！我们应该有一个工作的资源服务器，默认情况下，所有请求都需要在`Authorization `头中有一个有效的访问令牌。

### 5.2。保护资源服务器 API

接下来，让我们添加一些值得保护的端点。

我们将添加一个具有两个端点的`RestController`,一个为具有`resource.read`范围的用户授权，另一个为具有`resource.write scope:`范围的用户授权

```java
@GetMapping("/read")
public String read(Principal principal) {
    return "Hello write: " + principal.getName();
}

@GetMapping("/write")
public String write(Principal principal) {
    return "Hello write: " + principal.getName();
}
```

接下来，**我们将覆盖默认的 Spring Boot 配置**来保护两个资源:

```java
@EnableWebSecurity
public class OAuth2ResourceServerSecurityConfiguration extends WebSecurityConfigurerAdapter {

   @Override
   protected void configure(HttpSecurity http) throws Exception {
      http.authorizeRequests()
        .antMatchers("/read/**").hasAuthority("SCOPE_resource.read")
        .antMatchers("/write/**").hasAuthority("SCOPE_resource.write")
        .anyRequest().authenticated()
      .and()
        .oauth2ResourceServer().jwt();
   }
}
```

**注意，当访问令牌中提供的作用域被翻译成 [Spring Security `GrantedAuthority`](/web/20220911130933/https://www.baeldung.com/role-and-privilege-for-spring-security-registration)** 时，它们的前缀是`SCOPE_`。

### 5.3。从客户端请求受保护的资源

在客户端应用程序中，我们将使用`RestTemplate.` 调用两个受保护的资源。在发出请求之前，我们从上下文中检索访问令牌，并将其添加到`Authorization` 头`:`

```java
private String callResourceServer(OAuth2AuthenticationToken authenticationToken, String url) {
    OAuth2AuthorizedClient oAuth2AuthorizedClient = this.authorizedClientService.
      loadAuthorizedClient(authenticationToken.getAuthorizedClientRegistrationId(), 
      authenticationToken.getName());
    OAuth2AccessToken oAuth2AccessToken = oAuth2AuthorizedClient.getAccessToken();

    HttpHeaders headers = new HttpHeaders();
    headers.add("Authorization", "Bearer " + oAuth2AccessToken.getTokenValue());

    // call resource endpoint

    return response;
}
```

不过，请注意，如果我们[使用`WebClient`而不是`RestTemplate`](https://web.archive.org/web/20220911130933/https://docs.spring.io/spring-security/site/docs/current/reference/htmlsingle/#servlet-webclient) ，我们可以删除这个样板文件。

然后，我们将向资源服务器端点添加两个调用:

```java
@GetMapping("/read")
public String read(OAuth2AuthenticationToken authenticationToken) {
    String url = remoteResourceServer + "/read";
    return callResourceServer(authenticationToken, url);
}

@GetMapping("/write")
public String write(OAuth2AuthenticationToken authenticationToken) {
    String url = remoteResourceServer + "/write";
    return callResourceServer(authenticationToken, url);
}
```

不出所料，**对/ `read` API 的调用会成功，但/ `write`不会成功。**HTTP 状态 403 告诉我们用户没有被授权。

## 6。结论

在本文中，我们首先简要概述了 OAuth 2.0，因为它是 OAuth 2.0 授权服务器 UAA 的基础。然后，我们对它进行了配置，以便为客户机颁发访问令牌并保护资源服务器应用程序。

Github 上的[提供了示例的完整源代码。](https://web.archive.org/web/20220911130933/https://github.com/eugenp/tutorials/tree/master/security-modules/cloud-foundry-uaa)