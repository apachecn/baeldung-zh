# docker:“build”需要 1 个参数错误

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/ops/docker-build-argument-error>

## 1.概观

[Docker](/web/20220919051541/https://www.baeldung.com/tag/docker/) 是一个开源的容器平台。它允许我们将应用程序打包到容器中，并将结合应用程序源代码和操作系统的可执行组件标准化。它是一个用于构建、共享和运行单个容器的软件开发工具包。

[Dockerfile](https://web.archive.org/web/20220919051541/https://docs.docker.com/engine/reference/builder/) 是一个文本文件，包含一系列可用于构建图像的命令。这是自动创建图像的最简单的方法。

**docker file 的一个好处就是我们只需要编写相当于 Linux shell 命令的命令，所以不需要为它学习任何新的语法。**

在使用 docker 文件构建映像时，我们可能会面临不同的问题。在本教程中，我们将学习解决一个非常常见的 [Docker 构建](https://web.archive.org/web/20220919051541/https://docs.docker.com/engine/reference/commandline/build/)问题。

我们先来理解错误“Docker 构建需要 1 个参数”。

## 2.理解问题

在这一节中，首先，我们 ' ll 创建一个示例 Dockerfile 来重现错误然后 使用 不同的 方法 到 解决 it 。 让s 创建一个新的 Dockerfile，命名为docker file，内容如下:

```
FROM        centos:7
RUN         yum -y install wget \
            && yum -y install unzip \
            && yum install -y nc \
            && yum -y install httpd && \
            && yum clean all
EXPOSE      6379
ENTRYPOINT  ["ping"]
CMD  ["google.com"]
```

这里，我们编写了一个示例 Dockerfile 文件，它使用“centos:7 `”` 作为基本图像。因此，它支持`centos`的所有命令。

我们还在 Docker 文件中安装了 Docker 的其他一些实用命令。

重现问题“Docker 构建需要 1 个参数”的命令是:

```
$ docker build
"docker build" requires exactly 1 argument.
See 'docker build --help'.
Usage:  docker build [OPTIONS] PATH | URL | -
Build an image from a Dockerfile
```

如果我们使用带有不同选项的 Docker build 命令，也会出现此问题。

让我们探索其中一个选项:

```
$ docker build -t test_image/centos
"docker build" requires exactly 1 argument.
See 'docker build --help'.
Usage:  docker build [OPTIONS] PATH | URL | -
Build an image from a Dockerfile
```

在这两种情况下，我们都面临着“Docker 构建需要 1 个参数”的问题。

## 3.建立码头工人形象的不同方法

在我们解决错误“Docker build 需要 1 个参数”之前，让我们先了解一下使用不同选项的 Docker build 命令。

Docker 构建由 Docker 守护进程运行。首先，Docker build 将整个上下文发送给守护进程。**理想的情况是从一个空目录开始，只添加构建 Docker 映像所需的文件，使用 Dockerfile:**

```
$ docker build .
Sending build context to Docker daemon  2.048kB
Step 1/4 : FROM        centos:7
14.04: Pulling from library/centos
2e6e20c8e2e6: Extracting [============>                                      ]  17.83MB/70.69MB
0551a797c01d: Download complete 
512123a864da: Download complete 
```

我们还可以使用`-f`标志通过从特定位置指向 Dockerfile 文件来构建图像:

```
$ docker build -f /root/dockerImage/Dockerfile .
```

为了给图像加标签，我们可以在 Docker build 命令中使用`-t`选项:

```
$ docker build -t test_image/centos .
```

我们还可以用 Docker build 命令传递构建参数:

```
$ docker build -t test_image/centos --build-arg JAVA_ENV=1.8  .
```

要清理图像的构建，我们可以在命令中使用`–no-cache`选项:

```
$ docker build -t test_image/centos --build-arg JAVA_ENV=1.8  --no-cache .
```

在 Docker 的旧版本中，我们需要通过`–no-cache=true`，但在新版本中却不是这样。我们还可以创建一个 Dockerfile，而不用 Dockerfile `:`作为文件名

```
$ docker build -f /root/dockerImage/DockerFile_JAVA .
```

这里我们使用`DockerFile_JAVA`文件名创建了一个 Docker 图像。

## 4.由于论据不足

“Docker 构建需要 1 个参数”错误最常见的原因是我们试图在没有提供足够参数的情况下构建图像。这里，在参数中，我们需要用命令提供目录。

默认情况下，我们提供了点(。)在指定 Docker 守护进程使用 shell 的当前工作目录作为构建上下文的命令中:

```
$ docker build .
```

圆点(。)基本上告诉 Docker 必须从当前目录使用 Dockerfile。我们还可以使用以下命令更改 Docker 构建上下文:

```
$ docker build /root/test
```

我们可能面临的另一个与 Docker 构建相关的问题是:

```
$ docker build -f /root/test/Dockerfile2 .
unable to prepare context: unable to evaluate symlinks in Dockerfile path:
  lstat /root/test/Dockerfile2: no such file or directory
```

在上面的命令中，我们试图使用文件名“Dockerfile2”构建一个 Docker 映像，如果这个文件不在当前目录中，我们将得到以下错误:

```
unable to prepare context: unable to evaluate symlinks in Dockerfile path:
  lstat /root/test/Dockerfile2: no such file or directory
```

为了解决这个问题，我们需要用`-f`选项提供正确的文件名。

## 5.结论

在本教程中，我们了解了与 Docker build 命令相关的问题。

最初，我们探索了构建 Docker 映像的不同方法，然后我们解决了 Docker build 参数的问题，最后，我们了解了与 Docker build 命令相关的一些问题。