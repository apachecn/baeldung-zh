# 减少 Docker 构建命令的构建上下文

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/ops/docker-reduce-build-context>

## 1.概观

[Docker](/web/20220524165945/https://www.baeldung.com/tag/docker/) 构建上下文由位于特定路径或 URL 的文件或文件夹组成。在构建期间，这些文件被发送到 Docker 守护进程，以便映像可以将它们作为文件使用。

在本教程中，我们将了解 Docker 构建上下文以及与之相关的问题。我们将探索各种方法来减少构建映像的构建上下文。

## 2.了解 Docker 构建上下文

在我们继续讨论这个问题之前，让我们先了解一下 Docker 构建环境及其工作原理。Docker 构建上下文是当我们运行`docker build`时 Docker 引擎可以访问的文件和目录的集合，任何不属于构建上下文的内容都不能被我们的[Docker 文件](https://web.archive.org/web/20220524165945/https://docs.docker.com/engine/reference/builder/)中的命令访问。

在某些情况下，Docker CLI 和 Docker 引擎可能不在同一台机器上运行。在`docker build`期间，Docker CLI 将文件发送到 Docker 引擎以构建映像。

我们还可以使用 GIT 存储库的 URL 作为构建上下文。因此，构建上下文成为指定 git 存储库的内容。

## 3.理解问题

在构建映像时，我们将理解与构建上下文相关的问题。为了进一步理解构建上下文的问题，让我们举一个例子并创建一个 docker 文件示例:

```java
FROM   centos:7
MAINTAINER [[email protected]](/web/20220524165945/https://www.baeldung.com/cdn-cgi/l/email-protection)
COPY jdk-8u202-linux-x64.rpm / 
```

要构建映像，我们需要运行以下命令:

```java
$ docker build -t javaapplication .
```

上述命令的输出:

```java
Sending build context to Docker daemon  178.4MB
Step 1/3 : FROM   centos:7
 ---> eeb6ee3f44bd
Step 2/3 : MAINTAINER [[email protected]](/web/20220524165945/https://www.baeldung.com/cdn-cgi/l/email-protection)
 ---> Using cache
 ---> 4c798858cf11
Step 3/3 : COPY jdk-8u202-linux-x64.rpm /
 ---> Using cache
 ---> 9c58f775bb80
Successfully built 9c58f775bb80
Successfully tagged test:latest
```

现在让我们将大小为`186M`的`jdk-8u202-linux-x64.tar.gz`放入同一个目录中，而不在 Dockerfile 文件中进行更改。

在运行`docker build`命令时，我们将得到以下输出:

```java
Sending build context to Docker daemon  372.5MB
Step 1/3 : FROM   centos:7
 ---> eeb6ee3f44bd
Step 2/3 : MAINTAINER [[email protected]](/web/20220524165945/https://www.baeldung.com/cdn-cgi/l/email-protection)
 ---> Using cache
 ---> 4c798858cf11
Step 3/3 : COPY jdk-8u202-linux-x64.rpm /
 ---> Using cache
 ---> 9c58f775bb80
Successfully built 9c58f775bb80
Successfully tagged test:latest
```

这里我们可以看到 Docker 守护进程的构建上下文从`178.4MB`增加到了`372.5MB`，尽管我们没有在 Docker 文件中做任何更改。

Docker 构建上下文的一个要点是，它递归地包含当前工作目录的所有文件和文件夹，并将它们发送到 Docker 守护进程。

## 4.解决方案使用。`dockerignore`文件

以便减少 Docker 构建上下文。我们可以使用`.dockerignore`文件来解决构建上下文问题。

**Docker CLI 在上下文的根目录中搜索名为`.dockerignore`的文件，然后将其发送到 Docker 守护进程。**如果文件存在，CLI 将排除与其模式匹配的目录和文件。这将防止大文件或敏感目录被不必要地发送到 Docker 守护进程。

这里，我们将把`jdk-8u202-linux-x64.tar.gz`文件添加到`.dockerignore` 文件中，以便在创建构建时忽略它:

```java
$ echo "jdk-8u202-linux-x64.tar.gz" > .dockerignore
```

现在让我们构建 Docker 映像:

```java
$ docker build -t baeldung  .
Sending build context to Docker daemon  178.4MB
Step 1/3 : FROM   centos:7
 ---> eeb6ee3f44bd
```

在这里，我们可以清楚地看到发送到 Docker 守护进程的 Docker 构建上下文已经从`372.5MB`减少到了`178.4MB.`

## 5.使用 EOF 文件创建

我们将使用带有 EOF 的`docker build`命令直接创建一个 Dockerfile。

让我们假设以下 Dockerfile 文件:

```java
FROM   centos:7
MAINTAINER [[email protected]](/web/20220524165945/https://www.baeldung.com/cdn-cgi/l/email-protection)
RUN echo "Welcome to Bealdung"
```

为上面的 docker 文件构建映像。我们运行`docker build`命令，得到以下输出:

```java
$ docker build -t baeldung .
Sending build context to Docker daemon  372.5MB
Step 1/3 : FROM   centos:7
 ---> eeb6ee3f44bd
Step 2/3 : MAINTAINER [[email protected]](/web/20220524165945/https://www.baeldung.com/cdn-cgi/l/email-protection)
 ---> Using cache
 ---> a7088e6a3e53
Step 3/3 : RUN echo "Welcome to Bealdung"
 ---> Using cache
 ---> 1fc84e62de75
Successfully built 1fc84e62de75
Successfully tagged baeldung:latest
```

这里，我们可以看到发送给 Docker 守护进程的 Docker 构建上下文是`372.5MB`。如果我们使用以下方式运行同一个 Dockerfile 文件:

```java
$ docker build -t test -<<EOF
FROM   centos:7
MAINTAINER [[email protected]](/web/20220524165945/https://www.baeldung.com/cdn-cgi/l/email-protection)
RUN echo "Welcome to Bealdung"
EOF
```

上述命令的输出如下:

```java
Sending build context to Docker daemon  2.048kB
Step 1/3 : FROM   centos:7
 ---> eeb6ee3f44bd
Step 2/3 : MAINTAINER [[email protected]](/web/20220524165945/https://www.baeldung.com/cdn-cgi/l/email-protection)
 ---> Using cache
 ---> a7088e6a3e53
Step 3/3 : RUN echo "Welcome to Bealdung"
 ---> Running in 6240c486fcb5
Welcome to Bealdung
Removing intermediate container 6240c486fcb5
 ---> 1fc84e62de75
Successfully built 1fc84e62de75
Successfully tagged baeldung:latest
```

在这里，我们可以看到构建上下文已经从`372 MB`减少到`2.048KB.` **这个解决方案只适用于我们不使用任何复制和添加命令的情况。**

## 6.优化 Docker 图像

现在让我们讨论优化现有 Docker 映像的各种方法。开发图像最具挑战性的方面是保持图像尽可能小。docker 文件中的每一条指令都向图像添加了一层，我们应该在继续下一层之前尝试移除任何不想要的伪像。

为了编写一个真正高效的 Dockerfile，我们传统上需要使用 shell 技巧和其他逻辑来保持层尽可能小，并确保每一层都有它需要的来自前一层的工件，而没有其他东西。

### 6.1.使用多阶段构建

在多阶段构建中，我们在一个 Dockerfile 文件中使用多个 FROM 语句。每个 FROM 指令使用不同的基础，每个 FROM 指令开始构建的新阶段。人工制品可以有选择地从一个阶段复制到另一个阶段。

让我们看一个在 docker 文件中使用多阶段的例子:

```java
FROM centos:7 as builder
RUN yum -y install maven
COPY spring-boot-application /spring-boot-application
WORKDIR /spring-boot-application/
RUN mvn clean package -Dmaven.test.skip=true
FROM centos:7
RUN yum install -y epel-release \
    && yum install -y maven wget \
    && yum -y install java-1.8.0-openjdk \
    && yum clean all 
COPY --from=builder /spring-boot-application/target/spring-boot-application-0.0.1-SNAPSHOT.jar /
CMD ["java -jar ","-c","/spring-boot-application-0.0.1-SNAPSHOT.jar && tail -f /dev/null"]
```

这里，在上面的 Dockerfile 文件中，我们使用了两个`From`语句。首先，我们在第一层使用 [`mvn`](/web/20220524165945/https://www.baeldung.com/maven) 命令构建了`spring-boot-application-0.0.1-SNAPSHOT.jar`。然后我们使用`“COPY –from=builder”`命令在第二层使用相同的 jar 文件。这样我们就可以去掉第一层，大大减小尺寸。

### 6.2.减少阶段数

在构建图像时，我们应该注意 Dockerfile 创建的层。**运行命令为每次执行创建一个新层。可以通过合并这些层来减小图像尺寸。**

通常，用户像这样运行命令:

```java
RUN apt-get -y update
RUN apt-get install -y python
```

上面的 Dockerfile 文件将创建两层。但是将这两个命令组合在一起会在最终图像中创建一个层:

```java
RUN apt-get -y update && apt-get install -y python
```

因此，巧妙的命令组合可以产生更小的图像。

### 6.3.构建自定义基础映像

Docker 缓存图像。每当我们有同一个层的多个实例时，我们应该优化层并创建一个定制的基础映像。加载时间将加快，跟踪将更加容易。

## 7.结论

在本教程中，我们学习了 Docker 中构建上下文的概念。我们首先讨论了 Docker 构建上下文的细节和执行。

然后，在使用`docker build`命令创建 Docker 映像时，我们解决了构建上下文增加的问题。首先，我们探索了问题的用例，然后在 Docker 中使用不同的方法来解决它们。

最后，我们研究了优化 Docker 映像的不同方法。