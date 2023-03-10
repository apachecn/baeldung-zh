# Dockerfile 文件中复制和添加的区别

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/ops/docker-copy-add>

## 1.介绍

创建 Docker 文件时，通常需要将文件从主机系统传输到 Docker 映像中。这些可能是属性文件、本地库或我们的应用程序在运行时需要的其他静态内容。

**docker file 规范提供了两种将文件从源系统复制到映像**的方法:`COPY`和`ADD`指令。

在本文中，我们将看看它们之间的区别，以及什么时候使用它们是有意义的。

## 2.`COPY`和`ADD`的区别

乍一看，`COPY`和`ADD`指令看起来是一样的。它们有相同的语法:

```java
COPY <source> <destination>
ADD <source> <destination>
```

并且都将文件从主机系统复制到 [Docker 映像](/web/20220727020704/https://www.baeldung.com/ops/efficient-docker-images)。

那么有什么区别呢？**总之，`ADD`指令比`COPY`更有能力。**

虽然功能相似，但`ADD`指令在两个方面更强大:

*   它可以处理远程 URL
*   它可以自动提取 [tar 文件](/web/20220727020704/https://www.baeldung.com/linux/tar-command)

让我们更仔细地看看这些。

首先，`ADD`指令可以接受一个远程 URL 作为它的`source`参数。另一方面，`COPY`指令只能接受本地文件。

**注意，使用`ADD`获取远程文件并复制通常不是理想的**。这是因为该文件会增加整个 Docker 图像的大小。相反，我们应该使用 [`curl`或`wget`](/web/20220727020704/https://www.baeldung.com/linux/curl-wget) 来获取远程文件，并在不再需要时删除它们。

第二，**`ADD`指令会自动将 tar 文件扩展到镜像文件系统**。虽然这可以减少构建映像所需的 Dockerfile 步骤的数量，但并不是所有情况下都需要这样做。

请注意，仅当源文件位于主机系统本地时，才会发生自动扩展。

## 3.何时使用`ADD`或`COPY`

根据 [Dockerfile 最佳实践指南](https://web.archive.org/web/20220727020704/https://docs.docker.com/develop/develop-images/dockerfile_best-practices/#add-or-copy)，我们应该总是**更喜欢`COPY`而不是`ADD`，除非我们特别需要`ADD`的两个附加特性之一**。

如上所述，使用`ADD`将远程文件复制到 Docker 映像会创建一个额外的层并增加文件大小。如果我们使用`wget`或`curl`来代替，我们可以在之后删除这些文件，它们不会成为 Docker 映像的永久部分。

此外，由于`ADD`命令会自动扩展 tar 文件和某些压缩格式，这可能会导致意外的文件被写入我们映像中的文件系统。

## 4.结论

在这个快速教程中，我们看到了将文件复制到 Docker 映像的两种主要方法:`ADD`和`COPY`。虽然功能相似，但大多数情况下还是首选`COPY`指令。这是因为`ADD`指令提供了额外的功能，应该谨慎使用并且只在需要的时候使用。