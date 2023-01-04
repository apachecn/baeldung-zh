# 使用 Docker Compose 的交互式 Shell

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/ops/docker-compose-interactive-shell>

## 1.概观

在本教程中，我们将学习如何用一个交互式外壳运行多个 [Docker](/web/20221025214318/https://www.baeldung.com/ops/docker-guide) 容器。首先，我们将使用简单的 [`docker run`](/web/20221025214318/https://www.baeldung.com/ops/running-docker-containers-indefinitely) 命令运行 Docker 容器。稍后，我们将使用 [`docker-compose`](/web/20221025214318/https://www.baeldung.com/ops/docker-compose) 命令运行同一个 Docker 容器。

## 2.坞站和坞站组成

Docker 容器允许开发人员打包应用程序，这些应用程序可以在不同的环境中无缝工作。事实上，生产环境中 web 应用程序的典型部署可能需要几项服务:

*   数据库服务器
*   负载平衡
*   网络服务器

在这种情况下，Docker Compose 是一个非常方便的工具。

Docker Compose 主要用于将多个容器作为单个服务运行，同时保持容器之间的平滑连接。

## 3.了解 Docker 撰写

要使用`docker-compose`命令运行 Docker 容器，我们需要将所有配置添加到单个`docker-compose.yml`配置文件中。重要的是，**与普通的`docker run`命令相比，使用`docker-compose`的一个主要好处是将配置整合到一个文件中，机器和人都可以读取这个文件。**

让我们创建一个简单的`docker-compose.yml`来展示如何使用 [`docker-compose up`](https://web.archive.org/web/20221025214318/https://docs.docker.com/engine/reference/commandline/compose_up/) 命令运行 Docker 容器:

```
version: "3"
services:
 server:
   image: tomcat:jre11-openjdk
   ports:
     - 8080:8080
```

这里，我们使用 [`tomcat`](/web/20221025214318/https://www.baeldung.com/tomcat) 作为基础映像，并在主机上显示端口`8080`。为了查看它的运行情况，让我们使用`docker-compose up`命令构建并运行这个映像:

```
$ docker-compose up
Pulling server (tomcat:jre11-openjdk)...
jre11-openjdk: Pulling from library/tomcat
001c52e26ad5: Pull complete
...
704b1ae41f0e: Pull complete
Digest: sha256:85bfe38b723bc864ed594973a63c04b112e20d6d33eee57cd5303610d8e3dc77
Status: Downloaded newer image for tomcat:jre11-openjdk
Creating dockercontainers_server_1 ... done
Attaching to dockercontainers_server_1
server_1  | NOTE: Picked up JDK_JAVA_OPTIONS:  --add-opens=java.base/java.lang=ALL-UNNAMED --add-opens=java.base/java.io=ALL-UNNAMED --add-opens=java.base/java.util=ALL-UNNAMED --add-opens=java.base/java.util.concurrent=ALL-UNNAMED --add-opens=java.rmi/sun.rmi.transport=ALL-UNNAMED
server_1  | 03-Aug-2022 06:22:17.259 INFO [main] org.apache.catalina.startup.VersionLoggerListener.log Server version name:   Apache Tomcat/10.0.23
```

重要的是，我们应该从包含`docker-compose.yml`文件的目录中运行上面的命令。

在上面的输出中，我们可以看到`dockercontainers_server_1`已经启动并运行。然而，**这种方法的一个问题是，一旦我们退出上面的 shell，容器也会停止**。

为了长期运行 Docker 容器，我们需要用一个交互式 shell 来运行它。

## 4.Docker 中的交互式外壳

**Docker 中的交互模式允许我们在容器处于`running`状态时执行命令。**为了在交互模式下运行 Docker 容器，我们使用了`-it`选项。此外，我们用`-it`标志将 [STDIN 和 STDOUT](/web/20221025214318/https://www.baeldung.com/linux/stream-redirections) 通道连接到我们的终端。

Docker Compose 使用单主机部署，具有多种优势:

*   快速且易于配置
*   支持快速部署
*   减少完成多项任务所需的时间
*   所有容器都独立运行，这降低了违规风险

现在让我们使用带有交互式 shell 的`docker-compose`来运行前面的`tomcat`容器:

```
version: "3"
services:
 server:
   image: tomcat:jre11-openjdk
   ports:
     - 8080:8080
   stdin_open: true 
   tty: true
```

在这种情况下， **我们在`docker-compose.yml`文件中添加了`stdin_open`和`tty`选项，这样我们就可以拥有一个与`docker-compose`设置**交互的 shell。

当然，要访问 Docker 容器，我们首先需要使用下面的命令运行容器:

```
$ docker-compose up --d
```

现在，我们可以得到一个运行`docker-compose`服务的交互外壳:

```
$ docker-compose exec server bash
```

**注意我们如何使用服务名而不是容器名**。

最后，我们使用上面的命令成功登录到容器。

## 5.结论

在本文中，我们演示了如何使用`docker-compose`命令获得交互式 shell。首先，我们学习了如何使用`docker-compose`运行 Docker 容器。之后，我们使用`docker exec`命令和`docker-compose` YAML 配置在一个交互式 shell 中进行了同样的探索。