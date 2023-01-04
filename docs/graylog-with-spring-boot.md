# 用 Spring Boot 记录到灰色日志

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/graylog-with-spring-boot>

## 1.介绍

[Graylog](https://web.archive.org/web/20220630141220/https://www.graylog.org/) 是一个日志聚合服务。简而言之，它能够从多个来源收集数百万条日志消息，并在一个界面中显示它们。

此外，它还提供了许多其他功能，如实时警报、带有图形和图表的仪表板等等。

在本教程中，我们将了解如何设置一个 Graylog 服务器，并从 Spring Boot 应用程序向其发送日志消息。

## 2.设置灰色日志

有几种方法可以安装和运行 Graylog。在本教程中，我们将讨论两种最快的方法:Docker 和 Amazon Web Services。

### 2.1.码头工人

以下命令将下载所有需要的 Docker 映像，并为每个服务启动一个容器:

```java
$ docker run --name mongo -d mongo:3
$ docker run --name elasticsearch -p 9200:9200 -p 9300:9300 \
    -e ES_JAVA_OPTS="-Xms2g -Xmx4g" \
    -e "discovery.type=single-node" -e "xpack.security.enabled=false" \
    -e "bootstrap.memory_lock=true" --ulimit memlock=-1:-1 \
    -d docker.elastic.co/elasticsearch/elasticsearch:5.6.11
$ docker run --name graylog --link mongo --link elasticsearch \
    -p 9000:9000 -p 12201:12201 -p 514:514 -p 5555:5555 \
    -e GRAYLOG_WEB_ENDPOINT_URI="http://127.0.0.1:9000/api" \
    -d graylog/graylog:2.4.6-1
```

现在可以使用 URL `http://localhost:9000/`访问 Graylog 仪表板，默认用户名和密码都是`admin`。

虽然 Docker 设置是最简单的，但它确实需要大量的内存。它也不能在 Mac 版 Docker 上运行，因此可能不适用于所有平台。

### 2.2.亚马逊网络服务

设置 Graylog 进行测试的下一个最简单的选项是 Amazon Web Services。Graylog 提供了一个官方的 AMI，包括所有需要的依赖关系，尽管它在安装后需要一些额外的配置。

我们可以通过点击[这里的](https://web.archive.org/web/20220630141220/https://github.com/Graylog2/graylog2-images/tree/2.4/aws)并选择一个区域来快速部署带有 Graylog AMI 的 EC2 实例。 **Graylog 建议使用至少 4GB 内存的实例**。

实例启动后，我们需要 SSH 到主机并做一些更改。以下命令将为我们配置 Graylog 服务:

```java
$ sudo graylog-ctl enforce-ssl
$ sudo graylog-ctl set-external-ip https://<EC2 PUBLIC IP>:443/api/
$ sudo graylog-ctl reconfigure
```

我们还需要更新使用 EC2 实例创建的安全组，以允许特定端口上的网络流量。下图显示了需要启用的端口和协议:

[![graylog ec2 security zone inbound](img/459357789a1109af4b0a32bfda99cc9f.png)](/web/20220630141220/https://www.baeldung.com/wp-content/uploads/2018/10/graylog-ec2-security-zone-inbound.jpg)

现在可以使用 URL `https://<EC2 PUBLIC IP>/`访问 Graylog 仪表板，默认用户名和密码都是`admin`。

### 2.3.其他 Graylog 安装

除了 Docker 和 AWS，还有针对各种操作系统的 [Graylog 包](https://web.archive.org/web/20220630141220/http://docs.graylog.org/en/latest/pages/installation/operating_system_packages.html#operating-system-packages)。**通过这种方法，我们还必须建立一个 ElasticSearch 和 MongoDB 服务**。

由于这个原因，Docker 和 AWS 更容易设置，尤其是用于开发和测试目的。

## 3.发送日志消息

随着 Graylog 的启动和运行，我们现在必须配置我们的 Spring Boot 应用程序向 Graylog 服务器发送日志消息。

任何 Java 日志框架都可以支持使用 GELF 协议向 Graylog 服务器发送消息。

### 3.1.Log4J

目前唯一官方支持的日志框架是 Log4J。Graylog 提供了一个 appender，可以在 [Maven central](https://web.archive.org/web/20220630141220/https://search.maven.org/artifact/org.graylog2/gelfj) 上获得。

我们可以通过向任何`pom.xml`文件添加以下 Maven 依赖项来启用它:

```java
<dependency>
    <groupId>org.graylog2</groupId>
    <artifactId>gelfj</artifactId>
    <version>1.1.16</version>
</dependency>
```

我们还必须在使用 Spring Boot 启动模块的任何地方排除日志启动模块:

```java
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
    <exclusions>
        <exclusion>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-logging</artifactId>
        </exclusion>
    </exclusions>
</dependency>
```

现在我们可以在我们的`log4j.xml`文件中定义一个新的 appender:

```java
<appender name="graylog" class="org.graylog2.log.GelfAppender">
    <param name="graylogHost" value="<GRAYLOG IP>"/>
    <param name="originHost" value="localhost"/>
    <param name="graylogPort" value="12201"/>
    <param name="extractStacktrace" value="true"/>
    <param name="addExtendedInformation" value="true"/>
    <param name="facility" value="log4j"/>
    <param name="Threshold" value="INFO"/>
    <param name="additionalFields" value="{'environment': 'DEV', 'application': 'GraylogDemoApplication'}"/>
</appender>
```

这将把所有信息级别或更高级别的日志消息配置到 Graylog appender，后者再将日志消息发送到 Graylog 服务器。

### 3.2.其他日志框架

Graylog marketplace 有额外的库，支持各种其他日志框架，比如 Logback、Log4J2 等等。**只是要小心这些库不是由 Graylog** 维护的。其中一些已被废弃，其他的几乎没有或根本没有记录。

依赖这些第三方库时应小心谨慎。

### 3.3.灰色原木收集器边车

日志收集的另一个选项是[灰色日志收集器边车](https://web.archive.org/web/20220630141220/https://docs.graylog.org/docs/sidecar)。sidecar 是一个沿着文件收集器运行的进程，它将日志文件内容发送到 Graylog 服务器。

对于不可能更改日志配置文件的应用程序，Sidecar 是一个很好的选择。因为它直接从磁盘读取日志文件，**它也可以用来集成来自任何平台和编程语言的日志消息**。

## 4.查看灰色日志中的消息

我们可以使用 Graylog 仪表板来确认日志消息的成功交付。使用过滤器`source:localhost`将显示来自上面示例`log4j`配置的日志消息:

[![graylog log messages dashboard](img/809d18d4a29c3d108fceb1712c1d4c26.png)](/web/20220630141220/https://www.baeldung.com/wp-content/uploads/2018/10/graylog-log-messages-dashboard.jpg)

## 5.结论

Graylog 只是众多日志聚合服务之一。它可以快速搜索数百万条日志消息，实时可视化日志数据，并在特定条件成立时发送警报。

将 Graylog 集成到 Spring Boot 应用程序中只需要几行配置，不需要任何新代码。

代码样本一如既往地可以在 GitHub 上找到[。](https://web.archive.org/web/20220630141220/https://github.com/eugenp/tutorials/tree/master/spring-boot-modules/spring-boot-logging-log4j2)