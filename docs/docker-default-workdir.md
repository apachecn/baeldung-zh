# docker 文件中的默认工作目录是什么？

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/ops/docker-default-workdir>

## 1.概观

在本教程中，我们将学习[docker 文件](/web/20221229073156/https://www.baeldung.com/category/docker/tag/dockerfile)中的`[WORKDIR](https://web.archive.org/web/20221229073156/https://docs.docker.com/develop/develop-images/dockerfile_best-practices/#workdir)`指令。我们将讨论它的作用、默认值以及如何在 does 文件中使用它。我们还将讨论它在 Dockerfile 文件创建中的重要性。

## 2.什么是`WORKDIR`指令

docker 文件中的`WORKDIR`指令为 docker 文件中的后续指令设置当前工作目录。**当执行`WORKDIR`指令时，Dockerfile 中所有后续指令都将相对于指定目录执行。**

例如，如果`WORKDIR`指令被设置为`/app`，那么所有后续指令都将相对于`/app`目录。`WORKDIR`指令切换到 [Docker](/web/20221229073156/https://www.baeldung.com/category/docker) 映像中的特定目录，如应用程序代码目录，以便更容易在后续指令中引用文件。

需要注意的是，`WORKDIR`指令只为 docker 文件中的后续指令设置当前工作目录。如果`WORKDIR`指令使用根目录的默认值(`/`)，但 Dockerfile 文件中没有后续指令，则不会对当前工作目录产生任何影响。

## 3.默认`WORKDIR`

如果 Dockerfile 文件中没有指定`WORKDIR`指令，默认的`WORKDIR`是根目录(`/`)。换句话说，如果 Dockerfile 中没有`WORKDIR`指令，那么所有后续指令都将相对于根目录执行。然而，我们可以通过在`WORKDIR`指令中指定一个不同的目录来覆盖这个默认值。

**注意，`WORKDIR`指令的默认值始终是根目录(`/`)，不管 [Docker 镜像](/web/20221229073156/https://www.baeldung.com/ops/docker-images-vs-containers)。**

`WORKDIR`指令的默认值会影响依赖于工作目录的命令的行为。例如，Dockerfile 中有一个命令使用`[cd](https://web.archive.org/web/20221229073156/https://www.gnu.org/software/bash/manual/html_node/Bourne-Shell-Builtins.html#index-cd)`命令来更改特定的目录。现在，如果我们不指定`WORKDIR`指令，那么 Dockerfile 将使用默认值“`/`”。现在，`cd`命令将尝试切换到 Dockerfile 中文件系统根目录下的那个目录。如果根目录中不存在该目录，该命令将失败。

## 4.改变`WORKDIR`

在 Docker 中，我们还可以更改容器的工作目录。我们可以使用`WORKDIR`指令为 [CMD/ `ENTRYPOINT`](/web/20221229073156/https://www.baeldung.com/ops/dockerfile-run-cmd-entrypoint) 指令中指定的命令指定工作目录。让我们看看如何在 docker 文件中使用`WORKDIR`:

```java
FROM centos 
WORKDIR /home
ENTRYPOINT ["sh", "-c", "pwd"]
```

在这种情况下，`WORKDIR`指令将工作目录设置为`/home`。`ENTRYPOINT`指令指定在容器的`/home`目录中执行 [`pwd`](https://web.archive.org/web/20221229073156/https://man7.org/linux/man-pages/man1/pwd.1.html) 命令。

或者，我们也可以在运行 Docker 容器时使用–`w`或`–workdir`选项来指定工作目录:

```java
$ docker run -w /home centos:latest sh -c "pwd"
```

这个 Docker run 命令将执行容器内`/home`目录中的`pwd`命令。

## 5.结论

在本文中，我们发现`WORKDIR`指令的默认值是“`/`”。一条`WORKDIR`指令在 Docker 映像中的不同目录之间切换。我们可以通过为`WORKDIR`指令指定一个自定义值来覆盖它。