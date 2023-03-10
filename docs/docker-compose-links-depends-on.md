# Docker 编写中链接和依赖的区别

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/ops/docker-compose-links-depends-on>

## 1.概观

Docker 容器在我们的系统中作为独立的进程运行。然而，我们通常希望它们相互交流和传递信息。

在本教程中，我们将通过一些使用 Docker Compose 的实际例子来了解 Docker `links`和`depends_on`之间的区别。

## 2.坞站组成〔t0〕

**[`depends_on`](https://web.archive.org/web/20220810172900/https://docs.docker.com/compose/compose-file/compose-file-v3/#depends_on) 是一个 [Docker Compose](/web/20220810172900/https://www.baeldung.com/ops/docker-compose) 关键字，用来设置服务必须启动和停止的顺序。**

例如，假设我们希望我们的 web 应用程序(我们将构建为一个`web-app`图像)在我们的 [Postgres](/web/20220810172900/https://www.baeldung.com/ops/postgresql-docker-setup) 容器之后启动。我们来看看`docker-compose.yml`文件:

```java
services:
  db:
    image: postgres:latest
    environment:
      - POSTGRES_USER=postgres
      - POSTGRES_PASSWORD=postgres
    ports:
      - 5432:5432
  web-app:
    image: web-app:latest
    ports:
      - 8080:8080
    depends_on:
      - db
```

Docker 将根据给定的依赖关系提取图像并运行容器。因此，在这种情况下，Postgres 容器是队列中第一个运行的。

**然而，有一些限制，因为`depends_on` 没有明确地等待依赖项准备好。**

假设我们的 web 应用程序需要在启动时运行一些迁移脚本。如果数据库不接受连接，尽管 Postgres 服务已经正确启动，我们也不能执行任何脚本。

然而，如果我们使用特定的工具或我们自己的托管脚本来控制启动或关闭顺序，我们就可以避免这种情况。

## 3.坞站组成〔t0〕

**[`links`](https://web.archive.org/web/20220810172900/https://docs.docker.com/network/links/) 指示码头工人通过网络链接集装箱。当我们链接容器时，Docker 创建环境变量并将容器添加到已知主机列表中，以便它们可以发现彼此。**

我们将检查一个运行 Postgres 容器的简单 Docker 示例，并将其链接到我们的 web 应用程序。

首先，让我们运行 Postgres 容器:

```java
docker run -d --name db -p 5342:5342 postgres:latest 
```

然后，我们将它链接到我们的 web 应用程序:

```java
docker run -d -p 8080:8080 --name web-app --link db 
```

让我们将示例转换为 Docker Compose:

```java
services:
  db:
    image: postgres:latest
    environment:
      - POSTGRES_USER=postgres
      - POSTGRES_PASSWORD=postgres
    ports:
      - 5432:5432
  web-app:
    images: web-app:latest
    ports:
      - 8080:8080
    links:
      - db
```

## 4.坞站组成〔t0〕

**我们可以找到仍在使用的 Docker `links`。然而，由于引入了`[network](https://web.archive.org/web/20220810172900/https://docs.docker.com/network/)`，Docker Compose 从版本 2 开始就不再使用它了。**

这样，我们可以将应用程序与复杂的网络连接起来，例如，[覆盖](https://web.archive.org/web/20220810172900/https://docs.docker.com/network/overlay/)网络。

然而，在一个独立的应用程序中，当我们不指定网络时，我们通常可以使用一个[桥](https://web.archive.org/web/20220810172900/https://docs.docker.com/network/bridge/)作为默认。

让我们删除`links `并用`network`替换它，同时为数据库添加一个卷和环境变量:

```java
services:
  db:
    image: postgres:latest
    restart: always
    environment:
      - POSTGRES_USER=postgres
      - POSTGRES_PASSWORD=postgres
    ports:
      - 5432:5432
    volumes: 
      - db:/var/lib/postgresql/data
    networks:
      - mynet

  web-app:
    image:web-app:latest
    depends_on:
      - db
    networks:
      - mynet
    ports:
      - 8080:8080
    environment:
      DB_HOST: db
      DB_PORT: 5432
      DB_USER: postgres
      DB_PASSWORD: postgres
      DB_NAME: postgres

networks:
  mynet:
    driver: bridge

volumes:
  db:
    driver: local
```

## 5.Docker `links`和`depends_on`的区别

虽然它们涉及到表达依赖关系，Docker `links`和`depends_on`有不同的含义。

**`depends_on`表示服务必须启动和停止的顺序，而`links`关键字处理网络上容器的通信。**

此外，`depends_on`是 Docker Compose 关键字，而我们可以类似地使用`links`作为 Docker 的遗留特性。

## 6.结论

在本文中，我们已经通过 Docker Compose 示例`.`看到了 Docker `links`和`depends_on`之间的区别

`depends_on`告诉 Docker 运行容器的顺序，而`links,`或新版 Docker Compose 中的`network`通过网络为容器设置连接。

和往常一样，我们可以在 GitHub 上找到我们的示例[的源文件`docker-compose.yml`。](https://web.archive.org/web/20220810172900/https://github.com/eugenp/tutorials/tree/master/docker-modules/docker-compose)