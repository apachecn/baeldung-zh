# Docker 中“公开”和“发布”的区别

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/ops/docker/expose-vs-publish>

## 1.概观

在 Docker 中，知道一个容器化的应用程序正在监听哪些端口是很重要的。我们还需要一种从容器外部访问应用程序的方法。

为了解决这些问题，Docker 使我们能够公开和发布这些端口。

在本文中，我们将学习公开和发布端口。我们将使用一个简单的 Nginx web 服务器容器作为例子。

## 2.暴露端口

公开端口是关于容器化应用程序的一段元数据。在大多数情况下，这显示了应用程序正在监听哪些端口。Docker 本身对暴露的端口不做任何事情。然而，当我们启动一个容器时，我们可以在发布端口时使用这些元数据。

### 2.1.用 Nginx 曝光

让我们使用 Nginx web 服务器来尝试一下。

如果我们看一下 [Nginx 官方 docker 文件](https://web.archive.org/web/20220727020703/https://github.com/nginxinc/docker-nginx/blob/dcaaf66e4464037b1a887541f39acf8182233ab8/mainline/debian/Dockerfile)，我们会看到端口 80 是用以下命令公开的:

```java
EXPOSE 80
```

这里公开了端口 80，因为它是`http` 协议的默认端口。让我们在本地机器上运行 Nginx 容器，看看能否通过端口 80 访问它:

```java
$ docker run -d nginx
```

上述命令将使用 Nginx 的最新映像并运行容器。我们可以使用以下命令仔细检查 Nginx 容器是否正在运行:

```java
$ docker container ls
```

这个命令将输出一些关于所有正在运行的容器的信息，包括 Nginx:

```java
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS               NAMES
cbc2f10f787f        nginx               "/docker-entrypoint..."   15 seconds ago      Up 15 seconds       80/tcp              dazzling_mclean 
```

这里我们看到 80 在港口部分。由于端口 80 是公开的，我们可能认为访问`localhost:80`(或者仅仅是`localhost`)会显示 Nginx 默认页面，但事实并非如此:

```java
$ curl http://localhost:8080
... no web page appears
```

虽然端口是公开的，但 Docker 并没有向主机开放它。

### 2.2.公开端口的方法

在 Docker 中公开端口主要有两种方式。我们可以在`Dockerfile`中用`EXPOSE` 命令来完成:

```java
EXPOSE 8765
```

或者，我们也可以在运行容器`:`时使用`–expose` 选项公开端口

```java
$ docker run --expose 8765 nginx
```

## 3.发布端口

为了使容器端口可以通过 docker 主机访问，我们需要发布它。

### 3.1.用 Nginx 发布

让我们使用映射端口运行 Nginx:

```java
$ docker run -d -p 8080:80 nginx
```

上述命令将主机的端口 8080 映射到容器的端口 80。该选项的一般语法是:

```java
-p <hostport>:<container port>
```

如果我们去`localhost:8080`，我们应该得到 Nginx 的默认欢迎页面:

```java
$ curl http://localhost:8080
StatusCode : 200
StatusDescription : OK
Content : <!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
... more HTML
```

让我们列出所有正在运行的容器:

```java
$ docker container ls
```

现在我们应该看到容器有一个端口映射:

```java
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS                  NAMES
38cfed3c61ea        nginx               "/docker-entrypoint..."   31 seconds ago      Up 30 seconds       0.0.0.0:8080->80/tcp   dazzling_kowalevski
```

在`ports`部分下，我们有`0.0.0.0:8080->80/tcp` 映射。

默认情况下，Docker 为主机`.` 添加了`0.0.0.0` 不可路由的元地址，这意味着该映射对主机的所有地址/接口都有效。

### 3.2.限制容器访问

**我们可以根据主机 IP 地址限制对容器的访问。**我们可以在映射中指定主机 IP 地址，而不是允许从所有接口访问容器(这是`0.0.0.0`所做的)。

让我们将对容器的访问仅限于来自`127.0.0.1`回送地址的流量:

```java
$ docker run -d -p 127.0.0.1:8081:80 nginx
```

在这种情况下，只能从主机本身访问容器。这使用扩展语法进行发布，其中包括地址绑定:

```java
-p <binding address>:<hostport>:<container port>
```

## 4.发布所有公开的端口

公开的端口元数据对于启动容器很有用，因为 Docker 使我们能够发布所有公开的端口:

```java
$ docker run -d --publish-all nginx
```

这里，Docker 将容器中所有暴露的端口绑定到主机上空闲的随机端口。

让我们看看这个命令启动的容器:

```java
$ docker container ls

CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS                   NAMES
0a23e78732ce        nginx               "/docker-entrypoint..."   6 minutes ago       Up 6 minutes        0.0.0.0:32768->80/tcp   pedantic_curran
```

正如我们所料，Docker 从主机中随机选择了一个端口(本例中为 32768 ),并将其映射到暴露的端口`.`

## 5.结论

在本文中，我们学习了在 Docker 中公开和发布端口。

我们还讨论了公开的端口是关于容器化应用程序的元数据，而发布端口是从主机访问应用程序的一种方式。