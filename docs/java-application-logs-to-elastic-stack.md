# 将 Java 应用程序的日志发送到弹性堆栈(ELK)

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-application-logs-to-elastic-stack>

## 1。概述

在这个快速教程中，我们将逐步讨论如何将应用程序日志发送到弹性堆栈(ELK)。

在之前的一篇文章中，我们关注于设置弹性堆栈并将 JMX 数据发送到其中。

## 2。配置回退

让我们从配置 Logback 开始，使用`FileAppender`将应用程序日志写入文件:

```java
<appender name="STASH" class="ch.qos.logback.core.rolling.RollingFileAppender">
    <file>logback/redditApp.log</file>
    <rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
        <fileNamePattern>logback/redditApp.%d{yyyy-MM-dd}.log</fileNamePattern>
        <maxHistory>7</maxHistory>
    </rollingPolicy>  
    <encoder class="net.logstash.logback.encoder.LogstashEncoder"/>
</appender>
<root level="DEBUG">
    <appender-ref ref="STASH" />        
</root>
```

请注意:

*   我们通过使用`RollingFileAppender`和`TimeBasedRollingPolicy` 将每天的日志保存在一个单独的文件中(这里有更多关于这个追加器[的信息](/web/20221205113211/http://www.baeldung.com/java-logging-rolling-file-appenders)
*   通过将`maxHistory`设置为 7，我们将只保留旧日志一周(7 天)

另外，注意我们是如何使用 [`LogstashEncoder`](https://web.archive.org/web/20221205113211/https://github.com/logstash/logstash-logback-encoder) 来编码成 JSON 格式的——这在 Logstash 中更容易使用。

为了使用这个编码器，我们需要向我们的`pom.xml`添加以下依赖项:

```java
<dependency> 
    <groupId>net.logstash.logback</groupId> 
    <artifactId>logstash-logback-encoder</artifactId> 
    <version>4.11</version> 
</dependency>
```

最后，让我们确保应用程序有权访问日志目录:

```java
sudo chmod a+rwx /var/lib/tomcat8/logback
```

## 3。配置日志存储

现在，我们需要配置 Logstash 从我们的应用程序创建的日志文件中读取数据，并将其发送到 ElasticSearch。

这是我们的配置文件`logback.conf`:

```java
input {
    file {
        path => "/var/lib/tomcat8/logback/*.log"
        codec => "json"
        type => "logback"
    }
}

output {
    if [type]=="logback" {
         elasticsearch {
             hosts => [ "localhost:9200" ]
             index => "logback-%{+YYYY.MM.dd}"
        }
    }
}
```

请注意:

*   输入`file`用作 Logstash 这次将从日志文件中读取日志
*   `path`设置为我们的日志目录和所有文件。将处理日志扩展
*   `index`被设置为新索引“logback-%{+YYYY。MM.dd} "而不是默认的" logstash-%{+YYYY。MM.dd} "

要使用新配置运行 Logstash，我们将使用:

```java
bin/logstash -f logback.conf
```

## 4。使用 Kibana 可视化日志

我们现在可以在'`logback-*`'索引中看到我们的回退数据。

我们将创建一个新的搜索“回退日志”，以确保通过使用以下查询来分离回退数据:

```java
type:logback
```

最后，我们可以为我们的回溯数据创建一个简单的可视化:

*   导航到“可视化”选项卡
*   选择“垂直条形图”
*   选择“从保存的搜索中”
*   选择我们刚刚创建的“日志回溯”搜索

对于 Y 轴，确保选择聚合:`Count`

对于 X 轴，选择:

*   聚合:`Terms`
*   字段:`level`

在运行可视化之后，您应该看到多个条代表每个级别的日志计数(调试、信息、错误等)

## 5。结论

在本文中，我们学习了在我们的系统中设置 Logstash 的基本知识，以便将它生成的日志数据推送到 Elasticsearch 中——并在 Kibana 的帮助下可视化这些数据。