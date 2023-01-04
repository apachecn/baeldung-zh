# 从 Docker 容器获取环境变量

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/ops/docker-get-environment-variable>

## 1.概观

Docker 是一个容器化的平台，它将应用程序及其所有依赖项打包在一起。理想情况下，这些应用程序需要一定的环境才能启动。在 Linux 中，我们使用环境变量来满足这个需求。这些变量决定了应用程序的行为。

在本教程中，我们将学习检索在运行 Docker 容器时设置的所有环境变量。就像有多种方式将环境变量传递给 Docker 容器一样，一旦设置好，也有多种方式获取这些变量。

在我们进一步讨论之前，让我们首先理解对环境变量的需求。

## 2.了解 Linux 中的环境变量

环境变量是一组动态的键值对，可在系统范围内访问。这些变量可以帮助系统定位一个包，配置任何服务器的行为，甚至使 bash 终端输出变得直观。

默认情况下，主机上的环境变量不会传递给 Docker 容器。原因是 Docker 容器应该与主机环境隔离。因此，如果我们想在 Docker 容器中使用一个环境，那么我们必须显式地设置它。

现在让我们看看从 Docker 容器内部获取环境变量的不同方法。

## 3.使用`docker` `exec`命令提取

出于演示目的，让我们首先运行一个 [Alpine](https://web.archive.org/web/20220928130110/https://hub.docker.com/_/alpine) Docker 容器，并向其传递一些环境变量:

```
docker run -itd --env "my_env_var=baeldung" --name mycontainer alpine
9de9045b5264d2de737a7ec6ba23c754f034ff4f35746317aeefcea605d46e84
```

这里，我们在名为`mycontainer`的 Docker 容器中传递值为`baeldung`的`my_env_var `。

现在让我们使用 [`docker exec`](https://web.archive.org/web/20220928130110/https://docs.docker.com/engine/reference/commandline/exec/) 命令来获取名为`my_env_var`的环境变量:

```
$ docker exec mycontainer /usr/bin/env
PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
HOSTNAME=9de9045b5264
my_env_var=baeldung
HOME=/root 
```

这里，我们在 Docker 容器中执行`/usr/bin/env`实用程序。使用这个工具，您可以查看 Docker 容器中设置的所有环境变量。注意，我们的`my_env_var`也出现在输出中。

我们也可以使用以下命令来实现类似的结果:

```
$ docker exec mycontainer /bin/sh -c /usr/bin/env
HOSTNAME=9de9045b5264
SHLVL=1
HOME=/root
my_env_var=baeldung
PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
PWD=/ 
```

**注意，与之前的输出相比，现在有了更多的环境变量。**这是因为这次我们在`/bin/sh`二进制的帮助下执行命令。这个二进制文件隐式地设置了一些额外的环境变量。

另外，`/bin/sh` shell 并不一定要出现在所有的 Docker 映像中。例如，在 [centos](https://web.archive.org/web/20220928130110/https://hub.docker.com/_/centos) Docker 图像中，其中包含了`/bin/bash` shell，我们将 检索环境 变量 使用跟随 命令:

```
$ docker run -itd --env "container_type=centos" --name centos_container centos
aee6f2718f18723906f7ab18ab9c37a539b6b2c737f588be71c56709948de9eb
$ docker exec centos_container bash -c /usr/bin/env
container_type=centos
HOSTNAME=aee6f2718f18
PWD=/
HOME=/root
SHLVL=1
PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
_=/usr/bin/env
```

我们还可以使用`docker exec`命令获取单个环境变量的值:

```
$ docker exec mycontainer printenv my_env_var
baeldung 
```

**[`printenv`](https://web.archive.org/web/20220928130110/https://man7.org/linux/man-pages/man1/printenv.1.html) 是另一个命令行实用程序，显示 Linux 中的环境变量。**这里，我们将环境变量名称`my_env_var`作为参数传递给`printenv`。这将打印出`my_env_var`的值。

这种方法的缺点是**Docker 容器必须处于运行状态**才能检索环境变量。

## 4.使用`docker` `inspect`命令提取

现在让我们来看看当 Docker 容器处于停止状态时获取环境变量的另一种方法。为此我们将使用 [`docker inspect`](https://web.archive.org/web/20220928130110/https://docs.docker.com/engine/reference/commandline/inspect/) 命令。

`docker inspect`提供所有 Docker 资源的详细信息。输出是 JSON 格式的。因此，我们可以根据需要过滤输出。

让我们操作`docker inspect`命令来只显示容器的环境变量:

```
$ docker inspect mycontainer --format "{{.Config.Env}}"
[my_env_var=baeldung PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin]
```

这里，我们使用`–format`选项从`docker inspect`输出中过滤了环境变量。同样，`my_env_var`出现在输出中。

我们还可以使用`docker inspect`命令获取一个环境变量:

```
$ docker inspect mycontainer | jq -r '.[].Config.Env[]|select(match("^my_env_var"))|.[index("=")+1:]'
baeldung
```

[`jq`](/web/20220928130110/https://www.baeldung.com/linux/jq-command-json) 是一个轻量级的 JSON 处理器，可以解析和转换 JSON 数据。这里，我们将把`docker inspect`的 JSON 输出传递给`jq`命令。然后，它搜索`my_env_var`变量，并通过将其拆分为“=”来显示其值。

请注意，我们可以在`docker exec`和`docker inspect`命令中使用容器 id。

与`docker exec`不同，`docker inspect`命令对停止和运行的容器都有效。

## 5.结论

在本文中，我们学习了如何从 Docker 容器中检索所有环境变量。我们从讨论环境变量在 Linux 中的重要性开始。然后，我们查看了用于检索环境变量的`docker exec`和`docker inspect`命令。

`docker exec`方法有一些限制，而`docker inspect`命令在所有情况下都可以运行。