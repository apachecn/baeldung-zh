# Docker 错误:“无法通过套接字连接到本地 MySQL 服务器”

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/docker-cant-connect-local-mysql>

## 1.概观

在当今的技术时代，组织必须快速发布应用程序来吸引和留住业务。这使得团队以更快的速度和更低的成本构建和配置部署环境。然而，集装箱化技术拯救了轻量级基础设施的建设。

本文将阐述在基于容器的环境中部署和连接 MySQL 服务器的方法。

现在，让我们进入它的本质。

## 2.MySQL 容器部署

首先，让我们看看 MySQL 容器部署中涉及的步骤。基本上，MySQL 遵循客户端-服务器架构模型。这里，服务器是一个包含数据库的容器映像，而客户机用于访问主机上的数据库。

我们将部署工作流分为三个部分。

### 2.1.MySQL 服务器部署

现在，让我们将 MySQL 服务器实例引入 Docker。我们可以简单地基于从 Docker Hub 获取的 MySQL 映像构建一个容器。在选择从 [Docker Hub](https://web.archive.org/web/20221024141849/https://hub.docker.com/_/mysql) 中提取哪个版本的图像时，我们应该考虑两点:

*   官方图像标记——这些是来自 MySQL 开发团队的更安全、更具策划性的图像。
*   最新标签——除非我们对 MySQL 版本有任何保留，否则我们可以使用存储库中可用的最新版本。

让我们使用 [`docker pull`](https://web.archive.org/web/20221024141849/https://docs.docker.com/engine/reference/commandline/pull/) 命令从 Docker Hub 获取官方的 MySQL 镜像:

```
$ docker pull mysql:latest
latest: Pulling from library/mysql
f003217c5aae: Pull complete
…
… output truncated …
…
70f46ebb971a: Pull complete
db6ea71d471d: Waiting
c2920c795b25: Downloading [=================================================> ]  105.6MB/107.8MB
26c3bdf75ff5: Download complete
```

通常，**[图像](/web/20221024141849/https://www.baeldung.com/ops/docker-images-vs-containers)是按照清单文件**中描述的有序形式紧密耦合的不同层。我们的`docker pull`命令将**从 blob 存储中获取图像层，并使用清单文件**自动创建图像:

```
…
… output truncated …
…
4607fa685ac6: Pull complete
Digest: sha256:1c75ba7716c6f73fc106dacedfdcf13f934ea8c161c8b3b3e4618bcd5fbcf195
Status: Downloaded newer image for mysql:latest
docker.io/library/mysql:latest
```

如上所述，捆绑的映像会获得一个哈希代码以供将来参考。

事不宜迟，让我们运行容器。 **[`docker run`](https://web.archive.org/web/20221024141849/https://docs.docker.com/engine/reference/commandline/run/) 命令通常在图像层**之上创建可写容器层。我们需要使用`-name`参数提供容器名称，并使用带有最新标签的 MySQL 图像。此外，我们将通过[环境变量](/web/20221024141849/https://www.baeldung.com/ops/docker-container-environment-variables) MYSQL_ROOT_PASSWORD 来设置 MySQL 服务器密码。在我们的例子中，密码被设置为“baeldung”。

最后，`-d`选项帮助我们将容器作为守护进程运行。输出为将来的容器管理抛出了另一个哈希代码:

```
$ docker run --name bael-mysql-demo -e MYSQL_ROOT_PASSWORD=baeldung -d mysql:latest
fedf880ce2b690f9205c7a37f32d75f669fdb1da2505e485e44cadd0b912bd35
```

我们可以通过 [`ps`](https://web.archive.org/web/20221024141849/https://docs.docker.com/engine/reference/commandline/ps/) 命令看到一台主机中所有[运行的容器](/web/20221024141849/https://www.baeldung.com/ops/docker-list-containers):

```
$ docker ps
CONTAINER ID   IMAGE          COMMAND                  CREATED          STATUS          PORTS                 NAMES
fedf880ce2b6   mysql:latest   "docker-entrypoint.s…"   17 seconds ago   Up 16 seconds   3306/tcp, 33060/tcp   bael-mysql-demo
```

### 2.2.MySQL 客户端安装

必须安装一个客户端才能轻松访问 MySQL 服务器。根据我们的需要，我们可以将客户机安装在主机上，也可以安装在与服务器容器具有 IP 可达性的任何其他机器或容器上:

```
$ sudo apt install mysql-client -y
Reading package lists... Done
Building dependency tree
Reading state information... Done
mysql-client is already the newest version (5.7.37-0ubuntu0.18.04.1).
…
… output truncated …
…
```

现在，让我们提取 MySQL 客户端的安装路径和版本:

```
$ which mysql
/usr/bin/mysql
$ mysql --version
mysql  Ver 14.14 Distrib 5.7.37, for Linux (x86_64) using  EditLine wrapper
```

### 2.3.建立通信

接下来，让我们使用安装的客户端登录到服务器。传统上，我们使用带有用户名和密码的 MySQL 命令登录服务器。然而，它在基于容器的解决方案中不起作用:

```
$ mysql -u root -p
ERROR 2002 (HY000): Can't connect to local MySQL server through socket '/var/run/mysqld/mysqld.sock' (2)
```

我们将以如上所示的套接字错误结束。这里，重要的是要理解 MySQL 服务器是一个容器，而不是简单地安装在主机上。正如上一节所强调的，容器是轻量级服务器，拥有自己的计算资源、网络和存储。

[`inspect`](https://web.archive.org/web/20221024141849/https://docs.docker.com/engine/reference/commandline/inspect/) 命令帮助分配一个 [IP 地址](/web/20221024141849/https://www.baeldung.com/ops/docker-network-information)给 MySQL 服务器实例:

```
$ docker inspect -f '{{range.NetworkSettings.Networks}}{{.IPAddress}}{{end}}' bael-mysql-demo
172.17.0.2
```

让我们在客户机的主机选项中提供上述 IP 地址，默认端口号和协议类型为 TCP:

```
$ mysql -h 172.17.0.2 -P 3306 --protocol=tcp -u root -p
Enter password:
Welcome to the MySQL monitor.  Commands end with ; or \g.
…
… output truncated …
…
mysql>
mysql> show databases;
+--------------------+
| Database           |
+--------------------+
| information_schema |
| mysql              |
| performance_schema |
| sys                |
+--------------------+
4 rows in set (0.00 sec)

mysql> exit
Bye
```

恭喜，我们已经成功登录 MySQL 服务器了！

## 3.结论

总之，我们已经详细了解了部署 MySQL 服务器容器、在主机上安装 MySQL 客户端以及最后使用容器信息在它们之间建立连接的步骤。