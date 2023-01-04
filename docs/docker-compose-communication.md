# 多个 Docker 合成项目之间通信

> 原文::1230【https://web . archive . org/web/202209930061024/https://www . BAE message . com/ops/docker-compose-communication】

## 1.概观

我们通常使用单个`docker-compose.yml`文件来引用 Docker Compose。然而，我们可能需要使用多个 YAML 文件，并且仍然能够让运行的容器成为同一个网络的一部分。

在这个简短的教程中，我们将通过一些例子来看看如何使用一个`network`来连接多个 Docker Compose 项目。

## 2.将网络用于多个 Docker 合成项目

自从 [Docker Compose](/web/20220904150434/https://www.baeldung.com/ops/docker-compose) 引入 [networking](https://web.archive.org/web/20220904150434/https://docs.docker.com/compose/networking/) 以来，我们可以让我们的容器知道一个现有的网络，并让它们加入其中。

例如，假设我们希望我们的 Redis 缓存和 web 应用程序属于同一个网络，但是它们是两个不同的 YAML 文件。

### 2.1.加入现有网络

首先，让我们创建一个网络:

```java
docker network create network-example
```

然后，我们可以在 YAML 模板中定义对现有网络的引用。

让我们看看我们的 Redis 定义:

```java
services:
  db:
    image: redis:latest
    container_name: redis
    ports:
      - '6379:6379'
    networks:
      - network-example

networks:
  network-example:
    external: true
```

让我们也在不同的文件中为我们的 web 应用程序定义它:

```java
services:
  my_app:
    image: "web-app:latest"
    container_name: web-app
    ports:
      - "8080:8080"
    networks:
      - network-example

networks:
  network-example:
    external: true
```

### 2.2.在 YAML 模板中定义网络

**同样，我们也可以在模板内部定义一个`network`，比如我们的`redis_network` :**

```java
services:
  db:
    image: redis:latest
    container_name: redis
    ports:
      - '6379:6379'
    networks:
      - network

networks:
  network:
    driver: bridge
    name: redis_network
```

这一次，当我们设置我们的 web 应用程序模板时，我们需要参考 Redis 现有的网络:

```java
services:
  my_app:
    image: "web-app:latest"
    container_name: web-app
    ports:
      - "8080:8080"
    networks:
      - my-app

networks:
  my-app:
    name: redis_network
    external: true
```

## 3.运行和检查容器

最后，我们可以使用 [up](https://web.archive.org/web/20220904150434/https://docs.docker.com/engine/reference/commandline/compose_up/) 或 [down](https://web.archive.org/web/20220904150434/https://docs.docker.com/engine/reference/commandline/compose_down/) 命令启动或停止我们的服务。

如果我们在一个服务定义中创建一个网络，我们需要首先启动那个服务，就像我们例子中的 Redis 一样:

```java
docker-compose -f docker-compose-redis-service.yaml up -d && docker-compose -f docker-compose-my-app-service.yaml up -d
```

让我们[检查](https://web.archive.org/web/20220904150434/https://docs.docker.com/engine/reference/commandline/inspect/)一个正在运行的容器以查看网络定义，例如，我们的 Redis 服务:

```java
docker inspect 5c7f8b28480b
```

我们将看到`redis_network`的一个条目。对于 web-app 检查，我们将得到相同的输出:

```java
"Networks": {
    "redis_network": {
        "IPAMConfig": null,
        "Links": null,
        "Aliases": [
            "redis",
            "4d23d918eb2c"
        ],
        "NetworkID": "e122aa15d5ad150a66d87f3145084520bde540447a14a73f446ec6ea0603aba9",
        "EndpointID": "8904a3389d0b20c6785884c702cb6ae1101522af1f99c079067171cbc9ca97e5",
        "Gateway": "172.28.0.1",
        "IPAddress": "172.28.0.2",
        "IPPrefixLen": 16,
        "IPv6Gateway": "",
        "GlobalIPv6Address": "",
        "GlobalIPv6PrefixLen": 0,
        "MacAddress": "02:42:ac:1c:00:02",
        "DriverOpts": null
    }
}
```

同样，我们可以考察一下`redis_network`:

```java
docker network inspect redis_network
```

我们的两种服务属于同一个网络:

```java
"Containers": {
    "5c7f8b28480ba638ce993c6714841265c0a98d746b27205a756936bbe1850ac2": {
        "Name": "redis",
        "EndpointID": "238bb136d634100eccb7677e87ba07eb43f33be1dc320c795685230f04b809f9",
        "MacAddress": "02:42:ac:1c:00:02",
        "IPv4Address": "172.28.0.2/16",
        "IPv6Address": ""
    },
    "8584ce3bb2ab1cd3d182346c67179a3aa5e40c71e806c35cc4ce7ea91cae7902": {
        "Name": "web-app",
        "EndpointID": "9cf0e484e5af1baf968249c312489a83f57a194098a51652c3f6eac19ed0d557",
        "MacAddress": "02:42:ac:1c:00:03",
        "IPv4Address": "172.28.0.3/16",
        "IPv6Address": ""
    }
}
```

## 4.结论

在本文中，我们看到了如何在同一个网络上连接多个 Docker Compose 服务。我们可以让他们加入现有的网络，或者根据服务定义创建一个网络。

和往常一样，我们可以在 GitHub 上找到我们的示例[的源文件`docker-compose.yml`。](https://web.archive.org/web/20220904150434/https://github.com/eugenp/tutorials/tree/master/docker-modules/docker-compose/)