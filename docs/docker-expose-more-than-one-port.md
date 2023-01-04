# 用 Docker 公开多个端口

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/ops/docker-expose-more-than-one-port>

## 1.概观

当我们对接应用程序时，我们通常需要暴露一个端口。应用程序使用该端口与其他容器或外部世界进行交互。有时，一个端口是不够的。可能需要一个或多个附加端口来服务于其他目的。例如，在 Spring Boot 应用程序中，我们需要一个单独的端口来发布管理端点，以便使用执行器来监控应用程序。

在本文中，**我们将看到如何声明多个端口来公开，以及如何将公开的端口与主机**的端口绑定，以实现上述目的。

## 2.申报港口

首先，我们需要声明要公开的端口。我们可以在构建 docker 映像的同时做到这一点。也可以在运行基于映像的容器时声明端口。让我们看看如何做到这一点。

我们将从一个示例 Spring Boot 应用程序的例子开始—`my-app`。在整篇文章中，我们将使用同一个例子来理解这些概念。我们的应用程序只有一个`GET`端点，它返回“`Hello buddy`”。它还启用了弹簧致动器。应用程序在端口`8080`上运行，管理端点在端口`8081`上运行。因此，当应用程序在本地计算机上运行时，这些命令起作用:

```java
$ curl http://localhost:8080
Hello buddy

$ curl http://localhost:8081/actuator/health
{"status":"UP"}
```

### 2.1.`Dockerfile`中的声明

由于我们的应用程序`my-app`在两个端口`8080`和`8081`中发布其端点，我们需要在我们的`Dockerfile`中公开这两个端口。`Dockerfile`中的`[EXPOSE](https://web.archive.org/web/20221126221746/https://docs.docker.com/engine/reference/builder/#expose)`动词暴露端口:

```java
FROM openjdk:8-jdk-alpine
EXPOSE 8080
EXPOSE 8081
ARG JAR_FILE=target/my-app-0.1.jar
ADD ${JAR_FILE} my-app.jar
ENTRYPOINT ["java","-jar","/my-app.jar"]
```

然而，**当我们使用这个`Dockerfile`** 构建图像时:

```java
$ docker build -t my-app:latest .
```

**这实际上并没有打开端口**，因为`Dockerfile`的作者无法控制容器将要运行的网络。相反，`EXPOSE`命令充当文档。这样，运行容器的人就知道需要在主机中发布容器的哪些端口来与应用程序通信。

我们还可以指定用于在该端口上通信的协议–`TCP`或`UDP`:

```java
EXPOSE 8080/tcp
EXPOSE 8081/udp
```

如果我们不指定任何东西，它就把`TCP`作为默认值。

该命令还支持范围内的端口声明:

```java
EXPOSE 8000-8009
```

上面的命令告诉应用程序需要打开从`8000`到`8009`的 10 个端口进行通信。

### 2.2.在`docker run`命令中声明

让我们假设我们已经有了一个`my-app`的 docker 映像，该映像在`Dockerfile`中使用`EXPOSE`命令只暴露了一个端口`8080`。现在，**如果我们想要暴露另一个端口，`8081`，我们应该使用`–expose`参数和`[run](https://web.archive.org/web/20221126221746/https://docs.docker.com/engine/reference/run/#expose-incoming-ports)`命令**:

```java
$ docker run --name myapp -d --expose=8081 my-app:latest
```

上面的命令从映像`my-app`运行一个名为`myapp`的容器，并公开`8081`和端口`8080`。我们可以使用以下方法对此进行检查:

```java
$ docker ps
CONTAINER ID   IMAGE           COMMAND                  CREATED             STATUS              PORTS              NAMES
2debb3c5345b   my-app:latest   "java -jar /my-app.j…"   5 seconds ago       Up 3 seconds        8080-8081/tcp      myapp
```

**重要的是要明白，该参数仅暴露——但不公布——主机中的端口。**为了更清楚地理解它，让我们执行:

```java
$ docker port myapp
```

这不会打印任何内容，因为主机中没有打开和映射任何端口。因此，即使应用程序正在容器内运行，我们也无法访问它:

```java
$ curl http://localhost:8080
curl: (7) Failed to connect to localhost port 8080: Connection refused
```

我们也可以选择以同样的方式公开一系列端口:

```java
$ docker run --name myapp -d --expose=8000-8009 my-app:latest
```

## 3.发布端口

我们已经学会了为 dockerized 应用程序公开端口。现在是时候出版它们了。

### 3.1.在运行命令中发布

让我们重用上一节中的`my-app`图像的例子。它的`Dockerfile`—`8080`和`8081`有两个暴露的端口。在基于这个映像运行容器时，**我们可以使用`-P`参数**一次发布所有公开的端口:

```java
$ docker run --name myapp -d -P myApp:latest
```

上述命令打开主机中的两个随机端口，并将它们与 Docker 容器的端口`8080`和`8081`进行映射。如果我们公开一系列端口，它的行为也是一样的。

为了检查映射的端口，我们使用:

```java
$ docker port myapp
8080/tcp -> 0.0.0.0:32773
8081/tcp -> 0.0.0.0:32772
```

现在，可以使用端口`32773,`访问应用程序，并且可以通过`32772`访问管理端点:

```java
$ curl http://localhost:32773
Hello buddy
$ curl http://localhost:32772/actuator/health
{"status":"UP"}
```

**我们可以使用`-p`参数**在主机中选择特定的端口，而不是随机分配端口:

```java
$ docker run --name myapp -d -p 80:8080 my-app:latest
```

上面的命令只发布端口`8080`并与主机服务器中的端口`80`进行映射。它不允许从容器外部访问执行器端点:

```java
$ curl http://localhost:80
Hello buddy
$ curl http://localhost:8081/actuator/health
curl: (7) Failed to connect to localhost port 8081: Connection refused
```

**为了发布多个端口映射，我们多次使用`-p`参数**:

```java
$ docker run --name myapp -d -p 80:8080 -p 81:8081 my-app:latest
```

这样，我们还可以控制集装箱的哪些端口对外开放。

### 3.2.在`docker-compose`发布

如果我们在`docker-compose`、**中使用我们的应用程序，我们可以提供一个需要在`docker-compose.yml`文件**中发布的端口列表:

```java
version: "3.7"
services:
  myapp:
    image: my-app:latest
    ports:
      - 8080
      - 8081
```

如果我们启动此设置，它会为主机服务器的随机端口分配给定的端口:

```java
$ docker-compose up -d
Starting my-app_myapp_1 ... done
$ docker port  my-app_myapp_1
8080/tcp -> 0.0.0.0:32785
8081/tcp -> 0.0.0.0:32784
```

但是，可以提供特定的端口选择:

```java
version: "3.7"
services:
  myapp:
    image: my-app:latest
    ports:
      - 80:8080
      - 81:8081
```

这里，`80`和`81`是主机端口，`8080`和`8081`是集装箱端口。

## 4.结论

在本文中，我们讨论了在 Docker 容器中公开和发布多个端口的不同方式。我们已经看到了公开实际上意味着什么，以及如何获得对容器和主机之间的端口发布的完全控制。