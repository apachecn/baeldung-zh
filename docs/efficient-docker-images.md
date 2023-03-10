# 创建高效 Docker 图像的技巧

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/ops/efficient-docker-images>

## 1.概观

在过去几年中，Docker 已经成为 Linux 上容器化的事实上的标准。Docker 易于使用，并提供轻量级虚拟化，随着越来越多的服务在云中运行，它非常适合构建应用程序和微服务。

尽管创建我们的第一张[图片](/web/20220727020704/https://www.baeldung.com/docker-images-vs-containers)可能相对容易，但构建高效的图片需要深谋远虑。在本教程中，我们将看到如何编写高效的 Docker 图像的例子，以及每个建议背后的原因。

先说官方图像的使用。

## 2.将你的形象建立在官方形象的基础上

### 2.1.什么是官方形象？

**[Docker 官方形象](https://web.archive.org/web/20220727020704/https://docs.docker.com/docker-hub/official_images/)是由 Docker 赞助的团队创建和维护的，或者至少是他们认可的**。他们在 GitHub 项目上公开管理 Docker 图像。他们还会在发现漏洞时进行更改，并确保映像是最新的并遵循最佳实践。

让我们通过一个使用 [Nginx](https://web.archive.org/web/20220727020704/https://hub.docker.com/_/nginx) 官方图片的例子来更清楚地了解这一点。网络服务器的创建者维护这一形象。

假设我们想使用 Nginx 来托管我们的静态网站。我们可以创建 docker 文件，并以官方图像为基础:

```java
FROM nginx:1.19.2
COPY my-static-website/ /usr/share/nginx/html
```

然后我们可以建立我们的形象:

```java
$ docker build -t my-static-website .
```

最后，运行它:

```java
$ docker run -p 8080:80 -d my-static-website
```

我们的 docker 文件只有两行长。基本的官方映像处理了 Nginx 服务器的所有细节，比如默认配置文件和应该公开的端口。

更具体地说，基础映像阻止 Nginx 成为守护进程并结束初始进程。这种行为在其他环境中是意料之中的，但是在 Docker 中，这被解释为应用程序的结束，因此容器终止。解决方案是配置 Nginx 不要成为守护进程。这是官方图中的[配置](https://web.archive.org/web/20220727020704/https://github.com/nginxinc/docker-nginx/blob/1.19.2/stable/buster/Dockerfile#L110):

```java
CMD ["nginx", "-g", "daemon off;"]
```

当我们以官方图片为基础时，我们会避免难以调试的意外错误。官方的图像维护人员是 Docker 和我们想要使用的软件方面的专家，所以我们可以从他们的知识中获益，还可以节省时间。

### 2.2.由创作者维护的图像

虽然在前面解释的意义上不是官方的，但是 Docker Hub 中还有其他图像也是由应用程序的创建者维护的。

让我们用一个例子来说明这一点。 [EMQX](https://web.archive.org/web/20220727020704/https://hub.docker.com/r/emqx/emqx) 是一个 MQTT 消息代理。假设我们想将这个代理作为应用程序中的一个微服务。我们可以基于他们的映像添加我们的配置文件。或者更好的是，我们可以通过[环境变量](/web/20220727020704/https://www.baeldung.com/ops/docker-container-environment-variables)使用他们的条款来配置 EMQX。

例如，要更改 EMQX 监听的默认端口，我们可以添加`EMQX_LISTENER__TCP__EXTERNAL`环境变量:

```java
$ docker run -d -e EMQX_LISTENER__TCP__EXTERNAL=9999 -p 9999:9999 emqx/emqx:v4.1.3
```

作为一个特定软件背后的社区，他们最有条件为自己的软件提供 Docker 形象。

在某些情况下，对于我们想要使用的应用程序，我们找不到任何形式的官方图像。即使在这种情况下，我们也可以通过搜索 Docker Hub 来获取可用作参考的图片。

让我们以 H2 为例。H2 是用 Java 编写的轻量级关系数据库。尽管没有 H2 的官方图片，一个第三方创建了一个并很好地记录了下来。我们可以使用他们的 [GitHub 项目](https://web.archive.org/web/20220727020704/https://github.com/oscarfonts/docker-h2)来学习如何使用 H2 作为一个独立的服务器，甚至合作来保持项目的最新。

即使我们仅使用 Docker 形象项目作为建立我们形象的起点，我们也可能学到比从头开始更多的东西。

## 3.尽可能避免构建新的映像

当使用 Docker 时，我们可能会养成总是创建新图像的习惯，即使与基本图像相比变化很小。对于这些情况，**我们可以考虑将我们的配置直接添加到运行容器中，而不是构建映像**。

每次发生变化时，都需要重新构建自定义 Docker 映像。之后还需要将它们上传到注册表中。如果图像包含敏感信息，我们可能需要将其存储在私有存储库中。在某些情况下，通过使用基本映像并动态配置它们，而不是每次都构建自定义映像，我们可能会获得更多好处。

让我们以 HAProxy 为例来说明这一点。和 Nginx 一样，HAProxy 可以作为反向代理。码头工人社区保持其官方形象。

假设我们需要配置 HAProxy，将请求重定向到应用程序中适当的微服务。所有这些逻辑都可以写在一个配置文件中，比如说`my-config.cfg`。Docker 映像要求我们将配置放在特定的路径上。

让我们看看如何使用安装在运行容器上的自定义配置来运行 HAProxy:

```java
$ docker run -d -v my-config.cfg:/usr/local/etc/haproxy/haproxy.cfg:ro haproxy:2.2.2
```

这样，即使升级 HAProxy 也变得更加简单，因为我们只需要更改标签。当然，我们还需要确认我们的配置仍然适用于新版本。

如果我们正在构建一个由许多容器组成的解决方案，我们可能已经在使用一个 orchestrator，比如 Docker Swarm 或 Kubernetes。它们提供了存储配置并将其链接到运行中的容器的方法。Swarm 称它们为[配置](https://web.archive.org/web/20220727020704/https://docs.docker.com/engine/swarm/configs/)，Kubernetes 称它们为[配置图](https://web.archive.org/web/20220727020704/https://kubernetes.io/docs/concepts/configuration/configmap/)。

**编排工具已经考虑到我们可能会在我们使用的映像之外存储一些配置**。在某些情况下，将我们的配置放在映像之外可能是最好的妥协。

## 4.创建精简的图像

图像大小很重要，原因有二。首先，**较亮的图像传输速度更快**。当我们在开发机器中构建映像时，它可能看起来不像是游戏规则改变者。尽管如此，当我们在 CI/CD 管道上构建几个映像并部署到几个服务器上时，每次部署节省的总时间可能是显而易见的。

第二，为了实现一个更精简的镜像版本，我们需要**删除镜像没有使用的额外的包。这将帮助我们减少攻击面**，从而提高图像的安全性。

让我们来看两种简单的方法来缩小 Docker 图片的尺寸。

### 4.1.尽可能使用超薄型

这里，我们有两个主要选择:Debian 的精简版本和 Alpine Linux 发行版。

精简版是 Debian 社区从标准映像中删除不必要文件的努力成果。许多 Docker 图片已经是基于 Debian 的精简版本。

例如，HAProxy 和 Nginx 映像基于 Debian 发行版的精简版`debian:buster-slim`。多亏了它，这些图像从几百 MB 变成了几十 MB。

在其他一些情况下，该图像提供了一个超薄版本以及标准的全尺寸版本。例如，最新的 [Python 图像](https://web.archive.org/web/20220727020704/https://hub.docker.com/_/python)提供了一个苗条的版本，目前是`python:3.7.9-slim`，它几乎比标准图像小十倍。

另一方面，许多图像提供了阿尔卑斯山版本，就像我们之前提到的 Python 图像。基于 Alpine 的图像大小通常在 10 MB 左右。

Alpine Linux 从一开始就考虑到了资源效率和安全性。这使得它非常适合基本 Docker 图像。

需要记住的一点是，Alpine Linux 在几年前选择将系统库从更常见的`glibc`改为`musl`。尽管大多数软件都可以正常工作，但是如果我们选择 Alpine 作为我们的基础映像，我们还是要对我们的应用程序进行彻底的测试。

### 4.2.使用多阶段构建

[多阶段构建](https://web.archive.org/web/20220727020704/https://docs.docker.com/develop/develop-images/multistage-build/#use-multi-stage-builds)功能**允许在同一个 docker 文件的多个阶段中构建映像，通常在下一个阶段中使用前一阶段的结果**。让我们看看这是如何有用的。

假设我们想使用 HAProxy，并用它的 REST API 动态配置它，即[数据平面 API](https://web.archive.org/web/20220727020704/https://www.haproxy.com/blog/announcing-haproxy-dataplane-api-20/) 。因为这个 API 二进制文件在基础 HAProxy 映像中不可用，所以我们需要在构建时下载它。

我们可以在一个阶段下载 HAProxy API 二进制文件，并将其提供给下一个阶段:

```java
FROM haproxy:2.2.2-alpine AS downloadapi
RUN apk add --no-cache curl
RUN curl -L https://github.com/haproxytech/dataplaneapi/releases/download/v2.1.0/dataplaneapi_2.1.0_Linux_x86_64.tar.gz --output api.tar.gz
RUN tar -xf api.tar.gz
RUN cp build/dataplaneapi /usr/local/bin/

FROM haproxy:2.2.2-alpine
COPY --from=downloadapi /usr/local/bin/dataplaneapi /usr/local/bin/dataplaneapi
...
```

第一阶段`downloadapi`，下载最新的 API 并解压缩 tar 文件。第二阶段复制二进制文件，以便 HAProxy 以后可以使用它。我们不需要卸载`curl`或删除下载的 tar 文件，因为第一阶段完全被丢弃，不会出现在最终的映像中。

在其他一些情况下，多级图像的优势更加明显。例如，如果我们需要从源代码构建，那么最终的图像将不需要任何构建工具。第一阶段可以安装所有构建工具并构建二进制文件，下一阶段将只复制这些二进制文件。

即使我们不总是使用这个功能，但知道它的存在也很好。在某些情况下，它可能是我们形象瘦身的最佳选择。

## 5.结论

容器化将会继续存在，Docker 是开始使用它的最简单的方法。它的简单性有助于我们快速高效地工作，尽管有些经验只能通过经验来学习。

在本教程中，我们回顾了一些构建更加健壮和安全的映像的技巧。由于容器是现代应用程序的组成部分，它们越可靠，我们的应用程序就越强大。