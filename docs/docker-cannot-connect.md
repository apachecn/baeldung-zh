# 无法连接到 Docker 守护程序

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/ops/docker-cannot-connect>

## 1.概观

Docker 帮助将应用程序及其所有依赖项打包成一个轻量级的实体，称为容器。我们可以在任何物理机、虚拟机甚至云中部署 Docker 容器。

在不同的环境中使用 Docker 服务时，我们可能会面临各种问题。

在本教程中，我们将了解 Docker 守护进程的连接问题。这是初学者很容易遇到的错误。我们还将研究导致该问题的原因以及解决该问题的方法。

## 2.理解问题

考虑这样一种情况，我们试图在 [Linux](/web/20220915205204/https://www.baeldung.com/linux/) 上运行一个不在机器上的命令。我们得到一条`command not found`错误消息作为回报。

出现此问题的原因可能是计算机上没有实际安装该命令，或者安装了该命令但配置不正确。

我们先来了解一下 Docker 守护进程(`dockerd`)。这是一个管理所有 Docker 对象的程序，包括图像、容器、卷等等。

另一个实体，Docker 客户机，通过 Docker 守护进程帮助将命令从用户传递到 Docker 服务。

在某些情况下，Docker 客户端无法连接到 Docker 守护程序。在这种情况下，Docker 抛出错误`Cannot connect to Docker daemon`。

Docker 客户端无法连接到 Docker 守护程序可能有多种原因。现在，让我们深入探讨一下问题的根本原因和不同的解决方案。

## 3.由于不活动的码头服务

**该错误最常见的原因是当我们试图访问 Docker 服务，但它没有启动**:

```java
$ docker ps
Cannot connect to the Docker daemon at unix:///var/run/docker.sock.
  Is the docker daemon running?
```

首先，我们将检查 Docker 服务的状态以及它是否正在运行:

```java
$ systemctl status docker
 docker.service - Docker Application Container Engine
   Loaded: loaded (/usr/lib/systemd/system/docker.service; disabled; vendor preset: disabled)
   Active: inactive (dead)
   Docs: https://docs.docker.com
```

这里的输出清楚地显示了 Docker 服务是不活动的。

我们现在将启动 Docker 服务。这将在大多数情况下解决问题。

### 3.1.使用服务启动 Docker

通常，当我们使用[包管理器](/web/20220915205204/https://www.baeldung.com/linux/yum-and-apt)安装 Docker 时，它会创建一个 Docker 服务。这使得管理 Docker 变得容易。

现在让我们使用`systemctl`服务命令启动 Docker:

```java
$ systemctl start docker
```

我们可以使用以下命令检查 Docker 的状态:

```java
$ systemctl status docker
 docker.service - Docker Application Container Engine
   Loaded: loaded (/usr/lib/systemd/system/docker.service; disabled; vendor preset: disabled)
   Active: active (running) since Thu 2022-02-17 19:14:51 UTC; 1min 38s ago
     Docs: https://docs.docker.com
 Main PID: 1831 (dockerd)
    Tasks: 8
   Memory: 126.5M
   CGroup: /system.slice/docker.service
           └─1831 /usr/bin/dockerd -H fd:// --containerd=/run/containerd/containerd.sock
```

该命令将 Docker 服务的当前状态显示为活动(正在运行)。

### 3.2.手动启动 Docker 守护进程

或者，我们也可以在没有服务的情况下启动 Docker。

我们需要做的就是在后台运行`dockerd`命令:

```java
$ sudo dockerd
INFO[2022-02-18T05:19:50.048886666Z] Starting up                                  
INFO[2022-02-18T05:19:50.050883459Z] libcontainerd: started new containerd process  pid=2331
INFO[2022-02-18T05:19:50.050943756Z] parsed scheme: "unix"                         module=grpc
```

我们需要确保使用 sudo 特权运行`dockerd`。

## 4.由于权限不足

当我们使用包管理器安装 Docker 时，默认情况下它会创建一个`docker`用户和组。为了访问 Docker，我们需要将当前用户添加到`docker`组。

如果我们试图从`docker`组之外的用户访问 Docker 服务，我们会得到以下错误:

```java
$ docker ps
Got permission denied while trying to connect to the Docker daemon socket
  at unix:///var/run/docker.sock: Get http://%2Fvar%2Frun%2Fdocker.sock/v1.40/containers/json:
  dial unix /var/run/docker.sock: connect: permission denied
```

为了解决这个问题，我们可以做两件事。我们可以将用户添加到`docker`组，或者更新 Docker 套接字文件的权限。

现在，让我们通过示例深入探讨这两种解决方案。

### 4.1.更新用户权限

用户特权是 Linux 中的一个重要概念。它们决定不同用户对资源的可访问性。

在 Linux 上安装 Docker 时，会创建一个新的`docker`组，所有与 Docker 服务相关的包都链接到这个`docker`组。

如果我们在机器上找不到默认的`docker`组，我们可以手动创建它:

```java
$ sudo groupadd docker
```

上述命令将创建一个`docker`组。

现在我们将当前用户添加到`docker`组中:

```java
$ sudo usermod -aG docker docker-test
```

这里的`-a`选项将把用户`docker-test`添加到`docker `组。`-G`选项用于提及组名。

最后，我们将重新启动 Docker 服务以使更改生效:

```java
$ sudo service docker restart
```

### 4.2.更新 Docker 套接字文件权限

我们还可以通过更改` /var/run/docker.sock `文件的所有者来解决这个问题:

```java
$ sudo chown docker-test /var/run/docker.sock
```

注意，我们使用 sudo 权限运行这个命令。否则，该文件的权限将不会更新。

## 5.结论

在本文中，我们了解了经常遇到的 Docker 守护程序连接问题。

当 Docker 服务未正确启动或我们没有访问 Docker 服务的适当用户权限时，会出现此问题。

我们研究了各种方法来解决这个问题，通过将用户添加到`docker`组并更改`sock`文件的权限。