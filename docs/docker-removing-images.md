# 移除 Docker 图像

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/ops/docker-removing-images>

## 1.介绍

[在之前的文章](/web/20221126234722/https://www.baeldung.com/docker-images-vs-containers)中，我们解释了 Docker 图像和 Docker 容器之间的区别。简而言之:**图像就像 Java 类，容器就像 Java 对象。**

在本教程中，我们将看看各种方式来删除 Docker 图像。

## 2.为什么删除 Docker 图像？

Docker 引擎存储图像并运行容器。为此，**Docker 引擎保留一定量的磁盘空间作为“存储池”**用于存储图像、容器和其他任何东西(如全局 Docker 卷或网络)。

一旦存储池满了，Docker 引擎就停止工作:我们不能再创建或下载新的映像，我们的容器也无法运行。

**Docker 映像占据了 Docker 引擎存储池**的大部分。因此**我们移除 Docker 映像以保持 Docker 运行**。

我们还**删除图像，以保持 Docker 引擎的有序和干净**。例如，我们可以很容易地在开发过程中创建许多我们很快就不再需要的图像。或者，我们下载一些软件镜像用于测试，我们可以稍后处理。

我们**可以很容易地删除我们从 Docker 库**中提取的 Docker 映像:如果我们还需要它，我们只需再从库中提取一次。

但是我们必须小心我们自己创建的 Docker 图像:**一旦删除，o** **你自己的图像就消失了，除非我们保存它们！**我们可以通过将 Docker 图像推送到存储库或[导出到 TAR 文件](https://web.archive.org/web/20221126234722/https://docs.docker.com/engine/reference/commandline/save/)来保存它们。

## 3.下载 PostgreSQL 13 测试版映像

[PostgreSQL](https://web.archive.org/web/20221126234722/https://www.postgresql.org/) 是一个开源的关系数据库。我们将使用[的前两个 PostgreSQL 13 beta Docker 映像](https://web.archive.org/web/20221126234722/https://www.postgresql.org/about/news/2047/)作为例子。这两张图比较小，我们可以快速下载。因为它们是测试版软件，我们的 Docker 引擎中还没有它们。

我们将使用 beta 2 映像创建一个容器。我们不会直接使用 beta 1 图像。

但是在我们下载这两个映像之前，让我们先检查一下 Docker 映像在存储池中占用了多少空间:

```java
docker system df --format 'table {{.Type}}\t{{.TotalCount}}\t{{.Size}}'
```

这是测试机器的输出。第一行显示我们的 71 个 Docker 映像使用 7.8 GB:

```java
TYPE                TOTAL               SIZE
Images              71                  7.813GB
Containers          1                   359.1MB
Local Volumes       203                 14.54GB
Build Cache         770                 31.54GB
```

现在，我们下载两个 PostgreSQL 映像，并重新检查 Docker 存储池:

```java
docker pull postgres:13-beta1-alpine
docker pull postgres:13-beta2-alpine
docker system df --format 'table {{.Type}}\t{{.TotalCount}}\t{{.Size}}' 
```

不出所料，图像数量从 71 个增加到 73 个。整个图像大小从 7.8 GB 变成了 8.1 GB。

为了简洁起见，我们只显示第一行:

```java
TYPE                TOTAL               SIZE
Images              73                  8.119GB 
```

## 4.移除单个图像

让我们用 PostgreSQL 13 beta 2 映像开始一个容器。我们将`secr3t`设置为数据库根用户的密码，因为 PostgreSQL 容器不会在没有密码的情况下启动:

```java
docker run -d -e POSTGRES_PASSWORD=secr3t postgres:13-beta2-alpine
docker ps --format 'table {{.ID}}\t{{.Image}}\t{{.Status}}'
```

下面是测试机器上的运行容器:

```java
CONTAINER ID        IMAGE                      STATUS
527bfd4cfb89        postgres:13-beta2-alpine   Up Less than a second
```

现在我们来移除 PostgreSQL 13 beta 2 镜像。我们**使用 [`docker image rm`](https://web.archive.org/web/20221126234722/https://docs.docker.com/engine/reference/commandline/image_rm/) 移除一个码头工人图像**。该命令会删除一个或多个图像:

```java
docker image rm postgres:13-beta2-alpine 
```

此命令失败，因为正在运行的容器仍在使用该映像:

```java
Error response from daemon: conflict: unable to remove repository reference "postgres:13-beta2-alpine" (must force) - container 527bfd4cfb89 is using its referenced image cac2ee40fa5a
```

因此，让我们使用从`docker ps`获得的 ID 来停止正在运行的容器:

```java
docker container stop 527bfd4cfb89
```

我们现在再次尝试删除图像——并得到相同的错误消息:**我们不能删除容器使用的图像，无论是否运行**。

所以让我们把容器移走。然后我们终于可以移除图像了:

```java
docker container rm 527bfd4cfb89
docker image rm postgres:13-beta2-alpine 
```

Docker 引擎打印图像删除的详细信息:

```java
Untagged: postgres:13-beta2-alpine
Untagged: [[email protected]](/web/20221126234722/https://www.baeldung.com/cdn-cgi/l/email-protection):b3a4ebdb37b892696a7bd7e05763b938345f29a7327fc17049c7148c03ff6a92
removed: sha256:cac2ee40fa5a40f0abe53e0138033fe7a9bcee28e7fb6c9eaac4d3a2076b1a86
removed: sha256:6a14bab707274a8007da33fe08ea56a921f356263d8fd5e599273c7ee4880170
removed: sha256:5e6ef40b9f6f8802452dbca622e498caa460736d890ca20011e7c79de02adf28
removed: sha256:dbd38ed4b347c7f3c81328742a1ddeb1872ad52ac3b1db034e41aa71c0d55a75
removed: sha256:23639f6bd6ab4b786e23d9d7c02a66db6d55035ab3ad8f7ecdb9b1ad6efeec74
removed: sha256:8294c0a7818c9a435b8908a3bcccbc2171c5cefa7f4f378ad23f40e28ad2f843
```

`docker system df`确认删除:图像数量从 73 减少到 72。整体图像大小从 8.1 GB 变为 8.0 GB:

```java
TYPE                TOTAL               SIZE
Images              72                  7.966GB
```

## 5.按名称删除多个图像

让我们再次下载我们在上一节中刚刚删除的 PostgreSQL 13 beta 2 映像:

```java
docker pull postgres:13-beta2-alpine
```

现在，我们希望按名称删除 beta 1 映像和 beta 2 映像。到目前为止，我们只使用了 beta 2 图像。如前所述，我们没有直接使用 beta 1 映像，所以我们现在可以删除它。

遗憾的是，`docker image rm`没有提供按名称删除的过滤选项。相反，**我们将链接 Linux 命令，按名称**删除多个图像。

我们将通过存储库和标签引用图像，就像在`docker pull`命令中一样:存储库是`postgres`，标签是`13-beta1-alpine`和`13-beta2-alpine`。

因此，要按名称删除多个图像，我们需要:

*   按存储库和标签列出所有图像，例如`postgres:13-beta2-alpine`
*   然后，使用 [`grep`](https://web.archive.org/web/20221126234722/https://www.linux.org/docs/man1/grep.html) 命令:`^postgres:13-beta`通过正则表达式过滤那些输出行
*   最后，将这些行输入到`docker image rm`命令中

让我们开始把这些放在一起。为了测试正确性，让我们只运行其中的前两段:

```java
docker image ls --format '{{.Repository}}:{{.Tag}}' | grep '^postgres:13-beta'
```

在我们的测试机上，我们得到:

```java
postgres:13-beta2-alpine
postgres:13-beta1-alpine 
```

鉴于此，我们可以将它添加到我们的`docker image rm`命令中:

```java
docker image rm $(docker image ls --format '{{.Repository}}:{{.Tag}}' | grep '^postgres:13-beta')
```

和以前一样，我们只能在没有容器(运行或停止)使用图像的情况下删除它们。然后，我们会看到与上一节相同的图像移除细节。`docker system df`显示我们在测试机上返回了 71 个 7.8 GB 的图像:

```java
TYPE                TOTAL               SIZE
Images              71                  7.813GB
```

这个图像删除命令在 Linux 和 Mac 上的终端中有效。在 Windows 上，它需要 [Docker 工具箱](https://web.archive.org/web/20221126234722/https://docs.docker.com/toolbox/toolbox_install_windows/)的“Docker 快速启动终端”。将来，更新的 [Docker 桌面 Windows 版](https://web.archive.org/web/20221126234722/https://docs.docker.com/docker-for-windows/)可能[也会在 Windows 10 上使用这个 Linux 命令](https://web.archive.org/web/20221126234722/https://code.visualstudio.com/blogs/2020/03/02/docker-in-wsl2)。

## 6.按大小移除图像

节省磁盘空间的一个好方法是首先删除最大的 Docker 映像。

现在`docker image ls`也不能按大小排序了。因此，**我们列出所有图像，并用 [`sort`](/web/20221126234722/https://www.baeldung.com/linux/sort-command) 命令对输出进行排序，以便按大小查看图像**:

```java
docker image ls | sort -k7 -h -r
```

在我们的测试机器上输出:

```java
collabora/code   4.2.5.3         8ae6850294e5   3 weeks ago  1.28GB
nextcloud        19.0.1-apache   25b6e2f7e916   6 days ago   752MB
nextcloud        latest          6375cff75f7b   5 weeks ago  750MB
nextcloud        19.0.0-apache   5c44e8445287   7 days ago   750MB 
```

接下来，我们手动检查以找到我们想要删除的内容。ID(第三列)比存储库和标签(第一列和第二列)更容易复制和粘贴。Docker 允许一次删除多个图像。

假设我们想要删除`nextcloud:latest`和`nextcloud:19.0.0-apache`。简单地说，我们可以在我们的表中查看它们对应的 id，并在我们的`docker image rm`命令中列出它们:

```java
docker image rm 6375cff75f7b 5c44e8445287
```

和以前一样，我们只能删除没有被任何容器使用的图像，并查看通常的图像删除细节。现在，我们在测试机上减少到 69 个 7.1 GB 的图像:

```java
TYPE                TOTAL               SIZE
Images              69                  7.128GB
```

## 7.按创建日期删除图像

**Docker 可以根据图像的创建日期**删除图像。为此，我们将使用新的`docker image prune`命令。与`docker image rm`不同的是，它被设计成移除多个图像甚至所有图像。

现在，让我们删除 2020 年 7 月 7 日之前创建的所有映像:

```java
docker image prune -a --force --filter "until=2020-07-07T00:00:00"
```

我们仍然只能删除没有被任何容器使用的图像，我们仍然可以看到通常的图像删除细节。该命令删除了测试机器上的两个映像，因此我们在测试机器上有 67 个映像和 5.7 GB:

```java
TYPE                TOTAL               SIZE
Images              67                  5.686GB
```

按创建日期删除图像的另一种方法是指定时间跨度，而不是截止日期。假设我们想删除一周前的所有图像:

```java
docker image prune -a --force --filter "until=168h"
```

请注意，Docker 过滤器选项要求我们将时间跨度转换为小时。

## 8.修剪容器和图像

[`**docker image prune**`](https://web.archive.org/web/20221126234722/https://docs.docker.com/engine/reference/commandline/image_prune/) **批量删除未使用的图像**。它与 **[`docker container prune`](https://web.archive.org/web/20221126234722/https://docs.docker.com/engine/reference/commandline/container_prune/) 一起批量移除停止的集装箱**。让我们从最后一个命令开始:

```java
docker container prune
```

这会打印一条警告消息。我们必须输入`y`并按下`Enter`才能继续:

```java
WARNING! This will remove all stopped containers.
Are you sure you want to continue? [y/N] y
removed Containers:
1c3be3eba8837323820ecac5b82e84ab65ad6d24a259374d354fd561254fd12f

Total reclaimed space: 359.1MB
```

所以在测试机器上，这个移除了一个停止的容器。

现在我们需要简单讨论一下图像关系。我们的 Docker 映像扩展其他映像来获得它们的功能，就像 Java 类扩展其他 Java 类一样。

让我们看看 PostgreSQL beta 2 映像的[docker 文件的顶部，看看它扩展了什么映像:](https://web.archive.org/web/20221126234722/https://github.com/docker-library/postgres/blob/bb0d97951918e6d281f510adb3896da433a52bc4/13/alpine/Dockerfile)

```java
FROM alpine:3.12
```

所以 beta 2 镜像用`[alpine:3.12](https://web.archive.org/web/20221126234722/https://hub.docker.com/_/alpine)`。这就是为什么 Docker 在我们一开始拉 beta 2 镜像的时候隐式下载了`alpine:3.12`。我们看不到这些带有`docker image ls`的隐式下载图像。

现在假设我们删除了 PostgreSQL 13 beta 2 映像。如果没有其他 Docker 映像扩展了`alpine:3.12`，那么 Docker 会认为`alpine:3.12`是一个所谓的“悬空映像”:一个曾经隐式下载的映像，现在不再需要了。 **`docker image prune`去掉这些悬空的图像:**

```java
docker image prune
```

该命令还要求我们输入`y`并按下`Enter`继续:

```java
WARNING! This will remove all dangling images.
Are you sure you want to continue? [y/N] y
Total reclaimed space: 0B
```

在测试机器上，这并没有删除任何图像。

**`docker image prune -a`删除容器**未使用的所有图像。所以**如果我们没有任何容器(运行或不运行)，那么这将删除所有 Docker 图像**！这的确是一个危险的命令:

```java
docker image prune -a
```

在测试机器上，这删除了所有图像。`docker system df`确认没有留下容器或图像:

```java
TYPE                TOTAL               SIZE
Images              0                   0B
Containers          0                   0B 
```

## 9.强行移除容器和图像

`docker prune`命令删除停止的容器和悬挂的图像。但是，如果我们希望从我们的机器上删除所有的 Docker 图像呢？为此，我们首先需要删除我们机器上运行的所有 Docker 容器，然后删除 Docker 映像:

```java
docker rm -f $(docker ps -qa)
```

该命令将删除所有容器。`-f`标志用于强制移除正在运行的 Docker 容器。

现在让我们使用`docker rmi`命令删除所有 Docker 图像:

```java
docker rmi -f $(docker images -aq)
```

`docker images -qa`将返回所有 Docker 图像的图像 id。然后,`docker rmi`命令将逐个删除所有图像。再次使用`-f`标志强制删除 Docker 图像。

因为所有的 Docker 容器都已经从机器上移除了，所以我们现在也可以使用`docker image prune`命令来移除所有的 Docker 图像。

## 10.结论

在本文中，我们首先看到了如何删除单个 Docker 图像。接下来，我们学习了如何根据名称、大小或创建日期删除图像。最后，我们学习了如何移除所有未使用的容器和图像。