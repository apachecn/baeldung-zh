# 一个项目中的多个 docker 文件

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/ops/multiple-dockerfiles>

## 1.介绍

码头工人是集装箱运输的便利工具。它非常有用，有时我们希望在项目中有不止一个 [Dockerfile](https://web.archive.org/web/20220524070211/https://docs.docker.com/engine/reference/builder/) 。不幸的是，这违背了将所有 docker 文件命名为“Dockerfile”的简单惯例。

在本教程中，我们将着眼于解决它，并保持一个干净的项目结构。

## 2\. Dockerfile

Dockerfile 是一个包含组装 Docker 映像所需的所有指令的文件。Docker 可以使用它自动构建图像，不需要任何额外的命令或参数。由于命名约定，我们甚至不需要(直到版本 1.8.0，我们实际上不能)指定文件的路径。

我们可以从 Docker 文件所在的目录中调用 Docker 的`build`命令:

```
$ docker build .
```

## 3.指定 Dockerfile 文件名

从版本 1.8.0 开始，**我们可以更改 Dockerfile 的名称，并使用`“-f”`参数**将其传递给`build`命令。假设我们有两个 docker 文件，一个用于构建后端，另一个用于构建前端。

我们可以适当地命名它们，并调用两次`build`命令，每次传递一个 docker 文件的名称:

```
$ docker build -f Dockerfile.frontend .
...
$ docker build -f Dockerfile.backend .
```

**这种解决方案很好，但有一些不便之处**。第一个是我们需要为每个文件分别调用`build`命令。这对于两个文件来说并不可怕，但是如果我们有更多的组件，可能会很快产生对额外构建脚本的需求。第二个缺点是，一些 ide 可能会偏离常规，停止提供语法突出显示。

## 4.使用`docker-compose`

**不用改变 Dockerfiles 的名字，我们可以把它们放在单独的文件夹里。**然后，我们可以使用 [`docker-compose`](https://web.archive.org/web/20220524070211/https://baeldung.com/ops/docker-compose) 来触发他们所有人的构建。假设我们有这样一个目录结构:

```
docker-compose.yml
docker
├── frontend
│   └── Dockerfile
└── backend
    └── Dockerfile
```

虽然`docker-compose`文件的最基本用法通常意味着使用存储库中的图像，但是我们也可以为`build`提供目录路径:

```
version: '3'
services:
  frontend:
    build: ./docker/frontend
    ports:
     - "8081:8081"
  backend:
    build: ./docker/backend
    ports:
      - "8080:8080"
```

现在运行`docker-compose`将从 docker 文件构建图像:

```
$ docker-compose up
```

此外，在一个 `docker-compose`文件中，我们可以用不同的方式创建一个图像。有些可以从我们的源代码动态构建，有些可以由外部注册中心提供。例如，我们可能希望构建后端映像，但是从公共注册中心获取数据库映像:

```
services:
  backend:
    build: ./docker/backend
    ports:
      - "8080:8080"
  postgres:
    image: 'postgres:latest'
```

现在，`docker-compose`不仅会构建我们的服务并运行它们，还会为我们提供一个数据库。

## 5.结论

在本教程中，我们学习了在一个项目中处理多个 docker 文件的两种不同策略。

第一个是基于更改不同 docker 文件的名称，并为该问题提供了一个快速而优雅的解决方案。第二个依赖于`docker-compose` ，并给出了构建和自动化构建过程的附加选项。