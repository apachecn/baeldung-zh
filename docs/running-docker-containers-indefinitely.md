# 无限期运行 Docker 容器

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/ops/running-docker-containers-indefinitely>

## 1.概观

在本教程中，我们将探索保持 Docker 容器无限期运行的方法。

默认情况下，容器只在默认命令执行期间运行，但是一个常见的用例是为了调试和故障排除的目的无限期地运行它们。

## 2.码头运行基础

让我们看看`docker run `命令的一些基础知识，以及在启动时将命令传递给容器的方法。

### 2.1.指定要运行的命令

创建 Dockerfile 文件时，有两种方法可以指定要运行的命令。

*   **[`ENTRYPOINT`指令](/web/20220629135738/https://www.baeldung.com/ops/dockerfile-run-cmd-entrypoint#the-entrypoint-command)指定了容器**的命令。这对于我们总是想运行的命令很有帮助，除非用户显式地覆盖它们。
*   我们也可以使用 [`CMD`指令](/web/20220629135738/https://www.baeldung.com/ops/dockerfile-run-cmd-entrypoint#the-cmd-command)来指定命令。这用于定义要传递给容器的默认命令或参数。**用户在图像名称后包含的任何参数都将替换整个`CMD`指令。**

### 2.2.覆盖默认命令

要覆盖 CMD 指令，我们可以简单地在`docker run [image_name]`后添加另一个命令。例如:

```java
docker run ubuntu echo "Hello World"
```

要覆盖 ENTRYPOINT 指令，我们需要在图像名之前添加`–entrypoint`标志和所需的命令，并在图像名之后添加任何参数。例如:

```java
docker run --entrypoint echo ubuntu "Hello World"
```

两个例子都将在容器启动时运行命令`echo “Hello World”` 。

### 2.3.Docker 用命令运行

`docker run`命令的默认行为可以总结如下:

*   默认情况下，容器在前台运行，除非使用`-d`标志显式分离。
*   只要指定的命令一直运行，容器就会运行，然后停止。

让我们看一个小例子:

```java
docker run ubuntu bash 
```

上面的命令将运行`ubuntu`图像中的`bash`命令。`bash`命令将在容器中启动一个 shell。

命令运行，但是容器在命令完成后停止，这几乎是立即完成的。我们可以使用`docker ps`命令对此进行测试:

```java
docker ps -a
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS               NAMES
c8f8f8f8f8f8        ubuntu              bash                2 minutes ago      Exited (0)          22/tcp               mystifying_snyder 
```

## 3.无限期运行 Docker 容器

默认情况下，容器可能会也可能不会被设计为无限期运行。例如，只要 web 服务器在运行，包含 web 服务器的容器就会运行。但是，包含一次性作业(如 cron 作业)的容器仅运行很短一段时间。

让我们看看无限期运行容器的一些方法。

### 3.1.永不停止的命令

保持容器运行的最简单方法是传递一个永不结束的命令。

这里有几个例子:

我们可以使用`tail -f `命令来读取`/dev/null`文件。该命令一直在文件中寻找要显示的新更改。因此，只要文件存在，它就不会结束。让我们来看看这个命令:

```java
docker run ubuntu tail -f /dev/null 
```

我们可以使用下面的命令运行一个什么也不做的无限循环:

```java
docker run ubuntu while true; do sleep 1; done
```

下面的命令使容器保持空闲，不做任何事情:

```java
docker run ubuntu sleep infinity 
```

我们可以通过以下任何方式使用永不停止的命令:

*   docker 文件中的 ENTRYPOINT 或 CMD 指令。
*   覆盖 docker run 命令中的 ENTRYPOINT 或 CMD。

此外，在前台运行一个永无止境的命令并得到一个卡住的终端是没有意义的。在这种情况下，**我们可以使用`-d`标志在后台运行容器**。

### 3.2.开始一个伪 TTY

永无止境的命令在很久以前是一种黑客行为，当时没有其他选项存在。然而，在 Docker 的最新版本中，可以通过在前台和后台启动与容器的终端会话来保持容器的运行。

伪 tty 用于在容器内部运行命令。**要启动与容器的伪 TTY 会话，我们可以使用`-t`标志。**直到会话结束，容器才会退出。

如果我们想与容器交互，我们可以将它与`-i`标志结合起来。这将允许我们使用终端在容器中运行命令。下面是该命令的一个示例:

```java
docker run -it ubuntu bash 
```

或者，如果我们的目的只是无限期地运行容器，我们可以使用`-d`标志:

```java
docker run -d -t ubuntu 
```

## 4.结论

在本教程中，我们介绍了一些无限期运行 Docker 容器的方法。我们研究了将命令传递给容器的方式，然后修改这些命令来解决我们的问题。我们还探索了使用伪 TTY 会话来保持容器运行的现代解决方案。