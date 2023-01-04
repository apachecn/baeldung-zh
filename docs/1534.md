# 运行和启动 Docker 容器的区别

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/ops/docker-run-start-commands>

## 1.概观

[Docker](/web/20220722073354/https://www.baeldung.com/ops/docker-guide) 提供了一个有用的 [CLI](https://web.archive.org/web/20220722073354/https://docs.docker.com/engine/reference/commandline/cli/) 来与容器进行交互。在本教程中，我们将看到`r` `un` 和 s `tart` 命令，并通过一些实际例子强调它们的不同之处。

## 2.运行集装箱

**Docker 的`[run](https://web.archive.org/web/20220722073354/https://docs.docker.com/engine/reference/commandline/run/)`命令是其`create`和`start`命令的组合。它在其特定的图像上创建一个容器，然后启动它**。例如，让我们运行一个 [Postgres](https://web.archive.org/web/20220722073354/https://hub.docker.com/_/postgres) 容器:

```
docker run --name postgres_example -p 5432:5432 -v /volume:/var/lib/postgresql/data -e POSTGRES_PASSWORD=my_password -d postgres 
```

让我们用 [`docker ps`](https://web.archive.org/web/20220722073354/https://docs.docker.com/engine/reference/commandline/ps/) 来看看我们正在运行的容器:

```
CONTAINER ID   IMAGE     COMMAND                  CREATED          STATUS         PORTS                                       NAMES
52b7c79bfaa8   postgres  "docker-entrypoint.s…"   22 seconds ago   Up 20 seconds  0.0.0.0:5432->5432/tcp, :::5432->5432/tcp   postgres_example 
```

如果我们使用`[docker logs](https://web.archive.org/web/20220722073354/https://docs.docker.com/engine/reference/commandline/logs/)`，我们还可以检查关于一个已启动的容器的更多信息，例如:

```
starting PostgreSQL 13.2
listening on IPv4 address "0.0.0.0", port 5432
listening on IPv6 address "::", port 5432
listening on Unix socket "/var/run/postgresql/.s.PGSQL.5432"
database system is ready to accept connections
```

## 3.启动容器

**码头工人的** `[**start**](https://web.archive.org/web/20220722073354/https://docs.docker.com/engine/reference/commandline/start/)` **命令启动一个被停止的集装箱** 。容器可能因为不同的原因而停止，例如，当它消耗了太多内存并被主机操作系统终止时。

为了演示这一点，让我们手动停止我们之前创建的容器:

```
docker stop 52b7c79bfaa8
```

在这种情况下，我们的容器的运行列表将显示一个退出的容器:

```
CONTAINER ID   IMAGE     COMMAND                  CREATED          STATUS                    PORTS                                       NAMES
52b7c79bfaa8   postgres  "docker-entrypoint.s…"   2 minutes ago    Exited (0) 2 seconds ago  0.0.0.0:5432->5432/tcp, :::5432->5432/tcp   postgres_example 
```

让我们看看日志:

```
received fast shutdown request
aborting any active transactions
shutting down
database system is shut down
```

如果容器关闭，我们可能希望使用`docker start`再次启动它:

```
docker start 52b7c79bfaa8
```

如果启动容器时没有出现错误，我们将返回到运行容器状态。Docker 还提供了 [`docker restart`](https://web.archive.org/web/20220722073354/https://docs.docker.com/engine/reference/commandline/restart/) 命令，将 `stop`和`start` 组合成一个命令。

## 4.结论

在本教程中，我们简要讨论了 Docker 中的 `run` 和 `start` 命令。

我们已经看到了一个使用 `docker run` 运行容器的例子。如果一个容器停止了，我们可以用 `docker start` 重新启动它。