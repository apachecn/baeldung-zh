# 进入 Docker 容器的外壳

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/ops/docker-container-shell>

## 1.概观

我们知道 Docker 是一个强大的工具，可以轻松地创建、部署和运行应用程序。

在[图像与容器教程](/web/20221206020718/https://www.baeldung.com/docker-images-vs-containers)中，我们讨论了 Docker 图像是如何使用层构建的。我们还讨论了第一层通常是操作系统。

那么，有没有可能连接到容器的操作系统呢？是的，它是。现在我们要学习如何去做。

## 2.连接到现有容器

如果我们想要连接到一个现有的容器，我们应该让任何容器处于运行状态。为此，我们必须用`docker ps`命令检查系统中的容器状态:

```
$ docker ps -a
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS                  PORTS               NAMES
4b9d83040f4a        hello-world         "/hello"            8 days ago          Exited (0) 8 days ago                       dazzling_perlman
```

因为我们没有运行的容器，所以让我们以 RabbitMQ 容器为例:

```
$ docker run -d rabbitmq:3
```

一旦容器被启动，我们将从对`docker ps`的另一个调用中看到它:

```
$ docker ps
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS                                NAMES
b7a9f5eb6b85        rabbitmq:3          "docker-entrypoint.s…"   25 minutes ago      Up 25 minutes       4369/tcp, 5671-5672/tcp, 25672/tcp   trusting_bose
```

现在，连接到这个容器就像执行以下命令一样简单:

```
$ docker exec -it b7a9f5eb6b85 sh
```

此时，我们在容器中有了一个交互式外壳:

*   [`docker exec`](https://web.archive.org/web/20221206020718/https://docs.docker.com/engine/reference/commandline/exec/) 告诉 Docker 我们要执行一个命令到一个正在运行的容器中
*   `-it`参数意味着它将以交互模式执行——它保持 STIN 打开
*   `b7a9f5eb6b85`是容器 ID
*   是我们想要执行的命令

让我们探索一下我们新创建的容器的操作系统:

```
$ cat /etc/*-release
DISTRIB_ID=Ubuntu
DISTRIB_RELEASE=18.04
DISTRIB_CODENAME=bionic
DISTRIB_DESCRIPTION="Ubuntu 18.04.4 LTS"
NAME="Ubuntu"
VERSION="18.04.4 LTS (Bionic Beaver)"
ID=ubuntu
ID_LIKE=debian
PRETTY_NAME="Ubuntu 18.04.4 LTS"
VERSION_ID="18.04"
HOME_URL="https://www.ubuntu.com/"
SUPPORT_URL="https://help.ubuntu.com/"
BUG_REPORT_URL="https://bugs.launchpad.net/ubuntu/"
PRIVACY_POLICY_URL="https://www.ubuntu.com/legal/terms-and-policies/privacy-policy"
VERSION_CODENAME=bionic
UBUNTU_CODENAME=bionic
```

好像是仿生海狸 Ubuntu。如果我们检查[rabbit MQ docker 文件](https://web.archive.org/web/20221206020718/https://github.com/docker-library/rabbitmq/blob/1bc288f77525425dfb5b58a0d5dbb3834c7dc53c/3.8/ubuntu/Dockerfile)中的`FROM`指令，我们意识到这个映像是使用 Ubuntu 18.04 构建的。

我们可以用`exit`命令或仅仅用`CTRL+d`来断开与容器的连接。

这个例子很简单，因为当我们启动 RabbitMQ 容器时，它会一直运行，直到我们停止它。另一方面，有时我们不得不处理不能存活的容器，例如操作系统容器。让我们看看如何进入它们。

## 3.以交互模式运行容器

如果我们尝试启动一个新的操作系统容器，例如 18.04 版的 Ubuntu，我们会发现它无法保持活动状态:

```
$ docker run ubuntu:18.04
$ docker ps -a
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS                     PORTS                                NAMES
08c26636709f        ubuntu:18.04        "/bin/bash"              10 seconds ago      Exited (0) 7 seconds ago                                        heuristic_dubinsky
b7a9f5eb6b85        rabbitmq:3          "docker-entrypoint.s…"   About an hour ago   Up About an hour           4369/tcp, 5671-5672/tcp, 25672/tcp   trusting_bose
```

RabbitMQ 容器还在运行的时候，Ubuntu one 就停止了。因此，我们不能使用`docker exec`命令连接到这个容器。

避免这种情况的方法是以交互模式运行该容器:

```
$ docker run -it ubuntu:18.04
```

现在我们在容器内部，我们可以检查外壳类型:

```
$ echo $0
/bin/bash
```

实际上，当我们在交互模式下启动容器时，使用`–rm`参数很方便。它将确保在我们退出时移除容器:

```
$ docker run -it --rm ubuntu:18.04
```

## 4.保持集装箱运转

有时我们会遇到一些奇怪的情况，我们需要启动并连接到一个容器，但是交互模式不起作用。

如果我们遇到这些情况中的一种，很可能是因为某些事情是错误的，应该被纠正。

但是，如果我们需要一个快速的解决方法，我们可以在容器中运行`tail`命令:

```
$ docker run -d ubuntu:18.04 tail -f /dev/null
```

使用这个命令，我们在分离/后台模式(`-d`)下启动一个新的容器，并在容器内执行`tail -f /dev/null`命令。结果，这将迫使我们的容器永远运行。

现在我们只需要使用`docker exec`命令以我们之前看到的相同方式进行连接:

```
$ docker exec -it CONTAINER_ID sh
```

请记住，这是一种变通方法，只应在开发环境中使用。

## 5.结论

在本教程中，我们看到了如何连接到一个正在运行的容器的外壳，以及如何以交互方式启动容器。