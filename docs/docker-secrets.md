# 码头工人的秘密介绍

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/ops/docker-secrets>

## 1.介绍

[12 因素应用](https://web.archive.org/web/20221121061416/https://12factor.net/)的众多理念之一是配置应存储在环境中。实际上，**意味着将配置与我们的代码**分开存储。

在本教程中，我们将看看 Docker 秘密如何帮助我们实现这一目标。我们将看看如何创建和管理 Docker 秘密。然后，我们将看看如何在应用程序部署中利用 Docker 秘密。

## 2.什么是秘密？

一般来说，**秘密提供了一种安全存储数据的机制，这些数据可以在运行时被应用程序读取**。机密在将敏感数据与应用程序代码分开存储方面起着至关重要的作用。这包括密码、主机名、SSH 密钥等数据。

例如，假设我们的应用程序需要一个数据库连接。为此，它需要主机名、用户名和密码。此外，还有不同的数据库服务器用于开发、测试和生产。

有了秘密，每个环境都可以向应用程序提供自己的数据库信息。应用程序代码不需要知道它运行在哪个环境中。它只需要一种一致的方法来查找值。

虽然 Docker secrets 是一个相对较新的功能，**大多数云平台多年来一直提供某种形式的秘密**。

例如，亚马逊网络服务和谷歌云都有一个秘密管理器工具，而 Azure 提供了一个密钥库服务。 [Kubernetes](/web/20221121061416/https://www.baeldung.com/ops/kubernetes) 也为秘密提供一流的支持。

Docker 的 secrets 实现使用了许多与前面提到的系统相同的功能:

*   机密是与应用程序分开创建和管理的
*   遵循最低特权和需要知道的访问原则
*   存储各种不同数据类型的灵活性

对 Docker 秘密有了基本的了解，我们来看看如何管理它们。

## 3.管理 Docker 机密

在这一节中，我们将看看如何从创建到删除管理 Docker 机密。

### 3.1.码头工人要求

目前， **Docker secrets 只对群服务**开放。这意味着独立容器不能访问机密。

因此，要使用这些秘密，我们必须使用命令为 swarm 配置我们的集群:

```java
docker swarm init --advertise-addr <MANAGER-IP>
```

其中`<manager-ip>`是 Docker 分配给管理节点的 IP 地址。

在 Windows 和 Mac 的 Docker 桌面上，我们可以简化命令:

```java
docker swarm init
```

有了为 swarm 配置的集群，我们现在可以创建秘密了。

### 3.2.创建 Docker 机密

Docker secrets 几乎可以存储任何可以用字符串或二进制表示的数据类型:

*   用户名和密码
*   主机名和端口
*   SSH 密钥
*   TLS 证书

**唯一真正的限制是数据大小必须小于 500 kb**。

创建密码时，该命令接受来自命令行的输入:

```java
docker secret create my_secret -
```

在这种形式中，该命令允许我们键入秘密的值，甚至支持多行数据。要完成数据输入，我们必须给它一个 EOF 信号(在基于 Unix 的系统上是 Ctrl+D)。

然而，当与自动化流程结合使用时，用键盘手动键入输入不仅容易出错，而且不切实际。因此，**我们也可以用一个文件的内容来创建一个秘密**:

```java
docker secret create my_secret /path/to/secret/file
```

### 3.3.显示 Docker 机密

一旦我们创造了一个秘密，我们可以确认它是成功的:

```java
docker secret ls
ID                          NAME        DRIVER    CREATED          UPDATED
2g9z0nabsi6v7hsfra32unb1o   my_secret             30 minutes ago   30 minutes ago
```

这显示了我们所有的秘密，以及分配给它们的唯一 id。我们也可以检查个人秘密:

```java
docker secret inspect my_secret
[
    {
        "ID": "2g9z0nabsi6v7hsfra32unb1o",
        "Version": {
            "Index": 15
        },
        "CreatedAt": "2022-05-13T00:34:41.2802246Z",
        "UpdatedAt": "2022-05-13T00:34:41.2802246Z",
        "Spec": {
            "Name": "my_secret",
            "Labels": {}
        }
    }
]
```

### 3.4.删除 Docker 机密

一旦不再需要某个秘密，就删除它被认为是一种最佳做法。例如，我们可以从命令行永久删除一个秘密:

```java
docker secret rm my_secret
```

请注意，如果任何服务正在使用该密码，则该命令不起作用。

## 4.使用 Docker 机密

了解了如何管理秘密之后，我们现在可以看看如何在我们的应用程序中实际使用它们。正如 Docker 生态系统中的大多数事情一样，有多种方法可以做到这一点。

### 4.1.码头服务

因为 Docker 机密要求我们的集群处于群体模式，所以我们不能从普通的 Docker `run`命令访问机密。相反，我们必须创建服务，我们可以用命令行指定一个或多个秘密:

```java
docker service create --name my_app --secret my_secret openjdk:19-jdk-alpine
```

### 4.2 .复合坞站

在 Docker Compose 3 和更高版本中，我们有两个使用秘密的选项。下面是定义服务和机密的简单示例:

```java
version: '3.1'
services:
  my_app:
    image: my_app:latest
    secrets:
     - my_external_secret
     - my_file_secret
secrets:
  my_external_secret:
    external: true
  my_file_secret:
    file: /path/to/secret/file.txt
```

在这个例子中，我们定义了两个秘密。第一个是外部的，意思是指使用 Docker `secret`命令创建的秘密。第二个引用一个文件，不需要 Docker 的任何初始设置。

**请记住，使用文件方法绕过了使用 Docker 机密的大部分好处**。此外，该文件必须对服务可能运行的所有主机可用，并且我们必须注意使用 Docker 之外的机制来保护其内容。

### 4.3.获取秘密

Docker 将秘密作为文件提供给我们的应用程序。默认行为是将每个秘密放在目录`/run/secrets`中自己的文件中。使用我们之前的例子，`my_secret`的内容可以在文件`/run/secrets/my_secret`中找到。

我们可以通过使用我们的服务来指定文件的位置来更改它:

```java
docker service create
  --name my_app
  --secret source=my_secret,target=/different/path/to/secret/file,mode=0400
```

如果秘密包含我们的应用程序在特定位置期望的信息，这是有用的。例如，我们可以使用一个秘密来存储一个 Nginx 配置文件，然后使它在标准位置(`/etc/nginx/conf.d/site.conf`)可用。

## 5.结论

对于任何基于容器的架构来说，秘密都是一个重要的工具，因为它们帮助我们实现将代码和配置分开的目标。此外，Docker secrets 提供了一种安全存储敏感数据的方法，并使需要它的应用程序可以使用它。