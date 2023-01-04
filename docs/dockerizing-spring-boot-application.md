# 将 Spring Boot 的申请归档

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/dockerizing-spring-boot-application>

## 1。概述

在本教程中，我们将重点关注如何对一个`Spring Boot Application`进行 dockerize，以便在一个隔离的环境中运行它，也就是所谓的`container`。

我们将学习如何创建容器的组合，这些容器在虚拟专用网络中相互依赖并相互链接。我们还将了解如何使用单个命令来管理它们。

让我们首先创建一个简单的 Spring Boot 应用程序，然后我们将在运行 [`Alpine Linux`](https://web.archive.org/web/20221001124930/https://hub.docker.com/_/alpine/) 的轻量级基础映像中运行它。

## 2。对接一个独立的 Spring Boot 应用程序

作为一个我们可以 dockerize 的应用程序的例子，我们将创建一个简单的 Spring Boot 应用程序，`docker-message-server,`,它公开一个端点并返回一个静态消息:

```java
@RestController
public class DockerMessageController {
    @GetMapping("/messages")
    public String getMessage() {
        return "Hello from Docker!";
    }
}
```

有了正确配置的 Maven 文件，我们就可以创建一个可执行的 jar 文件:

```java
$> mvn clean package
```

接下来，我们将启动 Spring Boot 应用程序:

```java
$> java -jar target/docker-message-server-1.0.0.jar
```

现在我们有了一个可以在`localhost:8888/messages.`访问的 Spring Boot 应用程序

为了将应用程序归档，我们首先创建一个名为`Dockerfile`的文件，其内容如下:

```java
FROM openjdk:8-jdk-alpine
MAINTAINER baeldung.com
COPY target/docker-message-server-1.0.0.jar message-server-1.0.0.jar
ENTRYPOINT ["java","-jar","/message-server-1.0.0.jar"]
```

该文件包含以下信息:

*   中的**:作为我们图像的基础，我们将采用上一节中创建的`Java`使能的`Alpine Linux`。**
*   **维护者**:镜像的维护者。
*   **复制**:我们让`Docker`将我们的 jar 文件复制到映像中。
*   **入口点**:这将是容器启动时启动的可执行文件。我们必须将它们定义为`JSON-Array`，因为对于一些应用程序参数，我们将结合使用`ENTRYPOINT`和`CMD`。

为了从我们的`Dockerfile`创建一个图像，我们必须像前面一样运行`‘docker build'`:

```java
$> docker build --tag=message-server:latest .
```

最后，我们能够从我们的映像运行容器:

```java
$> docker run -p8887:8888 message-server:latest
```

这将在 Docker 中启动我们的应用程序，我们可以在`localhost:8887/messages`从主机访问它。这里定义端口映射很重要，它将主机(8887)上的一个端口映射到 Docker ( `8888`)内部的端口。这是我们在 Spring Boot 应用程序的属性中定义的端口。

注意:端口 8887 在我们启动容器的机器上可能不可用。在这种情况下，映射可能不起作用，我们需要选择一个仍然可用的端口。

如果我们以分离模式运行容器，我们可以使用以下命令检查它的详细信息、停止它并删除它:

```java
$> docker inspect message-server
$> docker stop message-server
$> docker rm message-server
```

### 2.1.更改基础图像

为了使用不同的 Java 版本，我们可以很容易地改变基本映像。例如，如果我们想使用亚马逊的 Corretto 发行版，我们可以简单地更改`Dockerfile`:

```java
FROM amazoncorretto:11-alpine-jdk
MAINTAINER baeldung.com
COPY target/docker-message-server-1.0.0.jar message-server-1.0.0.jar
ENTRYPOINT ["java","-jar","/message-server-1.0.0.jar"]
```

此外，我们可以使用自定义的基础映像。我们将在本教程的后面部分研究如何做到这一点。

## 3。将应用程序分类到一个组合中

`Docker`命令和`Dockerfiles`特别适合创建单独的容器。然而，如果我们想要`operate on a network of isolated applications`，容器管理很快就会变得杂乱无章。

**为了解决这个问题，`Docker` 提供了一个名为`Docker Compose`的工具。这个工具自带了一个`YAML`格式的构建文件，更适合管理多个容器。例如，它能够在一个命令中启动或停止服务的组合，或者将多个服务的日志输出合并到一个`pseudo-tty`中。**

### 3.1.第二个 Spring Boot 应用

让我们构建一个在不同 Docker 容器中运行的两个应用程序的例子。它们将相互通信，并作为“单一单元”呈现给主机系统。举个简单的例子，我们将创建第二个 Spring Boot 应用程序`docker-product-server`:

```java
@RestController
public class DockerProductController {
    @GetMapping("/products")
    public String getMessage() {
        return "A brand new product";
    }
}
```

我们可以用与我们的`message-server`相同的方式构建和启动应用程序。

### 3.2.Docker 撰写文件

我们可以将两种服务的配置合并到一个名为`docker-compose.yml`的文件中:

```java
version: '2'
services:
    message-server:
        container_name: message-server
        build:
            context: docker-message-server
            dockerfile: Dockerfile
        image: message-server:latest
        ports:
            - 18888:8888
        networks:
            - spring-cloud-network
    product-server:
        container_name: product-server
        build:
            context: docker-product-server
            dockerfile: Dockerfile
        image: product-server:latest
        ports:
            - 19999:9999
        networks:
            - spring-cloud-network
networks:
    spring-cloud-network:
        driver: bridge
```

*   **版本**:指定应该使用哪个格式版本。这是必填字段。这里我们使用较新的版本，而`legacy format`是‘1’。
*   **服务**:这个键中的每个对象定义了一个`service`，也称为容器。此部分是强制性的。
    *   **构建**:如果给定，`docker-compose`能够从`Dockerfile`构建一个图像
        *   **上下文**:如果给定，它指定编译目录，在那里查找`Dockerfile`。
        *   **dockerfile** :如果给定，它为一个`Dockerfile.`设置一个替换名
    *   **图像**:告诉`Docker`当构建特性被使用时，它应该给图像起什么名字。否则，它将在库中搜索该图像或`remote-registry.`
    *   **网络**:这是要使用的命名网络的标识符。给定的`name-value`必须在`networks`部分列出。
*   **网络**:在本节中，我们将指定对我们的服务`.`可用的`networks` 。在本例中，我们让`docker-compose`为我们创建一个类型为`‘bridge'`的命名的`network`。如果选项`external`被设置为`true`，它将使用具有给定名称的现有选项。

在继续之前，我们将检查构建文件中的语法错误:

```java
$> docker-compose config
```

然后，我们可以构建我们的映像，创建已定义的容器，并在一个命令中启动它:

```java
$> docker-compose up --build
```

这将一次启动`message-server`和`product-server`。

要停止容器，从`Docker`上取下容器，并从容器上取下连接的`networks`。为此，我们可以使用相反的命令:

```java
$> docker-compose down
```

关于`docker-compose,`更详细的介绍，我们可以阅读我们的文章[Docker Compose](/web/20221001124930/https://www.baeldung.com/docker-compose)简介。

### 3.3.扩展服务

`docker-compose`的一个很好的特性是**扩展服务的能力**。例如，我们可以告诉`Docker`为`message-server`运行三个容器，为`product-server`运行两个容器。

然而，为了正常工作，我们必须从我们的`docker-compose.yml`中删除`container_name`，这样`Docker`可以选择名字，并更改`exposed port configuration`以避免冲突。

对于端口，我们可以告诉 Docker 将主机上的一系列端口映射到 Docker 内部的一个特定端口:

```java
ports:
    - 18800-18888:8888
```

之后，我们能够像这样扩展我们的服务(注意，我们使用的是修改过的`yml-file`):

```java
$> docker-compose --file docker-compose-scale.yml up -d --build --scale message-server=1 product-server=1
```

该命令将旋转单个`message-server`和单个`product-server`。

为了扩展我们的服务，我们可以运行以下命令:

```java
$> docker-compose --file docker-compose-scale.yml up -d --build --scale message-server=3 product-server=2
```

该命令将启动**两个**附加消息服务器和**一个**附加产品服务器。正在运行的容器不会停止。

## 4。自定义基础图像

我们目前使用的基本映像(`openjdk:8-jdk-alpine`)包含了已经安装了 JDK 8 的 Alpine 操作系统的发行版。或者，我们可以构建自己的基础映像(基于 Alpine 或任何其他操作系统)。

为此，我们可以使用带有 Alpine 的`Dockerfile`作为基础映像，并安装我们选择的 JDK:

```java
FROM alpine:edge
MAINTAINER baeldung.com
RUN apk add --no-cache openjdk8
```

*   **来自**:关键字`FROM`告诉`Docker`使用带有标签的给定图像作为构建基础。如果这个映像不在本地库中，就在`[DockerHub](https://web.archive.org/web/20221001124930/https://hub.docker.com/explore/)`或任何其他配置的远程注册表上进行在线搜索。
*   **维护者**:一个`MAINTAINER`通常是一个电子邮件地址，标识一张图片的作者。
*   **运行**:使用`RUN`命令，我们在目标系统中执行一个 shell 命令行。这里我们利用`Alpine Linux's`包管理器、`apk,`来安装`Java 8 OpenJDK.`

为了最终构建映像并将其存储在本地库中，我们必须运行:

```java
docker build --tag=alpine-java:base --rm=true .
```

**注意:**`–tag` 选项将为图像命名，而`–rm=true`将在图像成功构建后移除中间图像。这个 shell 命令的最后一个字符是一个点，作为构建目录的参数。

现在我们可以使用创建的图像来代替`openjdk:8-jdk-alpine`。

## 5.Spring Boot 2.3 中的构建包支持

**Spring Boot 2.3 增加了对** [**buildpacks**](https://web.archive.org/web/20221001124930/https://buildpacks.io/) 的支持。简而言之，我们不用创建自己的 Dockerfile 并使用类似于`docker build`的东西来构建它，我们所要做的就是发出以下命令:

```java
$ mvn spring-boot:build-image
```

同样，在《格雷尔:

```java
$ ./gradlew bootBuildImage
```

要做到这一点，我们需要安装并运行 Docker。

buildpacks 背后的主要动机是创建一些知名云服务(如 Heroku 或 Cloud Foundry)已经提供了一段时间的相同部署体验。我们只是运行`build-image `目标，然后平台本身负责构建和部署工件。

此外，它可以帮助我们更有效地改变建立 Docker 形象的方式。我们所要做的不是对不同项目中的大量 docker 文件应用相同的更改，而是更改或调整 buildpacks 映像构建器。

除了易用性和更好的整体开发体验，它还可以提高效率。例如，buildpacks 方法将创建一个分层的 Docker 映像，并使用 Jar 文件的分解版本。

让我们看看运行上述命令后会发生什么。

当我们列出可用的 docker 图像时:

```java
docker image ls -a
```

我们看到我们刚刚创建的图像有一条线:

```java
docker-message-server 1.0.0 b535b0cc0079
```

这里，映像名称和版本与我们在 Maven 或 Gradle 配置文件中定义的名称和版本相匹配。哈希代码是图像哈希的简短版本。

然后，要启动我们的容器，我们只需运行:

```java
docker run -it -p9099:8888 docker-message-server:1.0.0
```

与我们构建的映像一样，我们需要映射端口，以便从 Docker 外部访问我们的 Spring Boot 应用程序。

## 6。结论

在本文中，我们学习了如何构建定制的`Docker`图像，将`Spring Boot Application`作为`Docker`容器运行，并使用`docker-compose`创建容器。

要进一步阅读构建文件，我们可以参考官方的`[Dockerfile reference](https://web.archive.org/web/20221001124930/https://docs.docker.com/engine/reference/builder/)`和`[docker-compose.yml reference](https://web.archive.org/web/20221001124930/https://docs.docker.com/compose/compose-file/)`。

和往常一样，本文的源代码可以在`[on Github](https://web.archive.org/web/20221001124930/https://github.com/eugenp/tutorials/tree/master/spring-cloud-modules/spring-cloud-docker)`中找到。