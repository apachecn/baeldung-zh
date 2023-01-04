# 将文件复制到 Docker 容器和从 Docker 容器复制文件

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/ops/docker-copying-files>

## 1.介绍

随着我们越来越多的应用程序被部署到云环境中，使用 [Docker](/web/20220926200722/https://www.baeldung.com/docker-java-api) 正在成为开发人员的一项必要技能。通常在调试应用程序时，将文件复制到 Docker 容器中或从 Docker 容器中复制出来是很有用的。

在本教程中，我们将看看一些不同的方法，我们可以从 Docker 容器复制文件。

## 2.Docker `cp`命令

从 Docker 容器复制文件最快的方法是使用 [`docker cp`命令](https://web.archive.org/web/20220926200722/https://docs.docker.com/engine/reference/commandline/cp/)。**该命令非常类似于 Unix cp 命令**，其语法如下:

`docker cp <SRC> <DEST>`

在我们看这个命令的一些例子之前，让我们假设我们有以下 Docker 容器在运行:

```
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS                    NAMES
1477326feb62        grafana/grafana     "/run.sh"                2 months ago        Up 3 days           0.0.0.0:3000->3000/tcp   grafana
8c45029d15e8        prom/prometheus     "/bin/prometheus --c…"   2 months ago        Up 3 days           0.0.0.0:9090->9090/tcp   prometheus
```

第一个示例将文件从主机上的 `/tmp`目录复制到`grafana`容器中的 Grafana 安装目录:

```
docker cp /tmp/config.ini grafana:/usr/share/grafana/conf/
```

我们还可以使用容器 id 来代替它们的名称:

```
docker cp /tmp/config.ini 1477326feb62:/usr/share/grafana/conf/
```

要将文件从`grafana`容器复制到主机上的`/tmp`目录，我们只需切换参数的顺序:

```
docker cp grafana:/usr/share/grafana/conf/defaults.ini /tmp
```

我们还可以复制整个目录，而不是单个文件。这个例子将整个`conf`目录从`grafana`容器复制到主机上的`/tmp`目录:

```
docker cp grafana:/usr/share/grafana/conf /tmp
```

`docker cp`命令确实有一些限制。首先，**我们不能用它在两个容器**之间复制。它只能用于在主机系统和单个容器之间复制文件。

第二，虽然它与 [Unix cp 命令](/web/20220926200722/https://www.baeldung.com/linux/copy-file-to-multiple-directories)具有相同的语法，但它不支持相同的标志。事实上，它只支持两种:

`-a`:存档模式，保存被复制文件的所有 uid/gid 信息
`-L`:始终跟随 SRC 中的符号链接

## 3.卷装载

将文件复制到 Docker 容器或从 Docker 容器复制文件的另一种方法是使用卷挂载。这意味着我们在容器内部提供了一个来自主机系统的目录。

为了使用卷挂载，**我们必须运行带有`-v`标志的容器:**

```
docker run -d --name=grafana -p 3000:3000 grafana/grafana -v /tmp:/transfer
```

上面的命令运行一个`grafana`容器，并从主机上将`/tmp`目录挂载到名为`/transfer`的容器中作为一个新目录。如果我们愿意，**我们可以提供多个`-v`标志**来在容器内创建多个卷挂载。

这种方法有几个优点。首先，**我们可以使用 Unix cp 命令，它比`docker cp`命令有更多的标志和选项**。

第二个优势是**我们可以为所有 Docker 容器**创建一个共享目录。这意味着我们可以在容器之间直接复制，只要它们都有相同的卷装载。

请记住，这种方法的缺点是所有文件都必须经过卷挂载。**这意味着我们不能用一个命令**复制文件。相反，我们首先将文件复制到挂载的目录中，然后复制到它们最终需要的位置。

这种方法的另一个缺点是我们可能会遇到文件所有权的问题。Docker 容器通常只有一个根用户，这意味着在容器内部创建的**文件将默认拥有根用户**。如果需要，我们可以在主机上使用 [Unix `chown`命令](https://web.archive.org/web/20220926200722/http://www.linfo.org/chown.html)来恢复文件所有权。

## 4.`Dockerfile`

[Dockerfiles](https://web.archive.org/web/20220926200722/https://docs.docker.com/engine/reference/builder/) 用于构建 Docker 映像，然后将其实例化到 Docker 容器中。 **Dockerfiles 可以包含几个不同的指令，其中一个是`COPY`。**

`COPY`指令让我们将一个(或多个)文件从主机系统复制到映像中。这意味着文件成为从该映像创建的每个容器的一部分。

`COPY`指令的语法类似于我们上面看到的其他复制命令:

```
COPY <SRC> <DEST>
```

就像其他复制命令一样，`SRC`可以是主机上的单个文件或目录。它还可以包含通配符来匹配多个文件。

让我们看一些例子。

这将把当前 Docker 构建上下文中的一个复制到映像中:

```
COPY properties.ini /config/
```

这将把所有 XML 文件复制到 Docker 映像中:

```
COPY *.xml /config/
```

这种方法的主要缺点是**我们不能用它来运行 Docker 容器**。 [Docker 映像不是 Docker 容器](/web/20220926200722/https://www.baeldung.com/docker-images-vs-containers)，所以这种方法只有在预先知道映像中需要的文件集时才有意义。

## 5.结论

在本教程中，我们看到了如何在 Docker 容器中复制文件。每种方法都有优点和缺点，所以我们必须选择最适合我们需要的方法。