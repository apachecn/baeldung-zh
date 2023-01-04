# Docker 编写重启策略

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/ops/docker-compose-restart-policies>

## 1.概观

在本教程中，我们将学习如何通过 [Docker Compose](/web/20220926125221/https://www.baeldung.com/ops/docker-compose) 使用重启策略。

首先，我们将介绍如何用重启策略重启 Docker 容器。然后，我们将介绍 Docker Compose 如何在普通模式和群体模式下定义重启策略，作为多容器 Docker 应用程序的配置。

## 2.Docker 重启策略

重启策略是我们可以用来自动重启 Docker 容器并管理其生命周期的策略。

考虑到容器可能会意外失败， **Docker 有安全措施来防止服务进入重启循环**。出现故障时，除非容器成功运行至少 10 秒钟，否则重启策略不会生效。

我们还可以假设，当提供了重启策略时，手动停止容器将使 Docker 自动重启服务。但是，在这种情况下，这些策略被取消，以防止容器在被任意停止后重新启动。

要使用重启策略，Docker 提供了以下选项:

*   `no`:集装箱不会自动重启
*   `on-failure[:max-retries]`:如果容器以非零退出代码退出，则重启容器，并为 Docker 守护程序提供重启容器的最大尝试次数
*   `always` **:** 如果容器停止，总是重新启动
*   `unless-stopped`:总是重启容器，除非它被任意停止或者被 Docker 守护进程停止

现在，让我们看看如何使用 Docker CLI 为单个容器设置重启策略的示例:

```java
docker run --restart always my-service
```

**从上面的例子来看，如果容器停止运行，`my-service`将`always` 重新启动。**然而，**如果我们显式地停止容器，重启策略只有在 Docker 守护进程重启或者我们使用 [`restart `命令](/web/20220926125221/https://www.baeldung.com/ops/docker-compose-restart-container)时才会生效。**

前面的例子演示了`restart` 标志如何配置策略来自动重启单个容器。然而， **Docker Compose 允许我们通过在普通模式或群组模式下使用`restart` 或`restart_policy` 属性来配置重启策略以管理多个容器。**

## 3.设置

在开始使用 Docker Compose 实现重启策略之前，让我们设置一个工作环境。

我们必须有一个正在运行的 Docker 容器来测试我们将要指定的重启策略。我们将使用前一篇文章中的一个项目， [`spring-cloud-docker`](https://web.archive.org/web/20220926125221/https://github.com/eugenp/tutorials/tree/master/spring-cloud-modules/spring-cloud-docker) ，它是一个 dockered Spring Boot 应用程序。这个项目有两个 Docker 服务，我们将使用它们通过 Docker Compose 实现不同的重启策略。

首先，我们必须通过从项目根目录运行以下命令来确认我们可以运行这两个容器:

```java
docker-compose up --detach --build
```

现在，我们应该能够通过执行`docker-compose ps`看到两个服务都在运行:

```java
$ docker ps
     Name                   Command              State            Ports         
--------------------------------------------------------------------------------
message-server   java -jar /message-server.jar   Up      0.0.0.0:18888->8888/tcp
product-server   java -jar /product-server.jar   Up      0.0.0.0:19999->9999/tcp 
```

此外，我们可以转到浏览器中的`localhost:18888`或`localhost:19999`，验证我们是否看到了应用服务显示的消息。

## 4.在 Docker 编写中重新启动策略

与`restart ` Docker 命令一样， **Docker Compose 包含了 `restart` 属性来自动重启容器。** 

此外，我们可以在 Docker Compose 中通过在`docker-compose.yml` 文件中向服务提供`restart`属性来定义重启策略。 **Docker Compose 使用 Docker CLI `restart`命令提供的相同值来定义重启策略。**

现在，让我们为容器创建一个重启策略。在 [`spring-cloud-docker`](https://web.archive.org/web/20220926125221/https://github.com/eugenp/tutorials/tree/master/spring-cloud/spring-cloud-docker) 项目中，我们必须通过添加重启策略属性来更改`docker-compose.yml`配置文件。例如:

```java
message-server:
    container_name: message-server
    build:
        context: docker-message-server
        dockerfile: Dockerfile
    image: message-server:latest
    ports:
        - 18888:8888
    networks:
        - spring-cloud-network
    restart: no
product-server:
    container_name: product-server
    build:
        context: docker-product-server
        dockerfile: Dockerfile
    image: product-server:latest
    ports:
        - 19999:9999
    networks:
        - spring-cloud-network
    restart: on-failure
```

注意我们是如何将`restart`属性添加到两个服务中的。在这种情况下，`message-server`永远不会自动重启。只有当`on-failure` 值指定的非零代码退出时，`product-server `才会重启。

接下来，让我们看看如何在 swarm 模式下使用 Docker Compose 声明相同的策略。

## 5.Docker 编写群模式下的重启策略

在指定容器如何自动重启时，群组模式下的 Docker Compose 提供了更大的选项集。然而，下面的实现只在 Docker Compose v3 中有效，它在配置中引入了`deploy`键值对。

下面，我们可以找到不同的属性来进一步扩展群模式下重启策略的配置:

*   `condition`**:**`none``on-failure,` 或`any` (默认)
*   `delay` **:** 重启尝试之间的持续时间
*   `max_attempts` **:** 重启以外的最大尝试次数`window`
*   `window` **:** 判断重启是否成功的持续时间

现在，让我们定义我们的重启策略。首先，我们必须确保我们正在使用 Docker Compose v3，方法是像这样更改`version` 属性:

```java
version: '3'
```

一旦我们更改了版本，我们就可以将`restart_policy` 属性添加到我们的服务中。与上一节类似，我们的`message-server` 容器将总是通过在`condition`中提供`any` 值来自动重启，如下所示:

```java
deploy:
    restart_policy:
        condition: any
        delay: 5s
        max_attempts: 3
        window: 120s
```

类似地，我们向`product-server`添加一个`on-failure`重启策略:

```java
deploy:
    restart_policy:
        condition: on-failure
        delay: 3s
        max_attempts: 5
        window: 60s
```

**注意`restart_policy` 属性是如何在`deploy` 键中的，这表明重启策略是我们提供的一个部署配置，用于以 swarm 模式管理一个容器集群。**

此外，两种服务中的重启策略都包括额外的配置元数据，这使得策略成为容器的更健壮的重启策略。

## 6.结论

在本文中，我们学习了如何用 Docker Compose 定义重启策略。在介绍了 Docker 重启策略之后，我们使用了之前的 Baeldung 项目来介绍为容器配置重启策略的两种方法。

首先，我们使用 Docker Compose `restart` 属性配置，它包含与 Docker CLI 中的本机`restart` 命令相同的选项。最后，我们使用了仅在 swarm 模式和 Docker Compose 版本 3 中可用的`restart_policy` 属性，以及定义重启策略的其他配置值。

和往常一样，本教程中使用的所有示例代码都可以在 GitHub 上的[处获得。](https://web.archive.org/web/20220926125221/https://github.com/eugenp/tutorials/tree/master/spring-cloud-modules/spring-cloud-docker)