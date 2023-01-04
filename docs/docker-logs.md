# 码头日志指南

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/ops/docker-logs>

## 1.概观

Docker 是一个操作系统级的虚拟化平台，允许我们在容器中托管应用程序。此外，它有助于应用程序和基础设施的分离，从而实现快速软件交付。

由 [Docker 容器](https://web.archive.org/web/20221021012638/https://docs.docker.com/engine/reference/commandline/container/)生成的日志文件包含各种有用的信息。每当事件发生时，Docker 容器都会创建日志文件。

Docker 为 STDOUT 或 STDERR 生成日志，包括日志来源、输出流数据和时间戳。使用日志文件可以调试和找到问题的根本原因。

在本教程中，我们将研究以不同的方式访问 Docker 日志。

## 2.了解 Docker 日志

在 Docker 中，主要有两种日志文件。Docker 守护进程日志提供了对 Docker 服务整体状态的深入了解。Docker 容器日志涵盖了与特定容器相关的所有日志。

我们将主要探索访问 Docker 容器日志的不同命令。我们将使用 [`docker logs`](https://web.archive.org/web/20221021012638/https://docs.docker.com/engine/reference/commandline/logs/) 命令并通过直接访问系统上的日志文件来检查容器日志。

日志文件对于调试问题非常有用，因为它们提供了所发生问题的详细信息。通过分析 Docker 日志，我们可以更快地诊断和解决问题。

## 3.使用`docker logs`命令

在我们继续之前，让我们先运行一个示例 Postgres Docker 容器:

```
$ docker run -itd -e POSTGRES_USER=baeldung -e POSTGRES_PASSWORD=baeldung -p 5432:5432 -v /data:/var/lib/postgresql/data --name postgresql-baedlung postgres
Unable to find image 'postgres:latest' locally
latest: Pulling from library/postgres
214ca5fb9032: Pull complete 
...
95df4ec75c64: Pull complete 
Digest: sha256:2c954f8c5d03da58f8b82645b783b56c1135df17e650b186b296fa1bb71f9cfd
Status: Downloaded newer image for postgres:latest
bce34bb3c6175fe92c50d6e5c8d2045062c2b502b9593a258ceb6cafc9a2356a 
```

为了说明这一点，让我们检查一下`postgresql-baedlung`容器的`containerId`:

```
$ docker ps 
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS                    NAMES
bce34bb3c617        postgres            "docker-entrypoint.s…"   12 seconds ago      Up 10 seconds       0.0.0.0:5432->5432/tcp   postgresql-baedlung
```

从上面命令的输出中我们可以看到，`postgresql-baedlung`正在用 container id“BCE 34 bb 3c 617”运行。现在让我们研究一下用于监控日志的`docker logs`命令:

```
$ docker logs bce34bb3c617
2022-05-16 18:13:58.868 UTC [1] LOG:  starting PostgreSQL 14.2 (Debian 14.2-1.pgdg110+1)
  on x86_64-pc-linux-gnu, compiled by gcc (Debian 10.2.1-6) 10.2.1 20210110, 64-bit
2022-05-16 18:13:58.869 UTC [1] LOG:  listening on IPv4 address "0.0.0.0", port 5432
2022-05-16 18:13:58.869 UTC [1] LOG:  listening on IPv6 address "::", port 5432
```

这里，日志包含带有时间戳的输出流的数据。上面的命令不包含连续的日志输出。为了查看容器的连续日志输出，我们需要在`docker logs`命令中使用`“–follow”`选项。

`“–follow”`选项是最有用的 Docker 选项之一，因为它允许我们监控容器的实时日志:

```
$ docker logs --follow  bce34bb3c617
2022-05-16 18:13:58.868 UTC [1] LOG:  starting PostgreSQL 14.2 (Debian 14.2-1.pgdg110+1)
  on x86_64-pc-linux-gnu, compiled by gcc (Debian 10.2.1-6) 10.2.1 20210110, 64-bit
...
2022-05-16 18:13:59.018 UTC [1] LOG:  database system is ready to accept connections
```

上面命令的一个缺点是它从一开始就包含了所有的日志。我们来看看这个命令，查看包含最近记录的连续日志输出:

```
$ docker logs --follow --tail 1 bce34bb3c617
2022-05-16 18:13:59.018 UTC [1] LOG:  database system is ready to accept connections 
```

我们还可以使用带有`docker log`命令的`“since”`选项来查看特定时间的文件:

```
$ docker logs --since 2022-05-16  bce34bb3c617
2022-05-16 18:13:58.868 UTC [1] LOG:  starting PostgreSQL 14.2 (Debian 14.2-1.pgdg110+1)
  on x86_64-pc-linux-gnu, compiled by gcc (Debian 10.2.1-6) 10.2.1 20210110, 64-bit
...
2022-05-16 18:13:59.018 UTC [1] LOG:  database system is ready to accept connections
```

或者，我们也可以使用 [`docker container logs`](https://web.archive.org/web/20221021012638/https://docs.docker.com/engine/reference/commandline/container_logs/) 命令代替`docker logs`命令:

```
$ docker container logs --since 2022-05-16  bce34bb3c617
2022-05-16 18:13:58.868 UTC [1] LOG:  starting PostgreSQL 14.2 (Debian 14.2-1.pgdg110+1)
  on x86_64-pc-linux-gnu, compiled by gcc (Debian 10.2.1-6) 10.2.1 20210110, 64-bit
...
2022-05-16 18:13:59.018 UTC [1] LOG:  database system is ready to accept connections
```

这里，我们可以从上面的输出中看到，这两个命令的工作方式完全相同。**`docker container logs`命令在新版本中已被弃用。**

## 4.使用默认日志文件

Docker 以 JSON 格式存储所有的 STDOUT 和 STDERR 输出。此外，还可以从主机监控所有实时 Docker 日志。默认情况下，Docker 使用`json-file`日志驱动程序将日志文件存储在主机上的专用目录中。**日志文件目录是运行容器的主机上的/var/lib/docker/containers/<container _ id>。**

为了演示，让我们检查一下我们的`postgress-baeldung`容器的日志文件:

```
$ cat /var/lib/docker/containers/bce34bb3c6175fe92c50d6e5c8d2045062c2b502b9593a258ceb6cafc9a2356a/
  bce34bb3c6175fe92c50d6e5c8d2045062c2b502b9593a258ceb6cafc9a2356a-json.log 
{"log":"\r\n","stream":"stdout","time":"2022-05-16T18:13:58.833312658Z"}
{"log":"PostgreSQL Database directory appears to contain a database; Skipping initialization\r\n","stream":"stdout","time":"2022-05-16T18:13:58.833360038Z"}
{"log":"\r\n","stream":"stdout","time":"2022-05-16T18:13:58.833368499Z"}
```

在上面的输出中，我们可以看到数据是 JSON 格式的。

## 5.清除日志文件

有时我们用完了系统上的磁盘空间，我们注意到 Docker 日志文件占用了大量空间。为此，我们首先需要找到日志文件，然后删除它们。此外，确保清除日志文件不会影响正在运行的容器的状态。

下面是清除存储在主机上的所有日志文件的命令:

```
$ truncate -s 0 /var/lib/docker/containers/*/*-json.log 
```

请注意，上面的命令不会删除日志文件。相反，它将删除日志文件中的所有内容。通过执行以下命令，我们可以删除与特定容器相关联的日志文件:

```
$ truncate -s 0 /var/lib/docker/containers/dd207f11ebf083f97355be1ae18420427dd2e80b061a7bf6fb0afc326ad04b10/*-json.log 
```

在容器的开始，我们还可以使用`docker run`命令的`“–log-opt max-size”`和`–log-opt max-file”` 选项在外部限制日志文件的大小:

```
$ docker run --log-opt max-size=1k --log-opt max-file=5 -itd -e POSTGRES_USER=baeldung -e POSTGRES_PASSWORD=baeldung -p 5432:5432
  -v /data:/var/lib/postgresql/data --name postgresql-baedlung postgres
3eec82654fe6c6ffa579752cc9d1fa034dc34b5533b8672ebe7778449726da32
```

现在，让我们检查一下`/var/lib/docker/containers/3eec82654fe6c6ffa579752cc9d1fa034dc34b5533b8672ebe7778449726da32`目录中日志文件的数量和大小:

```
$ ls -la
total 68
drwx------. 4 root root 4096 May 17 02:06 .
drwx------. 5 root root  222 May 17 02:07 ..
drwx------. 2 root root    6 May 17 02:02 checkpoints
-rw-------. 1 root root 3144 May 17 02:02 config.v2.json
-rw-r-----. 1 root root  587 May 17 02:06 3eec82654fe6c6ffa579752cc9d1fa034dc34b5533b8672ebe7778449726da32-json.log
-rw-r-----. 1 root root 1022 May 17 02:06 3eec82654fe6c6ffa579752cc9d1fa034dc34b5533b8672ebe7778449726da32-json.log.1
-rw-r-----. 1 root root 1061 May 17 02:06 3eec82654fe6c6ffa579752cc9d1fa034dc34b5533b8672ebe7778449726da32-json.log.2
-rw-r-----. 1 root root 1056 May 17 02:06 3eec82654fe6c6ffa579752cc9d1fa034dc34b5533b8672ebe7778449726da32-json.log.3
-rw-r-----. 1 root root 1058 May 17 02:06 3eec82654fe6c6ffa579752cc9d1fa034dc34b5533b8672ebe7778449726da32-json.log.4
-rw-r--r--. 1 root root 1501 May 17 02:02 hostconfig.json
-rw-r--r--. 1 root root   13 May 17 02:02 hostname
-rw-r--r--. 1 root root  174 May 17 02:02 hosts
drwx------. 2 root root    6 May 17 02:02 mounts
-rw-r--r--. 1 root root   69 May 17 02:02 resolv.conf
-rw-r--r--. 1 root root   71 May 17 02:02 resolv.conf.hash
```

在这里，我们可以看到创建了五个日志文件，每个日志文件的大小最大为 1 kb。如果我们删除一些日志文件，在这种情况下，我们将生成一个具有相同日志文件名的新日志。

我们也可以在`/etc/docker/daemon.json`文件中提供日志`max-size`和`max-file`的配置。让我们看看 daemon.json 文件的配置:

```
{
    "log-driver": "json-file",
    "log-opts": {
        "max-size": "1k",
        "max-file": "5" 
    }
}
```

这里，我们在`daemon.json,`中提供了相同的配置，重要的是，所有新的容器都将使用这种配置运行。更新完`daemon.json`文件后，我们需要重启 Docker 服务。

## 6.将 Docker 容器日志重定向到单个文件

**默认情况下，Docker 容器日志文件存储在`/var/lib/docker/containers/<containerId>`目录中。此外，我们还可以将 Docker 容器日志重定向到其他文件。**

为了说明这一点，我们来看看重定向容器日志的命令:

```
$ docker logs -f containername &> baeldung-postgress.log &
```

这里，在上面的命令中，我们将所有实时日志重定向到`baeldung-postgress.log`文件。此外，我们使用`&`在后台运行该命令，因此它将一直运行，直到被显式停止。

## 7.结论

在本教程中，我们学习了监视容器日志的不同方法。首先，我们查看了用于监控实时日志的`docker logs,`和`docker container logs`命令。后来，我们使用默认的容器日志文件来监控日志。

最后，我们研究了日志文件的清除和重定向。简而言之，我们研究了监视和截断日志文件。