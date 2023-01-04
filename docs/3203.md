# 使用 Shell 命令列出 Kafka 集群中的活动代理

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/ops/kafka-list-active-brokers-in-cluster>

## 1.概观

监控使用 Apache Kafka 集群的事件驱动系统通常需要我们获取活动代理的列表。在本教程中，我们将探索几个 **shell 命令来获取正在运行的集群中的活动代理**的列表。

## 2.设置

出于本文的目的，让我们使用下面的 **`docker-compose.yml`文件来建立一个双节点 Kafka 集群**:

```
$ cat docker-compose.yml
---
version: '2'
services:
  zookeeper-1:
    image: confluentinc/cp-zookeeper:latest
    environment:
      ZOOKEEPER_CLIENT_PORT: 2181
      ZOOKEEPER_TICK_TIME: 2000
    ports:
      - 2181:2181

  kafka-1:
    image: confluentinc/cp-kafka:latest
    depends_on:
      - zookeeper-1
    ports:
      - 29092:29092
    environment:
      KAFKA_BROKER_ID: 1
      KAFKA_ZOOKEEPER_CONNECT: zookeeper-1:2181
      KAFKA_ADVERTISED_LISTENERS: PLAINTEXT://kafka-1:9092,PLAINTEXT_HOST://localhost:29092
      KAFKA_LISTENER_SECURITY_PROTOCOL_MAP: PLAINTEXT:PLAINTEXT,PLAINTEXT_HOST:PLAINTEXT
      KAFKA_INTER_BROKER_LISTENER_NAME: PLAINTEXT
      KAFKA_OFFSETS_TOPIC_REPLICATION_FACTOR: 1
  kafka-2:
    image: confluentinc/cp-kafka:latest
    depends_on:
      - zookeeper-1
    ports:
      - 39092:39092
    environment:
      KAFKA_BROKER_ID: 2
      KAFKA_ZOOKEEPER_CONNECT: zookeeper-1:2181
      KAFKA_ADVERTISED_LISTENERS: PLAINTEXT://kafka-2:9092,PLAINTEXT_HOST://localhost:39092
      KAFKA_LISTENER_SECURITY_PROTOCOL_MAP: PLAINTEXT:PLAINTEXT,PLAINTEXT_HOST:PLAINTEXT
      KAFKA_INTER_BROKER_LISTENER_NAME: PLAINTEXT
      KAFKA_OFFSETS_TOPIC_REPLICATION_FACTOR: 1 
```

现在，让我们使用 [`docker-compose`](/web/20220526054627/https://www.baeldung.com/ops/docker-compose) 命令加速 Kafka 集群:

```
$ docker-compose up -d
```

我们可以验证 Zookeeper 服务器正在监听端口`2181`，而 Kafka 代理正在分别监听端口`29092`和`39092,`:

```
$ ports=(2181 29092 39092)
$ for port in $ports
do
nc -z localhost $port
done
Connection to localhost port 2181 [tcp/eforward] succeeded!
Connection to localhost port 29092 [tcp/*] succeeded!
Connection to localhost port 39092 [tcp/*] succeeded!
```

## 3.使用 Zookeeper APIs

在 Kafka 集群中， **[Zookeeper 服务器](/web/20220526054627/https://www.baeldung.com/java-zookeeper)存储与 Kafka broker 服务器**相关的元数据。因此，让我们使用 Zookeeper 公开的文件系统 API 来获取代理细节。

### 3.1.`zookeeper-shell`命令

大多数 Kafka 发行版都附带了`zookeeper-shell`或`zookeeper-shell.sh`二进制文件。因此，使用这个二进制文件与 Zookeeper 服务器进行交互是一个事实上的标准。

首先，让我们连接到运行于`localhost:2181`的 Zookeeper 服务器:

```
$ /usr/local/bin/zookeeper-shell localhost:2181
Connecting to localhost:2181
Welcome to ZooKeeper! 
```

一旦我们连接到 Zookeeper 服务器，我们就可以执行**典型的文件系统命令，比如`ls`来获取存储在服务器中的元数据信息**。让我们找到目前还活着的经纪人的 id:

```
ls /brokers/ids
[1, 2]
```

我们可以看到目前有两个活跃的代理，id 分别为 1 和 2。使用`get`命令，我们可以获取具有给定 id 的特定代理的更多细节:

```
get /brokers/ids/1
{"features":{},"listener_security_protocol_map":{"PLAINTEXT":"PLAINTEXT","PLAINTEXT_HOST":"PLAINTEXT"},"endpoints":["PLAINTEXT://kafka-1:9092","PLAINTEXT_HOST://localhost:29092"],"jmx_port":-1,"port":9092,"host":"kafka-1","version":5,"timestamp":"1625336133848"}
get /brokers/ids/2
{"features":{},"listener_security_protocol_map":{"PLAINTEXT":"PLAINTEXT","PLAINTEXT_HOST":"PLAINTEXT"},"endpoints":["PLAINTEXT://kafka-2:9092","PLAINTEXT_HOST://localhost:39092"],"jmx_port":-1,"port":9092,"host":"kafka-2","version":5,"timestamp":"1625336133967"}
```

请注意，`id=1`代理正在监听端口`29092`，而第二个`id=2`代理正在监听端口`39092`。

最后，要退出 Zookeeper shell，我们可以使用`quit`命令:

```
quit
```

### 3.2.`zkCli`命令

就像 Kafka 发行版附带了`zookeeper-shell`二进制文件一样，Zookeeper 发行版附带了`zkCli`或`zkCli.sh`二进制文件。

因此，**与`zkCli`的交互就像与`zookeeper-shell`** 的交互一样，所以让我们继续确认我们能够通过`id=1`获得经纪人所需的详细信息:

```
$ zkCli -server localhost:2181 get /brokers/ids/1
Connecting to localhost:2181

WATCHER::

WatchedEvent state:SyncConnected type:None path:null
{"features":{},"listener_security_protocol_map":{"PLAINTEXT":"PLAINTEXT","PLAINTEXT_HOST":"PLAINTEXT"},"endpoints":["PLAINTEXT://kafka-1:9092","PLAINTEXT_HOST://localhost:29092"],"jmx_port":-1,"port":9092,"host":"kafka-1","version":5,"timestamp":"1625336133848"}
```

正如所料，我们可以看到使用`zookeeper-shell`获取的代理细节与使用`zkCli`获取的相匹配。

## 4.使用代理版本 API

有时，我们可能**有一个不完整的活跃经纪人列表，**，我们希望**得到集群中所有可用的经纪人**。在这种情况下，我们可以**使用 Kafka 发行版附带的`kafka-broker-api-versions`命令**。

假设我们知道一个代理在`localhost:29092`运行，那么让我们试着找出参与 Kafka 集群的所有活动代理:

```
$ kafka-broker-api-versions --bootstrap-server localhost:29092 | awk '/id/{print $1}'
localhost:39092
localhost:29092
```

值得注意的是，我们使用了`[awk](/web/20220526054627/https://www.baeldung.com/linux/awk-guide)`命令来过滤输出，只显示代理地址。此外，结果正确地显示了集群中有两个活动的代理。

尽管这种方法看起来比 Zookeeper CLI 方法简单，但`kafka-broker-api-versions `二进制文件只是 Kafka 发行版的一个新成员。

## 5.命令过程

在任何实际场景中，**为每个经纪人手动执行`zkCli`或`zookeeper-shell`命令将会增加**的负担。因此，让我们编写一个 [Shell 脚本](/web/20220526054627/https://www.baeldung.com/linux/linux-scripting-series)，它将 Zookeeper 服务器地址作为输入，作为回报，它给出了所有活动代理的列表。

### 5.1.助手功能

让我们在`functions.sh` 脚本中编写所有的**助手函数:**

```
$ cat functions.sh
#!/bin/bash
ZOOKEEPER_SERVER="${1:-localhost:2181}"

# Helper Functions Below
```

首先，让我们编写 **`get_broker_ids`函数来获取将在内部调用`zkCli`命令的一组活动代理 id**:

```
function get_broker_ids {
broker_ids_out=$(zkCli -server $ZOOKEEPER_SERVER <<EOF
ls /brokers/ids
quit
EOF
)
broker_ids_csv="$(echo "${broker_ids_out}" | grep '^\[.*\]

接下来，让我们编写 **`get_broker_details`函数，使用`broker_id`获取详细的代理细节**:

```
function get_broker_details {
broker_id="$1"
echo "$(zkCli -server $ZOOKEEPER_SERVER <<EOF
get /brokers/ids/$broker_id
quit
EOF
)"
}
```

现在我们有了详细的代理细节，让我们编写`parse_broker_endpoint`函数来获取代理的端点细节:

```
function parse_endpoint_detail {
broker_detail="$1"
json="$(echo "$broker_detail"  | grep '^{.*}

在内部，我们使用 **[`jq`](/web/20220526054627/https://www.baeldung.com/linux/jq-command-json) 命令对**进行 JSON 解析。

### 5.2.主脚本

现在，让我们编写使用`functions.sh`中定义的助手函数的主脚本`get_all_active_brokers.sh`:

```
$ cat get_all_active_brokers.sh
#!/bin/bash
. functions.sh "$1"

function get_all_active_brokers {
broker_ids=$(get_broker_ids)
for broker_id in $broker_ids
do
    broker_details="$(get_broker_details $broker_id)"
    broker_endpoint=$(parse_endpoint_detail "$broker_details")
    echo "broker_id="$broker_id,"endpoint="$broker_endpoint
done
}

get_all_active_brokers
```

我们可以注意到**我们已经遍历了`get_all_active_brokers`函数中的所有`broker_ids`** 来聚合所有活动代理的端点。

最后，让我们执行`get_all_active_brokers.sh`脚本，这样我们就可以看到双节点 Kafka 集群的活动代理列表:

```
$ ./get_all_active_brokers.sh localhost:2181
broker_id=1,endpoint="PLAINTEXT_HOST://localhost:29092"
broker_id=2,endpoint="PLAINTEXT_HOST://localhost:39092"
```

我们可以看到结果是准确的。看起来我们成功了！

## 6.结论

在本教程中，我们学习了 shell 命令，如`zookeeper-shell`、`zkCli`和`kafka-broker-api-versions `来获取 Kafka 集群中活动代理的列表。此外，我们编写了一个 **shell 脚本来自动执行在真实场景中查找代理细节**的过程。)"
echo "$broker_ids_csv" | sed 's/\[//;s/]//;s/,/ /'
}
```

接下来，让我们编写 **`get_broker_details`函数，使用`broker_id`获取详细的代理细节**:

[PRE11]

现在我们有了详细的代理细节，让我们编写`parse_broker_endpoint`函数来获取代理的端点细节:

[PRE12]

在内部，我们使用 **[`jq`](/web/20220526054627/https://www.baeldung.com/linux/jq-command-json) 命令对**进行 JSON 解析。

### 5.2.主脚本

现在，让我们编写使用`functions.sh`中定义的助手函数的主脚本`get_all_active_brokers.sh`:

[PRE13]

我们可以注意到**我们已经遍历了`get_all_active_brokers`函数中的所有`broker_ids`** 来聚合所有活动代理的端点。

最后，让我们执行`get_all_active_brokers.sh`脚本，这样我们就可以看到双节点 Kafka 集群的活动代理列表:

[PRE14]

我们可以看到结果是准确的。看起来我们成功了！

## 6.结论

在本教程中，我们学习了 shell 命令，如`zookeeper-shell`、`zkCli`和`kafka-broker-api-versions `来获取 Kafka 集群中活动代理的列表。此外，我们编写了一个 **shell 脚本来自动执行在真实场景中查找代理细节**的过程。)"
json_endpoints="$(echo $json | jq .endpoints)"
echo "$(echo $json_endpoints |jq . |  grep HOST | tr -d " ")"
}
```

在内部，我们使用 **[`jq`](/web/20220526054627/https://www.baeldung.com/linux/jq-command-json) 命令对**进行 JSON 解析。

### 5.2.主脚本

现在，让我们编写使用`functions.sh`中定义的助手函数的主脚本`get_all_active_brokers.sh`:

[PRE13]

我们可以注意到**我们已经遍历了`get_all_active_brokers`函数中的所有`broker_ids`** 来聚合所有活动代理的端点。

最后，让我们执行`get_all_active_brokers.sh`脚本，这样我们就可以看到双节点 Kafka 集群的活动代理列表:

[PRE14]

我们可以看到结果是准确的。看起来我们成功了！

## 6.结论

在本教程中，我们学习了 shell 命令，如`zookeeper-shell`、`zkCli`和`kafka-broker-api-versions `来获取 Kafka 集群中活动代理的列表。此外，我们编写了一个 **shell 脚本来自动执行在真实场景中查找代理细节**的过程。)"
echo "$broker_ids_csv" | sed 's/\[//;s/]//;s/,/ /'
}
```

接下来，让我们编写 **`get_broker_details`函数，使用`broker_id`获取详细的代理细节**:

[PRE11]

现在我们有了详细的代理细节，让我们编写`parse_broker_endpoint`函数来获取代理的端点细节:

[PRE12]

在内部，我们使用 **[`jq`](/web/20220526054627/https://www.baeldung.com/linux/jq-command-json) 命令对**进行 JSON 解析。

### 5.2.主脚本

现在，让我们编写使用`functions.sh`中定义的助手函数的主脚本`get_all_active_brokers.sh`:

[PRE13]

我们可以注意到**我们已经遍历了`get_all_active_brokers`函数中的所有`broker_ids`** 来聚合所有活动代理的端点。

最后，让我们执行`get_all_active_brokers.sh`脚本，这样我们就可以看到双节点 Kafka 集群的活动代理列表:

[PRE14]

我们可以看到结果是准确的。看起来我们成功了！

## 6.结论

在本教程中，我们学习了 shell 命令，如`zookeeper-shell`、`zkCli`和`kafka-broker-api-versions `来获取 Kafka 集群中活动代理的列表。此外，我们编写了一个 **shell 脚本来自动执行在真实场景中查找代理细节**的过程。