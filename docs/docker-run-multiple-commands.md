# 在 Docker Run 中运行多个命令

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/ops/docker-run-multiple-commands>

## 1。概述

[Docker](/web/20221221193922/https://www.baeldung.com/ops/docker-guide) 是在隔离环境下打包应用的有用工具。它简化了在多个平台上部署应用程序的过程。[*docker run*](https://web.archive.org/web/20221221193922/https://docs.docker.com/engine/reference/commandline/run/)命令用于从 Docker 镜像启动一个新的容器。默认情况下， *docker 运行* 命令只执行[容器](/web/20221221193922/https://www.baeldung.com/ops/docker-container-shell)中的一条命令。但是，有些情况下我们可能需要在一个 *docker 中运行多个* 命令。

在本教程中，我们将讨论如何在 Docker 容器启动时运行多个命令。

## 2。使用 ***docker 运行*** **命令**

要执行 *docker 中的多个命令运行* 命令，我们可以使用[`&&`](/web/20221221193922/https://www.baeldung.com/linux/conditional-expressions-shell-script)运算符将这些命令链接在一起。**`&&`操作符执行第一条命令，如果成功，则执行第二条命令。**

例如，为了在一个单独的运行命令中运行 [`whoami`](/web/20221221193922/https://www.baeldung.com/linux/get-current-user) 和 `[date](/web/20221221193922/https://www.baeldung.com/linux/date-command)` 命令，我们可以使用下面的命令:

```
$ docker run centos:latest whoami && date
root
Sun Dec 18 10:09:30 UTC 2022 
```

在上面的输出中，我们可以看到`whoami`和`date`命令都以正确的顺序执行。

或者，**我们可以使用`-c`选项和****[shell](/web/20221221193922/https://www.baeldung.com/linux/sh-vs-bash)****同时执行多个命令**。`sh` `-c`选项允许我们传递一个包含多个命令的字符串作为参数。让我们使用`sh -c` : 运行`whoami`和`date`命令

```
$ docker run centos:latest sh -c "whoami && date"
root
Sun Dec 18 10:10:12 UTC 2022
```

在一个运行命令中同时执行`whoami`和`date`命令。我们也可以使用`;` 操作符和`sh`的`-c`选项来运行多个命令。此外，让我们使用`-w`选项来指定命令在 Docker 容器中执行的工作目录:

```
$ docker run -w /home centos:latest sh -c "whoami ; pwd"
root
/home
```

在这里，我们可以看到`whoami`和 [`pwd`](/web/20221221193922/https://www.baeldung.com/linux/run-script-different-working-dir) 命令都被执行，但是这次`/home`是 docker 容器的默认工作目录来执行这些命令。

## 3。使用`Dockerfile` 中的`CMD/ENTRYPOINT`

除了在 run 命令中运行多个命令外，**我们还可以在一个`[Dockerfile](/web/20221221193922/https://www.baeldung.com/category/docker/tag/dockerfile)`的** **[`CMD/ENTRYPOINT`](/web/20221221193922/https://www.baeldung.com/ops/dockerfile-run-cmd-entrypoint) 段中指定多个命令。**

`Dockerfile`的`CMD`和`ENTRYPOINT`定义了集装箱投放时执行的默认命令。如果我们向`ENTRYPOINT`和`CMD`部分添加多个命令，Docker 会按顺序运行它们。

让我们看看在`ENTRYPOINT`部分指定多个命令的`Dockerfile`:

```
FROM centos 
ENTRYPOINT ["sh", "-c", "whoami && date"]
```

为了运行容器，我们需要首先构建映像:

```
$ docker build -f Dockerfile -t baeldung_run .
Sending build context to Docker daemon  2.048kB
Step 1/2 : FROM centos
 ---> 5d0da3dc9764
Step 2/2 : ENTRYPOINT ["sh", "-c", "whoami && date"]
 ---> Running in dd027b5ba1e9
Removing intermediate container dd027b5ba1e9
 ---> a43dfa09d48b
Successfully built a43dfa09d48b
Successfully tagged baeldung_run:latest
```

让我们使用上面的 *baeldung_run:最新的* image:

```
$ docker run -itd --name baeldung_run baeldung_run 
b2a8ff012797d6110fd73dfffbf3c39e081f111dc50aac5d9d62fa73845b8a59
```

现在，我们可以通过查看*bael dung _ run*容器的日志文件来验证命令的执行:

```
$ docker logs -f baeldung_run
root
Sun Dec 18 10:13:44 UTC 2022
```

从上面的输出中我们可以看到，两个命令都成功执行了。

我们也可以使用`CMD`指令运行多个命令。让我们看看带有`CMD`指令的`Dockerfile`:

```
FROM centos 
CMD ["sh", "-c", "whoami && date"]
```

同样，我们需要首先创建 Docker 映像，然后运行一个容器。在容器日志中，我们将看到使用`ENRTYPOINT`指令生成的相同输出。

**我们应该注意到`CMD`和`ENTRYPOINT`指令不能互相替代。它们都有不同的用途。但是，在`ENTRYPOINT`和`CMD`中运行多个命令遵循相同的语法。**

## 4。结论

在本文中，我们已经讨论了如何在一个正在运行的 Docker 容器中运行多个命令。

首先，我们学习了使用`docker run`命令执行多个命令。之后，我们使用`Dockerfile`中的`ENTRYPOINT/CMD`指令进行了同样的探索。