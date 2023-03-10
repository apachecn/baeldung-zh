# 如何让 Docker-Compose 总是使用最新的图像

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/ops/docker-compose-latest-image>

## 1.介绍

在我们的项目中，我们经常使用[docker-compose](/web/20221010234555/https://www.baeldung.com/ops/docker-compose)来部署我们的容器化应用。有了 CI 和 CD，现在代码变更和部署非常频繁。 因此，确保 docker-compose 始终使用应用程序 的最新图像至关重要。

在本教程中，我们将检查几个选项来达到同样的目的。

## 2.显式提取图像

让我们举一个 docker-compose 文件的简单例子:

```java
version: '2.4'
services:
  db:
    image: postgres
  my_app:
    image: "eugen/test-app:latest"
    ports:
      - "8080:8080"
```

这里，我们使用了两个服务——一个是 PostgreSQL 数据库，另一个是测试应用程序。我们使用下面的[命令](https://web.archive.org/web/20221010234555/https://docs.docker.com/compose/reference/up/)来部署我们的应用程序:

```java
$ docker-compose up -d
```

这将从远程存储库(比如说， [dockerhub](https://web.archive.org/web/20221010234555/https://hub.docker.com/) )获取两个图像，并创建相应的容器。

当我们用相同的命令重新部署应用程序时，问题就出现了。由于图像已经存在于本地存储库中，所以它不会检查远程存储库中的这些图像是否有任何更改。**所以，我们的选择是将`docker-compose`文件中的`[pull](https://web.archive.org/web/20221010234555/https://docs.docker.com/compose/reference/pull/)`所有图像预先:**

```java
$ docker-compose pull
Pulling db     ... done
Pulling my_app ... done
```

该命令首先检查`dockerhub`中的两个图像是否有更新。然后，它只下载必要的图层，以保持图像与遥控器同步。

然而，我们可能并不总是更喜欢`Postgres`的形象。如果我们使用一个稳定版本的数据库，映像中不会有任何变化。因此，为每个部署下载它是没有意义的。有时，现有的数据文件与新版本不兼容。这可能会破坏部署。在这种情况下，**我们可以在`pull`命令中只使用特定服务的名称:**

```java
$ docker-compose pull my_app
Pulling my_app ... done
```

**现在，如果我们执行`up`命令，肯定会用最新的图像**重新创建容器:

```java
$ docker-compose up -d
Starting docker-test_db_1     ... done
Starting docker-test_my_app_1 ... done
```

当然，我们可以将这两个命令组合起来创建一个单行命令:

```java
$ docker-compose pull && docker-compose up -d
```

## 3.移除本地图像

让我们再次考虑同一个例子。我们已经知道，主要问题是本地图像的存在。**因此，我们这里的第二个选项是停止所有容器，并从本地存储库中删除它们的映像**:

```java
$ docker-compose down --rmi all
Stopping docker-test_my_app_1 ... done
Removing docker-test_db_1     ... done
Removing docker-test_my_app_1 ... done
Removing network docker-test_default
Removing image postgres
Removing image eugen/test-app:latest
```

`[down](https://web.archive.org/web/20221010234555/https://docs.docker.com/compose/reference/down/)`命令停止并移除集装箱。`–rmi`选项从本地存储库中删除图像。`rmi`类型`local`仅删除那些本地构建的图像。因此，最好使用`rmi`类型`all`来确保当前配置使用的所有图像都被移除。

**现在，我们用`up`命令:**再次启动我们的容器

```java
$ docker-compose up -d
Creating network "docker-test_default" with the default driver
Pulling db (postgres:)...
latest: Pulling from library/postgres
7d63c13d9b9b: Pull complete
cad0f9d5f5fe: Pull complete
...
Digest: sha256:eb83331cc518946d8ee1b52e6d9e97d0cdef6195b7bf25323004f2968e91a825
Status: Downloaded newer image for postgres:latest
Pulling my_app (eugen/test-app:latest)...
latest: Pulling from eugen/test-app
df5590a8898b: Already exists
705bb4cb554e: Already exists
...
Digest: sha256:31c05c8245192b32b8b359fc58b5e45d8397674ccf41f5f17a7d3109772ab5c1
Status: Downloaded newer image for eugen/test-app:latest
Creating docker-test_db_1     ... done
Creating docker-test_my_app_1 ... done
```

我们可以看到，它不再在本地存储库中找到图像。因此，它从远程存储库中提取最新的映像来重新创建容器。

这种方法的缺点是无法选择要删除的特定图像。因此，它总是删除`postgres`图像，并再次下载相同的图像，这是不必要的。为了解决这个问题，我们可以使用`docker rmi`来删除特定的图像:

```java
$ docker rmi -f eugen/test-app:latest
Untagged: eugen/test-app:latest
Untagged: eugen/[[email protected]](/web/20221010234555/https://www.baeldung.com/cdn-cgi/l/email-protection):31c05c8245192b32b8b359fc58b5e45d8397674ccf41f5f17a7d3109772ab5c1
Deleted: sha256:7bc07b4eb1c23f7a91afeb7133f107e0a8318fb77655d7d5f2f395a035a13eb7
```

## 4.重建图像

让我们看看具有不同风格的 docker-compose 文件的相同示例:

```java
version: '2.4'
services:
  db:
    image: postgres
  my_app:
    build: ./test-app
    ports:
      - "8080:8080"
```

这里，我们在 Postgres 中使用了相同的`db`服务。但是，对于服务`my_app`，我们给出了一个构建部分，而不是使用现成的映像。这个部分包含了`test-app`的构建上下文。驻留在`test-app`目录中的 docker 文件是这样的:

```java
FROM openjdk:11
COPY target/test-app-0.0.1-SNAPSHOT.jar app.jar
ENTRYPOINT ["java","-jar","/app.jar"]
```

在这个场景中，当我们使用`up`命令重新部署时，`docker-compose`再次重用本地映像(如果它存在的话)。因此，**我们需要确保`docker-compose`在每次触发部署时重建映像。** **我们可以使用选项`–build` :** 来完成

```java
$ docker-compose up -d --build
Building my_app
[+] Building 2.5s (8/8) FINISHED                                                                                            
 => [internal] load build definition from Dockerfile                                                                    0.0s
 => => transferring dockerfile: 41B                                                                                     0.0s
 => [internal] load .dockerignore                                                                                       0.0s
 => => transferring context: 2B                                                                                         0.0s
 => [internal] load metadata for docker.io/library/openjdk:11                                                           2.3s
 => [auth] library/openjdk:pull token for registry-1.docker.io                                                          0.0s
 => [1/2] FROM docker.io/library/openjdk:[[email protected]](/web/20221010234555/https://www.baeldung.com/cdn-cgi/l/email-protection):1b04b1958a4a61900feec994e3938a2a5d8f88db8ec9515f46a25cbe561b65d9     0.0s
 => [internal] load build context                                                                                       0.0s
 => => transferring context: 84B                                                                                        0.0s
 => CACHED [2/2] COPY target/test-app-0.0.1-SNAPSHOT.jar app.jar                                                        0.0s
 => exporting to image                                                                                                  0.0s
 => => exporting layers                                                                                                 0.0s
 => => writing image sha256:4867a3f0b0a043cd54e16086e2d3c81dbf4c418806399f60fc7d7ffc094c7159                            0.0s
 => => naming to docker.io/library/docker-test_my_app                                                                   0.0s

Starting docker-test_db_1 ... 
Starting docker-test_db_1 ... done
```

**另一种方法是在运行`up`命令之前使用`[build](https://web.archive.org/web/20221010234555/https://docs.docker.com/compose/reference/build/)`命令:**

```java
$ docker-compose build --pull --no-cache
db uses an image, skipping
Building my_app
...                                                                                                           0.0s
 => => naming to docker.io/library/docker-test_my_app 
```

该命令提供了另外两个选项。`–pull`选项要求在构建映像时从远程存储库中提取`Dockerfile`的基础映像。`–no-cache`选项表示跳过本地缓存。然而，它跳过了构建`db`服务，因为我们在那里使用了直接映像。

**现在，如果我们重新启动我们的组合配置，容器将使用最新的图像:**

```java
$ docker-compose up -d
Starting docker-test_db_1       ... done
Recreating docker-test_my_app_1 ... done
```

## 5.结论

在本文中，我们已经看到了为什么 docker-compose 在部署时可能不使用最新的映像。我们还学习了几种让 docker-compose 总是使用最新的映像来重新创建带有该映像的容器的方法。我们还讨论了这些方法的缺陷以及如何减轻它们。

与文章相关的代码可以在 GitHub 的[上找到。](https://web.archive.org/web/20221010234555/https://github.com/eugenp/tutorials/tree/master/docker-modules/docker-compose)