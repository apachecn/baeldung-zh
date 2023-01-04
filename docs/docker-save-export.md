# Docker 保存和导出的区别

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/ops/docker-save-export>

## 1.介绍

Docker 生态系统有许多工具和特性，有时会令人困惑。在这篇短文中，我们将看看 Docker `save`和`export`命令之间的区别。

## 2.Docker 图像与容器

为了理解这两个命令的区别，我们必须首先理解 [Docker 图像和容器](/web/20220530121225/https://www.baeldung.com/ops/docker-images-vs-containers)的区别。

Docker 映像是一个包含运行应用程序所需的所有文件的文件。这包括所有的操作系统文件，以及应用程序代码和任何需要的支持库。

**Docker 容器是已经开始**的 Docker 映像。容器本质上是一个正在运行的应用程序。容器像正常进程一样消耗内存和 CPU 资源，还可以访问文件系统并通过网络协议与其他容器通信。

Docker 容器和图像类似于 Java 类和对象。Java 类是如何创建对象的蓝图，就像 Docker 图像是创建容器的蓝图一样。就像一个类可以被实例化成多个对象一样，Docker 映像可以用来启动多个容器。

考虑到这一点，我们可以仔细看看 Docker `save`和`export`命令之间的区别。

## 3.`docker save`

**Docker`save`命令用于将 Docker 图像保存到 tar 文件**。这个命令有助于将 Docker 映像从一个注册表移动到另一个注册表，或者使用 Linux [`tar`命令](/web/20220530121225/https://www.baeldung.com/linux/tar-command)简单地检查映像的内容。

默认情况下，该命令将 tar 文件内容打印到 STDOUT，因此典型的用法是:

```java
docker save IMAGE > /path/to/file.tar
```

注意，我们还可以指定一个文件来打印内容，这样就不需要重定向了:

```java
docker save -o /path/to/file.tar IMAGE 
```

在任一情况下，`IMAGE`参数可以是以下两个值之一:

*   完全限定的映像名称，例如“ghcr . io/bael dung/my-application:1 . 2 . 3”
*   Docker 生成的图像哈希，例如“c85146bafb83”

## 4.`docker export`

**Docker`export`命令用于将 Docker 容器保存到 tar 文件**。这包括图像文件以及容器运行时所做的任何更改。

语法与`save`命令完全相同。与`save`一样，`export`命令将输出发送到 STDOUT，因此我们必须将其重定向到一个文件:

```java
docker export CONTAINER > /path/to/file.tar
```

或者我们可以指定输出文件名:

```java
docker export -o /path/to/file.tar CONTAINER
```

在这两种情况下，`CONTAINER`参数可以是下列值之一:

*   容器名称，可以是自动生成的，也可以是容器启动时指定的
*   Docker 引擎分配的唯一容器哈希

## 5.差异

虽然这些命令在本质上是相似的，但还是有一些不同之处需要注意。这两个命令都生成 tar 文件，但是包含的信息是不同的。

**`save`命令保存图像层信息，包括所有历史和元数据**。这允许我们将 tar 文件完全导入到任何 Docker 注册表中，并使用它来启动新的容器。

相反，**`export`命令不保存这个信息**。它包含与启动容器的映像相同的文件，但没有历史和元数据。

此外，`export`命令包括容器运行时所做的更改，比如新的或修改的文件。这意味着**来自同一个映像的不同容器在导出它们的时候可能会产生不同的 tar 文件**。

## 6.结论

在本教程中，我们已经看到了 Docker `save`和`export`命令之间的区别。**虽然它们有相似的语法并创建 tar 文件，但它们有两个不同的目的**。`save`命令用于图像，而`export`命令用于容器。