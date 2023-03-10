# 在 Docker 容器上安装多个卷

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/ops/docker-mounting-multiple-volumes>

## 1.概观

Docker 有多种选项来持久化和共享正在运行的容器的数据。然而，我们可能需要一个以上的文件存储来运行容器，例如，创建备份或授予不同的访问权限。或者对于同一个容器，我们可能需要添加命名卷并将它们绑定到特定路径。

在本教程中，我们将了解如何在一个容器上挂载多个卷。我们将看到一些命令行和 Docker Compose 的例子。

## 2.Docker 容器上的多个安装

Docker 使用[存储](https://web.archive.org/web/20221006124104/https://docs.docker.com/storage/)来保存数据，所以如果容器重启，我们不会丢失信息。此外，如果我们想要在集群环境中共享数据，数据持久性将是相关的。

**我们将使用[卷](https://web.archive.org/web/20221006124104/https://docs.docker.com/storage/volumes/)和[绑定挂载](https://web.archive.org/web/20221006124104/https://docs.docker.com/storage/bind-mounts/)来创建一些示例，以突出最常见的开发用例。**

### 2.1.使用多个卷

首先，让我们[创建](https://web.archive.org/web/20221006124104/https://docs.docker.com/engine/reference/commandline/volume_create/)两个不同命名的卷:

```java
docker volume create --name first-volume-data && docker volume create --name second-volume-data 
```

假设我们想为 web 应用程序安装两个不同的卷，但是其中一个路径必须是只读的。

如果我们使用命令行，我们可以使用`-v`选项:

```java
docker run -d -p 8080:8080 -v first-volume-data:/container-path-1 -v second-volume-data:/container-path-2:ro --name web-app web-app:latest 
```

**我们也可以使用`anonymous`卷，例如，通过包含`-v container-path`。Docker 会为我们创建它，但是一旦我们删除了容器，它就会被删除。**

让我们[检查](https://web.archive.org/web/20221006124104/https://docs.docker.com/engine/reference/commandline/inspect/)我们的集装箱，检查我们的装载是否正确:

```java
docker inspect 0050cda73c6f
```

我们可以看到相关信息，如源和目标、类型，以及第二个卷的只读状态:

```java
"Mounts": [
  {
      "Type": "volume",
      "Name": "first-volume-data",
      "Source": "/var/lib/docker/volumes/first-volume-data/_data",
      "Destination": "/container-path-1",
      "Driver": "local",
      "Mode": "z",
      "RW": true,
      "Propagation": ""
  },
  {
      "Type": "volume",
      "Name": "second-volume-data",
      "Source": "/var/lib/docker/volumes/second-volume-data/_data",
      "Destination": "/container-path-2",
      "Driver": "local",
      "Mode": "z",
      "RW": false,
      "Propagation": ""
  }
]
```

类似地，Docker 建议我们使用`–mount`选项:

```java
docker run -d \
  --name web-app \
  -p 8080:8080 \
  --mount source=first-volume-data,target=/container-path-1 \
  --mount source=second-volume-data,target=/container-path-2,readonly \
  web-app:latest
```

**结果与`-v`选项相同。然而，除了更清晰之外，`–mount` 是我们在[群集](https://web.archive.org/web/20221006124104/https://docs.docker.com/engine/swarm/)模式下使用带有[服务](https://web.archive.org/web/20221006124104/https://docs.docker.com/engine/swarm/how-swarm-mode-works/services/)的卷的唯一方式。**

因此，如果我们想为有多个挂载的 web 应用程序创建一个[服务](https://web.archive.org/web/20221006124104/https://docs.docker.com/engine/reference/commandline/service/)，我们需要使用`–mount`选项:

```java
docker service create --name web-app-service \
  --replicas 3 \
  --publish published=8080,target=80 \
  --mount source=first-volume-data,target=/container-path-1 \
  --mount source=second-volume-data,target=/container-path-2,readonly \
  web-app 
```

同样，我们可以检查我们的服务:

```java
docker service inspect web-app-service
```

同样，我们将在服务规范中获得一些关于容器的信息:

```java
"Mounts": [
  {
      "Type": "volume",
      "Source": "first-volume-data",
      "Target": "/container-path-1"
  },
  {
      "Type": "volume",
      "Source": "second-volume-data",
      "Target": "/container-path-2",
      "ReadOnly": true
  }
]
```

### 2.2.使用卷和绑定装载

我们可能还想在装载到主机中的特定文件夹或文件时使用卷。

假设我们有一个 [MySQL](/web/20221006124104/https://www.baeldung.com/ops/docker-mysql-container) 数据库映像，我们需要运行一个初始脚本来创建一个模式或者用一些数据填充它:

```java
docker run -d \
  --name db \
  -p 3306:3306 \
  --mount source=first-volume-data,target=/var/lib/mysql \
  --mount type=bind,source=/init.sql,target=/docker-entrypoint-initdb.d/init.sql \
  mysql:latest
```

如果我们检查我们的容器，我们现在可以看到两种不同的装载类型:

```java
"Mounts": [
  {
      "Type": "volume",
      "Name": "first-volume-data",
      "Source": "/var/lib/docker/volumes/first-volume-data/_data",
      "Destination": "/var/lib/mysql",
      "Driver": "local",
      "Mode": "z",
      "RW": true,
      "Propagation": ""
  },
  {
      "Type": "bind",
      "Source": "/init.sql",
      "Destination": "/docker-entrypoint-initdb.d/init.sql",
      "Mode": "",
      "RW": true,
      "Propagation": "rprivate"
  }
]
```

### 2.3.使用多个绑定装载

类似地，我们可以使用多个绑定挂载，例如，当我们使用本地 AWS 模拟器 [Localstack](https://web.archive.org/web/20221006124104/https://docs.localstack.cloud/get-started/) 时。

假设我们想要启动一个本地 S3 服务:

```java
docker run --name localstack -d \
  -p 4563-4599:4563-4599 -p 8055:8080 \
  -e SERVICES=s3 -e DEBUG=1 -e DATA_DIR=/tmp/localstack/data \
  -v /.localstack:/var/lib/localstack -v /var/run/docker.sock:/var/run/docker.sock \
  localstack/localstack
```

检查容器时，我们看到主机配置中有多个绑定:

```java
"Binds": [
  "/.localstack:/var/lib/localstack",
  "/var/run/docker.sock:/var/run/docker.sock"
]
```

## 3.复合坞站

让我们来看一个更紧凑的方法，用 [Docker Compose](/web/20221006124104/https://www.baeldung.com/ops/docker-compose) 实现多个挂载。

### 3.1.使用多个卷

首先，让我们从两卷开始，如我们的 YAML 模板所示:

```java
services:
  my_app:
    image: web-app:latest
    container_name: web-app
    ports:
      - "8080:8080"
    volumes:
      - first-volume-data:/container-path-1
      - second-volume-data:/container-path-2:ro

volumes:
  first-volume-data:
    driver: local
  second-volume-data:
    driver: local
```

一旦我们定义了前面创建的卷，我们就可以在服务中以`named-volume:container-path.`的形式添加挂载

[长语法](https://web.archive.org/web/20221006124104/https://docs.docker.com/compose/compose-file/#volumes)也可用于 Docker Compose，例如:

```java
volumes:
  - type: volume
    source: volume-data
    target: /container-path
```

### 3.2.使用卷和绑定装载

同样，这里有一个使用 MySQL 服务的例子:

```java
services:
  mysql-db:
    image: mysql:latest
    environment:
      - MYSQL_ROOT_PASSWORD=password
      - MYSQL_ROOT_HOST=localhost
    ports:
      - '3306:3306'
    volumes:
      - db:/var/lib/mysql
      - ./init.sql:/docker-entrypoint-initdb.d/init.sql

volumes:
  db:
    driver: local
```

### 3.3.使用多个绑定装载

最后，让我们将之前的 Localstack 示例转换为 Docker Compose:

```java
services:
  localstack:
    privileged: true
    image: localstack/localstack:latest
    container_name: localstack
    ports:
      - '4563-4599:4563-4599'
      - '8055:8080'
    environment:
      - SERVICES=s3
      - DEBUG=1
      - DATA_DIR=/tmp/localstack/data
    volumes:
      - './.localstack:/var/lib/localstack'
      - '/var/run/docker.sock:/var/run/docker.sock'
```

## 4.结论

在本文中，我们已经看到了如何使用 Docker 创建多个挂载。

我们已经看到了绑定装载和命名卷的一些组合。我们还看到了使用 Docker 命令行和 Docker Compose 的用例。

一如既往，我们可以在 GitHub 上找到工作代码示例[。](https://web.archive.org/web/20221006124104/https://github.com/eugenp/tutorials/tree/master/docker-modules/docker-compose/)