# Spring 云配置快速介绍

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-cloud-configuration>

## 1。概述

**`Spring Cloud Config`** 是 Spring 的客户机/服务器方法，用于存储和服务跨多个应用程序和环境的分布式配置。

这个配置存储在`Git`版本控制下被理想地版本化，并且可以在应用运行时被修改。虽然它非常适合使用所有支持的配置文件格式以及像`Environment`、`[PropertySource, or @Value](/web/20221108190012/https://www.baeldung.com/properties-with-spring)`这样的结构的 Spring 应用程序，但是它可以在任何运行任何编程语言的环境中使用。

在本教程中，我们将关注如何设置一个由`Git`支持的配置服务器，在一个简单的`REST`应用服务器中使用它，并设置一个包括加密属性值的安全环境。

## 2。项目设置和依赖关系

首先，我们将创建两个新的`Maven`项目。服务器项目依赖于`[spring-cloud-config-server](https://web.archive.org/web/20221108190012/https://search.maven.org/classic/#search%7Cgav%7C1%7Cg%3A%22org.springframework.cloud%22%20AND%20a%3A%22spring-cloud-config-server%22)`模块，以及`[spring-boot-starter-security](https://web.archive.org/web/20221108190012/https://search.maven.org/classic/#search%7Cgav%7C1%7Cg%3A%22org.springframework.boot%22%20AND%20a%3A%22spring-boot-starter-security%22)`和`[spring-boot-starter-web](https://web.archive.org/web/20221108190012/https://search.maven.org/classic/#search%7Cgav%7C1%7Cg%3A%22org.springframework.boot%22%20AND%20a%3A%22spring-boot-starter-web%22)`启动包:

```java
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-config-server</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-security</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
</dependency>
```

然而，对于客户端项目，我们只需要`[spring-cloud-starter-config](https://web.archive.org/web/20221108190012/https://search.maven.org/classic/#search%7Cgav%7C1%7Cg%3A%22org.springframework.cloud%22%20AND%20a%3A%22spring-cloud-starter-config%22)`和`[spring-boot-starter-web modules](https://web.archive.org/web/20221108190012/https://search.maven.org/classic/#search%7Cgav%7C1%7Cg%3A%22org.springframework.boot%22%20AND%20a%3A%22spring-boot-starter-web%22)`:

```java
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-config</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
</dependency>
```

## 3。一个配置服务器实现

应用程序的主要部分是一个配置类，更确切地说是一个 [`@SpringBootApplication`](/web/20221108190012/https://www.baeldung.com/spring-boot-application-configuration) ，它通过`auto-configure`注释`@EnableConfigServer:`获取所有需要的设置

```java
@SpringBootApplication
@EnableConfigServer
public class ConfigServer {

    public static void main(String[] arguments) {
        SpringApplication.run(ConfigServer.class, arguments);
    }
} 
```

现在我们需要配置我们的服务器正在监听的服务器`port`和一个`Git` -url，它提供我们的版本控制配置内容。后者可以与本地文件系统上的`http`、`ssh,`或简单的`file`等协议一起使用。

**提示:**如果我们计划使用指向同一个配置存储库的多个配置服务器实例，我们可以配置服务器将我们的 repo 克隆到本地临时文件夹中。但是要注意使用双因素认证的私有存储库；它们很难处理！在这种情况下，在我们的本地文件系统上克隆它们并使用副本会更容易。

还有一些`placeholder variables and search patterns`用于配置可用的`repository-url`；然而，这超出了本文的范围。如果您有兴趣了解更多，官方文档是一个很好的起点。

我们还需要为我们的`application.properties`中的`Basic-Authentication`设置用户名和密码，以避免每次应用程序重启时自动生成密码:

```java
server.port=8888
spring.cloud.config.server.git.uri=ssh://localhost/config-repo
spring.cloud.config.server.git.clone-on-start=true
spring.security.user.name=root
spring.security.user.password=s3cr3t
```

## 4。作为配置存储的 Git 存储库

为了完成我们的服务器，我们必须在配置的 url 下初始化一个`Git`存储库，创建一些新的属性文件，并用一些值填充它们。

配置文件的名称就像普通的 Spring `application.properties`一样，但是使用了一个配置名称，比如客户端的属性`‘spring.application.name',`的值，后面跟一个破折号和活动概要文件，而不是单词“application”。例如:

```java
$> git init
$> echo 'user.role=Developer' > config-client-development.properties
$> echo 'user.role=User'      > config-client-production.properties
$> git add .
$> git commit -m 'Initial config-client properties'
```

**故障排除:**如果我们遇到与`ssh`相关的认证问题，我们可以在我们的 ssh 服务器上再次检查`~/.ssh/known_hosts`和`~/.ssh/authorized_keys`。

## 5。查询配置

现在我们可以启动我们的服务器了。我们的服务器提供的`Git`支持的配置 API 可以使用以下路径进行查询:

```java
/{application}/{profile}[/{label}]
/{application}-{profile}.yml
/{label}/{application}-{profile}.yml
/{application}-{profile}.properties
/{label}/{application}-{profile}.properties
```

占位符`{label}`指的是 Git 分支，`{application}`指的是客户端的应用程序名称，`{profile}`指的是客户端当前活动的应用程序概要文件。

因此，我们可以通过以下方式检索在分支`master`中的开发概要文件下运行的计划配置客户机的配置:

```java
$> curl http://root:[[email protected]](/web/20221108190012/https://www.baeldung.com/cdn-cgi/l/email-protection):8888/config-client/development/master
```

## 6。客户端实现

接下来，我们来处理客户。这将是一个非常简单的客户端应用程序，由一个带有一个`GET` 方法的`REST`控制器组成。

要获取我们的服务器，配置必须放在`application.properties`文件中。Spring Boot 2.4 引入了一种使用`**spring.config.import**`属性加载配置数据的新方法，这是现在绑定到配置服务器的默认方式:

```java
@SpringBootApplication
@RestController
public class ConfigClient {

    @Value("${user.role}")
    private String role;

    public static void main(String[] args) {
        SpringApplication.run(ConfigClient.class, args);
    }

    @GetMapping(
      value = "/whoami/{username}",  
      produces = MediaType.TEXT_PLAIN_VALUE)
    public String whoami(@PathVariable("username") String username) {
        return String.format("Hello! 
          You're %s and you'll become a(n) %s...\n", username, role);
    }
}
```

除了应用程序名之外，我们还将活动概要文件和连接详细信息放入我们的`application.properties`:

```java
spring.application.name=config-client
spring.profiles.active=development
spring.config.import=optional:configserver:http://root:[[email protected]](/web/20221108190012/https://www.baeldung.com/cdn-cgi/l/email-protection):8888
```

这将连接到 http://localhost:8888 上的配置服务器，并在启动连接时使用 http 基本安全性。我们还可以分别使用`spring.cloud.config.username`和`spring.cloud.config.password`属性来设置用户名和密码。

在某些情况下，如果服务无法连接到配置服务器，我们可能希望服务启动失败。如果这是我们想要的行为，我们可以删除 **optional:** 前缀，让客户端异常暂停。

为了测试是否从我们的服务器正确接收到配置，以及是否在我们的控制器方法中注入了`role value`,我们只需在启动客户端后将其卷曲:

```java
$> curl http://localhost:8080/whoami/Mr_Pink
```

如果响应如下，我们的`Spring Cloud Config Server`和它的客户端目前工作正常:

```java
Hello! You're Mr_Pink and you'll become a(n) Developer...
```

## 7。加密和解密

**需求**:为了将加密的强密钥与 Spring 加密和解密特性结合使用，我们需要在`JVM.` 中安装`‘Java Cryptography Extension (JCE) Unlimited Strength Jurisdiction Policy Files'`，这些可以从 [Oracle](https://web.archive.org/web/20221108190012/http://www.oracle.com/technetwork/java/javase/downloads/jce8-download-2133166.html) 下载。要安装，请遵循下载中包含的说明。一些 Linux 发行版也通过它们的包管理器提供了一个可安装的包。

由于配置服务器支持属性值的加密和解密，我们可以使用公共存储库来存储敏感数据，比如用户名和密码。加密值以字符串`{cipher},`为前缀，如果服务器被配置为使用对称密钥或密钥对，则可以通过对路径`‘/encrypt'`的 REST 调用来生成加密值。

也可以使用要解密的端点。两个端点都接受一个路径，该路径包含应用程序名称及其当前配置文件的占位符:`‘/*/{name}/{profile}'.`这对于控制每个客户端的加密特别有用。然而，在它们有用之前，我们必须配置一个加密密钥，这将在下一节中进行。

**提示:**如果我们使用 curl 调用加密/解密 API，最好使用 `–data-urlencode` 选项(而不是`–data/-d`)，或者将‘Content-Type’头显式设置为`‘text/plain'`。这可以确保正确处理加密值中的特殊字符，如“+”。

如果一个值在通过客户端读取时不能被自动解密，那么它的`key`会被重命名为它自己的名字，加上前缀“invalid”这应该可以防止使用加密值作为密码。

**提示:**当建立一个包含 YAML 文件的存储库时，我们必须用单引号将加密值和前缀值括起来。然而，属性却不是这样。

### 7.1. **CSRF**

默认情况下，Spring Security 为发送到我们应用程序的所有请求启用 [CSRF](/web/20221108190012/https://www.baeldung.com/spring-security-csrf) 保护。

因此，为了能够使用`/encrypt`和`/decrypt`端点，让我们禁用它们的 CSRF:

```java
@Configuration
public class SecurityConfiguration {

    @Bean
    public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
        http.csrf()
          .ignoringAntMatchers("/encrypt/**")
          .ignoringAntMatchers("/decrypt/**");

        //...
    }
}
```

### 7.2。密钥管理

默认情况下，配置服务器能够以对称或非对称方式加密属性值。

**要使用对称加密**，我们只需将`application.properties` 中的属性`‘encrypt.key'`设置为我们选择的秘密`.` ，或者，我们可以传入环境变量`ENCRYPT_KEY`。

**对于非对称加密**，我们可以将`‘encrypt.key'`设置为一个`PEM`编码的字符串值或者配置一个`keystore`来使用。

由于我们的演示服务器需要一个高度安全的环境，我们将选择后一个选项，同时生成一个新的密钥库，包括一个`RSA`密钥对，首先是 Java `keytool` :

```java
$> keytool -genkeypair -alias config-server-key \
       -keyalg RSA -keysize 4096 -sigalg SHA512withRSA \
       -dname 'CN=Config Server,OU=Spring Cloud,O=Baeldung' \
       -keypass my-k34-s3cr3t -keystore config-server.jks \
       -storepass my-s70r3-s3cr3t
```

然后，我们将创建的密钥库添加到服务器的应用程序`.properties`中，并重新运行它:

```java
encrypt.keyStore.location=classpath:/config-server.jks
encrypt.keyStore.password=my-s70r3-s3cr3t
encrypt.keyStore.alias=config-server-key
encrypt.keyStore.secret=my-k34-s3cr3t
```

接下来，我们将查询加密端点，并将响应作为一个值添加到存储库中的配置中:

```java
$> export PASSWORD=$(curl -X POST --data-urlencode d3v3L \
       http://root:[[email protected]](/web/20221108190012/https://www.baeldung.com/cdn-cgi/l/email-protection):8888/encrypt)
$> echo "user.password={cipher}$PASSWORD" >> config-client-development.properties
$> git commit -am 'Added encrypted password'
$> curl -X POST http://root:[[email protected]](/web/20221108190012/https://www.baeldung.com/cdn-cgi/l/email-protection):8888/refresh
```

为了测试我们的设置是否正常工作，我们将修改`ConfigClient`类并重启我们的客户端:

```java
@SpringBootApplication
@RestController
public class ConfigClient {

    ...

    @Value("${user.password}")
    private String password;

    ...
    public String whoami(@PathVariable("username") String username) {
        return String.format("Hello! 
          You're %s and you'll become a(n) %s, " +
          "but only if your password is '%s'!\n", 
          username, role, password);
    }
}
```

最后，对我们的客户端的查询将显示我们的配置值是否被正确解密:

```java
$> curl http://localhost:8080/whoami/Mr_Pink
Hello! You're Mr_Pink and you'll become a(n) Developer, \
  but only if your password is 'd3v3L'!
```

### 7.3。使用多个按键

如果我们想要使用多个密钥进行加密和解密，比如为每个应用程序使用一个专用的密钥，我们可以在`{cipher}`前缀和`BASE64`编码的属性值之间以`{name:value}`的形式添加另一个前缀。

配置服务器几乎开箱即可理解前缀`{secret:my-crypto-secret}`或`{key:my-key-alias}`。后一个选项需要在我们的`application.properties`中配置一个密钥库。在这个密钥库中搜索匹配的密钥别名。比如:

```java
user.password={cipher}{secret:my-499-s3cr3t}AgAMirj1DkQC0WjRv...
user.password={cipher}{key:config-client-key}AgAMirj1DkQC0WjRv...
```

对于没有密钥库的场景，我们必须实现一个类型为`TextEncryptorLocator,`的`@Bean`，它处理查找并为每个密钥返回一个`TextEncryptor`对象。

### 7.4。提供加密属性

如果我们想禁用服务器端加密并在本地处理属性值的解密，我们可以在服务器的`application.properties`中放入以下内容:

```java
spring.cloud.config.server.encrypt.enabled=false
```

此外，我们可以删除所有其他的加密。* "属性来禁用`REST`端点。

## 8。结论

现在我们能够创建一个配置服务器，从一个`Git`存储库向客户端应用程序提供一组配置文件。使用这样的服务器，我们还可以做一些其他的事情。

例如:

*   以`YAML`或`Properties`格式而不是`JSON,` 格式提供配置，占位符也已解析。当在非 Spring 环境中使用它时，这可能是有用的，在非 Spring 环境中，配置不直接映射到`PropertySource`。
*   依次提供纯文本配置文件，可选地提供已解析的占位符。例如，这对于提供依赖于环境的日志记录配置非常有用。
*   将配置服务器嵌入到应用程序中，它从`Git`存储库配置自己，而不是作为服务于客户端的独立应用程序运行。因此，我们必须设置一些属性和/或删除`@EnableConfigServer`注释，这取决于用例。
*   使配置服务器在 Spring 网飞 Eureka 服务发现中可用，并在配置客户端中启用自动服务器发现。如果服务器没有固定的位置或者在其位置上移动，这就变得很重要。

和往常一样，这篇文章的源代码可以在`[Github](https://web.archive.org/web/20221108190012/https://github.com/eugenp/tutorials/tree/master/spring-cloud-modules/spring-cloud-config)`上找到。