# 将静态 IP 分配给 Docker 容器和 Docker-Compose

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/ops/docker-assign-static-ip-container>

## 1.概观

当我们运行 Docker 容器时，它使用一个 IP 地址连接一个虚拟网络。出于这个原因，我们希望服务能够动态地获取配置。但是，我们可能希望使用静态 IP，而不是自动 IP 分配。

在本教程中，我们将看到内置配置和为容器手动分配 IP 之间的区别。最后，我们将添加一些带有测试的 Docker 组合示例。

## 2.DHCP 和 DNS

让我们来看看 [Docker](/web/20221115175831/https://www.baeldung.com/ops/docker-guide) 使用 DHCP 和 DNS 解析主机名称的内置 IPs 分配给容器。

### 2.1.Docker 如何分配 IP

Docker 首先给每个容器分配一个 [IP](https://web.archive.org/web/20221115175831/https://docs.docker.com/config/containers/container-networking/#ip-address-and-hostname) ，作为 DHCP 服务器。此外，还有多个 DNS 服务器。

然后，容器通过 [`dockerd`](https://web.archive.org/web/20221115175831/https://docs.docker.com/engine/reference/commandline/dockerd/) 内部的服务器处理 DNS 请求，该服务器识别同一内部网络上其他容器的名称。这样，容器可以在不知道其内部 IP 地址的情况下进行通信。尽管每次当应用程序启动时，内部 IP 地址可能不同，但是由于`dockerd`中的内部 DNS 服务器，容器仍然可以很容易地使用人类可读的名称进行连接。

**然后，`dockerd`向[核心域名系统](https://web.archive.org/web/20221115175831/https://coredns.io/)(来自 [CNCF](https://web.archive.org/web/20221115175831/https://www.cncf.io/) )发送域名查询。最后，请求根据域名转移到主机。**

领域`docker.internal`还有一个旁门左道。它包括解析为当前主机的有效 IP 地址的 DNS 名称`host.docker.internal`。它允许容器联系那些主机服务，而不用担心硬编码 IP 地址。虽然不推荐，但对于开发来说，这很方便。

### 2.2.网络示例

例如，我们可以为 MySQL 服务运行一个容器。让我们来看看 [Docker 作曲](/web/20221115175831/https://www.baeldung.com/ops/docker-compose) YAML 的定义:

```java
services:
  db:
    image: mysql:latest
    environment:
      - MYSQL_ROOT_PASSWORD=password
      - MYSQL_ROOT_HOST=localhost
    ports:
      - 3306:3306
    volumes:
      - db:/var/lib/mysql
    networks:
      - network

volumes:
  db:
    driver: local

networks:
  network:
    driver: bridge
```

像往常一样，我们运行我们的容器:

```java
docker-compose up -d
```

让我们从容器的角度用`format` 语法检查网络，使用`[jq](/web/20221115175831/https://www.baeldung.com/linux/jq-command-json)`获得 JSON 输出:

```java
docker inspect --format='{{json .NetworkSettings.Networks}}' 2d3f4c69a213 | jq .
```

Docker Compose 根据当前目录分配网络名称。例如，如果我们在`project`目录中，我们可以看到类似的输出:

```java
{
  "project_network": {
    "IPAMConfig": null,
    "Links": null,
    "Aliases": [
      "project-db-1",
      "db",
      "2d3f4c69a213"
    ],
    "NetworkID": "39ffbd8155d11ba03d0b548307f549f06790fe045e121a6d862b070d4fb67fa7",
    "EndpointID": "0eba235239b06f7e0cb5065b7f2ebd83e7d227f8cfad4df8de73260472737500",
    "Gateway": "172.19.0.1",
    "IPAddress": "172.19.0.2",
    "IPPrefixLen": 16,
    "IPv6Gateway": "",
    "GlobalIPv6Address": "",
    "GlobalIPv6PrefixLen": 0,
    "MacAddress": "02:42:ac:13:00:02",
    "DriverOpts": null
  }
} 
```

容器从网络创建的子网中获得一个私有的`172.19.0.2` IP 地址。

**最重要的是，我们可以看到关于`IPAMConfig`的信息，也就是 IP 地址管理。当我们静态分配 IP 时，它将是相关的。**

现在，我们可以检查网络:

```java
docker inspect network project_network
```

这一次，我们对网络有了更深入的了解:

```java
[
    {
        "Name": "project_network",
        "Id": "39ffbd8155d11ba03d0b548307f549f06790fe045e121a6d862b070d4fb67fa7",
        "Created": "2022-09-09T16:19:26.27396468+02:00",
        "Scope": "local",
        "Driver": "bridge",
        "EnableIPv6": false,
        "IPAM": {
            "Driver": "default",
            "Options": null,
            "Config": [
                {
                    "Subnet": "172.19.0.0/16",
                    "Gateway": "172.19.0.1"
                }
            ]
        },
        "Internal": false,
        "Attachable": false,
        "Ingress": false,
        "ConfigFrom": {
            "Network": ""
        },
        "ConfigOnly": false,
        "Containers": {
            "2d3f4c69a2139dea9089a6d42907fdc085282c5df176b39bf7c20f5d0780179d": {
                "Name": "project-db-1",
                "EndpointID": "7447fe2550afb3f980f36449673724e9ed6dd16f41a085cc20ada3074a0d8e54",
                "MacAddress": "02:42:ac:13:00:02",
                "IPv4Address": "172.19.0.2/16",
                "IPv6Address": ""
            }
        },
        "Options": {},
        "Labels": {
            "com.docker.compose.network": "network",
            "com.docker.compose.project": "project",
            "com.docker.compose.version": "2.10.2"
        }
    }
]
```

值得注意的是，Docker Compose [网络](https://web.archive.org/web/20221115175831/https://docs.docker.com/compose/networking/)从版本 2 开始就可用了。

## 3.静态 IP

了解了更多关于自动 IP 分配的知识，我们现在将创建网络的子网。然后，我们可以将我们喜欢的 IP 分配给我们的服务。

### 3.1.分配一个静态 IP

如果我们使用 Docker CLI，我们将通过首先创建子网来实现这一结果:

```java
docker network create --subnet=10.5.0.0/16 mynet 
```

然后，我们使用静态 IP 运行容器，同样使用 MySQL 服务:

```java
docker run --net mynet --ip 10.5.0.1 -p 3306:3306 --mount source=db,target=/var/lib/mysql -e MYSQL_ROOT_PASSWORD=password mysql:latest
```

我们可以使用 Docker Compose 来总结一个完整的示例:

```java
services:
  db:
    container_name: mysql_db
    image: mysql:latest
    environment:
      - MYSQL_ROOT_PASSWORD=password
      - MYSQL_ROOT_HOST=10.5.0.1
    ports:
      - 3306:3306
    volumes:
      - db:/var/lib/mysql
      - ./init.sql:/docker-entrypoint-initdb.d/init.sql
    networks:
      network:
        ipv4_address: 10.5.0.5

volumes:
  db:
    driver: local

networks:
  network:
    driver: bridge
    ipam:
      config:
        - subnet: 10.5.0.0/16
          gateway: 10.5.0.1
```

我们现在已经通过关键字`ipam`定义了我们的[网络](https://web.archive.org/web/20221115175831/https://docs.docker.com/compose/compose-file/compose-file-v3/#networks)的子网，并为服务分配了一个 IPv4 地址。为了改变，我们使用了`10.5.0.5`。172.*和 10。* IP 地址通常用于专用网络。我们也可以使用 [IPv6](https://web.archive.org/web/20221115175831/https://docs.docker.com/config/daemon/ipv6/) 地址，它有 128 位地址长度，由于效率更高，将取代 IPv4。

按照建议，我们将网关地址分配给数据库主机`MYSQL_ROOT_HOST`。

最后，我们添加一个 SQL 脚本来创建一个用户、一个数据库和一个表:

```java
CREATE DATABASE IF NOT EXISTS test;
CREATE USER 'db_user'@'10.5.0.1' IDENTIFIED BY 'password';
GRANT ALL PRIVILEGES ON *.* TO 'db_user'@'10.5.0.1' WITH GRANT OPTION;
FLUSH PRIVILEGES;

use test;

CREATE TABLE IF NOT EXISTS TEST_TABLE (id int, name varchar(255));

INSERT INTO TEST_TABLE VALUES (1, 'TEST_1');
INSERT INTO TEST_TABLE VALUES (2, 'TEST_2');
INSERT INTO TEST_TABLE VALUES (3, 'TEST_3');
```

我们想让用户只在那个特定的地址访问数据库。

容器启动后，我们可以用 [`docker ps`](https://web.archive.org/web/20221115175831/https://docs.docker.com/engine/reference/commandline/ps/) 来看看它的定义:

```java
CONTAINER ID   IMAGE          COMMAND                  CREATED         STATUS         PORTS                                                  NAMES
97812e199512   mysql:latest   "docker-entrypoint.s…"   7 minutes ago   Up 7 minutes   0.0.0.0:3306->3306/tcp, :::3306->3306/tcp, 33060/tcp   mysql_db 
```

我们现在可以通过输入密码连接到数据库。我们使用容器名称或 ID 作为 DNS 的别名:

```java
mysql --host=mysql_db -u db_user -p
```

现在，使用`status`命令，我们可以测试我们的 MySQL 主机解析为容器 ID:

```java
Connection id:          10
Current database:       test
Current user:           [[email protected]](/web/20221115175831/https://www.baeldung.com/cdn-cgi/l/email-protection)
SSL:                    Not in use
Current pager:          stdout
Using outfile:          ''
Using delimiter:        ;
Server:                 MySQL
Server version:         8.0.30 MySQL Community Server - GPL
Protocol version:       10
Connection:             97812e199512 via TCP/IP
Server characterset:    utf8mb4
Db     characterset:    utf8mb4
Client characterset:    utf8mb3
Conn.  characterset:    utf8mb3
TCP port:               3306 
```

### 3.2.与内置 Docker IP 管理的区别

让我们检查一下集装箱。在静态 IP 的情况下，我们可以看到`IPAM`配置现在有了一个 IPv4 地址:

```java
{
  "project_network": {
    "IPAMConfig": {
      "IPv4Address": "10.5.0.5"
    },
    "Links": null,
    "Aliases": [
      "project_db",
      "db",
      "122c0c6bfcf9"
    ],
    "NetworkID": "7ac7a1d9e33dffc65bc867aee4db04b9b8fecaeb3bbb91c74c2f72e4611c6955",
    "EndpointID": "84145191a0327b777b6a31bacb2a0260d9a31e8c22cbfca1923775b3649b1d7e",
    "Gateway": "10.5.0.1",
    "IPAddress": "10.5.0.5",
    "IPPrefixLen": 16,
    "IPv6Gateway": "",
    "GlobalIPv6Address": "",
    "GlobalIPv6PrefixLen": 0,
    "MacAddress": "02:42:0a:05:00:05",
    "DriverOpts": null
  }
} 
```

从容器的角度来看，这是主要的区别。

如果我们需要一个静态的私有 IP 地址，我们应该考虑是否需要使用它。大多数时候，我们希望一个静态 IP 从另一个容器或主机与一个容器对话。Docker 的内置联网已经可以处理这个了。

但是，我们可能希望手动指定一个私有 IP 地址，例如，用于直接从主机访问容器。

值得注意的是使用 [Docker Swarm](https://web.archive.org/web/20221115175831/https://docs.docker.com/engine/swarm/networking/) 定制网络的可能性。

## 4.结论

在本文中，我们看到了 Docker 如何管理 IP 分配，以及如何向容器添加静态地址。我们还看到了 Docker Compose 配置在有或没有静态 IP 的情况下运行 MySQL 服务的例子。

一如既往，我们可以在 GitHub 上找到工作代码示例[。](https://web.archive.org/web/20221115175831/https://github.com/eugenp/tutorials/tree/master/docker-modules/docker-compose/)