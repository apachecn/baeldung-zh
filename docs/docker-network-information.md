# 从 Docker 获取网络信息

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/ops/docker-network-information>

## 1.概观

Docker 的主要功能之一是创建和隔离网络。

在本教程中，我们将了解如何提取关于网络及其容器的信息。

## 2.Docker 中的网络

当我们运行 Docker 容器时，我们可以定义要向外界公开哪些端口。这意味着我们使用(或创建)一个孤立的网络，并把我们的容器放在里面。我们可以决定如何与这个网络通信，以及如何在这个网络内部通信。

让我们创建几个容器，并在它们之间配置网络。他们将全部在端口 8080 上内部工作，并且他们将被放置在两个网络中。

它们每个都将托管一个简单的“Hello World”HTTP 服务:

```java
version: "3.5"

services:
  test1:
    image: node
    command: node -e "const http = require('http'); http.createServer((req, res) => { res.write('Hello from test1\n'); res.end() }).listen(8080)"
    ports:
      - "8080:8080"
    networks:
      - network1
  test2:
    image: node
    command: node -e "const http = require('http'); http.createServer((req, res) => { res.write('Hello from test2\n'); res.end() }).listen(8080)"
    ports:
      - "8081:8080"
    networks:
      - network1
      - network2
  test3:
    image: node
    command: node -e "const http = require('http'); http.createServer((req, res) => { res.write('Hello from test3\n'); res.end() }).listen(8080)"
    ports:
      - "8082:8080"
    networks:
      - network2

networks:
  network1:
    name: network1
  network2:
    name: network2 
```

以下是这些容器的示意图，以便更直观地展示:

[![](img/cede04647cb22abf8a25c9d6f5f0589c.png)](/web/20220727020703/https://www.baeldung.com/wp-content/uploads/2020/09/docker-compose-bael.png)

让我们用 [`docker-compose`](https://web.archive.org/web/20220727020703/https://docs.docker.com/compose/) 命令启动它们:

```java
$ docker-compose up -d
Starting bael_test2_1 ... done
Starting bael_test3_1 ... done
Starting bael_test1_1 ... done
```

## 3.检查网络

首先，让我们列出所有可用的 Docker 网络:

```java
$ docker network ls
NETWORK ID          NAME                DRIVER              SCOPE
86e6a8138c0d        bridge              bridge              local
73402de5766c        host                host                local
e943f7124776        network1            bridge              local
3b9a28673a16        network2            bridge              local
9361d16a834a        none                null                local
```

我们可以看到`bridge`网络，这是我们使用`docker run`命令时使用的默认网络。此外，我们可以看到我们用`docker-compose` 命令创建的网络。

让我们用`docker inspect`命令来检查它们:

```java
$ docker inspect network1 network2
[
    {
        "Name": "network1",
        "Id": "e943f7124776d45a1481ee26795b2dba3f2ab51f000d875a179a99ce832eee9f",
        "Created": "2020-08-22T10:38:22.198709146Z",
        "Scope": "local",
        "Driver": "bridge",
        "EnableIPv6": false,
        // output cutout for brevity
    }
],
    {
        "Name": "network2",
        // output cutout for brevity
    }
}
```

这将产生冗长、详细的输出。我们很少需要所有这些信息。**幸运的是，我们可以使用 [Go 模板](https://web.archive.org/web/20220727020703/https://golang.org/pkg/text/template/)对其进行格式化，并且只提取符合我们需求的元素。**让我们只得到`network1:`的子网

```java
$ docker inspect -f '{{range .IPAM.Config}}{{.Subnet}}{{end}}' network1
172.22.0.0/16
```

## 4.检查集装箱

同样，我们可以检查特定的容器。首先，让我们列出所有容器及其标识符:

```java
$ docker ps --format 'table {{.ID}}\t{{.Names}}'
CONTAINER ID        NAMES
78c10f03ad89        bael_test2_1
f229dde68f3b        bael_test3_1
b09a8f47e2a8        bael_test1_1
```

**现在我们将使用容器的 ID 作为`inspect`命令的参数来查找其 IP 地址。**与网络类似，我们可以格式化输出以获得我们需要的信息。我们将检查第二个容器及其在我们创建的两个网络中的地址:

```java
$ docker inspect 78c10f03ad89 --format '{{.NetworkSettings.Networks.network1.IPAddress}}'
172.22.0.2
$ docker inspect 78c10f03ad89 --format '{{.NetworkSettings.Networks.network2.IPAddress}}'
172.23.0.3
```

或者，我们可以使用 [`docker exec`](https://web.archive.org/web/20220727020703/https://docs.docker.com/engine/reference/commandline/exec/) 命令直接从容器中打印主机:

```java
$ docker exec 78c10f03ad89 cat /etc/hosts
127.0.0.1	localhost
::1	localhost ip6-localhost ip6-loopback
fe00::0	ip6-localnet
ff00::0	ip6-mcastprefix
ff02::1	ip6-allnodes
ff02::2	ip6-allrouters
172.22.0.2	78c10f03ad89
172.23.0.3	78c10f03ad89
```

## 5.集装箱之间的通信

利用关于我们 Docker 网络的知识，我们可以在同一个网络中的容器之间建立通信。

首先，让我们进入“test1”容器:

```java
$ docker exec -it b09a8f47e2a8 /bin/bash
```

然后，使用 curl 向“test2”容器发送一个请求:

```java
[[email protected]](/web/20220727020703/https://www.baeldung.com/cdn-cgi/l/email-protection):/# curl 172.22.0.2:8080
Hello from test2
```

由于我们在码头工人的网络中，我们也可以使用别名来代替 IP 地址。 Docker 的内置 DNS 服务将为我们解析地址:

```java
[[email protected]](/web/20220727020703/https://www.baeldung.com/cdn-cgi/l/email-protection):/# curl test2:8080
Hello from test2
```

注意，我们不能连接到“test3”容器，因为它在不同的网络中。通过 IP 地址连接将超时:

```java
[[email protected]](/web/20220727020703/https://www.baeldung.com/cdn-cgi/l/email-protection):/# curl 172.23.0.2:8080
```

通过别名连接也会失败，因为 DNS 服务无法识别它:

```java
[[email protected]](/web/20220727020703/https://www.baeldung.com/cdn-cgi/l/email-protection):/# curl test3:8080
curl: (6) Could not resolve host: test3
```

为此，我们需要将“test3”容器添加到“network1”中(从容器外部):

```java
$ docker network connect --alias test3 network1 f229dde68f3b
```

现在对“测试 3”的请求将正确工作:

```java
[[email protected]](/web/20220727020703/https://www.baeldung.com/cdn-cgi/l/email-protection):/# curl test3:8080
Hello from test3
```

## 6.结论

在本教程中，我们看到了如何为 Docker 容器配置网络，然后查询关于它们的信息。