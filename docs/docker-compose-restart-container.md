# 使用 Docker Compose 重新启动单个容器

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/ops/docker-compose-restart-container>

## 1.概观

在本教程中，我们将学习如何使用 [Docker Compose](/web/20220727020745/https://www.baeldung.com/ops/docker-compose) 重启单个 Docker 容器。

## 2.坞站组成〔t0〕命令

Docker Compose 是一个将多个容器作为单一服务进行管理的工具。然而，Docker Compose CLI 包括可以应用于单个容器的命令。例如，**`restart` 命令让我们提供我们想要重启的服务的名称，而不影响其他正在运行的服务:**

```java
docker-compose restart service-name
```

在开始执行`restart` 命令之前，让我们先设置一个工作环境。

## 3.设置

我们必须有一个 Docker 容器来运行 Docker 编写命令。我们将使用以前的 Baeldung 项目， [`spring-cloud-docker`](https://web.archive.org/web/20220727020745/https://github.com/eugenp/tutorials/tree/master/spring-cloud-modules/spring-cloud-docker) ，它是一个 dockerized Spring Boot 应用程序。这个项目有两个 Docker 容器，它们将帮助我们证明我们可以重启一个服务而不影响另一个。

首先，我们必须通过从项目根目录运行以下命令来确认我们可以运行这两个容器:

```java
docker-compose up --detach --build
```

现在，我们应该能够通过执行`docker-compose ps`看到两个服务都在运行:

```java
$ docker ps
     Name                   Command              State            Ports         
--------------------------------------------------------------------------------
message-server   java -jar /message-server.jar   Up      0.0.0.0:18888->8888/tcp
product-server   java -jar /product-server.jar   Up      0.0.0.0:19999->9999/tcp 
```

此外，我们可以转到浏览器中的`localhost:18888`或`localhost:19999`，验证我们是否看到了应用服务显示的消息。

## 4.重新启动单个容器

到目前为止，我们有两个容器由 Docker Compose 作为单个服务运行和管理。现在，让我们看看如何使用`restart` 命令来停止和启动两个容器中的一个。

首先，我们来看看如何在不重新构建容器的情况下实现这一点。 **但是，这个解决方案不会用最新的代码更新服务。然后，我们将看到另一种方法，在运行**之前，我们用最新的代码构建容器。

### 4.1.重新启动而不重建

两个容器都在运行，我们选择其中一个服务来重启。在这种情况下，我们将使用`message-server`容器:

```java
docker-compose restart message-server
```

在终端中运行该命令后，我们应该能够看到以下消息:

```java
Restarting message-server ... done
```

一旦终端提示输入另一个命令，我们可以通过运行 Docker 命令`ps`来检查所有正在运行的进程的状态，从而确认`message-server`已经成功重启:

```java
$ docker ps
CONTAINER ID   IMAGE                   COMMAND                  CREATED          STATUS          PORTS                     NAMES
b6541d1c4ddf   product-server:latest   "java -jar /product-…"   10 minutes ago   Up 42 seconds   0.0.0.0:19999->9999/tcp   product-server
1d07d2a7ed7d   message-server:latest   "java -jar /message-…"   10 minutes ago   Up 15 seconds   0.0.0.0:18888->8888/tcp   message-server 
```

最后，通过查看`STATUS`列，我们可以确定该命令成功地重启了`message-server` 容器。我们可以看到`message-server` 服务是如何在比`product-server` 服务更短的时间内启动并运行的，自从我们在上一节中运行了`docker-compose up` 命令后，`product-server` 服务就已经启动了。

### 4.2.**重建和重启**

**如果一个容器需要用最新的代码更新，运行`restart` 命令是不够的，因为服务需要首先构建以获得代码更改。**

在两个容器都运行的情况下，让我们首先更改代码，以确认我们将在重新启动之前用最新的代码更新服务。在`DockerProductController` 类中，让我们将`return`语句修改成这样:

```java
public String getMessage() {
    return "This is a brand new product";
}
```

现在，让我们构建 Maven 包:

```java
mvn clean package
```

现在，我们准备重启`product-server`容器。我们可以像启动服务一样实现这一点，但这一次是通过提供我们想要重启的容器的名称。

让我们运行命令来重建并重启容器:

```java
docker-compose up --detach --build product-server
```

现在，我们可以通过运行`docker ps`并查看输出来验证`product-server` 容器是否以最新的代码重启:

```java
$ docker ps
CONTAINER ID   IMAGE                   COMMAND                  CREATED          STATUS          PORTS                     NAMES
78a4364e75e6   product-server:latest   "java -jar /product-…"   6 seconds ago    Up 5 seconds    0.0.0.0:19999->9999/tcp   product-server
b559f742973b   message-server:latest   "java -jar /message-…"   22 minutes ago   Up 22 minutes   0.0.0.0:18888->8888/tcp   message-server 
```

正如我们所见，`product-server`改变了`CREATED` 值和`STATUS` 值，表明服务首先被重建，然后重新启动，对`message-server`没有任何影响。

此外，我们可以通过在浏览器中转至`localhost:19999`并检查输出是否是最新的来进一步确认代码是否已更新。

## 5.结论

在本文中，我们学习了如何使用 Docker Compose 重启单个容器。我们讨论了实现这一点的两种方法。

首先，我们使用带有服务名称的`restart` 命令来重启。然后，我们对代码进行了修改，以证明我们可以重新构建最新的代码，并在不影响另一个容器的情况下重新启动一个容器。