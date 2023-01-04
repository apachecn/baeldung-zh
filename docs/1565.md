# Docker 容器中的 Root 用户和密码

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/ops/root-user-password-docker-container>

## 1.概观

[Docker](/web/20221026042423/https://www.baeldung.com/ops/docker-guide) 的工作原理是将应用程序及其所需的所有依赖项打包成轻量级的 [容器](https://web.archive.org/web/20221026042423/https://www.docker.com/resources/what-container/) 。 除了在测试期间部署在本地集群上，我们还可以在生产环境中部署这些轻量级容器。

在本教程中，我们将研究如何使用不同的用户来执行 Docker 容器中的命令。

首先，我们将学习使用 root 用户来访问 Docker 容器，以获得一些额外的特权。我们还将讨论为根用户和非根用户设置密码，以保护容器免受易受攻击的来源的攻击。

## 2.设置 Docker 容器

在我们继续之前，让我们首先创建一个 docker 文件来添加一个用户 *john* :

```
FROM ubuntu:16.04
RUN apt-get update 
RUN useradd -m john
USER john
CMD /bin/bash
```

这里，我们使用“`ubuntu:16.04`”作为基础图像。让我们使用 [`docker build`](https://web.archive.org/web/20221026042423/https://docs.docker.com/engine/reference/commandline/build/) 命令来构建图像:

```
$ docker build -t baeldung .
Sending build context to Docker daemon  2.048kB
Step 1/5 : FROM ubuntu:16.04
16.04: Pulling from library/ubuntu
58690f9b18fc: Pull complete 
...
Step 5/5 : CMD /bin/bash
 ---> Running in d04af94585e2
Removing intermediate container d04af94585e2
 ---> 312faa93c781
Successfully built 312faa93c781
Successfully tagged baeldung:latest
```

我们现在将使用`baeldung`图像运行一个 Docker 容器:

```
$ docker run -id --name baeldung baeldung
34dbc77279a2a6244b0e4ee87890d79e814128391c6a4387d2e2fd10fa6e8f20
```

让我们使用 [`docker ps`](https://web.archive.org/web/20221026042423/https://docs.docker.com/engine/reference/commandline/ps/) 命令来验证容器是否按预期运行:

```
$ docker ps
CONTAINER ID        IMAGE               COMMAND                  CREATED              STATUS              PORTS               NAMES
34dbc77279a2        baeldung            "/bin/sh -c /bin/bash"   About a minute ago   Up About a minute                       baeldung
```

在这里，我们可以看到 Docker 容器正在运行，没有任何问题。

## 3.访问 Docker 容器

**Docker 容器被设计为以 root 用户身份访问，以执行非 root 用户无法执行的命令。**我们可以使用`docker exec`在运行容器中运行命令。我们将使用`docker exec`命令的`-i`和`-t`选项来获得带有 TTY 终端访问的交互 shell。

### 3.1.使用非超级用户

停靠容器"`baeldung`"已启动并正在运行。我们现在将使用`docker exec`命令来访问它:

```
$ docker exec -it baeldung bash
```

请仔细注意我们之前创建的 Dockerfile 文件。我们添加了一个新用户`john` ，它被设置为所有使用 Docker 映像运行的容器的默认用户。让我们使用 [`whoami`](/web/20221026042423/https://www.baeldung.com/linux/tag/whoami) 命令:来验证这一点

```
$ whoami
john 
```

现在，如果我们试图将任何包安装到容器中，我们将得到以下错误消息:

```
$ apt-get update
Reading package lists... Done
W: chmod 0700 of directory /var/lib/apt/lists/partial failed - SetupAPTPartialDirectory (1: Operation not permitted)
E: Could not open lock file /var/lib/apt/lists/lock - open (13: Permission denied)
E: Unable to lock directory /var/lib/apt/lists/
```

在这种情况下，非 root 用户无法访问`lock`文件。通常，该用户对容器的访问是受限的。

现在让我们退出容器，使用 root 用户重新登录。

### 3.2.使用根用户

为了在 Docker 容器中使用 root 用户来执行，我们将使用—`u`选项:

```
$ docker exec -it -u 0 baeldung bash
```

使用`docker exec`命令的 `-u”` 选项，我们定义了根用户的`id`。我们也可以在这个命令中使用用户名:

```
$ docker exec -it -u root baeldung bash
```

为了查看当前用户的详细信息，我们将运行 *whoami* 命令:

```
$ whoami
root
```

这次，我们以 root 用户的身份进入了容器。现在，我们可以对容器执行任何操作:

```
$ apt-get update
Hit:1 http://security.ubuntu.com/ubuntu xenial-security InRelease
Hit:2 http://archive.ubuntu.com/ubuntu xenial InRelease
Hit:3 http://archive.ubuntu.com/ubuntu xenial-updates InRelease
Hit:4 http://archive.ubuntu.com/ubuntu xenial-backports InRelease
Reading package lists... Done 
```

从上面的输出可以看出，更新命令成功，root 用户可以访问`lock`文件。有了 root 用户的全部权限，我们可以毫无问题地更改任何文件。

作为替代，我们也可以以 root 用户身份访问 Docker 容器。在本例中，我们将使用[*n 输入*](https://web.archive.org/web/20221026042423/https://manpages.ubuntu.com/manpages/xenial/man1/nsenter.1.html) 命令来访问 Docker 容器。 要使用*n 输入* 命令，我们必须知道运行容器的`PID`。

让我们看看获取容器 PID 的命令:

```
$ docker inspect --format {{.State.Pid}} baeldung
6491
```

一旦我们有了`PID`，我们将以如下方式将这个`PID`与*n 输入* 命令一起使用:

```
$ nsenter --target 6491 --mount --uts --ipc --net --pid
```

这允许我们作为根用户访问 Docker 容器，并运行任何命令来访问任何文件。

## 4.在容器内使用`sudo`命令

Docker 容器通常以 root 用户作为默认用户运行。 为了用不同的特权共享资源，我们可能需要在 Docker 容器中创建额外的用户。

在这里，我们将创建一个 docker 文件并添加一个新用户。重要的是，我们还将在构建映像时在 Docker 容器中安装 `[sudo](/web/20221026042423/https://www.baeldung.com/linux/sudo-command)` 包。当这个用户需要额外的特权时，可以使用 s *udo* 命令来访问它们。

我们来看看 Dockerfile:

```
FROM ubuntu:16.04
RUN apt-get update && apt-get -y install sudo
RUN useradd -m john && echo "john:john" | chpasswd && adduser john sudo
USER john
CMD /bin/bash 
```

此 Dockerfile 使用映像“*Ubuntu:16.04*”作为基础映像，安装 *sudo* 包，并创建一个新用户，“ *【约翰】* ”。我们使用 [`chpasswd`](https://web.archive.org/web/20221026042423/https://linux.die.net/man/8/chpasswd) 命令给*约翰*用户添加一个密码。在那之后的 ，我们用它作为默认用户。

让我们运行命令来构建映像:

```
$ docker build -t baeldung . 
```

上面的命令将创建`baeldung`图像。现在，让我们使用 `baeldung` 图像:来运行容器

```
$ docker run -id --name baeldung baeldung
b0f83a7e8b49ddf043c80792f21d5c483c0c5ab56c700815a83b0a40e5292754 
```

容器的默认用户是 `john` ，所以我们将使用它来访问容器:

```
$ docker exec -it baeldung bash
To run a command as administrator (user "root"), use "sudo <command>".
See "man sudo_root" for details. 
```

让我们运行 *whoami* 命令，找出登录用户的用户名:

```
$ whoami
john
```

这表明我们以非 root 用户身份登录。如果我们运行 [`apt-get update`](/web/20221026042423/https://www.baeldung.com/linux/yum-and-apt) 命令，我们将会遇到与我们在 3.2 节中遇到的权限相关的问题相同的问题。

这一次，我们将使用 *sudo* 命令为非根用户`john`获取特权:

```
$ sudo apt-get update
[sudo] password for john: 
Get:1 http://security.ubuntu.com/ubuntu xenial-security InRelease [99.8 kB]
Hit:2 http://archive.ubuntu.com/ubuntu xenial InRelease 
Get:3 http://archive.ubuntu.com/ubuntu xenial-updates InRelease [99.8 kB]
Get:4 http://archive.ubuntu.com/ubuntu xenial-backports InRelease [97.4 kB]
Fetched 297 kB in 1s (178 kB/s)
Reading package lists... Done
```

通过使用这种方法，我们可以使用 *sudo* 命令在非根帐户中运行任何命令。

## 5.结论

在这篇文章中，我们演示了如何在 Docker 容器中用不同的用户运行命令。 首先，我们讨论了根用户和非根用户在运行 Docker 容器中的角色。然后，我们 学习了如何作为根用户访问 Docker 容器以获得额外的特权。

理想情况下，我们不应该允许 root 用户访问 Docker 容器。这增加了更多的安全问题。相反，我们应该创建一个单独的用户来访问容器。这是容器世界中的标准安全步骤。