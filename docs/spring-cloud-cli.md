# Spring Cloud CLI 简介

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-cloud-cli>

## 1。简介

在本文中，我们来了解一下 Spring Boot 云 CLI(或简称为云 CLI)。该工具为 Spring Boot CLI 提供了一组命令行增强功能，有助于进一步抽象和简化 Spring Cloud 部署。

CLI 于 2016 年末推出，**允许使用命令行、`.yml`配置文件和 Groovy 脚本快速自动配置和部署标准 Spring 云服务。**

## 2。设置

Spring Boot 云 CLI 1.3.x 需要 Spring Boot CLI 1.5.x，所以一定要从 [Maven Central](https://web.archive.org/web/20221126215154/https://search.maven.org/classic/#search%7Cga%7C1%7Ca%3A%22spring-boot-cli%22) ( [安装说明](https://web.archive.org/web/20221126215154/https://docs.spring.io/spring-boot/docs/current/reference/html/getting-started-installing-spring-boot.html#getting-started-manual-cli-installation))和 [Maven 资源库](https://web.archive.org/web/20221126215154/https://search.maven.org/classic/#search%7Cga%7C1%7Ca%3A%22spring-cloud-cli%22) ( [官方 Spring 资源库](https://web.archive.org/web/20221126215154/https://repo.spring.io/snapshot/org/springframework/cloud/spring-cloud-cli/))获取最新版本的云 CLI！

要确保 CLI 已安装并可以使用，只需运行:

```java
$ spring --version
```

验证您的 Spring Boot CLI 安装后，安装最新稳定版本的云 CLI:

```java
$ spring install org.springframework.cloud:spring-cloud-cli:1.3.2.RELEASE
```

然后验证云 CLI:

```java
$ spring cloud --version
```

高级安装功能可以在官方云 CLI [页面](https://web.archive.org/web/20221126215154/https://cloud.spring.io/spring-cloud-cli/)找到！

## 3。默认服务和配置

CLI 提供了七种核心服务，可以使用单行命令运行和部署。

要在`http://localhost:8888`启动云配置服务器:

```java
$ spring cloud configserver
```

要在`http://localhost:8761`上启动 Eureka 服务器:

```java
$ spring cloud eureka
```

要在`http://localhost:9095`上启动 H2 服务器:

```java
$ spring cloud h2
```

要在`http://localhost:9091`上启动 Kafka 服务器:

```java
$ spring cloud kafka
```

要在`http://localhost:9411`上启动 Zipkin 服务器:

```java
$ spring cloud zipkin
```

要在 http://localhost:9393 上启动数据流服务器:

```java
$ spring cloud dataflow
```

在`http://localhost:7979`上启动 Hystrix 仪表板:

```java
$ spring cloud hystrixdashboard
```

列出当前运行的云服务:

```java
$ spring cloud --list
```

方便的帮助命令:

```java
$ spring help cloud
```

关于这些命令的更多细节，请查看官方[博客](https://web.archive.org/web/20221126215154/https://spring.io/blog/2016/11/02/introducing-the-spring-cloud-cli-launcher)。

## 4。用 YML 定制云服务

通过云 CLI 部署的每个服务也可以使用相应命名的`.yml` 文件进行配置:

```java
spring:
  profiles:
    active: git
  cloud:
    config:
      server:
        git:
          uri: https://github.com/spring-cloud-samples/config-repo
```

这构成了一个简单的配置文件，我们可以用它来启动云配置服务器。

例如，我们可以指定一个 Git 存储库作为 URI 源代码，当我们发出 `‘spring cloud configserver'`命令时，它将被自动克隆和部署。

云 CLI 使用的是引擎盖下的 Spring Cloud Launcher。这意味着**云 CLI 支持大多数 Spring Boot 配置机制。** [这里是](https://web.archive.org/web/20221126215154/https://docs.spring.io/spring-boot/docs/current/reference/html/appendix-application-properties.html)Spring Boot 房产的官方列表。

春云配置符合`‘spring.cloud…`约定。Spring Cloud 和 Spring Config Server 的设置可以在这个[链接](https://web.archive.org/web/20221126215154/https://cloud.spring.io/spring-cloud-static/spring-cloud-config/1.3.3.RELEASE/single/spring-cloud-config.html#_environment_repository)找到。

我们还可以直接在`cloud.yml`中指定几个不同的模块和服务:

```java
spring:
  cloud:
    launcher:
      deployables:
        - name: configserver
          coordinates: maven://...:spring-cloud-launcher-configserver:1.3.2.RELEASE
          port: 8888
          waitUntilStarted: true
          order: -10
        - name: eureka
          coordinates: maven:/...:spring-cloud-launcher-eureka:1.3.2.RELEASE
          port: 8761
```

`cloud.yml`允许添加定制服务或模块，并允许使用 Maven 和 Git 库。

## 5。运行定制的 Groovy 脚本

由于 Cloud CLI 可以编译和部署 Groovy 代码，所以可以用 Groovy 编写定制组件，并高效地进行部署。

下面是一个最小 REST API 实现的例子:

```java
@RestController
@RequestMapping('/api')
class api {

    @GetMapping('/get')
    def get() { [message: 'Hello'] }
}
```

假设脚本保存为`rest.groovy`，我们可以像这样启动我们的最小服务器:

```java
$ spring run rest.groovy
```

Pinging】应显示:

```java
{"message":"Hello"}
```

## 6。加密/解密

Cloud CLI 还提供了一个加密和解密工具(可以在包`org.springframework.cloud.cli.command.*`中找到)，可以通过命令行直接使用，也可以通过向云配置服务器端点传递一个值来间接使用。

我们来设置一下，看看怎么用。

### 6.1。设置

Cloud CLI 和 Spring Cloud Config Server 都使用`org.springframework.security.crypto.encrypt.*`f`or`handli`ng`加密和解密命令。

因此，两者都需要 Oracle [在这里](https://web.archive.org/web/20221126215154/http://www.oracle.com/technetwork/java/javase/downloads/jce8-download-2133166.html)提供的 JCE 无限强度扩展。

### 6.2。通过命令加密和解密

要通过终端加密“`my_value`”，调用:

```java
$ spring encrypt my_value --key my_key
```

可以使用“@”后跟路径(通常用于 RSA 公钥)来替换密钥名(如上面的“`my_key`”):

```java
$ spring encrypt my_value --key @${WORKSPACE}/foos/foo.pub
```

`my_value`'现在将被加密为类似于:

```java
c93cb36ce1d09d7d62dffd156ef742faaa56f97f135ebd05e90355f80290ce6b
```

此外，它将存储在内存中的键'`my_key`'下。这允许我们通过命令行将'`my_key`'解密回'`my_value`':

```java
$ spring decrypt --key my_key
```

我们现在还可以在配置 YAML 或属性文件中使用加密值，加载时云配置服务器会自动解密该值:

```java
encrypted_credential: "{cipher}c93cb36ce1d09d7d62dffd156ef742faaa56f97f135ebd05e90355f80290ce6b"
```

### 6.3。用配置服务器加密和解密

Spring Cloud Config Server 公开了 RESTful 端点，其中的密钥和加密值对可以存储在 Java 安全存储或内存中。

有关如何正确设置和配置您的云配置服务器以接受`symmetric`或`asymmetric`加密的更多信息，请查看我们的[文章](/web/20221126215154/https://www.baeldung.com/spring-cloud-configuration)或官方[文档](https://web.archive.org/web/20221126215154/https://cloud.spring.io/spring-cloud-static/spring-cloud-config/1.3.3.RELEASE/single/spring-cloud-config.html#_encryption_and_decryption)。

一旦 Spring Cloud Config Server 使用'`spring cloud configserver`'命令进行了配置并开始运行，您将能够调用它的 API:

```java
$ curl localhost:8888/encrypt -d mysecret
//682bc583f4641835fa2db009355293665d2647dade3375c0ee201de2a49f7bda
$ curl localhost:8888/decrypt -d 682bc583f4641835fa2db009355293665d2647dade3375c0ee201de2a49f7bda
//mysecret
```

## 7。结论

我们在这里重点介绍了 Spring Boot 云 CLI。更多信息，请查看官方[文档](https://web.archive.org/web/20221126215154/https://cloud.spring.io/spring-cloud-static/spring-cloud-cli/1.3.2.RELEASE/)。

本文中使用的配置和 bash 示例可以从 GitHub 上的[获得。](https://web.archive.org/web/20221126215154/https://github.com/eugenp/tutorials/tree/master/spring-cloud-modules/spring-cloud-cli)