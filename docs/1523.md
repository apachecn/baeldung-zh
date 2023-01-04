# 波德曼简介

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/ops/podman-intro>

## 1.介绍

在本教程中，我们将了解 Pod man(Pod Manager 的缩写)，它的功能和用法。

## 2.他征服了

Podman 是一个开源的容器管理工具，用于开发、管理和运行 [OCI](https://web.archive.org/web/20220626082433/https://www.opencontainers.org/) 容器。让我们来看看与其他容器管理工具相比，Podman 的一些优势:

*   由 Podman 创建的图像与其他容器管理工具兼容。Podman 创建的图像符合 OCI 标准，因此它们可以被推送到 Docker Hub 等其他容器注册中心
*   它可以作为普通用户运行，不需要 root 权限。当作为非 root 用户运行时，Podman 创建一个用户名称空间，并在其中获得 root 权限。这允许它挂载文件系统并设置所需的容器
*   它提供了管理 pod 的能力。与其他容器运行时工具不同，Podman 允许用户管理容器(一组一个或多个一起操作的容器)。用户可以在窗格上执行创建、列表、检查等操作

然而，波德曼有一定的局限性:

*   它只能在基于 Linux 的系统上运行。目前，Podman 只运行在基于 Linux 的操作系统上，没有针对 Windows 和 macOS 的包装器。
*   **Docker Compose 没有替代品。** Podman 不支持本地管理多个容器，类似于 Docker Compose 所做的。作为`[podman-compose](https://web.archive.org/web/20220626082433/https://github.com/containers/podman-compose)`项目的一部分，使用 Podman 后端的 Docker Compose 的实现正在开发中，但这仍在进行中。

## 3.与 Docker 的比较

现在我们已经了解了什么是 Podman，它的优点和局限性是什么，让我们将它与使用最广泛的容器管理工具之一 Docker 进行比较。

### 3.1.命令行界面(CLI)

Podman 提供了与 Docker 客户端相同的命令集。换句话说，这两个实用程序的命令之间存在一对一的映射。

然而，像`podman ps`和`podman images`这样的命令不会显示使用 Docker 创建的容器或图像。这是因为 Podman 的本地存储库是`/var/lib/containers` ，而不是由 Docker 维护的`/var/lib/docker` 。

### 3.2.集装箱模型

Docker 为容器使用客户机-服务器架构，而 Podman 使用 Linux 进程中常见的传统 fork-exec 模型。使用 Podman 创建的容器是父 Podman 流程的子流程。这就是为什么对 Docker 和 Podman 运行 version 命令时，Docker 会列出客户机和服务器的版本，而 Podman 只列出它的版本。

`docker version`的样本输出:

```
Client:
 Version:       17.12.0-ce
 API version:   1.35
 Go version:    go1.9.2
 Git commit:    c97c6d6
 Built: Wed Dec 27 20:11:19 2017
 OS/Arch:       linux/amd64

Server:
 Engine:
  Version:      17.12.0-ce
  API version:  1.35 (minimum version 1.12)
  Go version:   go1.9.2
  Git commit:   c97c6d6
  Built:        Wed Dec 27 20:09:53 2017
  OS/Arch:      linux/amd64
  Experimental: false
```

`podman version`的样本输出:

```
Version:       0.3.2-dev
Go Version:    go1.9.4
Git Commit:    "4f4a78abb40fa0e8407e8a55d5a67a2650d8fd96"
Built:         Mon Mar  5 11:10:35 2018
OS/Arch:       linux/amd64
```

由于 Podman 本身是作为一个进程运行的，所以它不需要后台的任何守护进程。与 Podman 不同，Docker 需要一个守护进程 Docker daemon 来协调客户端和服务器之间的 API 请求。

### 3.3.无根模式

如前所述，Podman 不需要 root 访问权限来运行它的命令。**另一方面，Docker 依赖于守护进程，需要 root 权限或要求用户成为`docker`组**的一部分，才能在没有 root 权限的情况下运行 Docker 命令**。**

```
$ sudo usermod -aG docker $USER
```

## 4.安装和使用

让我们从安装机器人开始。 `podman info`命令显示 Podman 系统信息，并帮助检查安装状态。

```
$ podman info
```

此命令显示与主机相关的信息，如内核版本、已用和可用的交换空间，以及与 Podman 相关的信息，如它有权获取和推送映像的注册表、它使用的存储驱动程序、存储位置等:

```
host:
  MemFree: 546578432
  MemTotal: 1040318464
  SwapFree: 4216320000
  SwapTotal: 4216320000
  arch: amd64
  cpus: 2
  hostname: base-xenial
  kernel: 4.4.0-116-generic
  os: linux
  uptime: 1m 2.64s
insecure registries:
  registries: []
registries:
  registries:
  - docker.io
  - registry.fedoraproject.org
  - registry.access.redhat.com
store:
  ContainerStore:
    number: 0
  GraphDriverName: overlay
  GraphOptions: null
  GraphRoot: /var/lib/containers/storage
  GraphStatus:
    Backing Filesystem: extfs
    Native Overlay Diff: "true"
    Supports d_type: "true"
  ImageStore:
    number: 0
  RunRoot: /var/run/containers/storage
```

让我们来看看一些基本的 Podman 命令。

### 4.1.创建图像

首先，我们将看看如何使用 Podman 创建图像。让我们从创建一个包含以下内容的`Dockerfile`开始:

```
FROM centos:latest
RUN yum -y install httpd
CMD ["/usr/sbin/httpd", "-D", "FOREGROUND"]
EXPOSE 80 
```

现在让我们使用`build`命令创建图像:

```
$ podman build .
```

在这里，我们首先提取 CentOS 的基本映像，在其上安装 Apache，然后作为前台进程运行它，并暴露端口 80。我们可以通过运行这个映像并将暴露的端口映射到主机端口来访问 Apache 服务器。

`build`命令递归地传递上下文目录中所有可用的文件夹。当没有指定目录时，默认情况下当前工作目录成为构建上下文。因此，建议不要在上下文目录中放置创建映像不需要的文件和文件夹。

### 4.2.列出可用图像

`podman images`命令列出了所有可用的图像。它还支持[各种选项](https://web.archive.org/web/20220626082433/https://podman.readthedocs.io/en/latest/markdown/podman-images.1.html#options)来过滤图像。

```
$ podman images
```

该命令列出了本地存储库中所有可用的图像。它包含图像来自哪个存储库、标记、图像 id、创建时间和大小等信息。

```
REPOSITORY                 TAG      IMAGE ID         CREATED         SIZE
docker.io/library/centos   latest  0f3e07c0138f    2 months ago      227MB
<none>                     <none   49030e844ce7   27 seconds ago     277MB
```

### 4.3.运行图像

`run`命令创建一个给定图像的容器，然后运行它。让我们运行之前创建的 CentOS 映像

```
$ podman run  -p 80:80 -dit centos
```

该命令首先检查 CentOS 是否有可用的本地映像。如果映像不在本地，它会尝试从已配置的注册表中提取映像。如果注册表中没有该图像，它会显示一个错误，说明找不到该图像。

**上面的运行命令指定将容器暴露的 80 端口映射到主机的 80 端口，而`dit` 标志指定以分离和交互模式**运行容器。所创建的容器的 id 将是输出。

### 4.4.删除图像

`rmi`命令删除本地存储库中的图像。通过在输入中以空格分隔的方式提供图像的 id，可以删除多个图像。指定`-a`标志会删除所有图像

```
$ podman rmi 785188cd988c
```

### 4.5.列出集装箱

使用`ps -a`命令可以列出所有可用的容器，包括没有运行的容器。类似于`images` 命令，这也可以与[各种选项](https://web.archive.org/web/20220626082433/https://podman.readthedocs.io/en/latest/markdown/podman-ps.1.html#options)一起使用。

```
$ podman ps -a
```

上述命令的输出列出了所有容器的信息，如创建它的映像、用于启动它的命令、它的状态、它运行的端口以及分配给它的名称。

```
CONTAINER ID   IMAGE    COMMAND     CREATED AT                      STATUS              PORTS                                    NAMES
eed30719cd37   centos   /bin/bash   2019-12-09 02:57:37 +0000 UTC   Up 14 minutes ago   0.0.0.0:80->80/udp, 0.0.0.0:80->80/tcp   reverent_liskov
```

### 4.6.删除容器

`rm`命令移除容器。此命令不会删除处于运行或暂停状态的容器。它们需要首先被阻止，然后被移除。

```
$ podman stop eed30719cd37

$ podman rm eed30719cd37
```

### 4.7.创建窗格

`pod create`命令创建一个 pod。创建命令支持[不同的选项](https://web.archive.org/web/20220626082433/https://podman.readthedocs.io/en/latest/markdown/podman-pod-create.1.html#options)。

```
$ podman pod create
```

`pod create`命令创建一个 pod，默认情况下有一个`infra`容器与之相关联，除非将 infra 标志明确设置为`false.`

```
$ podman pod create --infra = false
```

Infra container 允许 Podman 连接 pod 中的各种容器。

### 4.8.列表窗格

`pod list `命令显示所有可用的窗格

```
$ podman pod list
```

此命令的输出会显示诸如 pod id、其名称、关联容器的数量、基础容器的 id(如果可用)等信息:

```
POD ID         NAME             STATUS      CREATED       # OF CONTAINERS   INFRA ID
7e0a68528aed   gallant_raman    Running    5 seconds ago        1           c6d06673c667
```

所有可用的 Podman 命令及其用法可以在[官方文档](https://web.archive.org/web/20220626082433/https://podman.readthedocs.io/en/latest/Commands.html)中找到。

## 5.结论

在本教程中，我们已经了解了 Podman 的基础知识及其特性，它与 Docker 的比较以及一些可用的命令。

和往常一样，本文中使用的代码示例[可以通过 GitHub](https://web.archive.org/web/20220626082433/https://github.com/eugenp/tutorials/tree/master/podman) 获得。