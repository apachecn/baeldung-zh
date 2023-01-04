# 在 Docker 中访问 Spring Boot 日志

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/ops/spring-boot-logs-docker>

## 1.概观

在本教程中，我们将解释如何在 Docker 中访问 Spring Boot 日志，从本地开发到可持续的多容器解决方案。

## 2.基本控制台输出

首先，让我们从我们的[上一篇文章](/web/20220727020703/https://www.baeldung.com/spring-boot-docker-images)中建立我们的 Spring Boot 码头工人形象:

```java
$> mvn spring-boot:build-image
```

然后，**当我们运行我们的容器时，我们可以立即在控制台**中看到 STDOUT 日志:

```java
$> docker run --name=demo-container docker.io/library/spring-boot-docker:0.0.1-SNAPSHOT
Setting Active Processor Count to 1
WARNING: Container memory limit unset. Configuring JVM for 1G container.
```

该命令遵循类似 Linux shell `tail -f`命令的日志。

现在，让我们通过在`application.properties`文件中添加一行来配置带有日志文件附加器的 Spring Boot 应用程序:

```java
logging.file.path=logs
```

然后，我们可以通过在运行容器中运行`tail -f`命令来获得相同的结果:

```java
$> docker exec -it demo-container tail -f /workspace/logs/spring.log > $HOME/spring.log
Setting Active Processor Count to 1
WARNING: Container memory limit unset. Configuring JVM for 1G container.
```

这就是单容器解决方案。在接下来的章节中，我们将学习如何从组合容器中分析日志历史和日志输出。

## 3.日志文件的 Docker 卷

如果我们必须从主机文件系统访问日志文件，我们必须创建一个 Docker 卷。

为此，我们可以使用以下命令运行我们的应用程序容器:

```java
$> mvn spring-boot:build-image -v /path-to-host:/workspace/logs
```

然后，我们可以在`/path-to-host`目录中看到`spring.log`文件。

从我们上一篇关于 Docker Compose 的文章开始，我们可以从一个 Docker Compose 文件中运行多个容器。

如果我们使用 Docker 合成文件，我们应该添加卷配置:

```java
network-example-service-available-to-host-on-port-1337:
image: karthequian/helloworld:latest
container_name: network-example-service-available-to-host-on-port-1337
volumes:
- /path-to-host:/workspace/logs
```

然后，让我们运行文章`Compose`文件:

```java
$> docker-compose up
```

日志文件可以在`/path-to-host`目录中找到。

既然我们已经回顾了基本的解决方案，让我们探索更高级的`docker logs`命令。

在接下来的章节中，我们假设我们的 Spring Boot 应用程序被配置为打印日志到标准输出。

## 4.多个集装箱的码头日志

一旦我们一次运行多个容器，我们将不再能够从多个容器中读取混合日志。

我们可以在 Docker Compose 文档中发现，默认情况下，容器是用支持`docker logs`命令的`json-file`日志驱动程序设置的。

让我们用我们的 [Docker 编写示例](/web/20220727020703/https://www.baeldung.com/docker-compose)来看看它是如何工作的。

首先，让我们找到我们的容器 id:

```java
$> docker ps
CONTAINER ID        IMAGE                           COMMAND                  
877bb028a143        karthequian/helloworld:latest   "/runner.sh nginx" 
```

然后，**我们可以用`docker logs -f`命令**显示我们的容器日志。我们可以看到，尽管有了`json-file`驱动程序，输出仍然是纯文本——JSON 只在 Docker 内部使用:

```java
$> docker logs -f 877bb028a143
172.27.0.1 - - [22/Oct/2020:11:19:52 +0000] "GET / HTTP/1.1" 200 4369 "
172.27.0.1 - - [22/Oct/2020:11:19:52 +0000] "GET / HTTP/1.1" 200 4369 " 
```

`-f`选项的行为类似于`tail -f` shell 命令:它在日志产生时回显日志输出。

注意**如果我们在集群模式下运行我们的容器，我们应该使用`docker service ps`和`docker service logs`** 命令。

[在文档中，](https://web.archive.org/web/20220727020703/https://docs.docker.com/config/containers/logging/configure/#limitations-of-logging-drivers)我们可以看到`docker logs`命令支持有限的输出选项:`json-file`、`local,`或`journald`。

## 5.日志聚合服务的 Docker 驱动程序

`docker logs`命令对于即时观察特别有用:它不提供复杂的过滤器或长期统计。

为此， **Docker 支持几个[日志聚合服务驱动](https://web.archive.org/web/20220727020703/https://docs.docker.com/config/containers/logging/configure/#supported-logging-drivers)** 。正如我们在[的上一篇文章](/web/20220727020703/https://www.baeldung.com/graylog-with-spring-boot)中研究 Graylog 一样，我们将为这个平台配置合适的驱动程序。

**在`daemon.json`文件**中，该配置可以是主机的全局配置。它位于 Linux 主机上的`/etc/docker`或者 Windows 服务器上的`C:\ProgramData\docker\config`。

注意，如果`daemon.json`文件不存在，我们应该创建它:

```java
{ 
    "log-driver": "gelf",
    "log-opts": {
        "gelf-address": "udp://1.2.3.4:12201"
    }
}
```

Graylog 驱动程序被称为`GELF`——我们简单地指定了 Graylog 实例的 IP 地址。

**我们也可以在运行单个容器时覆盖此配置**:

```java
$> docker run \
      --log-driver gelf –-log-opt gelf-address=udp://1.2.3.4:12201 \
      alpine echo hello world
```

## 6.结论

在本文中，我们回顾了在 Docker 中访问 Spring Boot 日志的不同方法。

记录到 STDOUT 使得从单容器执行中观察日志变得非常容易。

然而，如果我们想从 Docker 日志特性中获益，使用文件附加器并不是最好的选择，因为容器没有适当的服务器那样的约束。