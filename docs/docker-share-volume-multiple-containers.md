# 在 Docker 编写中的多个容器之间共享卷

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/ops/docker-share-volume-multiple-containers>

## 1.概观

Docker 容器是隔离的环境。然而，容器有时需要持久化和共享数据。当第二个容器需要访问共享缓存或使用数据库数据时，可能会发生这种情况。我们可能还需要对用户生成的数据进行备份或执行操作。

在这个简短的教程中，我们将通过一个使用 Docker Compose 的例子来了解如何在 Docker 容器之间共享数据。

## 2.通过 Docker 存储持久化和共享数据

当容器运行时，所有文件都获得一个可写空间。然而，一旦我们停止容器，它们就不再存在了。

[Docker](https://web.archive.org/web/20221106163118/https://docs.docker.com/) 使用[存储](https://web.archive.org/web/20221106163118/https://docs.docker.com/storage/)，如果我们需要保存数据，可以选择持久存储和内存存储。

存储文件还可以提高性能，因为它直接写入主机文件系统，而不是使用容器的可写层。

### 2.1.Docker 卷

让我们快速浏览一下 [Docker 卷](/web/20221106163118/https://www.baeldung.com/ops/docker-volumes)。例如，让我们运行一个带有命名卷的 Nginx 容器。

首先，让我们[创建](https://web.archive.org/web/20221106163118/https://docs.docker.com/engine/reference/commandline/volume_create/)我们的卷:

```java
docker volume create --name volume-data
```

然后，让我们运行我们的容器:

```java
docker run -d -v volume-data:/data --name nginx-test nginx:latest
```

在这种情况下，Docker 将挂载在容器的`/data`文件夹中。如果容器在要挂载的路径中有文件或目录，它还会将目录的内容复制到卷中。

我们还可以看看 [`bind mounts`](https://web.archive.org/web/20221106163118/https://docs.docker.com/storage/bind-mounts/) 进行持久存储。

### 2.2.与卷共享数据

当多个容器需要访问共享数据时，它们可以在同一个卷上运行。

例如，让我们开始我们的 web 应用程序:

```java
docker run -d -v volume-data:/usr/src/app/public --name our-web-app web-app:latest
```

Docker 默认创建一个`local`卷。然而，我们可以使用一个[卷驱动器](https://web.archive.org/web/20221106163118/https://docs.docker.com/storage/volumes/#use-a-volume-driver)在多台机器上共享数据。

最后，Docker 还有 [`–volumes-from`](https://web.archive.org/web/20221106163118/https://docs.docker.com/storage/volumes/#backup-restore-or-migrate-data-volumes) 来链接正在运行的容器之间的卷。它可能有助于数据共享或更一般的备份使用。

## 3.与 Docker Compose 共享数据

我们已经了解了如何使用 Docker 创建卷。 **[Docker Compose](/web/20221106163118/https://www.baeldung.com/ops/docker-compose) 也支持 YAML 模板定义中的`[volumes](https://web.archive.org/web/20221106163118/https://docs.docker.com/compose/compose-file/compose-file-v3/#volume-configuration-reference)`关键字。**

让我们创建一个`docker-compose.yml`来运行共享同一个卷的 Nginx 容器和我们的 web 应用程序:

```java
services:
  nginx:
    container_name: nginx
    build: ./nginx/
    volumes:
      - shared-volume:/usr/src/app

  web-app:
    container_name: web-app
    env_file: .env
    volumes:
      - shared-volume:/usr/src/app/public
    environment:
      - NODE_ENV=production

volumes:
  shared-volume:
```

同样，在 Docker Compose 中，默认的`driver`将是`local`。我们还可以指定用于该卷的驱动程序:

```java
volumes:
  db:
    driver: some-driver
```

我们可能还需要使用 Docker Compose 外部的卷:

```java
volumes:
  data:
    external: true
    name: shared-data
```

## 4.结论

在本文中，我们看到了如何使用卷共享 Docker 容器的数据。我们还在一个使用 Docker Compose 的简单例子中看到了相同的概念。