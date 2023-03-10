# Docker 编写中 Expose 和 Ports 的区别

> 原文::1230]https://web . archive . org/web/202209930061024/https://www . BAE message . com/ops/docker-compose-expose-vs-ports

## 1.概观

正如我们所知， [Docker Compose](/web/20221006104139/https://www.baeldung.com/ops/docker-compose) 是一个一次定义和管理多个容器的工具。默认情况下，Docker Compose 为定义的容器建立一个专用网络，使它们之间能够通信。因此，我们可以使用一个命令创建和运行带有给定配置文件的服务。

在本教程中，我们将看到两个 YAML 属性，它们允许我们定制容器之间的网络连接——*expose*和`ports`。我们将详细描述它们，展示基本用例，并强调它们的主要区别。

## 2.`expose`一节

首先，我们来看一下`expose`的配置。该属性定义 Docker Compose 从容器中公开的端口。

这些**端口将被连接到同一网络的其他服务访问，但不会在主机**上发布。

我们可以通过在`services`部分指定端口号来公开端口:

```java
services:
  myapp1:
    ...
    expose:
      - "3000"
      - "8000"
  myapp2:
    ...
    expose:
      - "5000"
```

正如我们看到的，我们可以为每个服务指定多个值。我们刚刚公开了来自`myapp1` 容器的端口`3000`和`8000`，以及来自`myapp2` 容器的端口`5000`。对于同一网络中的其他容器，现在可以在这些端口上访问这些服务。

现在让我们检查暴露的端口:

```java
> docker ps
CONTAINER ID   IMAGE    COMMAND     CREATED     STATUS      PORTS               NAMES
8673c14f18d1   ...      ...         ...         ...         3000/tcp, 8000/tcp  bael_myapp1
bc044e180131   ...      ...         ...         ...         5000/tcp            bael_myapp2
```

在`docker ps`命令输出中，我们可以在`PORTS `列中找到暴露的端口。

最后，让我们验证容器之间的通信:

```java
> docker exec -it bc044e180131 /bin/bash

bash-5.1$ nc -vz myapp1 3000
myapp1 (172.18.0.1:3000) open
bash-5.1$ nc -vz myapp1 8000
myapp1 (172.18.0.1:8000) open
```

我们刚刚连接到`myapp2` 命令行界面。使用 [`netcat`](/web/20221006104139/https://www.baeldung.com/linux/netcat-command) 命令，我们检查了从`myapp1`公开的两个端口都可以到达。

## 3.*端口*部分

现在让我们检查一下`ports`部分。和以前一样，这个属性定义了我们希望从容器中公开的端口。但是与`expose`配置不同的是，**这些端口将是内部可访问的，并在主机**上发布。

和以前一样，我们可以在专用部分为每个服务定义端口，但是配置可能会更复杂。首先，我们必须在两种语法(短语法和长语法)之间做出选择来定义配置。

### 3.1.短语法

先从分析短的开始。**短语法是一个冒号分隔的字符串，用于设置主机 IP 地址、主机端口和容器端口**:

```java
[HOST:]CONTAINER[/PROTOCOL]
```

这里的`, HOST` 是一个主机端口号或一系列端口号，其前面可以有一个 IP 地址。如果我们不指定 IP 地址，Docker Compose 会将端口绑定到所有网络接口。

C `ONTAINER`定义一个容器端口号或端口号范围。

`PROTOCOL`将容器端口限制为指定的协议，如果为空，则将它们设置为`TCP`。只有`CONTAINER `部分是强制的。

现在我们知道了语法，让我们在 Docker 编写文件中定义端口:

```java
services:
  myapp1:
    ...
    ports:
    - "3000"                             # container port (3000), assigned to random host port
    - "3001-3005"                        # container port range (3001-3005), assigned to random host ports
    - "8000:8000"                        # container port (8000), assigned to given host port (8000)
    - "9090-9091:8080-8081"              # container port range (8080-8081), assigned to given host port range (9090-9091)
    - "127.0.0.1:8002:8002"              # container port (8002), assigned to given host port (8002) and bind to 127.0.0.1
    - "6060:6060/udp"                    # container port (6060) restricted to UDP protocol, assigned to given host (6060)
```

如上所述，我们还可以一次发布多个容器端口，使用短语法的不同变体并更精确地配置它。Docker Compose 公开了所有指定的容器端口，使它们可以从本地机器内部和外部访问。

像以前一样，让我们用`docker ps`命令检查暴露的端口:

```java
> docker ps -a
CONTAINER ID   ... PORTS                                                                        NAMES
e8c65b9eec91   ... 0.0.0.0:51060->3000/tcp, 0.0.0.0:51063->3001/tcp, 0.0.0.0:51064->3002/tcp,   bael_myapp1
                   0.0.0.0:51065->3003/tcp, 0.0.0.0:51061->3004/tcp, 0.0.0.0:51062->3005/tcp, 
                   0.0.0.0:8000->8000/tcp, 0.0.0.0:9090->8080/tcp, 0.0.0.0:9091->8081/tcp
                   127.0.0.1:8002->8002/tcp, 0.0.0.0:6060->6060/udp
```

同样，在`PORTS`列中，我们可以找到所有暴露的端口。箭头左边的值显示了我们可以从外部访问容器的主机地址。

### 3.2.长语法

使用长语法，我们可以用同样的方式配置端口。然而，我们没有使用冒号分隔的字符串，而是单独定义每个属性:

```java
services: 
  myapp1:
  ...
  ports:
  # - "127.0.0.1:6060:6060/udp"
  - target: 6060
    host_ip: 127.0.0.1
    published: 6060
    protocol: udp
    mode: host
```

这里，`target`是强制的，指定了将要公开的容器端口(或端口范围),相当于短语法中的`CONTAINER`。

`host_ip `和`published`是简短的`HOST`的一部分，在这里我们可以定义主机的 IP 地址和端口。

`protocol,`与短语法中的`PROTOCOL`相同，将容器端口限制为指定的协议(如果为空，则为`TCP`)。

`mode`是具有两个值的枚举，指定端口发布规则。我们应该使用`host`值在本地发布一个端口。第二个值—`ingress`—是为更复杂的集装箱环境保留的，意味着港口将进行负载平衡。

总之，任何短的语法字符串都可以很容易地用长结构来表示。然而，由于缺少`mode`属性，并不是所有的长语法配置都可以移动到短语法配置中。

## 4.结论

在本文中，我们刚刚讨论了 Docker Compose 中的一部分网络配置。我们使用`expose`和`ports`部分分析和比较了端口配置。

**`expose`部分允许我们将容器中的特定端口只暴露给同一网络**上的其他服务。我们可以简单地通过指定容器端口来做到这一点。

**`ports`部分还公开了容器**的指定端口。与前一个`,` **不同，端口不仅对同一网络上的其他服务开放，也对主机**开放。配置稍微复杂一点，我们可以配置公开的端口、本地绑定地址和受限协议。根据我们的喜好，我们可以在两种不同的语法之间进行选择。