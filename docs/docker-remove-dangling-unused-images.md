# docker–移除悬挂和未使用的图像

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/ops/docker-remove-dangling-unused-images>

## 1.概观

在本教程中，我们将看看为什么悬挂和未使用的图像在 [Docker](/web/20220919062318/https://www.baeldung.com/ops/docker-guide) 中常见的一些原因。然后，我们将看看一些方法来消除它们。

偶尔清理悬挂的和未使用的 Docker 图像是一个好习惯，因为大量未使用的图像会导致磁盘空间的浪费。

## 2.Docker 中未使用的对象

Docker 不会自动移除未使用的对象。相反，它会将它们保留在磁盘上，直到我们明确要求它删除它们。一些未使用的对象是:

*   没有活动容器的每个提取的图像
*   每个处于停止状态的容器
*   对应于停止和移除的容器的体积
*   构建缓存

让我们来探讨一下使用 Docker 会导致不必要的图像，以及如何删除它们。

### 2.1.悬挂的 Docker 图像

当我们用相同名称和标签的新图像覆盖悬空图像时，就会创建悬空图像。

让我们看一个小例子，看看更新一个图像会导致一个悬空的图像。下面是一个简单的 Dockerfile 文件:

```java
FROM ubuntu:latest
CMD ["echo", "Hello World"] 
```

让我们建立这样的形象:

```java
docker build -t my-image . 
```

我们可以通过运行以下命令来验证映像是否已创建:

```java
docker images
REPOSITORY   TAG       IMAGE ID       CREATED          SIZE
my-image     latest    7ed6e7202eca   32 seconds ago   72.8MB
ubuntu       latest    825d55fb6340   6 days ago       72.8MB 
```

假设我们对 Dockerfile 文件做了一点修改:

```java
FROM ubuntu:latest
CMD ["echo", "Hello, World!"] 
```

让我们使用与之前相同的命令重建图像，并再次列出图像:

```java
docker images
REPOSITORY   TAG       IMAGE ID       CREATED              SIZE
my-image     latest    da6e74196f66   4 seconds ago        72.8MB
<none>       <none>    7ed6e7202eca   About a minute ago   72.8MB
ubuntu       latest    825d55fb6340   6 days ago           72.8MB 
```

构建者创建了一个新`my-image` 图像。正如我们所看到的，旧的图像仍然在那里，但现在它晃来晃去。它的名字和标签被设置为`<none>:<none>`。

请注意，如果我们没有对 docker 文件进行更改，则在运行`build`命令时不会重建映像。它将从缓存中被重用。

### 2.2.未使用的 Docker 图像

**未使用的图像是没有与其相关联的运行或停止容器的图像。**

未使用的图像示例有:

*   从注册表中提取但尚未在任何容器中使用的图像
*   其容器已被移除的任何图像
*   标记有旧版本且不再使用的图像
*   所有悬挂图像

正如我们所看到的，未使用的图像不一定是悬空的。我们可能会在将来使用这些图像，并且我们可能希望保留它们。然而，保存大量未使用的图像会导致空间问题。

## 3.删除不必要的图像

我们研究了为什么在 Docker 中悬挂和未使用的图像很常见的几个原因。现在，让我们来看看几种去除它们的方法。

### 3.1.按 ID 或名称移除图像

如果我们知道图像 ID，我们可以使用`docker rmi`命令删除图像。

```java
docker rmi 7ed6e7202eca 
```

该命令将删除 ID 为`7ed6e7202eca`的图像(悬空图像)。让我们重新检查图像:

```java
docker images
REPOSITORY   TAG       IMAGE ID       CREATED          SIZE
my-image     latest    da6e74196f66   18 minutes ago   72.8MB
ubuntu       latest    825d55fb6340   6 days ago       72.8MB 
```

或者，如果我们想要删除一个特定的未使用的图像，我们可以使用带有图像名称和标签的`docker rmi`命令:

```java
docker rmi my-image:latest 
```

### 3.2.Docker 图像修剪

如果我们不想找到悬空的图像并逐个删除它们，我们可以使用 [`docker image prune`](https://web.archive.org/web/20220919062318/https://docs.docker.com/engine/reference/commandline/image_prune/) 命令。此命令移除所有悬挂的图像。

如果我们还想删除未使用的图像，我们可以使用`-a`标志。

让我们运行下面的命令:

```java
docker image prune -a
WARNING! This will remove all images without at least one container associated to them.
Are you sure you want to continue? [y/N] y 
```

该命令将返回已删除的映像 id 列表和已释放的空间。

如果我们再次列出图像，我们会发现没有图像留下，因为我们没有运行任何容器。

### 3.3.Docker 系统修剪

找到并移除所有未使用的对象可能很繁琐。为了方便起见，我们可以使用 [`docker system prune`](https://web.archive.org/web/20220919062318/https://docs.docker.com/engine/reference/commandline/system_prune/) 命令。

默认情况下，此命令将删除以下对象:

*   停止的容器–处于停止状态的容器
*   至少一个容器未使用的网络
*   悬空图像
*   悬空构建缓存–支持悬空图像的构建缓存

此外，我们可以添加`-a`标志来删除所有未使用的容器、图像、网络和整个构建缓存。当我们想要释放大量空间时，这很有用。

让我们来看一个例子:

```java
docker system prune -a
WARNING! This will remove:
  - all stopped containers
  - all networks not used by at least one container
  - all images without at least one container associated to them
  - all build cache

Are you sure you want to continue? [y/N]
```

## 4.结论

在本教程中，我们看了为什么悬空和未使用的图像在 Docker 中很常见。我们还研究了使用`rmi,` `image prune,` 和`system prune`命令删除它们的方法。