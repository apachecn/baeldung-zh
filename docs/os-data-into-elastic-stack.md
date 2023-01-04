# 将操作系统数据发送到弹性堆栈(ELK 堆栈)

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/ops/os-data-into-elastic-stack>

## 1.概观

在这个快速教程中，我们将讨论如何将 OS 级指标发送到 Elastic Stack。作为参考，我们将在这里使用 Ubuntu 服务器。

我们将使用 Metricbeat 从操作系统中收集数据，并定期发送给 Elasticsearch。

如果您对将其他类型的数据发送到 es 实例感兴趣，我们之前讨论过 [JMX 数据](/web/20220524124513/https://www.baeldung.com/tomcat-jmx-elastic-stack)和[应用程序日志](/web/20220524124513/https://www.baeldung.com/java-application-logs-to-elastic-stack)。

## 2.安装 Metricbeat

首先，我们需要在我们的 Ubuntu 机器上下载并安装标准的 [Metricbeat](https://web.archive.org/web/20220524124513/https://www.elastic.co/guide/en/beats/metricbeat/current/metricbeat-installation.html) 代理:

```java
curl -L -O https://artifacts.elastic.co/downloads/beats/metricbeat/metricbeat-6.0.1-amd64.deb
sudo dpkg -i metricbeat-6.0.1-amd64.deb
```

安装后，我们需要配置 Metricbeat 向 Elasticsearch 发送数据，方法是修改位于`/etc/metricbeat/`(在 Ubuntu 上)的`metricbeat.yml`:

```java
output.elasticsearch:
  hosts: ["localhost:9200"]
```

然后，我们可以通过修改`/etc/metricbeat/modules.d/system.yml`来定制我们想要跟踪的指标:

```java
- module: system
  period: 10s
  metricsets:
    - cpu
    - load
    - memory
    - network
    - process
    - process_summary
```

最后，我们将启动 Metricbeat 服务:

```java
sudo service metricbeat start
```

## 3.快速检查

为了确保 Metricbeat 正在向 Elasticsearch 发送数据，请快速检查索引:

```java
curl -X GET 'http://localhost:9200/_cat/indices'
```

以下是您应该获得的内容:

```java
yellow open metricbeat-6.0.1-2017.12.11 1 1  2185 0   1.7mb   1.7mb
```

现在，我们将从“设置”选项卡使用模式“`metricbeat-*`”创建新索引

## 4.可视化操作系统指标

现在，我们将随着时间的推移可视化我们的内存使用情况。

首先，我们将使用以下名为“System Memory”的查询在“`metricbeat-*`”索引上创建一个新的搜索，以分离我们的内存指标:

```java
metricset.name:memory
```

最后，我们可以创建一个简单的内存数据可视化:

*   导航到“可视化”选项卡
*   选择“折线图”
*   选择“从保存的搜索中”
*   选择我们刚刚创建的“系统内存”搜索

对于 Y 轴，选择:

*   聚集:平均值
*   字段:system.memory.used.pct

对于 X 轴，选择聚集:日期直方图

## 5.结论

在这篇简明扼要的文章中，我们学习了如何使用 Metricbeat 将 OS 级数据发送到弹性堆栈实例中。