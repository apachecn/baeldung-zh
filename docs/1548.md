# 与 Docker 容器连接和分离

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/ops/docker-attach-detach-container>

## 1.概观

在使用 docker 容器时，我们经常需要以交互模式运行它。这是我们将终端的标准输入、输出或错误流附加到容器的地方。

通常我们更喜欢在后台运行我们的容器。但是，我们可能希望稍后连接到它以检查其输出或错误，或者断开会话。

在这篇短文中，我们将学习一些有用的命令来实现这些。我们还将看到在不停止容器的情况下从会话中分离的不同方法。

## 2.以附加/分离模式运行容器

让我们看看如何在附加或分离模式下运行容器。

### 2.1.默认模式

默认情况下，Docker 在前台运行一个容器:

```
$ docker run --name test_redis -p 6379:6379 redis
```

这意味着在这个过程结束之前，我们不能返回到我们的 shell 提示符。

上面的命令将标准输出(`stdout`)和标准误差(`stderr`)流与我们的终端链接起来。因此，**我们可以在我们的终端中看到容器的控制台输出。**

`–name`选项给容器一个名字。我们以后可以在其他命令中使用相同的名称来引用这个容器。或者，我们可以通过执行 [`docker ps`](https://web.archive.org/web/20220812123411/https://docs.docker.com/engine/reference/commandline/ps/) 命令得到的容器 id 来引用它。

我们还可以使用`-a`选项从 stdin、stdout 和 stderr 中选择特定的流进行连接:

```
$ docker run --name test_redis -a STDERR -p 6379:6379 redis
```

上面的命令意味着我们只看到来自容器的错误消息。

### 2.2.对话方式

我们在交互模式下用 `-i` 和 `-t` 选项一起初始化一个容器:

```
$ docker run -it ubuntu /bin/bash
```

这里，`-i`选项将 bash shell 的标准输入流( stdin )附加到容器中，`-t`选项将一个伪终端分配给进程。这让我们从终端与容器进行交互。

### 2.3.分离模式

我们使用`-d`选项在分离模式下运行容器:

```
$ docker run -d --name test_redis -p 6379:6379 redis
```

该命令启动容器，打印其 id，然后返回到 shell 提示符。因此，当容器继续在后台运行时，我们可以继续其他任务。

我们可以稍后使用其名称或容器 id 连接到该容器。

## 3.与运行中的容器交互

### 3.1.执行命令

**[执行](https://web.archive.org/web/20220812123411/https://docs.docker.com/engine/reference/commandline/exec/)命令让我们在已经运行的容器内执行命令**:

```
$ docker exec -it test_redis redis-cli
```

该命令在已经运行的名为`test_redis`的 Redis 容器中打开一个`redis-cli`会话。我们也可以使用容器 id 来代替名称。如第 2.2 节所述，选项`-it`启用交互模式。

但是，我们可能只想获得一个键的值:

```
$ docker exec test_redis redis-cli get mykey
```

这将执行`redis-cli` `,` 中的`get`命令，返回键`mykey`的值，并关闭会话。

也可以在后台执行命令:

```
$ docker exec -d test_redis redis-cli set anotherkey 100
```

在这里，我们使用-d 来实现这个目的。它将键`anotherKey`的值设置为 100，但不显示命令的输出。

### 3.2.附加会话

**[`attach`](https://web.archive.org/web/20220812123411/https://docs.docker.com/engine/reference/commandline/attach/)命令将我们的终端连接到一个正在运行的容器**:

```
$ docker attach test_redis
```

默认情况下，该命令将标准输入、输出或错误流与主机外壳绑定在一起。

为了只查看输出和错误消息，我们可以使用`–no-stdin`选项省略`stdin`:

```
$ docker attach --no-stdin test_redis
```

## 4.从容器中取出

从 docker 容器分离的方式取决于它的运行模式。

### 4.1.默认模式

按 CTRL-c 是结束会话的常用方式。但是，如果我们在没有`-d`或`-it`选项的情况下启动了我们的容器，**`CTRL-c`命令会停止容器，而不是从它那里断开**。会话将`CTRL-c`即`SIGINT`信号传播给容器，并终止其主进程。

让我们覆盖通过`–sig-proxy=false`的行为:

```
$ docker run --name test_redis --sig-proxy=false -p 6379:6379 redis
```

现在，我们可以按下`CTRL-c`只分离当前会话，而容器继续在后台运行。

### 4.2.对话方式

在这种模式下，`CTRL-c`充当交互会话的命令，因此它不作为分离键。在这里，**我们应该使用`CTRL-p CTRL-q`来结束会话**。

### 4.3.后台方式

在这种情况下，我们需要在附加会话时用**覆盖`–sig-proxy`值:**

```
$ docker attach --sig-proxy=false test_redis
```

我们也可以通过`–detach-keys`选项定义一个单独的键:

```
$ docker attach --detach-keys="ctrl-x" test_redis
```

当我们按下`CTRL-x`时，这将分离容器并返回提示。

## 5.结论

在本文中，我们看到了如何在附加和分离模式下启动 docker 容器。

然后，我们看了一些命令来开始或结束一个活动容器的会话。