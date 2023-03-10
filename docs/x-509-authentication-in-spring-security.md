# Spring Security 中的 X.509 身份验证

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/x-509-authentication-in-spring-security>

## 1。概述

在本文中，我们将关注 X.509 证书认证的主要用例—**在使用 HTTPS (HTTP over SSL)协议时验证通信对等方的身份**。

简而言之——建立安全连接后，客户端根据其证书(由可信的证书颁发机构颁发)验证服务器。

但除此之外，Spring Security 中的 X.509 可以用来在连接时由服务器**验证客户端**的身份。这被称为 **`“mutual authentication”,`** ，我们也将看看这是如何实现的。

最后，当使用这种认证有意义时，我们将触及**。**

为了演示服务器验证，我们将创建一个简单的 web 应用程序，并在浏览器中安装一个定制的证书颁发机构。

此外，对于`mutual authentication`，我们将创建一个客户端证书，并修改我们的服务器，只允许经过验证的客户端。

强烈建议您按照本教程一步一步地学习，并根据下面几节中的说明自己创建证书以及密钥库和信任库。然而，所有现成的文件都可以在我们的 [GitHub 库](https://web.archive.org/web/20220822105735/https://github.com/eugenp/tutorials/tree/master/spring-security-modules/spring-security-web-x509/store)中找到。

## 2.自签名根 CA

为了能够签署我们的服务器端和客户端证书，我们需要首先创建我们自己的自签名根 CA 证书。这样**我们将充当我们自己的认证机构**。

为此，我们将使用 [openssl](https://web.archive.org/web/20220822105735/https://wiki.openssl.org/index.php/Binaries) 库，因此我们需要在进行下一步之前安装它。

现在让我们创建 CA 证书:

```java
openssl req -x509 -sha256 -days 3650 -newkey rsa:4096 -keyout rootCA.key -out rootCA.crt
```

当我们执行上面的命令时，我们需要为我们的私钥提供密码。出于本教程的目的，我们使用`changeit`作为密码短语。

此外，**我们需要输入形成所谓的识别名**的信息。在这里，我们只提供 CN(通用名称)-Baeldung.com-并留下其他部分为空。

[![rootCA](img/abe07bd3cfce212e4c409c88d6dbe4ec.png)](/web/20220822105735/https://www.baeldung.com/wp-content/uploads/2016/08/rootCA.jpg)

## 3。密钥库

**可选需求**:为了使用加密的强密钥以及加密和解密特性，我们需要在 JVM `.`中安装“`Java Cryptography Extension (JCE) Unlimited Strength Jurisdiction Policy Files`

例如，可以从 [Oracle](https://web.archive.org/web/20220822105735/http://www.oracle.com/technetwork/java/javase/downloads/jce8-download-2133166.html) 下载这些软件(遵循下载中包含的安装说明)。一些 Linux 发行版也通过它们的包管理器提供了一个可安装的包。

keystore 是一个存储库，我们的 Spring Boot 应用程序将使用它来保存服务器的私钥和证书。换句话说，我们的应用程序将在 SSL 握手期间使用密钥库向客户端提供证书。

在本教程中，我们使用 **Java Key-Store (JKS)格式和一个 [keytool](https://web.archive.org/web/20220822105735/https://docs.oracle.com/en/java/javase/11/tools/keytool.html) 命令行工具。**

### 3.1.服务器端证书

为了在我们的 Spring Boot 应用程序中实现服务器端 X.509 认证，我们首先需要创建一个服务器端证书。

让我们从创建所谓的证书签名请求(CSR)开始:

```java
openssl req -new -newkey rsa:4096 -keyout localhost.key –out localhost.csr
```

类似地，对于 CA 证书，我们必须提供私钥的密码。另外，让我们使用`localhost`作为常用名(CN)。

在我们继续之前，我们需要创建一个配置文件—`localhost.ext`。它将存储签名证书时需要的一些附加参数。

```java
authorityKeyIdentifier=keyid,issuer
basicConstraints=CA:FALSE
subjectAltName = @alt_names
[alt_names]
DNS.1 = localhost
```

在这里也有一个现成可用的文件[。](https://web.archive.org/web/20220822105735/https://github.com/eugenp/tutorials/blob/master/spring-security-modules/spring-security-web-x509/store/localhost.ext)

现在，是时候让**用我们的`rootCA.crt`证书和它的私钥**签署请求了:

```java
openssl x509 -req -CA rootCA.crt -CAkey rootCA.key -in localhost.csr -out localhost.crt -days 365 -CAcreateserial -extfile localhost.ext
```

请注意，我们必须提供创建 CA 证书时使用的相同密码。

在这个阶段，**我们终于有了一个由我们自己的证书颁发机构签署的可以使用的`localhost.crt`证书。**

要以人类可读的形式打印证书的详细信息，我们可以使用以下命令:

```java
openssl x509 -in localhost.crt -text
```

### 3.2.导入到密钥库

在这一节中，我们将看到如何**将签名的证书和相应的私钥导入到`keystore.jks`文件**中。

我们将使用 [PKCS 12 档案库](https://web.archive.org/web/20220822105735/https://en.wikipedia.org/wiki/PKCS_12)，将我们服务器的私钥和签名证书打包在一起。然后我们将它导入到新创建的`keystore.jks.`

我们可以使用下面的命令创建一个`.p12`文件:

```java
openssl pkcs12 -export -out localhost.p12 -name "localhost" -inkey localhost.key -in localhost.crt
```

所以我们现在将`localhost.key`和`localhost.crt`捆绑在一个`localhost.p12`文件中。

现在让我们使用 keytool 来**创建一个`keystore.jks`存储库，并使用一个命令**导入`localhost.p12`文件:

```java
keytool -importkeystore -srckeystore localhost.p12 -srcstoretype PKCS12 -destkeystore keystore.jks -deststoretype JKS
```

在这个阶段，我们已经为服务器认证部分做好了一切准备。让我们继续我们的 Spring Boot 应用程序配置。

## 4。应用示例

我们的 SSL 安全服务器项目由一个 [`@SpringBootApplication`](/web/20220822105735/https://www.baeldung.com/spring-boot-application-configuration) 带注释的应用程序类(是一种`[@Configuration](/web/20220822105735/https://www.baeldung.com/bootstraping-a-web-application-with-spring-and-java-based-configuration))`)、一个`application.properties`配置文件和一个非常简单的 MVC 风格的前端组成。

应用程序所要做的就是呈现一个带有`“Hello {User}!”`消息的 HTML 页面。这样，我们可以在浏览器中检查服务器证书，以确保连接得到验证和保护。

### 4.1.Maven 依赖性

首先，我们创建一个新的 Maven 项目，其中包含三个 Spring Boot 入门包:

```java
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-security</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-thymeleaf</artifactId>
</dependency>
```

**供参考:**我们可以在 Maven Central 上找到捆绑包([安全](https://web.archive.org/web/20220822105735/https://search.maven.org/classic/#search%7Cgav%7C1%7Cg%3A%22org.springframework.boot%22%20AND%20a%3A%22spring-boot-starter-security%22)、 [web](https://web.archive.org/web/20220822105735/https://search.maven.org/classic/#search%7Cgav%7C1%7Cg%3A%22org.springframework.boot%22%20AND%20a%3A%22spring-boot-starter-web%22) 、[百里叶](https://web.archive.org/web/20220822105735/https://search.maven.org/classic/#search%7Cgav%7C1%7Cg%3A%22org.springframework.boot%22%20AND%20a%3A%22spring-boot-starter-thymeleaf%22))。

### 4.2.Spring Boot 应用

下一步，我们创建主应用程序类和用户控制器:

```java
@SpringBootApplication
public class X509AuthenticationServer {
    public static void main(String[] args) {
        SpringApplication.run(X509AuthenticationServer.class, args);
    }
}

@Controller
public class UserController {
    @RequestMapping(value = "/user")
    public String user(Model model, Principal principal) {

        UserDetails currentUser 
          = (UserDetails) ((Authentication) principal).getPrincipal();
        model.addAttribute("username", currentUser.getUsername());
        return "user";
    }
}
```

现在，我们告诉应用程序在哪里可以找到我们的`keystore.jks` 以及如何访问它。我们将 SSL 设置为“enabled”状态，并将标准监听端口更改为**以指示安全连接。**

此外，我们配置了一些`user-details`来通过基本认证访问我们的服务器:

```java
server.ssl.key-store=../store/keystore.jks
server.ssl.key-store-password=${PASSWORD}
server.ssl.key-alias=localhost
server.ssl.key-password=${PASSWORD}
server.ssl.enabled=true
server.port=8443
spring.security.user.name=Admin
spring.security.user.password=admin
```

这将是 HTML 模板，位于`resources/templates`文件夹:

```java
<!DOCTYPE html>
<html xmlns:th="http://www.thymeleaf.org">
<head>
    <title>X.509 Authentication Demo</title>
</head>
<body>
    <h2>Hello <span th:text="${username}"/>!</h2>
</body>
</html>
```

### 4.3.根 CA 安装

在我们完成这一部分并查看站点之前，**我们需要在浏览器**中安装我们生成的根证书颁发机构作为可信证书。

我们为`Mozilla Firefox`安装的认证中心的示例如下所示:

1.  在地址栏中输入`about:preferences`
2.  打开`Advanced -> Certificates -> View Certificates -> Authorities`
3.  点击 **`Import`**
4.  找到`Baeldung tutorials`文件夹及其子文件夹`spring-security-x509/keystore`
5.  选择 **`rootCA.crt`** 文件，点击 **`OK`**
6.  选择`Trust this CA to identify websites”`，点击 **`OK`**

**注意:**如果你不想把我们的`certificate authority`加入到`**trusted authorities**`的名单中，你可以选择制作一个`exception`来显示网站的强硬，即使它被认为是不安全的。但是之后你会在地址栏看到一个‘黄色惊叹号’符号，表示连接不安全！

之后，我们将导航到`spring-security-x509-basic-auth`模块并运行:

```java
mvn spring-boot:run
```

最后，我们点击`[https://localhost:8443/user](https://web.archive.org/web/20220822105735/https://localhost:8443/user)`，输入来自`application.properties`的用户凭证，应该会看到一条`“Hello Admin!”`消息。现在，我们可以通过单击地址栏中的“绿锁”符号来检查连接状态，这应该是一个安全的连接。

[![Screenshot_20160822_205015](img/05fcccf6e302a9b98439682c9d43d5f2.png)](/web/20220822105735/https://www.baeldung.com/wp-content/uploads/2016/08/Screenshot_20160822_205015.png)

## 5。相互认证

在上一节中，我们介绍了如何实现最常见的 SSL 身份验证模式——服务器端身份验证。这意味着，只有服务器向客户端验证自己。

在本节中，**我们将描述如何添加认证的另一部分——客户端认证**。这样，只有拥有由我们的服务器信任的权威机构签署的有效证书的客户才能访问我们的安全网站。

但是在我们继续之前，让我们看看使用相互 SSL 认证的利弊。

**优点:**

*   X.509 **客户端证书的私钥比任何用户定义的密码**都强。但是必须保密！
*   有了证书，客户的**身份就众所周知，也很容易验证**。
*   再也不会忘记密码了！

**缺点:**

*   我们需要为每个新客户端创建一个证书。
*   客户端证书必须安装在客户端应用程序中。事实上: **X.509 客户端认证是依赖于设备的**，这使得在公共场所(例如网吧)无法使用这种认证。
*   必须有一种机制来撤销受损的客户端证书。
*   我们必须保留客户的证件。这很容易变得昂贵。

### 5.1.信任商店

在某种程度上，信任者与密钥库相反。**它持有我们信任的外部实体的证书**。

在我们的例子中，在信任库中保存根 CA 证书就足够了。

让我们看看如何创建一个`truststore.jks`文件并使用 keytool 导入`rootCA.crt`:

```java
keytool -import -trustcacerts -noprompt -alias ca -ext san=dns:localhost,ip:127.0.0.1 -file rootCA.crt -keystore truststore.jks
```

注意，我们需要为新创建的`trusstore.jks`提供密码。这里，我们再次使用了`changeit`密码。

就这样，我们已经导入了自己的 CA 证书，信任库已经可以使用了。

### 5.2.Spring 安全配置

为了继续，我们正在修改我们的`X509AuthenticationServer`以从 [`WebSecurityConfigurerAdapter`](/web/20220822105735/https://www.baeldung.com/spring-security-authentication-provider) 扩展，并覆盖所提供的配置方法之一。这里我们配置 x.509 机制来解析证书的`Common Name (CN)`字段，以提取用户名。

通过提取的用户名，Spring Security 在提供的`UserDetailsService`中查找匹配的用户。所以我们也实现了这个包含一个演示用户的服务接口。

**提示:**在生产环境中，这个`UserDetailsService`可以从一个 [JDBC 数据源](/web/20220822105735/https://www.baeldung.com/spring-jdbc-jdbctemplate) `.`加载它的用户

您必须注意到，我们用启用预授权/后授权的`@EnableWebSecurity`和`@EnableGlobalMethodSecurity`来注释我们的类。

对于后者，我们可以用`@PreAuthorize`和`@PostAuthorize`来注释我们的资源，以实现细粒度的访问控制:

```java
@SpringBootApplication
@EnableWebSecurity
@EnableGlobalMethodSecurity(prePostEnabled = true)
public class X509AuthenticationServer extends WebSecurityConfigurerAdapter {
    ...

    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http.authorizeRequests().anyRequest().authenticated()
          .and()
          .x509()
            .subjectPrincipalRegex("CN=(.*?)(?:,|$)")
            .userDetailsService(userDetailsService());
    }

    @Bean
    public UserDetailsService userDetailsService() {
        return new UserDetailsService() {
            @Override
            public UserDetails loadUserByUsername(String username) {
                if (username.equals("Bob")) {
                    return new User(username, "", 
                      AuthorityUtils
                        .commaSeparatedStringToAuthorityList("ROLE_USER"));
                }
                throw new UsernameNotFoundException("User not found!");
            }
        };
    }
}
```

如前所述，我们现在可以在控制器中使用`Expression-Based Access Control`。更具体地说，我们的授权注释受到重视，因为我们的`@Configuration` :中有`@EnableGlobalMethodSecurity`注释

```java
@Controller
public class UserController {
    @PreAuthorize("hasAuthority('ROLE_USER')")
    @RequestMapping(value = "/user")
    public String user(Model model, Principal principal) {
        ...
    }
}
```

所有可能的授权选项的概述可在`[official documentation](https://web.archive.org/web/20220822105735/https://docs.spring.io/spring-security/reference/6.0/servlet/authorization/method-security.html).` 中找到

作为最后的修改步骤，我们必须告诉应用程序我们的`truststore`在哪里，并且`SSL client authentication`是必需的(`server.ssl.client-auth=need`)。

因此，我们将以下内容放入我们的`application.properties`:

```java
server.ssl.trust-store=store/truststore.jks
server.ssl.trust-store-password=${PASSWORD}
server.ssl.client-auth=need
```

现在，如果我们运行应用程序，将浏览器指向`[https://localhost:8443/user](https://web.archive.org/web/20220822105735/https://localhost:8443/user)`，我们会被告知对方无法验证，它拒绝打开我们的网站。

### 5.3.客户端证书

现在是创建客户端证书的时候了。我们需要采取的步骤与我们已经创建的服务器端证书非常相似。

首先，我们必须创建一个证书签名请求:

```java
openssl req -new -newkey rsa:4096 -nodes -keyout clientBob.key -out clientBob.csr
```

我们必须提供将被纳入证书的信息。在这个练习中，**我们只输入常用名(CN)–Bob**。这很重要，因为我们在授权过程中使用这个条目，并且只有 Bob 被我们的示例应用程序识别。

接下来，我们需要向我们的 CA 签署请求:

```java
openssl x509 -req -CA rootCA.crt -CAkey rootCA.key -in clientBob.csr -out clientBob.crt -days 365 -CAcreateserial
```

我们需要采取的最后一步是将签名的证书和私钥打包到 PKCS 文件中:

```java
openssl pkcs12 -export -out clientBob.p12 -name "clientBob" -inkey clientBob.key -in clientBob.crt
```

最后，**我们准备在浏览器**中安装客户端证书。

同样，我们将使用 Firefox:

1.  在地址栏中输入`about:preferences`
2.  打开`Advanced -> View Certificates -> Your Certificates`
3.  点击 **`Import`**
4.  找到`Baeldung tutorials`文件夹及其子文件夹`spring-security-x509/store`
5.  选择 **`clientBob.p12`** 文件，点击 **`OK`**
6.  输入证书密码，点击 **`OK`**

现在，当我们刷新网站时，系统会提示我们选择要使用的客户端证书:

[![clientCert](img/a250b94186c90709f903880c07e1e44e.png)](/web/20220822105735/https://www.baeldung.com/wp-content/uploads/2016/08/clientCert.jpg)

如果我们看到类似`“Hello Bob!”`的欢迎信息，这意味着一切都按预期运行！

[![bob](img/8076c82c950ddf639388192fcb77704c.png)](/web/20220822105735/https://www.baeldung.com/wp-content/uploads/2016/08/bob.jpg)

## 6。使用 XML 进行相互认证

将 X.509 客户端认证添加到`XML` 中的 [`http`安全配置也是可能的:](/web/20220822105735/https://www.baeldung.com/spring-security-digest-authentication)

```java
<http>
    ...
    <x509 subject-principal-regex="CN=(.*?)(?:,|$)" 
      user-service-ref="userService"/>

    <authentication-manager>
        <authentication-provider>
            <user-service id="userService">
                <user name="Bob" password="" authorities="ROLE_USER"/>
            </user-service>
        </authentication-provider>
    </authentication-manager>
    ...
</http>
```

要配置一个底层的 Tomcat，我们必须将我们的`keystore`和`truststore`放到它的`conf`文件夹中，并编辑`server.xml`:

```java
<Connector port="8443" protocol="HTTP/1.1" SSLEnabled="true" scheme="https" secure="true"
    clientAuth="true" sslProtocol="TLS"
    keystoreFile="${catalina.home}/conf/keystore.jks"
    keystoreType="JKS" keystorePass="changeit"
    truststoreFile="${catalina.home}/conf/truststore.jks"
    truststoreType="JKS" truststorePass="changeit"
/>
```

**提示:`clientAuth`设置为`“want”`时，`SSL`仍然启用，即使客户端不提供有效证书。但是在这种情况下，我们必须使用第二种身份验证机制，例如登录表单，来访问受保护的资源。**

## 7。结论

总之，我们已经学习了**如何创建一个自签名的 CA 证书，以及如何使用它来签署其他证书**。

此外，我们还创建了服务器端和客户端证书。然后，我们介绍了如何相应地将它们导入到密钥库和信任库。

此外，您现在应该能够**将证书及其私钥打包成 PKCS12 格式**。

我们还讨论了何时使用 Spring Security X.509 客户端身份验证是有意义的，因此由您来决定是否在您的 web 应用程序中实现它。

最后，在 Github 上找到本文[的源代码。](https://web.archive.org/web/20220822105735/https://github.com/eugenp/tutorials/tree/master/spring-security-modules/spring-security-web-x509)