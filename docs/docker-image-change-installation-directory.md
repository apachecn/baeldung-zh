# 更改 Docker 映像安装目录

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/ops/docker-image-change-installation-directory>

## 1.概观

在使用 [Docker](https://web.archive.org/web/20221222091408/https://www.docker.com/) 容器时，我们经常需要创建各种持久对象，比如卷和映像。默认情况下，这些对象占用启动磁盘的磁盘空间。这种默认配置可能会导致一些重大的数据问题，例如用于其他应用程序的磁盘空间不足，或者在硬件出现故障时数据丢失。

在本教程中，我们将讨论如何在 Docker 中配置数据根目录。这允许我们更改映像安装目录并缓解上面提到的问题。

## 2.树立榜样

让我们创建几个持久 Docker 对象作为例子。

首先，我们将拉出 [NGINX](https://web.archive.org/web/20221222091408/https://www.nginx.com/) 和 [Redis](https://web.archive.org/web/20221222091408/https://redis.io/) 图像:

```java
$ docker image pull nginx
$ docker image pull redis
```

让我们验证它们是否拉好:

```java
$ docker image list 
REPOSITORY   TAG       IMAGE ID       CREATED       SIZE
nginx        latest    76c69feac34e   2 weeks ago   142MB
redis        latest    c2342258f8ca   2 weeks ago   117MB
```

接下来，我们将创建一个卷并找到它的挂载点:

```java
$ docker volume create dangling-volume

$ docker volume inspect dangling-volume -f '{{ .Mountpoint }}'
/var/lib/docker/volumes/dangling-volume/_data
```

最后，让我们在一个`/var/lib/docker/volumes/dangling-volume/_data`目录中创建一个示例文件:

```java
$ echo "Dangling volume" | sudo tee /var/lib/docker/volumes/dangling-volume/_data/sample.txt

$ sudo cat /var/lib/docker/volumes/dangling-volume/_data/sample.txt
Dangling volume
```

在下一节中，我们将看到如何更改 Docker 根目录，以便在不同的位置存储这样的持久对象。

## 3.更改映像安装目录

在 Docker 中，**镜像安装目录由`DockerRootDir`属性**表示。我们可以使用`[info](https://web.archive.org/web/20221222091408/https://docs.docker.com/engine/reference/commandline/info/)`子命令找到它的值:

```java
$ docker info -f '{{ .DockerRootDir }}'
/var/lib/docker
```

在这个例子中，引导盘的`/var/lib/docker`目录代表 Docker 根目录。

现在，让我们讨论改变这个默认根目录的各种方法。

### 3.1.使用守护程序配置文件

我们可以通过更新守护程序配置文件来更改默认根目录。该配置文件在 Linux 上的**默认位置是`/etc/docker/daemon.json`** 。

因此，让我们创建一个新目录，并通过编辑守护程序配置文件将其配置为根目录:

```java
$ mkdir -p /tmp/new-docker-root
$ sudo vi /etc/docker/daemon.json
```

然后我们编辑文件，使它有一个指向新创建的目录的`data-root`。当我们保存它时:

```java
$ sudo cat /etc/docker/daemon.json
{ 
   "data-root": "/tmp/new-docker-root"
}
```

最后，我们必须重启 docker 服务，并使用`info`子命令检查更新后的根目录:

```java
$ sudo systemctl restart docker
$ docker info -f '{{ .DockerRootDir}}'
/tmp/new-docker-root
```

现在，我们可以看到 Docker 根目录被设置为`/tmp/new-docker-root`。

### 3.2.恢复默认配置

在上一节中，我们通过更新守护程序配置文件更改了默认配置。但是，在移动到下一部分之前，我们必须将其恢复到以前的状态。

首先，我们通过删除守护程序配置文件和`/tmp/new-docker-root`目录将设置重置为默认值:

```java
$ sudo rm /etc/docker/daemon.json
$ sudo rm -rf /tmp/new-docker-root/
```

接下来，我们必须重新启动 docker 服务以使更改生效:

```java
$ sudo systemctl restart docker 
```

最后，我们可以验证 Docker 根目录是否设置为其默认位置:

```java
$ docker info -f '{{ .DockerRootDir}}'
/var/lib/docker
```

### 3.3.使用`systemd`配置文件

类似地，我们也可以修改 docker 服务的 [systemd](/web/20221222091408/https://www.baeldung.com/linux/create-remove-systemd-services) 配置来达到相同的结果。Docker 使用 **`/lib/systemd/system/docker.service`单元文件存储其配置**。让我们看看如何更新这个文件。

首先，我们将创建一个新目录，并从`/lib/systemd/system/docker.service` 文件`:`中更新`ExecStart`属性

```java
$ mkdir -p /tmp/new-docker-root/
$ sudo vi /lib/systemd/system/docker.service
```

现在，更新后的文件如下所示:

```java
$ grep ExecStart /lib/systemd/system/docker.service
ExecStart=/usr/bin/dockerd --data-root /tmp/new-docker-root -H fd:// --containerd=/run/containerd/containerd.sock
```

在这个例子中，我们已经使用 `–data-root /tmp/new-docker-root` 属性**配置了 Docker 根目录。**

接下来，我们必须重新加载单元文件并重新启动 docker 服务，以使更改生效:

```java
$ sudo systemctl daemon-reload
$ sudo systemctl restart docker 
```

最后，让我们使用`info`子命令检查更新的数据根目录:

```java
$ docker info -f '{{ .DockerRootDir}}'
/tmp/new-docker-root
```

在这里，我们可以看到数据根目录指向`/tmp/new-docker-root` 位置`.`

### 3.4.限制

在前面几节中，我们看到了如何更改 Docker 数据根目录。然而，仅仅适当地改变数据根是不够的。因为这个配置**没有定位先前创建的持久对象**。

为了理解这一点，让我们列出图像和卷:

```java
$ docker image list
REPOSITORY   TAG       IMAGE ID   CREATED   SIZE

$ docker volume list
DRIVER    VOLUME NAME 
```

正如我们所看到的，Docker 无法从以前的数据根目录中定位映像和卷。

## 4.迁移持久对象

对于改变持久对象位置的完整解决方案，我们可能还希望迁移现有的对象。

### 4.1.使用`rsync`命令复制数据

是一个命令行工具，可以高效地复制和同步文件和目录。我们可以使用这个命令从`/var/lib/docke` r 目录中复制内容:

让我们使用`rsync`命令复制数据并重启 docker 服务:

```java
$ sudo rsync -aqxP /var/lib/docker/ /tmp/new-docker-root
$ sudo systemctl restart docker
```

在本例中，我们使用了以下内容:

*   –`a`选项启用存档模式
*   –`q`选项抑制非错误消息
*   –`x`在递归复制目录时避免跨越文件系统边界的选项
*   –`P`选项保留部分复制的文件/目录

现在，让我们列出图像和卷:

```java
$ docker image list
REPOSITORY   TAG       IMAGE ID       CREATED       SIZE
nginx        latest    76c69feac34e   3 weeks ago   142MB
redis        latest    c2342258f8ca   3 weeks ago   117MB

$ docker volume list
DRIVER    VOLUME NAME
local     dangling-volume
```

正如我们所看到的，Docker 现在能够识别之前创建的持久对象。

### 4.2.恢复默认配置

首先，让我们停止 docker 服务并从`/lib/systemd/system/docker.service`单元文件中删除`–data-root`属性:

```java
$ sudo systemctl stop docker 
$ sudo vi /lib/systemd/system/docker.service
```

修改后，单元文件如下所示:

```java
$ grep ExecStart /lib/systemd/system/docker.service
ExecStart=/usr/bin/dockerd -H fd:// --containerd=/run/containerd/containerd.sock
```

现在，我们将重新加载单元文件，重新启动 docker 服务，并检查更新后的数据根目录:

```java
$ sudo systemctl daemon-reload
$ sudo systemctl restart docker

$ docker info -f '{{ .DockerRootDir}}'
/var/lib/docker
```

## 5.结论

在本文中，我们看到了如何通过配置`data-root`属性来更改 Docker 中的镜像安装目录。

首先，我们使用守护程序配置文件来更改 Docker 根目录。接下来，我们讨论了如何使用`systemd`单元文件获得相同的结果。然后，我们讨论了这些方法的局限性。

最后，我们讨论了如何通过迁移持久对象来克服这些限制。