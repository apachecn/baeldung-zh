# 列出码头集装箱

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/ops/docker-list-containers>

## 1.概观

Docker 提供了各种选项来列出和过滤不同状态的容器，甚至提供了定制列表输出的选项。

在本教程中，我们将看到如何以各种方式过滤 Docker [容器](/web/20220917053734/https://www.baeldung.com/docker-images-vs-containers)。

## 2.列出容器

**为了列出 Docker 容器，我们可以使用`“docker ps”`或`“docker container ls”`命令。**该命令提供了多种方式来列出和过滤特定 Docker 引擎上的所有容器。

让我们从列出所有正在运行的容器开始。

### 2.1.别名

**从 [Docker 1.13](https://web.archive.org/web/20220917053734/https://www.docker.com/blog/whats-new-in-docker-1-13/) 开始，Docker 团队将每个命令重新分组，放在与之交互的逻辑对象下。**例如，为了[列出 Docker 容器](https://web.archive.org/web/20220917053734/https://github.com/docker/docker-ce/blob/81c95466da0057d166c8ae3230edc935d18af694/components/cli/cli/command/container/list.go#L29)，除了`“` `docker ps”`之外，我们还可以使用`“` `docker container list” `甚至`“` `docker container ls” `命令。

所有这三个别名都支持同一组选项。然而，采用新语法是个好主意。

### 2.2.运行容器

**如果我们使用不带选项的`“` `docker container ls” `命令，它将列出所有正在运行的容器**:

```java
$ docker container ls
CONTAINER ID        IMAGE                COMMAND                  CREATED             STATUS              PORTS                                NAMES
1addfea727b3        mysql:5.6            "docker-en.."   2 seconds ago       Up 1 second         0.0.0.0:32801->3306/tcp              dazzling_hellman
09c4105cb356        nats:2.1.0-scratch   "/nats-…"       17 minutes ago      Up 17 minutes       4222/tcp, 6222/tcp, 8222/tcp         nats-1
443fc0c41710        rabbitmq:3.7         "docker-…"      17 minutes ago      Up 17 minutes       4369/tcp, 5671-5672/tcp, 25672/tcp   rabbit-1
b06cfe3053e5        postgres:11          "docker-…"      29 minutes ago      Up 29 minutes       0.0.0.0:32789->5432/tcp              pg-2
4cf774b9e4a4        redis:5              "docker-…"      30 minutes ago      Up 30 minutes       0.0.0.0:32787->6379/tcp              redis-2
```

到目前为止，我们已经运行了五个容器——Nats、RabbitMQ、PostgreSQL、MySQL 和 Redis。

默认情况下，输出显示每个运行容器的几个详细信息:

*   容器 ID 是容器的唯一标识符。这个标识符是一个很长的 SHA-256 散列的截断版本。
*   IMAGE 是用冒号分隔的容器图像名及其标签，比如`postgres:11`。
*   COMMAND 是负责运行容器的命令。
*   创建时间显示容器的创建时间。
*   状态显示集装箱的状态。如上所述，所有这些容器都在运行。
*   PORTS 显示主机和容器内部之间的端口映射。例如，`0.0.0.0:32789->5432/tcp` 意味着主机中的端口`32789 `被映射到容器中的端口`5432`。此外，我们可以看到我们没有为 Nats 容器映射任何端口— `4222/tcp, 6222/tcp, 8222/tcp`。
*   NAMES 表示 Docker 容器的可读名称，例如`pg-2`。

现在，我们应该知道每一列的含义。因此，从现在开始，为了简洁起见，我们将只显示这些列的子集。

### 2.3.所有容器

默认情况下，`“` `docker container ls” `命令只显示正在运行的容器。

然而，**如果我们通过`-a `或`–all `选项，它将列出所有(停止和运行的)容器**:

```java
$ docker container ls -a
CONTAINER ID        IMAGE                STATUS
1addfea727b3        mysql:5.6            Up 4 hours
32928d81a65f        mysql:5.6            Exited (1) 4 hours ago
09c4105cb356        nats:2.1.0-scratch   Up 4 hours
443fc0c41710        rabbitmq:3.7         Up 4 hours
b06cfe3053e5        postgres:11          Up 4 hours
16d3c67ebd40        postgres:11          Exited (0) 4 hours ago
4cf774b9e4a4        redis:5              Up 4 hours
99c537a3dd86        redis:5              Exited (0) 4 hours ago
```

我们现在有三个几小时前被停止的容器——Redis、MySQL 和 PostgreSQL。

### 2.4.最新容器

要查看最后的`n` Docker 容器(运行的和停止的)，我们可以使用`-n <number> `或`–last <number> `选项:

```java
$ docker container ls -n 2
CONTAINER ID        IMAGE               STATUS
1addfea727b3        mysql:5.6           Up 4 hours
32928d81a65f        mysql:5.6           Exited (1) 4 hours ago
```

也可以通过`-l` 或`–latest `选项查看最新的容器:

```java
$ docker container ls --latest
CONTAINER ID        IMAGE               STATUS
1addfea727b3        mysql:5.6           Up 4 hours
```

当然，我们可以用`-n 1 `选项实现同样的事情。

### 2.5.禁用截断

默认情况下，如果输出列的值超过某个阈值，Docker 会截断输出列。事实上，我们已经看到了容器 id 的截断值。

尽管这在大多数情况下是一个很好的特性，但我们可以使用`–no-trunc `选项禁用它:

```java
$ docker container ls --latest --no-trunc
CONTAINER ID                                                       COMMAND
1addfea727b38f484a2e0023ed7f47dcb9bbfc6e053f094c349391bb38cb3af7   "docker-entrypoint.sh mysqld"
```

我们可以看到输出列现在更加详细了。

### 2.6.安静模式

也可以只查看集装箱的集装箱 id。

为此，我们可以使用`-q `或`–quiet `选项:

```java
$ docker container ls -q
1addfea727b3
09c4105cb356
443fc0c41710
b06cfe3053e5
4cf774b9e4a4
```

我们可以混合搭配选项，并查看完整的容器标识符:

```java
$ docker container ls --quiet --no-trunc
1addfea727b38f484a2e0023ed7f47dcb9bbfc6e053f094c349391bb38cb3af7
09c4105cb3567ba0070dacf7381b9946165908c819c0841cffaa1855766537c7
443fc0c41710ee3811e72fe5079bf4696b9318e0754c38eeab1c960f5c5a7007
b06cfe3053e521704c67a1902a7302665ae05f66ef592419f32b8c73b2a066fd
4cf774b9e4a487a2ba658de37273994161378cffe7e69fe5c928ad29e6946372
```

当我们要将一组 id 传递给另一个命令时，安静模式尤其有用。

这里有一种强制删除所有容器的方法:

```java
$ docker container rm -f $(docker container ls -aq)
```

当然，这些组合应该非常谨慎地使用。

### 2.7.容器尺寸

我们可以通过`-s `或`–size `选项查看容器的大小及其在磁盘上的图像:

```java
$ docker container ls --latest -s
CONTAINER ID        IMAGE               SIZE
1addfea727b3        mysql:5.6           2B (virtual 256MB)
```

第一个值(2B)表示用于每个容器的可写层的字节数。第二个值是磁盘上的图像大小，在本例中是 256 MB。

### 2.8.定制输出

**如果我们对默认的输出格式不满意，我们可以使用 [Go 模板](https://web.archive.org/web/20220917053734/https://golang.org/pkg/text/template/)定制输出。**我们所要做的就是将想要的格式传递给`–format`选项。

让我们来看看实际情况:

```java
$ docker container ls --format "{{.ID}} -> Based on {{.Image}}, named {{.Names}}, ({{.Status}})"
1addfea727b3 -> Based on mysql:5.6, named dazzling_hellman, (Up 3 hours)
09c4105cb356 -> Based on nats:2.1.0-scratch, named nats-1, (Up 4 hours)
443fc0c41710 -> Based on rabbitmq:3.7, named rabbit-1, (Up 4 hours)
b06cfe3053e5 -> Based on postgres:11, named pg-2, (Up 4 hours)
4cf774b9e4a4 -> Based on redis:5, named redis-2, (Up 4 hours)
```

这里我们使用了 Go 模板格式中的模板化字符串。`ID`、`Image`、`Names`和`Status `是占位符，其余文本按原样呈现。

此外，还可以用表格的形式显示各列。

我们只需使用`table `前缀:

```java
$ docker container ls --format "table {{.ID}}\t{{.Image}}\t{{.Names}}"
CONTAINER ID        IMAGE                NAMES
1addfea727b3        mysql:5.6            dazzling_hellman
09c4105cb356        nats:2.1.0-scratch   nats-1
443fc0c41710        rabbitmq:3.7         rabbit-1
b06cfe3053e5        postgres:11          pg-2
4cf774b9e4a4        redis:5              redis-2
```

让我们仔细看看可以在模板字符串中使用的字段:

*   `.ID`–集装箱 id
*   `.Image`–图像名称和标签
*   `.Command`–负责运行该容器的命令
*   `.CreatedAt`–容器创建时间
*   `.Running`–集装箱启动后经过的时间
*   `.Ports`–端口映射
*   `.Status`–集装箱执行状态
*   `.Size`–容器及其镜像磁盘大小
*   `.Names`–集装箱名称
*   `.Labels`–分配给容器的所有标签
*   `.Mounts`–集装箱的容积
*   `.Networks`–附加到容器的所有网络名称

### 2.9.高级过滤

到目前为止，我们只根据运行或停止状态过滤了容器。事实证明，`“` `docker container ls” `提供的远不止这种基本的过滤。

**为了过滤容器，我们可以使用`-f `或`–filter `选项。**

这里我们将过滤状态为`exited `的容器:

```java
$ docker container ls --filter "status=exited"
CONTAINER ID        IMAGE               STATUS
32928d81a65f        mysql:5.6           Exited (1) 8 hours ago
16d3c67ebd40        postgres:11         Exited (0) 9 hours ago
99c537a3dd86        redis:5             Exited (0) 9 hours ago
```

注意，我们应该用`key=value `格式传递过滤标准。

**如果我们想同时应用多个过滤器，我们应该传递多个`–filter `选项。**

让我们更进一步，只保留退出状态等于 1 的`exited `容器:

```java
$ docker container ls --filter "status=exited" --filter "exited=1"
CONTAINER ID        IMAGE               STATUS
32928d81a65f        mysql:5.6           Exited (1) 8 hours ago
```

正如我们所料，现在只有 MySQL 符合过滤标准。Docker 容器状态不仅仅是运行或停止。

假设我们暂停一个 Docker 容器:

```java
$ docker container pause redis-2
```

然后，我们可以过滤所有暂停的容器:

```java
$ docker container ls --filter "status=paused"
CONTAINER ID        IMAGE               STATUS
4cf774b9e4a4        redis:5             Up 45 minutes (Paused)
```

此外，我们可以根据任何可能的状态来过滤容器，这些状态包括`created`、`restarting`、`running`、`removing`、`paused`、`exited`或`dead`。

如果我们知道容器名称的某个部分，我们可以搜索它:

```java
$ docker container ls -a --filter "name=pg"
CONTAINER ID        IMAGE               STATUS
b06cfe3053e5        postgres:11         Up 18 minutes
16d3c67ebd40        postgres:11         Exited (0) 9 hours ago
```

我们还可以根据容器的基本图像过滤容器:

```java
$ docker container ls -a --filter "ancestor=postgres"
CONTAINER ID        IMAGE               STATUS
b06cfe3053e5        postgres:11         Up 28 minutes
16d3c67ebd40        postgres:11         Exited (0) 9 hours ago
```

这里我们只列出了基于`postgres `图像的容器。

甚至可以根据创建时间过滤容器。

让我们只保留在 Nats 容器之前创建的容器:

```java
$ docker container ls --filter "before=nats-1"
CONTAINER ID        IMAGE               STATUS
443fc0c41710        rabbitmq:3.7        Up 52 minutes
b06cfe3053e5        postgres:11         Up 52 minutes
4cf774b9e4a4        redis:5             Up 52 minutes (Paused)
```

另一方面，我们可以使用`since `过滤器列出在 Nats 容器之后创建的所有 Docker 容器:

```java
$ docker container ls --filter "since=nats-1"
CONTAINER ID        IMAGE               STATUS
2fdc65a6effb        mysql:5.6           Exited (137) 4 days ago
1addfea727b3        postgres:11         Exited (0) 3 days ago
```

## 3.结论

在本文中，我们看到了如何使用`“docker container ls”` 命令及其有用的选项来列出和过滤 Docker 容器。

对于更详细的讨论，查看官方文档总是一个好主意。