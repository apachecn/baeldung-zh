# Docker、Dockerfile 和 Docker Compose 之间的区别

> 原文::1230]https://web . archive . org/web/202209930061024/https://www . BAE message . com/ops/docker-dockerfile-docker-compose

## 1.介绍

随着越来越多的应用程序转向云计算，术语有时会变得令人困惑。

在本文中，**我们将讨论 Docker、Dockerfile 和 Docker Compose** 之间的区别。

## 2.码头工人

让我们先来看看 Docker，它是任何云计算平台的核心组件之一。Docker 是一个容器引擎，它允许我们高效、安全地将我们的应用程序与运行它们的基础设施分离开来。

这到底是什么意思？ **Docker 允许我们在虚拟化环境中运行任何应用**，使用我们想要的任何硬件或操作系统。这意味着我们可以在开发、测试和生产中为我们的应用程序使用相同的环境。

Docker 由几个关键组件组成:

*   守护进程:Docker 守护进程监听 Docker API 请求，并管理图像和容器等对象。
*   客户机:Docker 客户机是向守护进程发出命令的主要接口。
*   桌面:Docker 桌面是 Docker for Mac 和 Windows 的专用版本，它简化了与守护进程的交互。

此外，还有几个我们应该熟悉的 Docker 对象:

*   映像:映像是一个自包含文件，包含运行应用程序所需的所有文件(包括操作系统和应用程序代码)。图像有层，每层提供一组或多组文件和目录。
*   容器:容器是一个图像的可运行实例。容器通常是相互隔离的，尽管卷和网络允许它们进行交互。
*   注册表:注册表存储图像。它可以是私有的或公共的，并且可选地需要认证。
*   卷:卷是一个文件系统，可用于一个或多个容器。卷可以是持久的，也可以是短暂的(仅在容器处于活动状态时才持续)。
*   网络:网络允许容器使用标准网络协议(TCP/IP)进行通信。

既然我们已经理解了 Docker 本身的基本概念，我们可以看看另外两个直接相关的技术。

## 3\. Dockerfile

Dockerfile 是一个纯文本文件，包含构建 Docker [图像](/web/20220727020730/https://www.baeldung.com/ops/docker-images-vs-containers)的指令。他们遵循 Dockerfile 标准，Docker 守护进程最终负责执行 Dockerfile 并生成映像。

典型的 [Dockerfile](https://web.archive.org/web/20220727020730/https://docs.docker.com/develop/develop-images/dockerfile_best-practices/) 通常从包含另一个图像开始。例如，它可能构建在特定的操作系统或 Java 发行版上。

从那里，Dockerfile 可以执行各种操作来构建映像:

*   将文件从主机系统复制到容器中。例如，我们可能想要复制一个包含我们的应用程序代码的 JAR 文件。
*   运行与图像相关的任意命令。例如，我们可能希望运行典型的 Unix 命令来更改文件权限或使用包管理器安装新的包。
*   定义创建容器时应该执行的命令。例如，一个`java`命令加载我们的 JAR 文件并启动所需的 main 方法。

让我们来看一个 docker 文件示例:

```java
FROM openjdk:17-alpine
ARG JAR_FILE=target/my-app.jar
COPY ${JAR_FILE} my-app.jar
ENTRYPOINT ["java","-jar","/my-app.jar"]
```

该示例基于现有的 OpenJDK 17 Alpine 映像创建一个 Docker 映像。然后，它将编译后的 JAR 文件复制到映像中，并将启动命令定义为带有`-jar`选项的`java`命令以及编译后的 JAR 文件。

**值得注意的是，Docker 文件只是创建 Docker 图像的一种方式**。其他工具，如 [Buildpacks](/web/20220727020730/https://www.baeldung.com/spring-boot-docker-images) ，可以帮助自动化将我们的代码编译成 Docker 映像的过程，而无需使用 Docker 文件。

## 4.复合坞站

Docker Compose 是一个定义和运行多容器 Docker 应用程序的工具。使用 YAML 配置文件， [Docker Compose](/web/20220727020730/https://www.baeldung.com/dockerizing-spring-boot-application#2-the-docker-compose-file) 允许我们在一个地方配置多个容器。然后，我们可以使用一个命令立即启动和停止所有这些容器。

另外， **Docker Compose 允许我们定义容器**共享的公共对象。例如，我们可以定义一个卷，然后将它安装在每个容器中，这样它们就可以共享一个公共的文件系统。或者，我们可以定义一个或多个容器用来通信的网络。

让我们编写一个简单的 Docker 编写文件:

```java
version: "3.9"
services:
  database:
    image: mysql:5.7
    volumes:
      - db_data:/var/lib/mysql
    ports:
      - "3306:3306"
  web:
    image: my-application:latest
    ports:
      - "80:5000"
volumes:
  db_data: {}
```

这会启动两个容器:一个 MySQL 数据库服务器和一个 web 应用程序。它定义了一个由数据库容器使用的卷。此外，它还定义了每个容器应该公开哪些端口，以便网络流量可以到达这些端口。

**Dockerfiles 和 Docker Compose 并不互斥**。事实上，他们合作得很好。Docker Compose 提供了一个`build`指令，我们可以使用它在启动容器之前构建 Docker 文件。

最后，记住 **Docker Compose 只是编排多个容器**的一个工具。其他选项包括 Kubernetes、Openshift 和 Apache Mesos。

## 5.结论

在本文中，我们讨论了 Docker、Dockerfile 和 Docker Compose 之间的区别。**虽然所有这些技术都是相关的，但它们都是一个更大的技术生态系统的不同部分**。

了解每一个部分以及它们所扮演的角色可以帮助我们在云计算平台上工作时做出更明智的决策。