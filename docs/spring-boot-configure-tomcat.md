# 如何配置 Spring Boot Tomcat

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-boot-configure-tomcat>

## 1.概观

默认情况下，Spring Boot web 应用程序包括一个预先配置的嵌入式 web 服务器。然而，在某些情况下，我们想要**修改默认配置**来满足定制需求。

在本教程中，我们将看看通过`application.properties`文件配置 Tomcat 嵌入式服务器的一些常见用例。

## 2.常见的嵌入式 Tomcat 配置

### 2.1.服务器地址和端口

我们可能希望更改的最常见配置**是端口号**:

```java
server.port=80
```

如果我们不提供`server.port`参数，它默认设置为`8080``.`

在某些情况下，我们可能希望设置一个服务器应该绑定的网络地址。换句话说，我们定义了一个 **IP 地址，我们的服务器将在这里监听** :

```java
server.address=my_custom_ip
```

默认情况下，该值设置为`0.0.0.0, `，允许通过所有 IPv4 地址进行连接。设置另一个值，例如 localhost-`127.0.0.1`-将使服务器更具选择性。

### 2.2.错误处理

**默认情况下，Spring Boot 提供一个标准的错误网页**。这个页面叫做`Whitelabel`。默认情况下它是启用的，但是如果我们不想显示任何错误信息，我们可以禁用它:

```java
server.error.whitelabel.enabled=false
```

到`Whitelabel`的默认路径是`/error`。我们可以通过设置`server.error.path`参数来定制:

```java
server.error.path=/user-error
```

我们还可以设置属性来决定显示哪些错误信息。例如，我们可以包括错误消息和堆栈跟踪:

```java
server.error.include-exception=true
server.error.include-stacktrace=always
```

我们的教程【REST 的异常消息处理和[自定义白标错误页面](/web/20221205171134/https://www.baeldung.com/spring-boot-custom-error-page)解释了更多关于在 Spring Boot 处理错误的内容。

### 2.3.服务器连接

当在低资源容器上运行时，我们可能希望**减少 CPU 和内存负载。**一种方法是限制我们的应用程序可以处理的并发请求的数量。相反，我们可以增加这个值，以使用更多的可用资源来获得更好的性能。

在 Spring Boot，我们可以定义 Tomcat 工作线程的最大数量:

```java
server.tomcat.threads.max=200
```

在配置 web 服务器时，**设置服务器连接超时**可能也是有用的。这表示服务器在连接后、连接关闭前等待客户端发出请求的最长时间:

```java
server.connection-timeout=5s
```

我们还可以定义请求报头的最大大小:

```java
server.max-http-header-size=8KB
```

请求正文的最大大小:

```java
server.tomcat.max-swallow-size=2MB
```

或者整个 post 请求的最大大小:

```java
server.tomcat.max-http-post-size=2MB
```

### 2.4.加密套接字协议层

**为了在我们的 Spring Boot 应用程序中启用 SSL 支持**，我们需要将`server.ssl.enabled`属性设置为`true`并定义一个 SSL 协议:

```java
server.ssl.enabled=true
server.ssl.protocol=TLS
```

我们还应该配置保存证书的密钥库的密码、类型和路径:

```java
server.ssl.key-store-password=my_password
server.ssl.key-store-type=keystore_type
server.ssl.key-store=keystore-path
```

我们还必须定义在密钥库中标识密钥的别名:

```java
server.ssl.key-alias=tomcat
```

有关 SSL 配置的更多信息，请访问我们的 [HTTPS，使用 Spring Boot](/web/20221205171134/https://www.baeldung.com/spring-boot-https-self-signed-certificate) 文章中的自签名证书。

### 2.5.Tomcat 服务器访问日志

Tomcat 访问日志在测量页面命中计数、用户会话活动等方面非常有用。

**要启用访问日志，**只需设置:

```java
server.tomcat.accesslog.enabled=true
```

我们还应该配置附加到日志文件的其他参数，如目录名、前缀、后缀和日期格式:

```java
server.tomcat.accesslog.directory=logs
server.tomcat.accesslog.file-date-format=yyyy-MM-dd
server.tomcat.accesslog.prefix=access_log
server.tomcat.accesslog.suffix=.log
```

## 3.嵌入式 Tomcat 的版本

我们不能通过配置我们的`application.properties`文件来改变正在使用的 Tomcat 版本。这有点复杂，主要取决于我们是否使用`spring-boot-starter-parent`。

然而，在我们继续之前，我们必须意识到每个 **Spring Boot 版本都是针对特定的 Tomcat 版本设计和测试的。如果我们改变它，我们可能会面临一些意想不到的兼容性问题**。

### 3.1.使用`spring-boot-starter-parent`

如果我们使用 Maven 并将项目配置为从`spring-boot-starter-parent`继承，我们可以通过覆盖`pom.xml`中的特定属性来覆盖单个依赖项。

记住这一点，要更新 Tomcat 版本，我们必须使用`tomcat.version`属性:

```java
<properties>
    <tomcat.version>9.0.44</tomcat.version>
</properties>
```

### 3.2.使用`spring-boot-dependencies`

有些情况下，我们不想或不能使用`spring-boot-starter-parent`。例如，如果我们在我们的 Spring Boot 项目中使用一个[定制父节点。在这种情况下，我们使用`spring-boot-dependency`仍然有很大的机会从依赖管理中获益。](/web/20221205171134/https://www.baeldung.com/spring-boot-dependency-management-custom-parent)

然而，这种设置不允许我们通过使用 Maven 属性来覆盖单个依赖项，如前一节所示。

为了实现相同的目标，并且**仍然使用不同的 Tomcat 版本，我们需要在 pom 文件**的`dependencyManagement`部分添加一个条目。需要记住的重要一点是，我们必须将**放在**和`spring-boot-dependencies`之前:

```java
<dependencyManagement>
    <dependencies>
        <dependency>
            <groupId>org.apache.tomcat.embed</groupId>
            <artifactId>tomcat-embed-core</artifactId>
            <version>9.0.44</version>
        </dependency>

        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-dependencies</artifactId>
            <version>2.4.5</version>
            <type>pom</type>
            <scope>import</scope>
        </dependency>
    </dependencies>
</dependencyManagement>
```

## 4.结论

在本教程中，我们学习了一些常见的`Tomcat`嵌入式服务器配置。要查看更多可能的配置，请访问官方的 [Spring Boot 应用属性文档](https://web.archive.org/web/20221205171134/https://docs.spring.io/spring-boot/docs/current/reference/html/appendix-application-properties.html)页面。

和往常一样，这些例子的源代码可以在 [GitHub](https://web.archive.org/web/20221205171134/https://github.com/eugenp/tutorials/tree/master/spring-boot-modules/spring-boot-runtime) 上找到。