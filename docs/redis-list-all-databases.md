# 列出所有重定向数据库

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/redis-list-all-databases>

## 1。简介

在这个简短的教程中，我们将看看在 [Redis](https://web.archive.org/web/20220625222057/https://redis.io/) 中列出所有可用数据库的不同方式。

## 2。列出所有数据库

首先，**Redis 中的数据库数量是固定的**。因此，我们可以用一个简单的 [`grep`](/web/20220625222057/https://www.baeldung.com/linux/common-text-search) 命令从配置文件中提取这些信息:

```java
$ cat redis.conf | grep databases
databases 16
```

但是如果我们没有访问配置文件的权限呢？在这种情况下，我们可以通过 [`redis-cli`](https://web.archive.org/web/20220625222057/https://redis.io/topics/rediscli) 在运行时读取配置来获取我们需要的信息:

```java
127.0.0.1:6379> CONFIG GET databases
1) "databases"
2) "16"
```

最后，尽管它更适合底层应用程序，但我们可以通过 telnet 连接使用 [Redis 序列化协议(RESP)](https://web.archive.org/web/20220625222057/https://redis.io/topics/protocol) :

```java
$ telnet 127.0.0.1 6379
Trying 127.0.0.1...
Connected to 127.0.0.1.
Escape character is '^]'.
*3
$6
CONFIG
$3
GET
$9
databases
*2
$9
databases
$2
16
```

## 3。列出包含条目的所有数据库

有时我们会想获得更多关于包含键的数据库的信息。为了做到这一点，**我们可以利用 Redis `INFO`命令，用来获取关于服务器**的信息和统计数据。这里，我们特别希望将注意力集中在`keyspace`部分，它包含与数据库相关的数据:

```java
127.0.0.1:6379> INFO keyspace
# Keyspace
db0:keys=2,expires=0,avg_ttl=0
db1:keys=4,expires=0,avg_ttl=0
db2:keys=9,expires=0,avg_ttl=0 
```

输出列出了至少包含一个键的数据库，以及一些统计信息:

*   包含的密钥数
*   过期密钥的数量
*   密钥的平均生存时间

## 4。结论

总之，本文介绍了在 Redis 中列出数据库的不同方法。正如我们所看到的，有不同的解决方案，我们选择哪一个取决于我们想要达到的目标。

如果我们可以访问配置文件，那么`grep`通常是最好的选择。否则，我们可以使用`redis-cli`。RESP 通常不是一个好的选择，除非我们正在构建一个需要底层协议的应用程序。最后，如果我们只想检索包含键的数据库，那么`INFO`命令非常有用。