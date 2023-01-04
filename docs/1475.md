# 在 Docker Compose 中执行多个命令

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/ops/docker-compose-multiple-commands>

## 1.概观

在 [Docker](/web/20221002153728/https://www.baeldung.com/ops/docker-guide) 中，开发人员可以通过将应用程序及其所有依赖项打包到一个容器中来构建、部署和测试应用程序。 [Docker Compose](/web/20221002153728/https://www.baeldung.com/ops/docker-compose) 是使用服务管理多个容器的基本工具。

在本教程中，我们将了解如何在 Docker 容器中执行多个命令，并使用 Docker Compose 进行管理。此外，我们将探索在 Docker 容器中运行多个命令的不同方式。

## 2.运行单个命令

Docker Compose 允许我们在 Docker 容器中执行命令。在容器启动期间，我们可以通过`[command](https://web.archive.org/web/20221002153728/https://docs.docker.com/compose/compose-file/#command)`指令设置任何命令。

让我们看一下`docker-compose.yml`，它在容器中运行一个简单的命令:

```
version: "3"
services:
 server:
   image: alpine
   command: sh -c "echo "baeldung""
```

在上面的`docker-compose.yml`文件中，我们正在执行`[alpine](https://web.archive.org/web/20221002153728/https://hub.docker.com/_/alpine)` Docker 映像中的单个 [`echo`](/web/20221002153728/https://www.baeldung.com/linux/echo-command) 命令。

## 3.运行多个命令

我们可以使用 Docker Compose 通过在`docker-compose.yml`文件中创建服务来管理多个应用程序。具体来说，**我们将使用`&&`和`|`操作符。**让我们来看看这两者。

### 3.1.使用`&&`操作符

我们将首先创建一个简单的`docker-compose.yml`文件来演示 Docker Compose 如何运行多个命令:

```
version: "3"
services:
 server:
   image: alpine
   command: sh -c "echo "baeldung" && echo "docker" "
```

这里，我们使用`alpine`作为 Docker 容器的基本图像。注意最后一行，我们执行了两个命令:`echo baeldung`和`echo docker`。它们被`&&`操作符分割。

为了演示结果，让我们使用 [`docker-compose up`](https://web.archive.org/web/20221002153728/https://docs.docker.com/engine/reference/commandline/compose_up/) 命令运行这个图像:

```
$ docker-compose up
Creating dockercompose_server_1 ... done
Attaching to dockercompose_server_1
server_1  | baeldung
server_1  | docker
dockercompose_server_1 exited with code 0
```

这里，两个`echo `语句的输出都打印在`stdout`上。

### 3.2.使用|运算符

我们还可以使用|操作符在 Docker Compose 中运行多个命令。`|`操作符的语法与`&&`操作符略有不同。

为了说明`|`操作符是如何工作的，让我们更新一下我们的`docker-compose.yml`:

```
version: "3"
services:
 server:
   image: alpine
   command:
      - /bin/sh
      - -c
      - |
        echo "baeldung"
        echo "docker"
```

这里，我们在单独的行上添加了命令。除了`command`指令，其他都一样。

推荐使用这种方法，因为它可以保持我们的 YAML 文件干净，因为所有的命令都在单独的一行中。

让我们用`docker-compose up`命令再次运行 Docker 容器:

```
$ docker-compose up
Creating dockercompose_server_1 ... done
Attaching to dockercompose_server_1
server_1  | baeldung
server_1  | docker
dockercompose_server_1 exited with code 0
```

从上面的输出中我们可以看到，两个命令都是一个接一个执行的。

## 4.结论

在本文中，我们演示了如何在 Docker Compose 中执行多个命令。首先，我们看到了如何使用`&&`操作符执行多个命令。后来，我们使用了`|`操作符来实现类似的结果。