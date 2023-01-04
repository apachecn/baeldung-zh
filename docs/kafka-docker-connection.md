# 连接到运行在 Docker 中的 Apache Kafka

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/kafka-docker-connection>

## 1.概观

Apache Kafka 是一个非常受欢迎的事件流媒体平台，经常与 [Docker](https://web.archive.org/web/20220529025840/https://www.docker.com/) 一起使用。通常，人们会遇到 Kafka 的连接建立问题，尤其是当客户端不在同一个 Docker 网络或同一个主机上运行时。这主要是由于卡夫卡的广告听众的错误配置。

在本教程中，我们将学习如何配置监听器，以便客户端可以连接到 Docker 中运行的 Kafka broker。

## 2.设置卡夫卡

在我们尝试建立连接之前，我们需要使用 Docker 运行一个 [Kafka 代理。下面是我们的](/web/20220529025840/https://www.baeldung.com/ops/kafka-docker-setup) [docker-compose.yaml](/web/20220529025840/https://www.baeldung.com/ops/docker-compose) 文件的一个片段:

```
version: '2'
services:
  zookeeper:
    container_name: zookeeper
    networks: 
      - kafka_network
    ...

  kafka:
    container_name: kafka
    networks: 
      - kafka_network
    ports:
      - 29092:29092
    environment:
      KAFKA_LISTENERS: EXTERNAL_SAME_HOST://:29092,INTERNAL://:9092
      KAFKA_ADVERTISED_LISTENERS: INTERNAL://kafka:9092,EXTERNAL_SAME_HOST://localhost:29092
      KAFKA_LISTENER_SECURITY_PROTOCOL_MAP: INTERNAL:PLAINTEXT,EXTERNAL_SAME_HOST:PLAINTEXT
      KAFKA_INTER_BROKER_LISTENER_NAME: INTERNAL
     ... 

networks:
  kafka_network:
    name: kafka_docker_example_net
```

在这里，我们定义了两个必备服务——Kafka 和 Zookeeper。我们还定义了一个定制网络–`kafka_docker_example_net,` ,我们的服务将使用该网络`.`

稍后我们将更详细地查看`KAFKA_LISTENERS`、`KAFKA_ADVERTISED_LISTENERS,`和`KAFKA_LISTENER_SECURITY_PROTOCOL_MAP`属性。

使用上面的`docker-compose.yaml` 文件，我们启动服务:

```
docker-compose up -d
Creating network "kafka_docker_example_net" with the default driver
Creating zookeeper ... done
Creating kafka ... done
```

此外，我们将使用 Kafka [控制台生成器](https://web.archive.org/web/20220529025840/https://kafka-tutorials.confluent.io/kafka-console-consumer-producer-basics/kafka.html)实用程序作为样本客户端来测试与 Kafka 代理的连接。要在没有 Docker 的情况下使用 Kafka-console-producer 脚本，我们需要下载 [Kafka](https://web.archive.org/web/20220529025840/https://kafka.apache.org/downloads) 。

## 3.听众

在与 Kafka brokers 连接时，侦听器、广告侦听器和侦听器协议起着相当大的作用。

我们用`KAFKA_LISTENERS`属性管理监听器，在这里我们声明了一个逗号分隔的 URIs 列表，它指定了代理应该监听传入 TCP 连接的套接字。

每个 URI 包括一个协议名，后跟一个接口地址和一个端口:

```
EXTERNAL_SAME_HOST://0.0.0.0:29092,INTERNAL://0.0.0.0:9092
```

这里，我们指定了一个`0.0.0.0`元地址来将套接字绑定到所有接口。此外，`EXTERNAL_SAME_HOST`和`INTERNAL`是我们在 URI 格式中定义侦听器时需要指定的定制侦听器名称。

### 3.2.拔靴带

对于初始连接，Kafka 客户端需要一个引导服务器列表，我们在其中指定代理的地址。该列表应该包含至少一个指向群集中随机代理的有效地址。

客户端将使用该地址连接到代理。如果连接成功，代理将返回关于集群的元数据，包括集群中所有代理的公告侦听器列表。对于后续的连接，客户端将使用该列表来联系代理。

### 3.3.广告听众

仅仅声明侦听器是不够的，因为它只是代理的一个套接字配置。我们需要一种方法来告诉客户(消费者和生产者)如何连接到卡夫卡。

在`KAFKA_ADVERTISED_LISTENERS`属性的帮助下，这就是被广告的侦听器进入画面的地方。它的格式类似于侦听器的属性:

**<监听器协议> :// <通告主机名> : <通告端口>**

在初始引导过程**之后，客户端使用被指定为广告监听器的地址。**

### 3.4.监听器安全协议映射

除了侦听器和广告侦听器之外，我们还需要告诉客户端在连接到 Kafka 时要使用的安全协议。在`KAFKA_LISTENER_SECURITY_PROTOCOL_MAP,` 中，我们将自定义协议名称映射到有效的安全协议。

在上一节的配置中，我们声明了两个自定义协议名称——`INTERNAL`和`EXTERNAL_SAME_HOST.` ,我们可以随意命名它们，但是我们需要将它们映射到有效的安全协议。

我们指定的安全协议之一是`PLAINTEXT`，这意味着客户端不需要向 Kafka 代理进行认证。此外，交换的数据没有加密。

## 4.从同一 Docker 网络连接的客户端

让我们从另一个容器启动 Kafka 控制台生成器，并尝试向代理生成消息:

```
docker run -it --rm --network kafka_docker_example_net confluentinc/cp-kafka /bin/kafka-console-producer --bootstrap-server kafka:9092 --topic test_topic
>hello
>world
```

这里，我们将这个容器附加到现有的`kafka_docker_example_net` 网络上，以便与我们的代理自由通信。我们还指定代理的地址–`kafka:9092 `和主题的名称，这将自动创建。

我们能够为主题生成消息，这意味着与代理的连接是成功的。

## 5.客户端从同一台主机连接

当客户机没有容器化时，让我们从主机连接到代理。对于外部连接，我们公布了`EXTERNAL_SAME_HOST`监听器，我们可以用它来建立来自主机的连接。从公布的侦听器属性中，我们知道我们必须使用`localhost:29092 `地址来到达 Kafka broker。

为了测试同一台主机的连接性，我们将使用一个非 Dockerized Kafka 控制台生成器:

```
kafka-console-producer --bootstrap-server localhost:29092 --topic test_topic_2
>hi
>there 
```

由于我们成功地生成了主题，这意味着最初的引导和随后的到代理的连接(客户端使用广告的侦听器)都是成功的。

我们之前在 `docker-compose.yaml`中配置的端口号 29092 使得 Kafka 代理可以在 Docker 之外访问。

## 6.客户端从不同的主机连接

如果 Kafka 代理运行在不同的主机上，我们如何连接到它？不幸的是，我们不能重用现有的侦听器，因为它们只用于同一个 Docker 网络或主机连接。因此，我们需要定义一个新的侦听器并公布它:

```
KAFKA_LISTENERS: EXTERNAL_SAME_HOST://:29092,EXTERNAL_DIFFERENT_HOST://:29093,INTERNAL://:9092
KAFKA_ADVERTISED_LISTENERS: INTERNAL://kafka:9092,EXTERNAL_SAME_HOST://localhost:29092,EXTERNAL_DIFFERENT_HOST://157.245.80.232:29093
KAFKA_LISTENER_SECURITY_PROTOCOL_MAP: INTERNAL:PLAINTEXT,EXTERNAL_SAME_HOST:PLAINTEXT,EXTERNAL_DIFFERENT_HOST:PLAINTEXT 
```

我们创建了一个名为`EXTERNAL_DIFFERENT_HOST`的新监听器，使用安全协议`PLAINTEXT` 和端口 29093 关联`KAFKA_ADVERTISED_LISTENERS`中的`.` ，我们还添加了运行 Kafka 的云机器的 IP 地址。

我们必须记住，我们不能使用`localhost` ，因为我们是从不同的机器连接的(在这种情况下是本地工作站)。此外，端口 29093 发布在 ports 部分下，因此可以在 Docker 之外访问它。

让我们尝试生成一些消息:

```
kafka-console-producer --bootstrap-server 157.245.80.232:29093 --topic test_topic_3
>hello
>REMOTE SERVER
```

我们可以看到，我们能够成功地连接到 Kafka 代理并生成消息。

## 7.结论

在本文中，我们学习了如何配置监听器，以便客户端可以连接到 Docker 中运行的 Kafka 代理。我们研究了不同的场景，其中客户端运行在相同的 Docker 网络、相同的主机、不同的主机等。我们看到侦听器、广告侦听器和安全协议映射的配置决定了连接性。