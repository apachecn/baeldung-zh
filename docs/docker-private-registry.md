# Docker 私人注册指南

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/ops/docker-private-registry>

## 1.概观

Docker 是一个软件平台，工作在操作系统级虚拟化，在容器中运行应用程序。**Docker 的独特之处之一是 Docker 容器提供了相同的虚拟环境来运行应用程序。** CI/CD 工具还可用于从注册表中自动推送或提取映像，以部署到生产环境中。

在本教程中，我们将学习理解[公共](https://web.archive.org/web/20221128042924/https://hub.docker.com/)和[私有 Docker 注册表](https://web.archive.org/web/20221128042924/https://docs.docker.com/registry/deploying/)的使用。我们还将建立一个私人码头登记处。此外，我们将把一个 Docker 映像推送到一个私有的 Docker 注册中心，然后从同一个注册中心拉这个映像。

## 2.私人和公共码头登记处

Docker 支持在私有服务器上创建、存储和管理 Docker 映像。此外，Docker 还有一个免费的公共注册表。Docker Hub 可以托管我们的图像，但它们将对公众开放。在大多数情况下，映像包含运行应用程序所需的所有代码和配置。在这种情况下，我们可以使用 Docker Hub 私有帐户，或者在一台机器上建立一个私有 Docker 注册中心。

Docker Hub 私人账户是付费的，在云中存储多个图像是一个昂贵的选择。虽然私有 Docker 注册中心的设置是免费的，但是从私有注册中心访问图像的所有命令都很简单，几乎与 Docker Hub 中的命令相同。使用私有注册中心，我们可以平衡负载，定制身份验证和日志记录，并进行更多的配置更改。它创建了一个定制的管道，帮助在个人位置存储图像。这里，我们将简要介绍如何在服务器上秘密管理图像。

典型的 Docker 映像包含应用程序代码、安装、配置和所需的依赖项。一个 Docker 图像通常由多层组成。我们还可以将这些层推送到私有或公共注册中心。此外，我们将研究一些安全和存储选项，以便我们定制配置。使用这些工具，我们可以安全地管理映像，并快速、安全地推送映像。

## 3.建立一个私有注册表

我们可以通过将容器映像集中在私有或公共注册中心来减少构建时间。我们还可以从一个注册表中下载一个压缩映像，该注册表以捆绑的形式包含应用程序的所有组件，而不是在不同的环境中安装不同的依赖项。要设置私有 Docker 注册中心，我们首先需要对 Docker 守护进程的默认配置进行更改。

### 3.1.配置私有 Docker 注册表

**在 Docker 中，我们可以通过运行一个`registry`图像的容器来建立一个注册表。**在我们继续之前，让我们首先更新 Docker 安装的默认配置。

在`/etc/docker/daemon.json`中添加以下配置:

```java
{
    "insecure-registries":[
        "localhost:5000"
    ]
}
```

在上面的 JSON 中，我们在“`insecure-registries`”属性中添加了带有端口`5000`的`localhost`。要应用上述更改，让我们使用命令行重新加载 Docker 守护进程:

```java
$ sudo systemctl daemon-reload
```

现在，我们将重新启动 Docker 服务:

```java
$ sudo systemctl restart docker
```

到目前为止，我们已经成功配置了私有注册中心。

### 3.2.运行一个私有的 Docker 注册表

要运行一个私有注册中心，我们必须将一个 [`registry`](https://web.archive.org/web/20221128042924/https://hub.docker.com/_/registry) 映像存储在公共 Docker Hub 上:

```java
$ docker pull registry
Using default tag: latest
latest: Pulling from library/registry
2408cc74d12b: Pull complete 
...
fc30d7061437: Pull complete 
Digest: sha256:bedef0f1d248508fe0a16d2cacea1d2e68e899b2220e2258f1b604e1f327d475
Status: Downloaded newer image for registry:latest
docker.io/library/registry:latest
```

我们还可以提取特定版本的注册表。现在让我们使用`docker images`命令来验证注册表映像:

```java
$ docker images
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
registry            latest              773dbf02e42e        21 hours ago        24.1MB
```

现在，让我们使用`registry`图像运行一个 Docker 容器:

```java
$ docker run -itd -p 5000:5000 --name baeldung-registry registry
e2d09cd3a5ef9c88e17e0393f7125b6eeffad175fa0ce69fa3daa7803a0b3067 
```

容器的内部服务器使用端口 T1。因此，我们暴露了主机上的`5000`端口:

```java
$ docker ps
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS                    NAMES
e2d09cd3a5ef        registry            "/entrypoint.sh /etc…"   3 minutes ago       Up 2 minutes        0.0.0.0:5000->5000/tcp   baeldung-registry
```

上面的命令确认注册表已经启动并正在运行。

## 4.将映像推送到私有注册表

要将映像推送到私有注册中心，我们首先从公共 Docker 注册中心获取最新的 [centos](https://web.archive.org/web/20221128042924/https://hub.docker.com/_/centos) 映像:

```java
$ docker pull centos
Using default tag: latest
latest: Pulling from library/centos
a1d0c7532777: Pull complete 
Digest: sha256:a27fd8080b517143cbbbab9dfb7c8571c40d67d534bbdee55bd6c473f432b177
Status: Downloaded newer image for centos:latest
docker.io/library/centos:latest
```

这里，我们提取了一个 Docker 图像样本，可以将它推送到 Docker 私有注册中心。首先，我们将标记`centos`图像，稍后将它推送到私有 docker 注册中心。在这里，我们将它标记为`localhost:5000/baeldung-centos`:

```java
$ docker tag centos:latest localhost:5000/baeldung-centos
```

现在，让我们检查主机上的所有图像:

```java
$ docker images
REPOSITORY                          TAG                 IMAGE ID            CREATED             SIZE
registry                            latest              773dbf02e42e        22 hours ago        24.1MB
localhost:5000/baeldung-centos   latest              5d0da3dc9764        8 months ago        231MB
centos                              latest              5d0da3dc9764        8 months ago        231MB
```

在这里，我们可以看到 imageId `5d0da3dc9764`有两个不同的存储库。Docker 中的标签类似于符号[链接](/web/20221128042924/https://www.baeldung.com/linux/symbolic-and-hard-links)。要删除图像，我们必须删除 imageId 或显式删除这两个标记。

让我们来看看将图像推送到 docker 私有注册表的命令:

```java
$ docker push localhost:5000/baeldung-centos
The push refers to repository [localhost:5000/baeldung-centos]
74ddd0ec08fa: Pushed 
latest: digest: sha256:a1801b843b1bfaf77c501e7a6d3f709401a1e0c83863037fa3aab063a7fdb9dc size: 529
```

使用上面的命令，我们已经成功地将`baeldung-centos`映像推送到私有注册中心，它是在端口`5000`上本地设置的。类似地，我们也可以在私有注册表中存储多个图像。

## 5.从私有注册表中提取图像

从私有注册表中提取映像的命令类似于从 Docker Hub 中提取映像。这里，首先，我们将删除所有 imageId 为`5d0da3dc9764`的图像:

```java
$ docker rmi 5d0da3dc9764
```

让我们来看看存储在主机上的所有图像:

```java
$ docker-registry]# docker images
REPOSITORY                       TAG                 IMAGE ID            CREATED             SIZE
registry                         latest              773dbf02e42e        22 hours ago        24.1MB
```

我们可以看到，imageId 为`5d0da3dc9764`的图像已经被删除。让我们看看从私有 Docker 注册表中提取图像的命令:

```java
$ docker pull  localhost5000/baeldung-centos
Using default tag: latest
latest: Pulling from baeldung-centos
a1d0c7532777: Pull complete 
Digest: sha256:a1801b843b1bfaf77c501e7a6d3f709401a1e0c83863037fa3aab063a7fdb9dc
Status: Downloaded newer image for localhost:5000/baeldung-centos:latest
localhost:5000/baeldung-centos:latest
```

上面的命令将从私有注册表中提取`baeldung-centos`图像。

## 6.为私有注册表设置身份验证

Docker 允许我们将图像存储在本地的中央服务器上，但有时，保护图像免受外部滥用是必要的。在这种情况下，我们需要用基本的 [`htpasswd`](https://web.archive.org/web/20221128042924/https://httpd.apache.org/docs/2.4/programs/htpasswd.html) 认证来认证注册中心。

让我们首先创建一个单独的目录来存储 Docker 注册表凭证:

```java
$ mkdir -p Docker_registry/auth
```

接下来，让我们运行一个`httpd`容器来创建一个带有密码的`htpasswd`受保护用户:

```java
$ cd Docker_registry &&docker run \
  --entrypoint htpasswd \
  httpd:2 -Bbn baeldung-user baeldung > auth/htpasswd
```

上面的命令将创建一个拥有`htpasswd`认证密码的用户。凭证的详细信息存储在`auth/htpasswd`文件中。

现在，让我们使用`auth/htpasswd`认证文件运行同一个 Docker 注册容器:

```java
$ docker run -itd \
  -p 5000:5000 \
  --name registry \
  -v "$(pwd)"/auth:/auth \
  -e "REGISTRY_AUTH=htpasswd" \
  -e "REGISTRY_AUTH_HTPASSWD_REALM=Registry Realm" \
  -e REGISTRY_AUTH_HTPASSWD_PATH=/auth/htpasswd \
  registry
  3a497bafed4adb21a5a3f0b52307b4beaa261c6abe265e543cd8f5a15358e29d
```

由于 Docker 注册中心使用基本身份验证运行，我们现在可以使用以下命令测试登录:

```java
$ docker login  localhost:5000 -u baeldung-user -p baeldung
WARNING! Using --password via the CLI is insecure. Use --password-stdin.
WARNING! Your password will be stored unencrypted in /root/.docker/config.json.
Configure a credential helper to remove this warning. See
https://docs.docker.com/engine/reference/commandline/login/#credentials-store
Login Succeeded
```

一旦成功登录到 Docker 注册中心，我们就可以像上面讨论的那样推送和拉取图像。

## 7.结论

本教程演示了如何创建我们自己的私有 Docker 注册表和推送 Docker 映像。

首先，我们建立了一个私人注册中心。后来，我们在注册表中推送和提取图像。最后，我们在私有 Docker 注册表中启用了身份验证。