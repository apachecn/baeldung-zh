# 如何在 Docker 中更改目录

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/docker-change-directory>

## 1.概观

Docker 映像包含一组顺序指令，用作构建容器的模板。在本教程中，**我们将学习如何在构建 Docker 映像或使用映像运行容器时更改目录。**

## 2.使用`WORKDIR`指令

首先，让我们从使用现成的`ubuntu:latest` [图像](/web/20221030130212/https://www.baeldung.com/ops/docker-images-vs-containers#docker-images)生成 Docker 容器开始:

```java
$ docker run -it ubuntu:latest
[[email protected]](/web/20221030130212/https://www.baeldung.com/cdn-cgi/l/email-protection):/# pwd
/
```

我们可以看到，容器一启动，当前目录就被设置为`/.`

接下来，假设我们想在容器启动时将这个目录更改为`/tmp`。我们可以通过在使用`ubuntu:latest`作为基础图像的自定义图像中使用`WORKDIR`指令来实现这一点:

```java
$ cat custom-ubuntu-v1.dockerfile
FROM ubuntu:latest
WORKDIR /tmp
```

在使用这个映像运行容器之前，我们需要构建这个映像。因此，让我们继续构建`custom-ubuntu:v1`图像:

```java
$ docker build -t custom-ubuntu:v1 - < ./custom-ubuntu-v1.dockerfile
```

最后，让我们[使用`custom-ubuntu:v1`映像运行一个容器](/web/20221030130212/https://www.baeldung.com/ops/docker-images-vs-containers#running-images),并验证当前目录:

```java
$ docker run -it custom-ubuntu:v1
[[email protected]](/web/20221030130212/https://www.baeldung.com/cdn-cgi/l/email-protection):/tmp# pwd
/tmp
```

看起来我们做对了！

## 3.使用`–workdir`选项

使用 [`WORKDIR`](https://web.archive.org/web/20221030130212/https://docs.docker.com/develop/develop-images/dockerfile_best-practices/#workdir) 指令是大多数情况下我们希望在构建 Docker 映像时更改目录的推荐做法。然而，**如果我们的用例仅限于在运行容器时更改目录，那么我们可以通过使用`–workdir`选项**来实现:

```java
$ docker run --workdir /tmp -it ubuntu:latest
[[email protected]](/web/20221030130212/https://www.baeldung.com/cdn-cgi/l/email-protection):/tmp# pwd
/tmp
```

看到这里，我们可以欣赏这个命令的简洁，以及在这种情况下我们不必创建自定义图像的事实。

## 4.使用`cd`命令

在 Linux 中， [`cd`命令](/web/20221030130212/https://www.baeldung.com/linux/cd-command-bash-script)是大多数用例改变目录的标准方式。同样，当使用一些 docker 指令如`RUN`、`CMD`和`ENTRYPOINT`时，我们可以使用`cd`命令来改变上下文中当前命令的目录。

让我们从编写`custom-ubuntu-v2.dockerfile`开始，通过`cd`命令使用`RUN`指令:

```java
FROM ubuntu:latest
RUN cd /tmp && echo "sample text" > data.txt
```

我们可以看到其意图是将“样本文本”写入到`/tmp/data.txt`文件中。

接下来，让我们添加`ENTRYPOINT`指令来运行`bash`作为容器启动时的默认命令。此外，我们使用`cd`命令将当前目录更改为`/tmp`目录:

```java
ENTRYPOINT ["sh", "-c", "cd /tmp && bash"]
```

接下来，让我们构建自定义图像:

```java
$ docker build -t custom-ubuntu:v2 - < ./custom-ubuntu-v2.dockerfile
```

最后，让我们使用`custom-ubuntu:v2`映像运行容器，并验证命令的执行:

```java
$ docker run -it custom-ubuntu:v2
[[email protected]](/web/20221030130212/https://www.baeldung.com/cdn-cgi/l/email-protection):/tmp# pwd
/tmp
[[email protected]](/web/20221030130212/https://www.baeldung.com/cdn-cgi/l/email-protection):/tmp# cat /tmp/data.txt
random text
```

我们可以看到两个更改目录命令的结果都符合预期。此外，我们必须记住`WORKDIR`仍然是推荐的方式。尽管如此，对于**简单的用例，我们可以将`cd`命令与`RUN`、[、`CMD`指令](/web/20221030130212/https://www.baeldung.com/ops/dockerfile-run-cmd-entrypoint)、**结合使用。

## 5.结论

在本文中，我们**学习了在使用 Docker 图像或启动容器**时改变目录的不同方法。