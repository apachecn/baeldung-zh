# Docker 中的标签指南

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/ops/docker-tag>

## 1.概观

在本教程中，我们将学习 [Docker](/web/20220727020745/https://www.baeldung.com/ops/docker-guide) 中标签的概念。

Docker 为在 [Docker Hub 存储库](https://web.archive.org/web/20220727020745/https://hub.docker.com/)上存储图像提供支持。Docker 标签为 Docker 图像提供了唯一的身份。Docker 储存库中有由标签标识的不同版本的相似图像集。

在这里，我们将学习使用`docker build`和`docker tag`命令来标记图像。

## 2.了解 Docker 标签

Docker 标签帮助维护构建版本，以将映像推送到 Docker Hub **。Docker Hub 允许我们根据名称和标签将图像分组。**多个 Docker 标签可以指向一个特定的图片。基本上，和在 Git 中一样，Docker 标签类似于特定的提交。Docker 标签只是图像 ID 的别名。

标记名必须是 ASCII 字符串，可以包含小写和大写字母、数字、下划线、句点和破折号。此外，标记名不得以句点或破折号开头，并且只能包含 128 个字符。

## 3.使用 Docker 标签构建图像

在我们继续之前，让我们首先创建一个示例 docker 文件来演示标记:

```java
FROM centos:7
RUN yum -y install wget 
RUN yum -y install unzip 
RUN yum -y install java-1.8.0-openjdk 
RUN yum clean all
ENV JAVA_HOME /usr/lib/jvm/java-8-openjdk-amd64/
RUN export JAVA_HOME
```

在上面的 Dockerfile 文件中，我们使用`“centos:7”`作为基础映像运行所有必要的命令来安装 java。

### 3.1.用单个 Docker 标签构建图像

在 Docker 中，我们可以在构建时标记图像。为了说明这一点，我们来看看标记图像的命令:

```java
$ docker build -t baeldung-java:5 .
Sending build context to Docker daemon  2.048kB
Step 1/2 : FROM centos:7
 ---> eeb6ee3f44bd
Step 2/2 : RUN yum -y install wget
 ---> Using cache
 ---> 46ee47a7422d
Successfully built 46ee47a7422d
Successfully tagged baeldung-java:5
```

这里，在上面的命令中，我们提供了“`baeldung-java:5`”作为 Docker 图像的标签。**Docker 中的标签对于维护 build 的版本以将映像推送到 DockerHub 很有用。**版本控制通常用于部署任何 Docker 映像或回到旧版本。

我们还可以使用以下语法提供带有用户名和图像名称的标记:

```java
$ docker build -t baeldung/baeldung-java:5 .
Sending build context to Docker daemon  2.048kB
Step 1/2 : FROM centos:7
 ---> eeb6ee3f44bd
....
Successfully built 46ee47a7422d
Successfully tagged baeldung/baeldung-java:5
```

这里，在上面的命令中，我们为用户名`“baeldung” `提供了图像名`baeldung-java`，标记为`5`。

### 3.2.用多个 Docker 标签构建图像

**在 Docker 中，我们还可以给一张图片分配多个标签。**这里，我们将使用`docker build`命令在一个命令中给一个图像分配多个标签。

为了进行演示，我们来看看上面 docker 文件的命令:

```java
$ docker build -t baeldung-java:5 -t baeldung-java:6 .
Sending build context to Docker daemon  2.048kB
Step 1/2 : FROM centos:7
 ---> eeb6ee3f44bd
....
Successfully built 46ee47a7422d
Successfully tagged baeldung-java:5
Successfully tagged baeldung-java:6
```

在这里，我们可以看到为`imageId``46ee47a7422d`创建了两个标签`“baeldung-java:5”`和`“baeldung-java:6”`。

### 3.3.构建没有任何标签的图像

我们还可以不使用任何标签来构建 Docker 图像。但是为了跟踪图像，我们应该始终提供一个带有图像名称的标签。让我们来看看构建没有标签的图像的命令:

```java
$ docker build -t baeldung-java .
Sending build context to Docker daemon  2.048kB
Step 1/2 : FROM centos:7
 ---> eeb6ee3f44bd
...
Successfully built 46ee47a7422d
Successfully tagged baeldung-java:latest
```

这里，在上面的命令中，我们构建了没有任何标签的图像，所以默认情况下，Docker 为图像提供了一个标签作为最新的"`baeldung-java:latest”`。

Docker 总是使用最新的标签指向最新的稳定版本。旧版本甚至可以称为最新版本。但我们无法预测它是主要版本还是次要版本。

## 4.使用`docker tag`命令标记图像

到目前为止，我们已经讨论了使用`docker build`命令标记图像。但是我们也可以使用`[docker tag](https://web.archive.org/web/20220727020745/https://docs.docker.com/engine/reference/commandline/tag/)`命令显式标记图像。给图像添加标签只是为图像名称或`imageId`创建一个别名。在这里，我们将探索标记图像的两种方法。

Docker 图像名称的一般格式如下:

```java
<user-name>/<image-name>:<tag-name>
```

在上面的代码片段中，冒号后面的组件表示附加到图像的标签。

让我们看看使用图像名称标记图像的命令:

```java
$ docker tag baeldung-java:6 baeldung-java:8 
```

使用`imageId`标记图像的命令如下:

```java
$ docker tag 46ee47a7422d baeldung-java:9 
```

让我们来看看到目前为止创建的所有图像:

```java
$ docker images
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
baeldung-java       5                   46ee47a7422d        13 minutes ago      370MB
baeldung-java       6                   46ee47a7422d        13 minutes ago      370MB
baeldung-java       8                   46ee47a7422d        13 minutes ago      370MB
baeldung-java       9                   46ee47a7422d        13 minutes ago      370MB
baeldung-java       latest              46ee47a7422d        13 minutes ago      370MB
centos              7                   eeb6ee3f44bd        7 months ago        204MB
```

在这里，我们将找到迄今为止创建的所有图像。

## 5.在`docker pull`命令中使用标签

Docker 标签在创建图像或从 Docker Hub 存储库中提取图像时非常有用。
在我们的 docker 文件中，我们使用了命令`FROM centos:7.`这将拉版本`7″`的`centos`公共图像。

我们也可以提取带有或不带有标签的图像。

让我们研究一个带有特定标记的命令:

```java
$ docker pull centos:7
```

`docker pull`没有任何标记的命令:

```java
$ docker pull centos
```

上面的命令将从公共 Docker Hub 存储库中提取`“centos:latest”`图像。我们还可以对一个图像应用多个标签，通常用来指定主要版本和次要版本。

## 6.结论

在本文中，我们学习了如何在 Docker 中创建和管理标签。我们探索了标记图像的各种方法。

使用`docker build`命令，我们首先标记一个图像。后来，我们研究了`docker tag`命令。此外，我们探索了使用标签的`docker pull`命令。