# 如何修复 Docker 中的“名称已被容器使用”错误

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/ops/docker-name-already-in-use>

## 1.概观

在启动 Docker 容器时，我们有时可能会遇到 `“name already in use by container”`错误。

在这个简短的教程中，我们将看看这个很容易解决的常见问题。

首先，我们将展示如何导致这个错误 `. `，然后，我们将解释它的原因。最后，我们将展示如何修复它。

## 2.如何导致错误

### 2.1.获取 Docker 图像

让我们首先为我们的例子选择一个 Docker 图像。

我们将使用免费公开的 [Nginx 演示图像](https://web.archive.org/web/20220914190252/https://hub.docker.com/r/nginxdemos/hello/)。 [NGINX](https://web.archive.org/web/20220914190252/https://www.nginx.com/resources/glossary/nginx/) 是一款免费开源的 web 服务器，被`Netflix``CloudFare``Airbnb`等多家公司使用。我们将要使用的 Docker 演示图像服务于一个具有一些基本属性的 web 页面，例如主机名、IP 地址和端口。

### 2.2.运行多个容器并导致错误

**为了避免这个错误，我们需要运行两个使用相同名字**、`baeldung_nginx`的实例。

值得考虑的是，为什么我们甚至需要一个容器的名称。名称是向运行容器列表添加含义的一种便捷方式。更何况**这个名字可以在 Docker 网**做参考。

让我们开始第一个容器:

```java
docker run --name baeldung_nginx -p 80:80 -d nginxdemos/hello:plain-text
```

我们以分离模式运行容器，这意味着它将在后台运行。我们将容器的端口 80 发布到主机上的同一个端口。最后，我们已经为容器指定了我们的自定义名称—`baeldung_nginx`。

现在，如果我们在浏览器中打开`http://localhost`，我们应该会看到类似这样的内容:

```java
Server address: 123.45.6.7:80
Server name: e378ad49d49d
Date: 08/Apr/2022:22:08:44 +0000
URI: /
Request ID: 7bda7e3234cb6d1e51900fccc89320d5
```

现在让我们尝试运行第二个容器。我们将为第二个实例分配端口 81，因为主机端口 80 已经被第一个容器占用:

`docker run --name baeldung_nginx -p 81:80 -d nginxdemos/hello:plain-text`

可惜这不管用。我们得到一个错误:

```java
docker: Error response from daemon: Conflict. The container name "/baeldung_nginx" is already in use by container "76da8f6d3accc9b6d41c8a98fd492d4b8622804220ee628a438264b8cf4ae3d4". 
You have to remove (or rename) that container to be able to reuse that name.
See 'docker run --help'.
```

## 3.解释错误的根本原因

每个 Docker 容器都有一个唯一的名称。如果我们在`docker run`命令中不使用可选的`name` 参数，Docker 会分配一个随机的名称。

在我们的例子中，我们想给两个不同的容器分配相同的名称，`baeldung_nginx, `。我们应该注意，即使我们使用相同的 Docker 映像，每个`docker run`命令都会创建一个新的容器。

由于第二个容器不能使用已经在使用的名称，我们得到了错误。

## 4.怎么修

### 4.1.重新启动容器

此解决方案适用于系统中已经存在名为 `baeldung_nginx ` **的 Docker 容器的情况，这才是正确的状态**。在这种情况下，我们不希望有两个同名的不同实例。相反，我们想要重启已经存在的容器。

为了重启现有的 容器，我们必须使用 `docker start` 命令来代替 `docker run` 命令。 

`docker run` 创建一个图像的新容器。我们可以创造尽可能多的相同图像的克隆。另一方面， **`docker start `启动之前停止的集装箱**。

因此，我们可能不会尝试启动一个新的容器，而是重启现有的容器，在这种情况下，这就是解决方案。然而，有时我们想用一个新的映像来替换现有的容器。

### 4.2.移除现有容器

当我们确定希望新容器接管该名称，并且已经停止了任何其他同名容器时，我们只需删除以前的同名容器:

```java
docker rm baeldung_nginx
```

不幸的是，这个命令并不总是有效。例如，其他容器可能需要我们的容器才能正常工作。如果是这种情况，我们仍然想删除我们的容器，我们可以在 remove 命令中使用`force`标志:

```java
docker rm -f baeldung_nginx
```

一旦先前的容器被移除，我们就可以自由地用我们选择的名称启动一个新的容器。

### 4.3.对容器使用不同的名称

如果我们想运行同一个图像的两个实例，该怎么办？这种情况的解决方案很简单。我们所要做的就是使用两个不同的名称和端口:

```java
docker run --name baeldung_nginx_1 -p 80:80 -d nginxdemos/hello:plain-text
```

```java
docker run --name baeldung_nginx_2 -p 81:80 -d nginxdemos/hello:plain-text
```

现在，让我们使用下面的 Docker 命令来列出正在运行的容器:

```java
docker ps
```

我们应该会看到类似这样的内容:

```java
CONTAINER ID   IMAGE                         COMMAND                  CREATED          STATUS          PORTS                NAMES
f341bb9fe165   nginxdemos/hello:plain-text   "/docker-entrypoint.…"   2 seconds ago    Up 2 seconds    0.0.0.0:81->80/tcp   baeldung_nginx_2
33883c2b31a7   nginxdemos/hello:plain-text   "/docker-entrypoint.…"   12 seconds ago   Up 11 seconds   0.0.0.0:80->80/tcp   baeldung_nginx_1
```

## 5.结论

在本文中，我们学习了如何修复 Docker 中的`“name already in use by container”`错误。

首先，我们看到了如何重现错误。然后，我们研究了错误的根本原因。

最后，我们看到了解决这个问题的三种不同的解决方案。