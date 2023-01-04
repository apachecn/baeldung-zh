# 如何在 Docker 中处理数据库？

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/ops/docker-databases>

## 1.概观

在本文中，我们将回顾如何使用 Docker 来管理数据库。

在第一章中，我们将介绍在本地机器上安装数据库。然后我们将发现数据持久性是如何跨容器工作的。

最后，我们将讨论在 Docker 生产环境中实现数据库的可靠性。

## 2.在本地运行 Docker 映像

### 2.1.从标准的 Docker 图像开始

首先，我们必须安装 [Docker 桌面](https://web.archive.org/web/20220525123245/https://www.docker.com/get-started)。**然后，我们应该从 [Docker Hub](https://web.archive.org/web/20220525123245/https://hub.docker.com/)** 中找到我们数据库的现有图像。一旦我们找到它，我们将从页面的右上角选择`docker pull`命令。

在本教程中，我们将使用 PostgreSQL，因此命令是:

```java
$docker pull postgres
```

下载完成后，**`docker run`命令将在 Docker 容器**中创建一个运行数据库。对于 PostgreSQL，必须使用`-e`选项指定`POSTGRES_PASSWORD`环境变量:

```java
$docker run -e POSTGRES_PASSWORD=password postgres
```

接下来，我们将测试我们的数据库容器连接。

### 2.2.将 Java 项目连接到数据库

让我们做一个简单的测试。我们将使用 JDBC 数据源将本地 Java 项目连接到数据库。连接字符串应该使用`localhost`上的默认 PostgreSQL 端口`5432`:

```java
jdbc:postgresql://localhost:5432/postgres?user=postgres&password;=password
```

一个错误应该通知我们端口没有打开。事实上，数据库正在监听来自容器网络内部的连接，而我们的 Java 项目正在它的外部运行。

为了解决这个问题，**我们需要将集装箱港口映射到我们的`localhost`港口**。我们将为 PostgreSQL 使用默认端口 5432:

```java
$docker run -p 5432:5432 -e POSTGRES_PASSWORD=password postgres
```

现在连接正常了，我们应该可以使用我们的 JDBC 数据源了。

### 2.3.运行 SQL 脚本

现在，我们可以从 shell 连接到我们的数据库，例如，运行一个初始化脚本。

首先，让我们找到正在运行的容器 id:

```java
$docker ps
CONTAINER ID   IMAGE      COMMAND                  CREATED          STATUS          PORTS                    NAMES
65d9163eece2   postgres   "docker-entrypoint.s…"   27 minutes ago   Up 27 minutes   0.0.0.0:5432->5432/tcp   optimistic_hellman
```

然后，**我们将运行带有交互式`-it`选项的`docker exec`命令来运行容器**内的 shell:

```java
$docker exec -it 65d9163eece2 bash
```

最后，我们可以使用命令行客户端连接到数据库实例，并粘贴我们的 SQL 脚本:

```java
[[email protected]](/web/20220525123245/https://www.baeldung.com/cdn-cgi/l/email-protection):/# psql -U postgres
postgres=#CREATE DATABASE TEST;
CREATE TABLE PERSON(
  ID INTEGER PRIMARY KEY,
  FIRST_NAME VARCHAR(1000),
  LAST_NAME VARCHAR(1000)
);
...
```

例如，如果我们有一个大的转储文件要加载，我们必须避免复制粘贴。**我们可以使用`docker exec`命令**直接从主机运行导入命令:

```java
$docker exec 65d9163eece2 psql -U postgres < dump.sql
```

## 3.用 Docker 卷保存数据

### 3.1.为什么我们需要体积？

只要我们使用同一个容器，我们的基本设置就会工作，每次我们需要重新启动时使用`docker container stop/start`。**如果我们再次使用`docker run`，一个新的空容器将被创建，我们将丢失我们的数据**。事实上，默认情况下，Docker 将数据保存在临时目录中。

现在，我们将学习如何修改这个卷映射。

### 3.2.Docker 卷设置

**第一个任务是检查我们的容器，看看哪个卷被我们的数据库使用:**

```java
$docker inspect -f "{{ .Mounts }}" 65d9163eece2
[{volume f1033d3 /var/lib/docker/volumes/f1033d3/_data /var/lib/postgresql/data local true }] 
```

我们可以看到卷`f1033d3`已经将容器目录`/var/lib/postgresql/data`映射到在主机文件系统中创建的临时目录`/var/lib/docker/volumes/f1033d3/_data`。

**我们必须通过添加`-v`选项到我们在第 2.1 章中使用的`docker run`命令**来修改这个映射:

```java
$docker run -v C:\docker-db-volume:/var/lib/postgresql/data -e POSTGRES_PASSWORD=password postgres
```

现在，我们可以看到在`C:\docker-db-volume`目录中创建的数据库文件。我们可以在这篇[专用文章](/web/20220525123245/https://www.baeldung.com/ops/docker-volumes)中找到高级卷配置。

因此，每次我们使用`docker run` 命令`,` 时，数据都会随着不同的容器执行而持久化。

此外，我们可能希望在团队成员之间或跨不同环境共享配置。我们可以使用 Docker 编写文件，每次都会创建新的容器。在这种情况下，卷是必需的。

下一章将介绍 Docker 数据库在生产环境中的具体使用。

## 4.在生产中与 Docker 合作

Docker Compose 非常适合作为无状态服务共享配置和管理容器。如果服务失败或无法处理工作负载，我们可以配置 Docker Compose 来自动创建新的容器。这对于为 REST 后端构建生产集群非常有用，REST 后端在设计上是无状态的。

**然而，数据库是有状态的，它们的管理更加复杂**:让我们回顾一下不同的上下文。

### 4.1.单实例数据库

让我们假设我们正在构建一个非关键环境，用于测试或生产，该环境能够容忍停机时间(在部署、备份或故障期间)。

**在这种情况下，我们不需要高可用性集群，我们可以简单地对单实例数据库使用 Docker Compose:**

*   我们可以使用一个简单的卷来存储数据，因为容器将在同一台机器上执行
*   我们可以使用[全局模式](https://web.archive.org/web/20220525123245/https://docs.docker.com/compose/compose-file/compose-file-v3/#mode)限制它一次运行一个容器

让我们来看一个极简主义的例子:

```java
version: '3'
services:       
  database:
    image: 'postgres'
    deploy:
      mode: global
    environment:
      - POSTGRES_PASSWORD=password
    ports:
      - "5432:5432"
    volumes:
      - "C:/docker-db-volume:/var/lib/postgresql/data"
```

使用这个配置，我们的产品将一次只创建一个容器，并重用我们的`C:\docker-db-volume`目录中的数据文件。

然而，在这种配置下，定期备份更加重要。如果出现配置错误，这个目录可能会被容器删除或损坏。

### 4.2.复制的数据库

现在让我们假设我们的生产环境很关键。

在这种情况下，像 [Docker Swarm](https://web.archive.org/web/20220525123245/https://docs.docker.com/engine/swarm/) 和 [Kubernetes](https://web.archive.org/web/20220525123245/https://kubernetes.io/fr/) 这样的编排工具对无状态容器是有益的:它们提供垂直和水平集群，具有负载平衡、故障转移和自动伸缩能力。

不幸的是，由于我们的数据库容器是有状态的，这些解决方案不提供卷复制机制。

另一方面，构建自制配置是危险的，因为这会导致严重的数据丢失。例如:

*   对卷使用**共享存储，如 NFS 或 NAS，不能保证在另一个实例中重启数据库时不会丢失数据**
*   在主从集群上，**让 Docker 编排选择多个主节点**是一个常见的错误，这会导致数据损坏

到目前为止，我们的不同选择是:

*   不要对数据库使用 Docker，并且**实现特定于数据库或硬件的复制机制**
*   不要将 Docker 用于数据库，并且**订阅平台即服务解决方案**，如 OpenShift、Amazon AWS 或 Azure
*   使用 Docker 特有的复制机制，比如 [KubeDB](https://web.archive.org/web/20220525123245/https://kubedb.com/) 和 [Portworx](https://web.archive.org/web/20220525123245/https://portworx.com/)

## 5.结论

在本文中，我们回顾了适用于开发、测试和非关键生产的基本配置。

最后，我们得出结论，Docker 在高可用性环境中使用时有缺点。因此，应该避免使用它，或者与数据库集群中的专用解决方案结合使用。