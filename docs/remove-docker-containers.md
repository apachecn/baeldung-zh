# 移除码头集装箱

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/ops/remove-docker-containers>

## 1.概观

在上一篇文章中，我们学习了如何移除 Docker 图片。然而，一个 **Docker 镜像只有在没有 Docker 容器使用该镜像时才能被删除。**因此，要删除一个 Docker 映像，必须删除所有运行该映像的 Docker 容器。

在本教程中，我们将学习使用不同的方法移除 Docker 容器。

## 2.为什么要移除 Docker 容器？

当 Docker 容器完成它的执行时，它达到`exited`状态。**这样的容器不消耗任何 CPU 或内存，但是它们仍然使用机器的磁盘空间。**此外，停止的容器不会自动移除，除非我们在运行 Docker 容器时使用`–rm` 标志。

因此，随着越来越多的容器进入`exited`状态，它们消耗的总磁盘空间会增加。因此，我们可能无法启动新的容器，或者 Docker 守护进程将停止响应。

**为了避免这种情况，建议使用`–rm`标志运行 Docker 容器，或者定期手动移除 Docker 容器。**

现在让我们学习如何移除 Docker 容器。

## 3.移除单个码头集装箱

首先，我们将以非交互模式启动 CentOS Docker 容器。通过这样做，容器将在我们运行容器后立即停止:

```java
$ docker run --name mycontainer centos:7
$ docker ps -a
CONTAINER ID   IMAGE          COMMAND                  CREATED         STATUS                     PORTS              NAMES
418c28b4b04e   centos:7      "/bin/bash"              6 seconds ago   Exited (0) 5 seconds ago                       mycontainer
```

现在让我们使用`docker rm`命令删除 Docker 容器`mycontainer`:

```java
$ docker rm mycontainer
mycontainer
$ docker ps -a
CONTAINER ID   IMAGE     COMMAND   CREATED   STATUS    PORTS     NAMES 
```

我们还可以使用 Docker 容器 id 而不是 Docker 容器名称来删除 Docker 容器，方法是使用`docker rm`命令:

```java
$ docker rm 418c28b4b04e
```

## 4.移除多个 Docker 容器

我们还可以使用`docker rm`命令删除多个 Docker 容器。`docker rm`命令接受一个以空格分隔的 Docker 容器名称或 id 列表，并删除它们:

```java
$ docker ps -a
CONTAINER ID   IMAGE      COMMAND       CREATED          STATUS                      PORTS     NAMES
23c70ec6e724   centos:7   "/bin/bash"   6 seconds ago    Exited (0) 5 seconds ago              mycontainer3
fd0886458666   centos:7   "/bin/bash"   10 seconds ago   Exited (0) 9 seconds ago              mycontainer2
c223ec695e2d   centos:7   "/bin/bash"   14 seconds ago   Exited (0) 12 seconds ago             mycontainer1
$ docker rm c223ec695e2d mycontainer2 23c70ec6e724
c223ec695e2d
mycontainer2
23c70ec6e724
```

在上面的例子中，有三个 Docker 容器处于`exited`状态，我们使用`docker rm`命令删除了它们。

**我们可以在任何 Docker 命令中互换使用 Docker 容器名称和 id。**注意，我们对`mycontainer1`和`mycontainer3`使用了 Docker 容器 id，对`mycontainer2. `使用了容器名

## 5.移除所有码头集装箱

考虑一个场景，机器上有太多停止的码头集装箱，现在我们希望将它们全部移除。当然，我们可以使用上述方法，将所有容器 id 传递给`docker rm`命令。但是，让我们研究一个更优化、更简单的命令来删除所有 Docker 容器:

```java
$ docker ps -a
CONTAINER ID   IMAGE      COMMAND       CREATED          STATUS                      PORTS     NAMES
b5c45fa5764f   centos:7   "/bin/bash"   4 seconds ago    Exited (0) 3 seconds ago              mycontainer1
ed806b1743cd   centos:7   "/bin/bash"   9 seconds ago    Exited (0) 7 seconds ago              mycontainer2
2e00a052eb12   centos:7   "/bin/bash"   13 seconds ago   Exited (0) 12 seconds ago             mycontainer3
$ docker rm $(docker ps -qa)
b5c45fa5764f
ed806b1743cd
2e00a052eb12
```

命令`docker ps -qa `返回机器上所有容器的数字标识。所有这些 id 然后被传递给`docker rm`命令，该命令将迭代地删除 Docker 容器。

我们也可以使用 [`docker container prune`](https://web.archive.org/web/20220614151341/https://docs.docker.com/engine/reference/commandline/container_prune/) 命令来删除所有停止的容器:

```java
$ docker container prune -f
```

这里，我们使用`-f`标志来避免确认提示。

## 6.强行移除正在运行的 Docker 容器

我们在上面的例子中讨论的所有命令只有在 Docker 容器停止时才起作用。如果我们试图删除一个正在运行的容器，而没有先停止它，我们将得到一个类似如下的错误消息:

```java
$ docker ps
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS               NAMES
f84692b27b0a        centos:7            "/bin/bash"         59 seconds ago      Up 58 seconds                           mycontainer
$ docker rm mycontainer
Error response from daemon:
  You cannot remove a running container f84692b27b0a18266f34b35c90dad655faa10bb0d9c85d73b22079dde506b8b5.
  Stop the container before attempting removal or force remove
```

删除正在运行的 Docker 容器的一种方法是首先使用`docker stop`命令停止该容器，然后使用`docker rm`命令删除它。

另一种方法是使用`-f`选项强制移除此类容器:

```java
$ docker rm -f mycontainer
mycontainer
```

**我们可以使用`-f`选项删除单个 Docker 容器、多个 Docker 容器或所有 Docker 容器。**

## 7.结论

在本教程中，我们了解了为什么需要删除 Docker 容器。首先，我们学习了从 Linux 机器上移除容器。此外，我们使用`docker rm`和`docker prune`命令批量移除 Docker 容器。

最后，我们看了如何强行移除 Docker 容器，它们处于`running`状态。