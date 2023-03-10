# 停靠站停止和停靠站终止命令之间的区别

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/ops/docker-stop-vs-kill>

## 1.概观

Docker 是一个 os 级的软件框架，用于在服务器和云中创建、管理和运行容器。Docker 通过两种不同的方式提供了停止容器的支持。

在本教程中，我们将学习使用 [`docker stop`](https://web.archive.org/web/20221128034952/https://docs.docker.com/engine/reference/commandline/stop/) 和 [`docker kill`](https://web.archive.org/web/20221128034952/https://docs.docker.com/engine/reference/commandline/kill/) 命令来停止和终止容器。

我们将使用不同的 docker 命令来停止和移除容器。

## 2.理解`docker stop`和`docker kill`命令

启动和停止容器不同于启动和停止正常进程。为了终止一个容器，Docker 提供了`docker stop`和`docker kill`命令。`docker kill`和`docker stop`命令看起来相似，但是它们的内部执行是不同的。

**`docker stop`命令发出 SIGTERM 信号，而`docker kill`命令发送 SIGKILL 信号。**执行 [SIGTERM 和 SIGKILL](/web/20221128034952/https://www.baeldung.com/linux/sigint-and-other-termination-signals) 。是不同的。与 SIGKILL 不同，SIGTERM 优雅地终止一个进程，而不是立即终止它。可以处理、忽略或阻止 SIGTERM 信号，但不能阻止或处理 SIGKILL 信号。SIGTERM 允许子进程或父进程向其他进程发送信息。

使用 SIGKILL，我们可能会创建僵尸进程，因为被杀死的子进程不会通知它的父进程它收到了杀死信号。容器需要一些时间来完全关闭。

## 3.执行`docker stop`和`docker kill`命令

在我们继续理解`docker stop`和`docker kill`命令之前，让我们首先运行一个示例 Postgres Docker 容器:

```java
$ docker run -itd -e POSTGRES_USER=baeldung -e POSTGRES_PASSWORD=baeldung
  -p 5432:5432 -v /data:/var/lib/postgresql/data --name postgresql-baedlung postgres
Unable to find image 'postgres:latest' locally
latest: Pulling from library/postgres
214ca5fb9032: Pull complete 
...
95df4ec75c64: Pull complete 
Digest: sha256:2c954f8c5d03da58f8b82645b783b56c1135df17e650b186b296fa1bb71f9cfd
Status: Downloaded newer image for postgres:latest
0aece936b317984b5c741128ac88a891ffc298d48603cf23514b7baf9eeb981a
```

让我们来看看集装箱的细节:

```java
$ docker ps
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS                    NAMES
be2848539d76        postgres            "docker-entrypoint.s…"   4 seconds ago       Up 2 seconds        0.0.0.0:5432->5432/tcp   postgresql-baedlung
```

`docker ps`命令列出了主机上所有正在运行的进程。

### 3.1.停靠站停止命令

**`docker stop`命令优雅地停止容器，并提供一条安全的出路。如果一个`docker stop`命令未能在指定的超时时间内终止一个进程，Docker 会立即隐式发布一个 kill 命令。在 Docker 中，有两种方法可以使用`docker stop`命令来停止一个进程。我们可以使用`containerId`或容器名来停止一个容器。**

让我们演示如何使用容器名来停止容器:

```java
$ docker stop postgresql-baeldung
```

让我们举例说明如何使用`containerId`来停止容器:

```java
$ docker stop be2848539d76
```

有趣的是，我们也可以使用 containerId 的前缀来启动或停止容器。这里，我们只需要确保没有其他容器以“be”开始运行`containerId`:

```java
$ docker stop be
```

默认情况下，`docker stop`命令会等待 10 秒钟来终止进程。但是我们可以使用`-t`选项来配置等待时间:

```java
$ docker stop -t 60 be2848539d76
```

这里，容器将等待 60 秒，然后强制移除容器。我们也可以使用`docker container stop`命令停止容器:

```java
$ docker container stop -t 60 be2848539d76
```

这两个命令的工作方式完全相同。**`docker container stop`命令在 Docker 的新版本中已被弃用。**

### 3.2.码头工人取消命令

**`docker kill`命令突然终止进入点过程。`docker kill`命令导致不安全退出。**在某些情况下，docker 容器与主机上装载的卷一起运行。如果主进程停止时内存中仍有挂起的更改，这可能会导致文件系统损坏。

让我们来看看杀死一个容器的命令:

```java
$ docker kill be2848539d76
```

类似地，要杀死一个容器，我们也可以使用`docker container kill`命令:

```java
$ docker container kill be2848539d76
```

`docker container kill`命令的工作方式类似于`docker kill`命令。

## 4.停止容器的附加命令

`docker kill`和`docker stop`命令都停止集装箱。停止容器的另一种方法就是移除它。我们可以使用`docker rm`命令来移除一个容器。这将立即从本地存储中移除容器:

```java
$ docker rm be2848539d76
```

当我们运行`docker rm`命令时，容器从`docker ps -a`列表中删除。在使用`docker stop`命令时，我们可以保留容器以供重用。理想情况下，我们可以将容器置于两种状态。容器可以处于停止或暂停状态。**如果一个容器被停止，它所有被分配的资源都被释放，而一个被暂停的容器不释放内存，但是 CPU 被释放。在这种情况下，进程会暂停。**

让我们看看暂停容器的命令:

```java
$ docker pause be2848539d76
```

值得注意的是，即使在容器停止后，我们也可以检查它的细节。为了找到关于容器的更多信息，我们可以使用`docker inspect`命令。`docker inspect`命令显示集装箱状态下集装箱的退出代码。当使用`docker stop` 命令停止集装箱时，退出代码为 0。类似地， `docker kill`命令将容器状态显示为非零退出代码。

## 5.结论

在本教程中，我们讨论了执行`docker stop`和`docker kill`命令的区别。首先，我们讨论了使用不同的命令停止容器。接下来，我们讨论了在 docker stop 和 docker kill 命令中 SIGTERM 和 SIGKILL 的实现。

我们还研究了这两个命令的不同选项。后来，我们探索了`docker container stop,`和`docker container kill`命令。

**简而言之，我们学习了各种阻止和杀死码头集装箱的方法。**