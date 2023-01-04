# Docker 图像和容器之间的差异

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/ops/docker-images-vs-containers>

## 1.概观

Docker 是一个用于轻松创建、部署和运行应用程序的工具。它允许我们将我们的应用程序与所有的依赖项打包，并将它们作为单独的包分发。Docker 保证我们的应用程序将在每个 Docker 实例上以相同的方式运行。

当我们开始使用 Docker 时，有两个主要概念**我们需要弄清楚——**图像和容器**。**

在本教程中，我们将了解它们是什么以及它们有什么不同。

## 2.Docker 图像

映像是一个文件，它表示一个打包的应用程序，该应用程序具有正确运行所需的所有依赖关系。换句话说，我们可以说 **Docker 映像就像一个 Java 类**。

**图像由一系列层构成**。层被组装在彼此的顶部。那么，什么是层呢？简单来说，层就是一个图像。

假设我们想要创建一个 Hello World Java 应用程序的 Docker 映像。我们首先需要考虑的是我们的应用程序需要什么。

首先，它是一个 Java 应用程序，所以我们需要一个 JVM。好吧，这看起来很简单，但是 JVM 需要运行什么呢？它需要一个操作系统。因此，**我们的 Docker 映像将有一个操作系统层、一个 JVM 和我们的 Hello World 应用程序**。

Docker 的一个主要优势是其庞大的社区。如果我们想要构建一个图像，我们可以去 [Docker Hub](https://web.archive.org/web/20220727020632/https://hub.docker.com/) 搜索我们需要的图像是否可用。

假设我们想要创建一个数据库，使用 [PostgreSQL](https://web.archive.org/web/20220727020632/https://hub.docker.com/_/postgres) 数据库。我们不需要从头开始创建新的 PostgreSQL 映像。我们只要去 Docker Hub，搜索`postgres`，这是 PostgresSQL 的 Docker 官方镜像名，选择我们需要的版本，运行它。

我们从 Docker Hub 创建或提取的每个图像都存储在我们的文件系统中，并通过其名称和标签进行识别。也可以通过其图像 id 来识别**。**

使用`docker images`命令，我们可以查看文件系统中可用的图像列表:

```
$ docker images
REPOSITORY           TAG                 IMAGE ID            CREATED             SIZE
postgres             11.6                d3d96b1e5d48        4 weeks ago         332MB
mongo                latest              9979235fc504        6 weeks ago         364MB
rabbitmq             3-management        44c4867e4a8b        8 weeks ago         180MB
mysql                8.0.18              d435eee2caa5        2 months ago        456MB
jboss/wildfly        18.0.1.Final        bfc71fe5d7d1        2 months ago        757MB
flyway/flyway        6.0.8               0c11020ffd69        3 months ago        247MB
java                 8-jre               e44d62cf8862        3 years ago         311MB
```

## 3.运行 Docker 图像

使用带有图像名称和标签的`docker run`命令运行图像。假设我们想要运行 postgres 11.6 映像:

```
docker run -d postgres:11.6
```

注意我们提供了`-d`选项。这告诉 Docker 在后台运行图像—也称为分离模式。

使用`docker ps`命令，我们可以检查我们的映像是否正在运行。我们应该使用此命令:

```
$ docker ps
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS               NAMES
3376143f0991        postgres:11.6       "docker-entrypoint.s…"   3 minutes ago       Up 3 minutes        5432/tcp            tender_heyrovsky
```

注意上面输出中的`CONTAINER ID`。让我们看看什么是容器，以及它是如何与图像相关联的。

## 4.码头集装箱

容器是图像的一个实例。每个容器都可以通过其 ID 来识别。回到我们的 Java 开发类比，我们可以说**一个容器就像一个类**的实例。

Docker 为容器定义了七种状态:`created`、`restarting`、`running`、`removing`、`paused`、`exited`和`dead`。知道这一点很重要。因为容器只是图像的一个实例，所以它不需要运行。

现在让我们再想想上面看到的`run`命令。我们说过它是用来运行图像的，但这并不完全准确。事实是，`run`命令是用来给`create`和`start`的图像添加一个新的容器。

一个很大的优势是容器就像轻量级的虚拟机。他们的行为彼此完全隔绝。这意味着我们可以运行同一个图像的多个容器，使每个容器处于不同的状态，具有不同的数据和不同的 id。

能够同时运行同一个图像的多个容器是一个很大的优势，因为它允许我们以一种简单的方式扩展应用程序。举个例子，我们来想想微服务。如果每个服务都被打包成 Docker 映像，那么这意味着新服务可以按需作为容器部署。

## 5.容器生命周期

前面，我们提到了容器的七种状态，现在，让我们看看如何使用`docker`命令行工具来处理不同的生命周期状态。

启动一个新容器需要我们先`create`它，然后再`start`它。这意味着它必须经过创建状态才能运行。我们可以通过显式创建和启动容器来实现这一点:

```
docker container create <image_name>:<tag>
docker container start <container_id>
```

或者我们可以用`run`命令轻松做到这一点:

```
docker run <image_name>:<tag>
```

我们可以暂停一个正在运行的容器，然后再次将其置于运行状态:

```
docker pause <container_id>
docker unpause <container_id>
```

当我们检查流程时，暂停的容器将显示“暂停”状态:

```
$ docker ps
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS                  PORTS               NAMES
9bef2edcad7b        postgres:11.6       "docker-entrypoint.s…"   5 minutes ago       Up 4 minutes (Paused)   5432/tcp            tender_heyrovsky
```

我们还可以停止正在运行的容器，然后重新运行它:

```
docker stop <container_id>
docker start <container_id>
```

最后，我们可以移除一个容器:

```
docker container rm <container_id>
```

只能删除处于停止或已创建状态的容器。

关于 Docker 命令的更多信息，我们可以参考 [Docker 命令行参考](https://web.archive.org/web/20220727020632/https://docs.docker.com/engine/reference/commandline/cli/)。

## 6.结论

在本文中，我们讨论了 Docker 图像和容器以及它们之间的区别。图像描述了应用程序及其运行方式。容器是映像实例，其中可以运行同一映像的多个容器，每个容器处于不同的状态。

我们还讨论了容器的生命周期，并学习了管理它们的基本命令。

现在我们已经知道了基础知识，是时候了解更多关于 Docker 这个激动人心的世界了，开始增长我们的知识吧！