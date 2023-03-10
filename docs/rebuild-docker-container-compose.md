# 在 Docker 编写中重建 Docker 容器

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/rebuild-docker-container-compose>

## 1.概观

在本教程中，我们将看到如何用`[docker-compose](/web/20221024150313/https://www.baeldung.com/ops/docker-compose)`独立地重建一个容器。

## 2.问题的呈现

**让我们定义一个有两个容器**的`docker-compose.yml`配置文件:一个引用最新的`ubuntu`图像，另一个引用最新的`alpine`图像。我们将为每个带有“`tty: true`”的容器添加伪终端，以防止容器在启动时直接退出:

```java
version: "3.9"
services:
  ubuntu:
    image: "ubuntu:latest"
    tty: true
  alpine:
    image: "alpine:latest"
    tty: true
```

现在让我们构建容器并启动它们。我们将使用带有`-d`选项的`docker-compose up`命令让它们在后台运行:

```java
$ docker-compose up -d

Container {folder-name}-alpine-1  Creating
Container {folder-name}-ubuntu-1  Creating
Container {folder-name}-ubuntu-1  Created
Container {folder-name}-alpine-1  Created
Container {folder-name}-ubuntu-1  Starting
Container {folder-name}-alpine-1  Starting
Container {folder-name}-alpine-1  Started
Container {folder-name}-ubuntu-1  Started
```

我们可以快速检查我们的容器是否按预期运行:

```java
$ docker-compose ps
NAME                         COMMAND             SERVICE             STATUS              PORTS
{folder-name}-alpine-1   "/bin/sh"           alpine              running
{folder-name}-ubuntu-1   "bash"              ubuntu              running
```

我们现在来看看如何在不影响`alpine`容器的情况下重建并重启`ubuntu`容器。

## 3.独立地重建和重新启动容器

**将容器的名称添加到`docker-compose up`命令中即可。**我们将在启动容器之前添加`build`选项来构建图像。我们还将添加`force-recreate`旗帜，因为我们没有改变图像:

```java
$ docker-compose up -d --force-recreate --build ubuntu
Container {folder-name}-ubuntu-1  Recreate
Container {folder-name}-ubuntu-1  Recreated
Container {folder-name}-ubuntu-1  Starting
Container {folder-name}-ubuntu-1  Started
```

正如我们所见，`ubuntu`容器被重新构建和重新启动，对`alpine`容器没有任何影响。

## 4.如果容器依赖于另一个容器

现在让我们稍微更新一下我们的`docker-compose.yml`文件，使`ubuntu`容器依赖于`alpine`容器:

```java
version: "3.9"
services:
  ubuntu:
    image: "ubuntu:latest"
    tty: true
    depends_on:
      - "alpine"
  alpine:
    image: "alpine:latest"
    tty: true
```

我们将**停止以前的容器，并使用新配置从头开始重建它们**:

```java
$ docker-compose stop
Container {folder-name}-alpine-1  Stopping
Container {folder-name}-ubuntu-1  Stopping
Container {folder-name}-ubuntu-1  Stopped
Container {folder-name}-alpine-1  Stopped

$ docker-compose up -d
Container {folder-name}-alpine-1  Created
Container {folder-name}-ubuntu-1  Recreate
Container {folder-name}-ubuntu-1  Recreated
Container {folder-name}-alpine-1  Starting
Container {folder-name}-alpine-1  Started
Container {folder-name}-ubuntu-1  Starting
Container {folder-name}-ubuntu-1  Started
```

在这种情况下，我们需要**添加`no-deps`选项来明确告诉`docker-compose`不要重启链接的容器**:

```java
$ docker-compose up -d --force-recreate --build --no-deps ubuntu
Container {folder-name}-ubuntu-1  Recreate
Container {folder-name}-ubuntu-1  Recreated
Container {folder-name}-ubuntu-1  Starting
Container {folder-name}-ubuntu-1  Started
```

## 5.结论

在本教程中，我们已经看到了如何用`docker-compose`重建一个容器。

和往常一样，代码可以在 GitHub 的[上获得。](https://web.archive.org/web/20221024150313/https://github.com/eugenp/tutorials/tree/master/docker-modules/docker-compose)