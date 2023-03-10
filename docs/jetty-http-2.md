# 码头中的 HTTP/2

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/jetty-http-2>

## 1.概观

[HTTP/2](https://web.archive.org/web/20220627075556/https://en.wikipedia.org/wiki/HTTP/2) 协议带有一个**推送特性，允许服务器为一个请求**向客户端发送多个资源。因此，它减少了获取所有资源所需的多次往返，从而缩短了页面的加载时间。

Jetty 支持客户机和服务器实现的 HTTP/2 协议。

在本教程中，我们将探索 Jetty 中的 HTTP/2 支持，并创建一个 Java web 应用程序来检查 HTTP/2 推送特性。

## 2.入门指南

### 2.1.下载 Jetty

Jetty 需要 JDK 8 或更高版本和 [ALPN](https://web.archive.org/web/20220627075556/https://en.wikipedia.org/wiki/Application-Layer_Protocol_Negotiation) (应用层协议协商)支持来运行 HTTP/2。

通常，**Jetty 服务器部署在 SSL 上，并通过 TLS 扩展(ALPN)** 启用 HTTP/2 协议。

首先，我们需要下载最新的 [Jetty](https://web.archive.org/web/20220627075556/https://www.eclipse.org/jetty/download.html) 发行版，并设置`JETTY_HOME` 变量。

### 2.2.启用 HTTP/2 连接器

接下来，我们可以使用 Java 命令在 Jetty 服务器上启用 HTTP/2 连接器:

```java
java -jar $JETTY_HOME/start.jar --add-to-start=http2
```

该命令为端口`8443`上的 SSL 连接器添加 HTTP/2 协议支持。此外，它还通过 ALPN 模块进行协议协商:

```java
INFO  : server          transitively enabled, ini template available with --add-to-start=server
INFO  : alpn-impl/alpn-1.8.0_131 dynamic dependency of alpn-impl/alpn-8
INFO  : alpn-impl       transitively enabled
INFO  : alpn            transitively enabled, ini template available with --add-to-start=alpn
INFO  : alpn-impl/alpn-8 dynamic dependency of alpn-impl
INFO  : http2           initialized in ${jetty.base}/start.ini
INFO  : ssl             transitively enabled, ini template available with --add-to-start=ssl
INFO  : threadpool      transitively enabled, ini template available with --add-to-start=threadpool
INFO  : bytebufferpool  transitively enabled, ini template available with --add-to-start=bytebufferpool
INFO  : Base directory was modified
```

这里，日志显示了像`ssl`和`alpn-impl/alpn-8` 这样的模块的信息，这些模块是为 HTTP/2 连接器临时启用的。

### 2.3.启动 Jetty 服务器

现在，我们准备启动 Jetty 服务器:

```java
java -jar $JETTY_HOME/start.jar
```

当服务器启动时，日志将显示启用的模块:

```java
INFO::main: Logging initialized @228ms to org.eclipse.jetty.util.log.StdErrLog
...
INFO:oejs.AbstractConnector:main: Started [[email protected]](/web/20220627075556/https://www.baeldung.com/cdn-cgi/l/email-protection){SSL, (ssl, alpn, h2)}{0.0.0.0:8443}
INFO:oejs.Server:main: Started @872ms
```

### 2.4.启用附加模块

类似地，我们可以启用其他模块，如`http`和`http2c`:

```java
java -jar $JETTY_HOME/start.jar --add-to-start=http,http2c
```

让我们验证日志:

```java
INFO:oejs.AbstractConnector:main: Started [[email protected]](/web/20220627075556/https://www.baeldung.com/cdn-cgi/l/email-protection){SSL, (ssl, alpn, h2)}{0.0.0.0:8443}
INFO:oejs.AbstractConnector:main: Started [[email protected]](/web/20220627075556/https://www.baeldung.com/cdn-cgi/l/email-protection){HTTP/1.1, (http/1.1, h2c)}{0.0.0.0:8080}
INFO:oejs.Server:main: Started @685ms
```

此外，我们可以列出 Jetty 提供的所有模块:

```java
java -jar $JETTY_HOME/start.jar --list-modules
```

输出将类似于:

```java
Available Modules:
==================
tags: [-internal]
Modules for tag '*':
--------------------
     Module: alpn 
           : Enables the ALPN (Application Layer Protocol Negotiation) TLS extension.
     Depend: ssl, alpn-impl
        LIB: lib/jetty-alpn-client-${jetty.version}.jar
        LIB: lib/jetty-alpn-server-${jetty.version}.jar
        XML: etc/jetty-alpn.xml
    Enabled: transitive provider of alpn for http2
    // ...

Modules for tag 'connector':
----------------------------
     Module: http2 
           : Enables HTTP2 protocol support on the TLS(SSL) Connector,
           : using the ALPN extension to select which protocol to use.
       Tags: connector, http2, http, ssl
     Depend: ssl, alpn
        LIB: lib/http2/*.jar
        XML: etc/jetty-http2.xml
    Enabled: ${jetty.base}/start.ini
    // ...

Enabled Modules:
================
    0) alpn-impl/alpn-8 dynamic dependency of alpn-impl
    1) http2           ${jetty.base}/start.ini
    // ...
```

### 2.5.附加配置

类似于`–list-modules`参数，我们可以使用`–list-config` 列出每个模块的所有 XML 配置文件:

```java
java -jar $JETTY_HOME/start.jar --list-config
```

要为 Jetty 服务器配置像`host`和`port`这样的公共属性，我们可以在`start.ini`文件中进行修改:

```java
jetty.ssl.host=0.0.0.0
jetty.ssl.port=8443
jetty.ssl.idleTimeout=30000
```

此外，我们还可以配置一些`http2`属性，如`maxConcurrentStreams`和`maxSettingsKeys`:

```java
jetty.http2.maxConcurrentStreams=128
jetty.http2.initialStreamRecvWindow=524288
jetty.http2.initialSessionRecvWindow=1048576
jetty.http2.maxSettingsKeys=64
jetty.http2.rateControl.maxEventsPerSecond=20
```

## 3.设置 Jetty 服务器应用程序

### 3.1.Maven 配置

现在我们已经配置了 Jetty，是时候创建我们的应用程序了。

让我们将 [`jetty-maven-plugin`](https://web.archive.org/web/20220627075556/https://search.maven.org/search?q=g:org.eclipse.jetty%20a:jetty-maven-plugin) Maven 插件以及 Maven 依赖项添加到我们的`pom.xml`中，如 [`http2-server`](https://web.archive.org/web/20220627075556/https://search.maven.org/search?q=g:org.eclipse.jetty.http2%20a:http2-server) 、 [`jetty-alpn-openjdk8-server`](https://web.archive.org/web/20220627075556/https://search.maven.org/search?q=g:org.eclipse.jetty%20a:jetty-alpn-openjdk8-server) 和`[jetty-servlets](https://web.archive.org/web/20220627075556/https://search.maven.org/search?q=g:org.eclipse.jetty%20a:jetty-servlets)`:

```java
<build>
    <plugins>
        <plugin>
            <groupId>org.eclipse.jetty</groupId>
            <artifactId>jetty-maven-plugin</artifactId>
            <version>9.4.27.v20200227</version>
            <dependencies>
                <dependency>
                    <groupId>org.eclipse.jetty.http2</groupId>
                    <artifactId>http2-server</artifactId>
                    <version>9.4.27.v20200227</version>
                </dependency>
                <dependency>
                    <groupId>org.eclipse.jetty</groupId>
                    <artifactId>jetty-alpn-openjdk8-server</artifactId>
                    <version>9.4.27.v20200227</version>
                </dependency>
                <dependency>
                    <groupId>org.eclipse.jetty</groupId>
                    <artifactId>jetty-servlets</artifactId>
                    <version>9.4.27.v20200227</version>
                </dependency>
            </dependencies>
        </plugin>
    </plugins>
</build>
```

然后，我们将使用 Maven 命令编译这些类:

```java
mvn clean package
```

最后，我们可以将未组装的 Maven 应用程序部署到 Jetty 服务器:

```java
mvn jetty:run-forked
```

默认情况下，服务器使用 HTTP/1.1 协议从端口`8080`启动:

```java
oejmp.Starter:main: Started Jetty Server
oejs.AbstractConnector:main: Started [[email protected]](/web/20220627075556/https://www.baeldung.com/cdn-cgi/l/email-protection){HTTP/1.1, (http/1.1)}{0.0.0.0:8080}
oejs.Server:main: Started @1045ms
```

### 3.2.在`jetty.xml`中配置 HTTP/2

接下来，我们将通过添加适当的`Call`元素，在我们的`jetty.xml`文件中用 HTTP/2 协议配置 Jetty 服务器:

```java
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE Configure PUBLIC "-//Jetty//Configure//EN" "http://www.eclipse.org/jetty/configure_9_0.dtd">
<Configure id="Server" class="org.eclipse.jetty.server.Server">
    <!-- sslContextFactory and httpConfig configs-->

    <Call name="addConnector">
        <Arg>
            <New class="org.eclipse.jetty.server.ServerConnector">
                <Arg name="server"><Ref id="Server"/></Arg>
                <Arg name="factories">
                    <Array type="org.eclipse.jetty.server.ConnectionFactory">
                        <Item>
                            <New class="org.eclipse.jetty.server.SslConnectionFactory">
                                <Arg name="sslContextFactory"><Ref id="sslContextFactory"/></Arg>
                                <Arg name="next">alpn</Arg>
                            </New>
                        </Item>
                        <Item>
                            <New class="org.eclipse.jetty.alpn.server.ALPNServerConnectionFactory">
                                <Arg>h2</Arg>
                            </New>
                        </Item>
                        <Item>
                            <New class="org.eclipse.jetty.http2.server.HTTP2ServerConnectionFactory">
                                <Arg name="config"><Ref id="httpConfig"/></Arg>
                            </New>
                        </Item>
                    </Array>
                </Arg>
                <Set name="port">8444</Set>
            </New>
        </Arg>
    </Call>

    <!-- other Call elements -->
</Configure>
```

这里，HTTP/2 连接器在端口`8444`上配置了 ALPN 以及`sslContextFactory`和`httpConfig`配置。

此外，我们可以通过在`jetty.xml`中定义逗号分隔的参数来添加其他模块，如`h2-17`和`h2-16`(`h2`的草案版本):

```java
<Item> 
    <New class="org.eclipse.jetty.alpn.server.ALPNServerConnectionFactory"> 
        <Arg>h2,h2-17,h2-16</Arg> 
    </New> 
</Item>
```

然后，我们将在我们的`pom.xml`中配置 `jetty.xml`的位置:

```java
<plugin>
    <groupId>org.eclipse.jetty</groupId>
    <artifactId>jetty-maven-plugin</artifactId>
    <version>9.4.27.v20200227</version>
    <configuration>
        <stopPort>8888</stopPort>
        <stopKey>quit</stopKey>
        <jvmArgs>
            -Xbootclasspath/p:
            ${settings.localRepository}/org/mortbay/jetty/alpn/alpn-boot/8.1.11.v20170118/alpn-boot-8.1.11.v20170118.jar
        </jvmArgs>
        <jettyXml>${basedir}/src/main/config/jetty.xml</jettyXml>
        <webApp>
            <contextPath>/</contextPath>
        </webApp>
    </configuration>
    ...
</plugin>
```

注意:为了在我们的 Java 8 应用程序中启用 HTTP/2，**我们已经将`[alpn-boot](https://web.archive.org/web/20220627075556/https://search.maven.org/search?q=g:org.mortbay.jetty.alpn%20a:alpn-boot) jar`添加到 JVM BootClasspath** 中。然而， **ALPN 支持已经在 Java 9 或更高版本**中可用。

让我们重新编译我们的类并重新运行应用程序，以验证 HTTP/2 协议是否已启用:

```java
oejmp.Starter:main: Started Jetty Server
oejs.AbstractConnector:main: Started [[email protected]](/web/20220627075556/https://www.baeldung.com/cdn-cgi/l/email-protection){SSL, (ssl, http/1.1)}{0.0.0.0:8443}
oejs.AbstractConnector:main: Started [[email protected]](/web/20220627075556/https://www.baeldung.com/cdn-cgi/l/email-protection){SSL, (ssl, alpn, h2)}{0.0.0.0:8444}
```

在这里，我们可以观察到端口`8443`配置了 HTTP/1.1 协议，端口`8444`配置了 HTTP/2 协议。

### 3.3.配置`PushCacheFilter`

接下来，我们需要一个过滤器，将图像、JavaScript 和 CSS 等辅助资源推送到客户端。

为此，我们可以使用`org.eclipse.jetty.servlets`包中的 [`PushCacheFilter`](https://web.archive.org/web/20220627075556/https://www.eclipse.org/jetty/documentation/jetty-9/index.html#http2-configuring-push) 类。`PushCacheFilter`构建与`index.html`等主要资源相关联的次要资源的缓存，并将它们推送到客户端。

让我们在`web.xml`中配置`PushCacheFilter`:

```java
<filter>
    <filter-name>push</filter-name>
    <filter-class>org.eclipse.jetty.servlets.PushCacheFilter</filter-class>
    <init-param>
        <param-name>ports</param-name>
        <param-value>8444</param-value>
    </init-param>
</filter>
<filter-mapping>
    <filter-name>push</filter-name>
    <url-pattern>/*</url-pattern>
</filter-mapping>
```

### 3.4.配置 Jetty Servlet 和 Servlet 映射

然后，我们将创建`Http2JettyServlet`类来访问图像，并将`servlet-mapping`添加到我们的`web.xml`文件中:

```java
<servlet>
    <servlet-name>http2Jetty</servlet-name>
    <servlet-class>com.baeldung.jetty.http2.Http2JettyServlet</servlet-class>
</servlet>
<servlet-mapping>
    <servlet-name>http2Jetty</servlet-name>
    <url-patterimg/*</url-pattern>
</servlet-mapping>
```

## 4.设置 HTTP/2 客户端

最后，为了验证 HTTP/2 推送特性和改进的页面加载时间，我们将创建一个`http2.html`文件来加载一些图像(辅助资源):

```java
<!DOCTYPE html>
<html>
<head>
    <title>Baeldung HTTP/2 Client in Jetty</title>
</head>
<body>
    <h2>HTTP/2 Demo</h2>
    <div>
        <img src="images/homepage-latest_articles.jpg" alt="latest articles" />
        <img src="images/homepage-rest_with_spring.jpg" alt="rest with spring" />
        <img src="images/homepage-weekly_reviews.jpg" alt="weekly reviews" />
    </div>
</body>
</html>
```

## 5.测试 HTTP/2 客户端

为了获得页面加载时间的基线，让我们使用开发工具在 [`https://localhost:8443/http2.html`](https://web.archive.org/web/20220627075556/https://localhost:8443/http2.html) 访问 HTTP/1.1 应用程序来验证协议和加载时间:

[![](img/e92fb70b073c0dba523ef83e19b684b0.png)](/web/20220627075556/https://www.baeldung.com/wp-content/uploads/2020/04/http2-screenshot-1.png)

在这里，我们可以看到使用 HTTP/1.1 协议在 3-6 毫秒内加载了图像。

然后，我们将在 [`https://localhost:8444/http2.html`](https://web.archive.org/web/20220627075556/https://localhost:8444/http2.html) 访问启用了推送的 HTTP/2 应用程序:

[![](img/06626dfcb7f31869a38c80fd6f9cd8cc.png)](/web/20220627075556/https://www.baeldung.com/wp-content/uploads/2020/04/http2-screenshot-2.png)

这里我们观察到协议是`h2`，发起方是`Push`，所有镜像(二次资源)的加载时间都是 1ms。

因此，`PushCacheFilter`缓存了`http2.html,`的二级资源，并将它们推送到端口`8444`，极大地改善了页面的加载时间。

## 6.结论

在本教程中，我们探索了 Jetty 中的 HTTP/2。

首先，我们研究了如何使用 HTTP/2 协议启动 Jetty 及其配置。

然后，我们看到了一个具有 HTTP/2 Push 特性的 Java 8 web 应用程序，配置了一个`PushCacheFilter`，并观察了包含辅助资源的页面的加载时间如何比我们在 HTTP/1.1 协议中看到的有所改善。

像往常一样，所有的代码实现都可以在 GitHub 上获得[。](https://web.archive.org/web/20220627075556/https://github.com/eugenp/tutorials/tree/master/libraries-server-2)