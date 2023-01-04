# 码头集装箱的状态

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/ops/docker-container-states>

## 1.概观

一个 [Docker](/web/20221210235938/https://www.baeldung.com/ops/docker-guide) 容器是一个 Docker 映像的实例，在其中运行一些进程。当这个过程的状态改变时，容器的行为也会受到影响。因此，容器在其整个生命周期中可以处于不同的状态。

在本教程中，我们将了解 Docker 容器的所有可能状态。

让我们首先看看如何找到 Docker 容器的状态，然后我们将经历容器的不同阶段。

## 2.查找 Docker 容器的当前状态

在我们深入研究 Docker 容器的不同状态之前，让我们先看看如何找到任何 Docker 容器的状态。

默认情况下，`docker ps`命令显示所有 Docker 容器的当前状态:

```java
$ docker ps -a
CONTAINER ID   IMAGE      COMMAND       CREATED              STATUS                          PORTS     NAMES
8f0b524f2d32   centos:7   "/bin/bash"   46 seconds ago       Created                                   strange_beaver
e6d798254d45   centos:7   "/bin/bash"   About a minute ago   Exited (0) About a minute ago             wizardly_cohen
```

输出显示机器上的所有容器及其`STATUS`(第五列)和一系列其他细节。

我们还可以使用`docker inspect`命令来获取单个容器的状态:

```java
$ docker inspect -f '{{.State.Status}}' mycontainer
running
```

这里，`mycontainer`是我们希望找到其当前状态的容器名。我们也可以用 Docker 容器 id 替换它。

## 3.Docker 容器的可能状态

在任何特定的实例中，Docker 容器可以有 6 种可能的状态。现在，让我们深入了解这些状态:

### 3.1.创造

**Docker 将`created`状态分配给自创建以来从未启动过的容器。因此，在这种状态下，容器不使用 CPU 或内存。**

使用`docker create`命令创建的 Docker 容器将状态显示为 c `reated:`

```java
$ docker create --name mycontainer httpd
dd109e4be16219f1a6b9fc1cbfb050c1ae035d6a2c301ea0e93eb7d5252b8d2e
$ docker inspect -f '{{.State.Status}}' mycontainer
created
```

这里，我们使用`httpd` 的[官方 Docker 映像创建了一个 Docker 容器`mycontainer`。由于我们使用了`docker create`命令来启动容器，所以状态显示为`created. `](https://web.archive.org/web/20221210235938/https://hub.docker.com/_/httpd)

当我们需要为一些大任务做准备时，这样的容器很有用。在这种情况下，我们创建容器，这样当我们启动它时它就准备好了。

### 3.2.运转

当我们使用`e docker start`命令启动一个具有`created`状态的容器时，它会达到`running`状态。

**这个状态表示进程正在容器内部的隔离环境中运行。**

```java
$ docker create --name mycontainer httpd
8d60cb560afc1397d6732672b2b4af16a08bf6289a5a0b6b5125c5635e8ee749
$ docker inspect -f '{{.State.Status}}' mycontainer
created
$ docker start mycontainer
mycontainer
$ docker inspect -f '{{.State.Status}}' mycontainer
running
```

在上面的例子中，我们首先使用官方的`httpd` Docker 映像创建了一个 Docker 容器。然后，当我们启动`mycontainer,` `httpd`服务器进程和所有其他相关进程开始运行时，容器的状态是`created.` 。因此，同一集装箱的状态现在已更改为`running.`

使用`docker run`命令运行的容器也获得了相同的状态:

```java
$ docker run -itd --name mycontainer httpd
685efd4c1c4a658fd8a0d6ca66ee3cf88ab75a127b9b439026e91211d09712c7
$ docker inspect -f '{{.State.Status}}' mycontainer
running
```

在这种状态下，容器的 CPU 和内存消耗不会受到影响。

### 3.3.重新启动

简单地说，这种状态表示容器正在重启过程中。

Docker 支持四种类型的[重启策略](https://web.archive.org/web/20221210235938/https://docs.docker.com/config/containers/start-containers-automatically/)，即-`no`、`on-failure`、`always`、`unless-stopped.`重启策略决定容器退出时的行为。

默认情况下，重启策略设置为`no,` ，这意味着容器退出后不会自动启动。

让我们将重启策略更新为`always `，并使用以下示例验证 Docker 容器的状态:

```java
$ docker run -itd --restart=always --name mycontainer centos:7 sleep 5
f7d0e8becdac1ebf7aae25be2d02409f0f211fcc191aea000041d158f89be6f6
```

上述命令将运行`mycontainer`并执行`sleep 5`命令，然后退出。但是由于我们已经更新了这个容器的重启策略，它会在退出后自动重启容器。

5 秒钟后，容器的状态将为`restarting`:

```java
$ docker inspect -f '{{.State.Status}}' mycontainer
restarting
```

### 3.4.退出

当容器内的进程终止时，就达到了这种状态。 **在这种状态下，容器不消耗 CPU 和内存。**

运行中的容器退出可能有多种原因。让我们来看看其中的几个:

*   容器内部的进程已经完成，所以它退出了。
*   容器内的进程在运行时遇到异常。
*   使用`docker stop`命令有意停止容器。
*   没有为运行 bash 的容器设置交互终端。

```java
$ docker run -itd --name mycontainer centos:7 sleep 10
596a10ddb635b83ad6bb9daffb12c1e2f230280fe26be18559c53c1dca6c755f 
```

在这里，我们已经启动了一个 centos 容器，`mycontainer, `并通过了命令`sleep 10\.` ，它将在 10 秒钟的睡眠后退出容器。我们可以通过在 10 秒钟后运行以下命令来验证这一点:

```java
$ docker inspect -f '{{.State.Status}}' mycontainer
exited
```

使用`docker exec`命令无法访问处于`exited`状态的容器。然而，我们可以使用`docker start`或`docker restart `启动容器，然后访问它。

```java
$ docker start mycontainer
```

### 3.5.暂停

**`Paused`是无限期挂起所有进程的 Docker 容器的状态。**

可以使用`docker pause`命令暂停 Docker 容器。

```java
$ docker run -itd --name mycontainer centos:7 sleep 1000
1a44702cea17eec42195b057588cf72825174db311a35374e250d3d1da9d70c5 
```

在上面的例子中，我们使用 centos Docker 映像启动了一个 Docker 容器，并运行了命令`sleep 1000`。这将在休眠 1000 秒后退出容器。

现在让我们暂停这个容器几秒钟，比如 100 秒钟:

```java
$ docker pause mycontainer
mycontainer
$ docker inspect -f '{{.State.Status}}' mycontainer
paused
```

**一个`paused`容器消耗的内存与运行容器时消耗的内存相同，但是 CPU 被完全释放。**让我们使用`docker stat`命令来验证这一点:

```java
$ docker stats --no-stream
CONTAINER ID   NAME          CPU %     MEM USAGE / LIMIT    MEM %     NET I/O       BLOCK I/O   PIDS
1a44702cea17   mycontainer   0.00%     1.09MiB / 7.281GiB   0.01%     1.37kB / 0B   0B / 0B     1
```

请注意，CPU 使用率为 0%，但内存使用率不为零。

我们可以使用`docker unpause`命令来恢复容器:

```java
$ docker unpause mycontainer
mycontainer
```

**在取消暂停容器时，它将从我们暂停它的同一点恢复。**在上面的例子中，我们在休眠 100 秒后暂停了容器。所以当我们打开容器时，它将从 100 开始恢复睡眠。因此，容器将在 900 秒睡眠后停止。(总睡眠时间设置为 1000)。

现在的问题是什么时候暂停 Docker 容器？考虑 Docker 容器正在执行一些 CPU 密集型任务的情况。同时，我们希望以高优先级运行另一个 CPU 密集型容器。当然，我们可以同时运行这两个容器，但是由于缺乏资源，这会降低执行速度。

在这种情况下，我们可以暂停一个低优先级的容器一段时间，让另一个容器使用整个 CPU。一旦完成，我们就可以取消第一个容器的执行。

### 3.6.死亡的

Docker 容器的`dead`状态意味着容器不起作用。当我们试图移除容器时，会达到这种状态，但是无法移除，因为某些资源仍被外部进程使用。因此，容器被移动到`dead`状态。

**处于`dead`状态的容器不能重启。它们只能被移除。**

由于处于`dead`状态的容器被部分移除，所以它**不消耗任何内存或 CPU。**

## 4.结论

在本教程中，我们经历了 Docker 容器的不同阶段。

首先，我们研究了几种发现 Docker 容器状态的方法。后来，我们学习了每个状态的重要性，以及如何使用不同的 Docker 命令来实现这些状态。