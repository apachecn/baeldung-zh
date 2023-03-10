# 在 Docker 容器中部署 Java War

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/docker-deploy-java-war>

## 1.概观

在本教程中，我们将学习在一个 [Docker](/web/20220727020703/https://www.baeldung.com/ops/docker-guide) 容器中部署一个 Java WAR 文件。

我们将把 WAR 文件部署在 Apache Tomcat 上，这是一个免费的开源 web 服务器，在 Java 社区中被广泛使用。

## 2.将 WAR 文件部署到 Tomcat

WAR (Web 应用程序存档)是一个压缩存档文件，它打包了所有与 Web 应用程序相关的文件及其目录结构。

简单来说，[在 Tomcat](/web/20220727020703/https://www.baeldung.com/tomcat-deploy-war) 上部署 WAR 文件只不过是将 WAR 文件复制到 Tomcat 服务器的部署目录中。Linux 中的部署目录是`$CATALINA_HOME/webapps`。`$CATALINA_HOME`表示 Tomcat 服务器的安装目录。

在这之后，我们需要**重启 Tomcat 服务器**，它将提取部署目录中的 WAR 文件。

## 3.在 Docker 容器中部署 WAR

假设我们的应用程序`ROOT.war`有一个 WAR 文件，我们需要将它部署到 Tomcat 服务器上。

为了实现我们的目标，我们需要首先创建一个 docker 文件。这个 docker 文件将包含运行我们的应用程序所需的所有依赖项。

此外，我们将使用这个 Docker 文件创建一个 Docker 映像，然后执行启动 Docker 容器的步骤。

现在让我们一个接一个地深入研究这些步骤。

### 3.1.创建 Dockerfile 文件

我们将使用 Tomcat 的最新 [Docker 图像作为 Docker 文件的基础图像。使用这个映像的优点是所有必需的依赖项/包都是预先安装的。例如，如果我们使用最新的 Ubuntu/CentOS Docker 映像，那么我们需要手动安装 Java、Tomcat 和其他必需的包。](https://web.archive.org/web/20220727020703/https://hub.docker.com/_/tomcat)

因为已经安装了所有需要的包，所以我们需要做的就是将 WAR 文件`ROOT.war`复制到 Tomcat 服务器的部署目录中。就是这样！

让我们仔细看看:

```java
$ ls
Dockerfile  ROOT.war
$ cat Dockerfile 
FROM tomcat

COPY ROOT.war /usr/local/tomcat/webapps/
```

**`$CATALINA_HOME/webapps` 在这里表示 Tomcat `.`** 的部署目录，`CATALINA_HOME`表示 Tomcat 的官方 Docker 镜像为`/usr/local/tomcat`。结果，完整的部署目录将变成`/usr/local/tomcat/webapps`。

我们在这里使用的应用程序非常简单，不需要任何其他依赖。

### 3.2.建立 Docker 形象

现在，让我们使用刚刚创建的 Docker 文件来创建 Docker 映像:

```java
$ pwd
/baeldung
$ ls
Dockerfile  ROOT.war
$ docker build -t myapp .
Sending build context to Docker daemon  19.97kB
Step 1/2 : FROM tomcat
 ---> 710ec5c56683
Step 2/2 : COPY ROOT.war /usr/local/tomcat/webapps/
 ---> Using cache
 ---> 8b132ab37a0e
Successfully built 8b132ab37a0e
Successfully tagged myapp:latest 
```

`docker build`命令将创建一个带有标签`myapp. `的 Docker 图像

**确保从 Docker 文件所在的目录中构建 Docker 映像。**在上面的例子中，当我们构建 Docker 映像时，我们位于`/baeldung`目录中。

### 3.3.运行 Docker 容器

到目前为止，我们已经创建了一个 Docker 文件，并用它构建了一个 Docker 映像。现在让我们运行 Docker 容器:

```java
$ docker run -itd -p 8080:8080 --name my_application_container myapp
e90c61fdb4ac85b198903e4d744f7b0f3c18c9499ed6e2bbe2f39da0211d42c0
$ docker ps 
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS                    NAMES
e90c61fdb4ac        myapp               "catalina.sh run"   6 seconds ago       Up 5 seconds        0.0.0.0:8080->8080/tcp   my_application_container 
```

这个命令将使用 Docker 映像`myapp. `启动一个名为`my_application_container` 的 Docker 容器

Tomcat 服务器的默认端口是 8080。因此，在启动 Docker 容器时，确保总是将容器端口 8080 与任何可用的主机端口绑定。为了简单起见，我们在这里使用了主机端口 8080。

### 3.4.验证设置

现在，让我们验证一下到目前为止我们所做的一切。我们将在浏览器中访问 URL `http://<IP>:<PORT>` 来查看应用程序。

这里，`IP` 表示 Docker 主机的公共 IP(或者在某些情况下是私有 IP)。`PORT`是我们在运行 Docker 容器时公开的容器端口(在我们的例子中是 8080)。

我们还可以使用 Linux 中的 [`curl`](https://web.archive.org/web/20220727020703/https://curl.se/docs/manpage.html) 实用程序来验证设置:

```java
$ curl http://localhost:8080
Hi from Baeldung!!!
```

在上面的命令中，我们从 Docker 主机执行命令。因此，我们能够使用`localhost.` 连接到应用程序。作为响应，`curl`实用程序打印应用程序网页的原始 HTML。

## 4.结论

在本文中，我们学习了在 Docker 容器中部署 Java WAR 文件。我们首先使用官方的 Tomcat Docker 映像创建 Docker 文件。然后，我们构建 Docker 映像并运行应用程序容器。

最后，我们通过访问应用程序 URL 来验证设置。