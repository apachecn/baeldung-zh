# 设置自定义假装客户端超时

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/feign-timeout>

## 1.介绍

Spring Cloud Feign Client 是一个方便的声明式 REST 客户端，我们用它来实现微服务之间的通信。

在这个简短的教程中，我们将展示如何设置一个自定义的假装客户端连接超时，包括全局和每个客户端。

## 2.默认

假装客户端是非常可配置的。

就超时而言，它允许我们配置读取和连接超时。连接超时是 TCP 握手所需的时间，而读取超时是从套接字读取数据所需的时间。

默认情况下，连接和读取超时分别为 10 秒和 60 秒。

## 3.全球的

我们可以通过在我们的`application.yml`文件中设置的`feign.client.config.` `default` 属性，设置应用于应用程序中每个虚拟客户端的连接和读取超时:

```
feign:
  client:
    config:
      default:
        connectTimeout: 60000
        readTimeout: 10000
```

**这些值代表超时发生前的毫秒数。**

## 4.每个客户端

还可以通过命名客户端:来设置每个特定客户端的超时值

```
feign:
  client:
    config:
      FooClient:
        connectTimeout: 10000
        readTimeout: 20000
```

当然，我们可以一起列出全局设置和每个客户端的覆盖，这没有问题。

## 5.结论

在本教程中，我们解释了如何调整假装客户端超时以及如何通过`application.yml`文件设置自定义值。跟随[我们的主要假装介绍](/web/20220630124251/https://www.baeldung.com/spring-cloud-openfeign#properties)随意尝试这些。**