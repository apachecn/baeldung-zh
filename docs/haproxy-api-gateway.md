# 使用 HAProxy 作为路由和速率限制的 API 网关

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/devops/haproxy-api-gateway>

## 1.概观

在本教程中，我们将学习如何使用 HAProxy 作为路由和速率限制的 API 网关。

## 2.作为 API 网关的 HAProxy

API 网关是位于客户端和众多后端服务之间的应用程序。它就像一个反向代理。它将 API 调用路由到相应的服务。此外，它有能力承担许多责任，例如保护服务、限制 API 调用的速率、监控流量，有时还有负载平衡。

**[HAProxy](https://web.archive.org/web/20221212054534/https://www.haproxy.com/) 是一款开源软件负载平衡器和应用交付控制器**。它非常高效，在工业中得到广泛应用。

在接下来的章节中，我们将配置 HAProxy，使其充当 API 网关。

## 3.使用 HAProxy 的 HTTP API 路由

API 网关的主要职责之一是将 HTTP 请求路由到目的服务器。我们将开始配置 HAProxy，使它开始根据路径、源域和数据格式路由 HTTP 请求。

### 3.1.基本配置

HAProxy 的[基本配置](https://web.archive.org/web/20221212054534/https://www.haproxy.com/blog/haproxy-configuration-basics-load-balance-your-servers/)是一个负载均衡器。我们将为示例定义`frontend`和`backend` :

```java
frontend haproxy_as_api_gateway
    bind 127.0.0.1:80
    default_backend load_balancing
backend load_balancer
    server server1 127.0.0.1:8080
    server server2 127.0.0.1:8081
```

**我们有两台服务器将用于负载平衡**。客户端应用程序将在`127.0.0.1:80`解析请求。从那里，HAProxy 帮助路由到可用的服务器，因为我们已经定义了一个后端`load_balancer`。这个后端位于在`127.0.0.1:8080`和`127.0.0.1:8081`解析的两个服务器之上。

我们将添加进一步的配置，使其在用例方面像 API 网关一样工作。

### 3.2.按路径分离请求

API 网关的基本功能是将 HTTP API 调用路由到微服务架构中的相应服务。

让我们考虑一下，我们已经为我们的在线商店给了一个客户端应用程序两个端点。一个人在终点`127.0.0.1:80/order`接受订单。另一个在端点`127.0.0.1:80/invoicing`管理发票。

对于这两个端点，我们部署了两个独立的微服务。订单服务在`127.0.0.1:8080,`解析，发票服务在`127.0.0.1:8081`解析。

配置如下:

```java
frontend haproxy_as_api_gateway
    bind 127.0.0.1:80
    acl PATH_order path_beg -i /order
    acl PATH_invoicing path_beg -i /invoicing
    use_backend order_service if PATH_order
    use_backend invoicing_service if PATH_invoicing
backend order_service
    server server1 127.0.0.1:8080
backend invoicing_service    
    server server1 127.0.0.1:8081
```

在这种配置中，路径`/order` 将总是在后端`order_service`被解析。类似地，路径`/invoicing`将总是在后端`invoicing_service`被解析。

如果订单服务的一个服务器无法管理负载，我们可以在`127.0.0.1:8090`添加另一个服务器解析，并利用 HAProxy 的负载平衡特性:

```java
frontend haproxy_as_api_gateway
    bind 127.0.0.1:80
    acl PATH_order path_beg -i /order
    acl PATH_invoicing path_beg -i /invoicing
    use_backend order_service if PATH_order
    use_backend invoicing_service if PATH_invoicing
backend order_service
    server server1 127.0.0.1:8080
    server server2 127.0.0.1:8090
backend invoicing_service    
    server server1 127.0.0.1:8081
```

通过这种方式，我们不仅将订单服务路由到不同的位置，还对其进行了负载平衡。

### 3.3.按域分离请求

对于简单的场景，上述配置工作良好。然而，在现实世界中，我们会经历复杂的场景。**一个这样的场景是分离不同域的 API 路径**。

让我们继续网上商店的例子。订单 API 由消费者使用，而发票则由运营团队使用。**消费者网站和运营团队门户**有两个不同的领域。每个域都有各自的 API 路径。

如果消费者试图从网站访问发票，这样的请求不应该得到解决。类似地，来自运营团队门户的订单请求也不应该被解析。

下面是我们实现上述场景的方法:

```java
frontend haproxy_as_api_gateway
    bind :80
    acl consumerapi_host req.hdr(Host) -i -m dom 127.0.0.1
    acl operationapi_host req.hdr(Host) -i -m dom 127.0.0.2
    acl PATH_order path_beg -i /order
    acl PATH_invoicing path_beg -i /invoicing
    use_backend order_service if consumerapi_host PATH_order
    use_backend invoicing_service if operationapi_host PATH_invoicing
backend order_service
    server server1 127.0.0.1:8080
    server server2 127.0.0.1:8090
backend invoicing_service    
    server server1 127.0.0.1:8081
```

有了这个配置，我们已经分别为两个域`127.0.0.1`和`127.0.0.2,` `.` 定义了两个变量`consumerapi_host`和`operationapi_host`

只有当请求来自路径为`order,`的`consumerapi_host`时，HAProxy 才会将请求路由到`order_service`后端。类似地，只有当请求来自路径为`invoicing,`的`operationapi_host`时，它才会将请求路由到`invoicing_service`后端。

### 3.4.按数据格式分离请求

我们可能有这样一个场景，我们必须根据请求的内容类型来分离订单服务。例如，某个客户端可能以`XML`格式发送数据，而另一个客户端以`JSON`格式发送数据。

下面是我们如何满足上述要求:

```java
frontend haproxy_as_api_gateway
    bind :80
    acl consumerapi_host req.hdr(Host) -i -m dom 127.0.0.1
    acl operationapi_host req.hdr(Host) -i -m dom 127.0.0.2
    acl api_json req.hdr(Content-Type) -i -m dom application/json
    acl api_xml req.hdr(Content-Type) -i -m dom application/xml    
    acl PATH_order path_beg -i /order
    acl PATH_invoicing path_beg -i /invoicing
    use_backend order_service_json if consumerapi_host api_json PATH_order
    use_backend order_service_xml if consumerapi_host api_xml PATH_order
    use_backend invoicing_service if operationapi_host PATH_invoicing
backend order_service_json
    server server1 127.0.0.1:8080
backend order_service_xml    
    server server2 127.0.0.1:8090
backend invoicing_service    
    server server1 127.0.0.1:8081
```

我们现在已经为订单服务定义了两个不同的后端。`127.0.0.1:8080`将处理所有的`JSON`请求。`127.0.0.1:8090`会处理所有`XML`的请求。

使用请求头，我们为每个传入的请求定义了两个变量，`api_json`和`api_xml,` 。只有当请求来自消费者域并且路径中有订单时，我们才添加它们的用法。

## 4.限速

速率限制限制了来自客户端的请求数量。在我们的案例中，我们可能希望对订单数量进行费率限制。例如，我们在 10 秒内设置了 100 个订单的限制。除此之外，我们可能会想到一些恶意活动正在进行。或者，我们可能需要添加额外的服务器来管理负载。

首先，我们将创建一个文件`path_param_rates.map`。我们将添加具有各自限制的路径:

```java
/order 100
```

在 HAProxy 中，[棒表](https://web.archive.org/web/20221212054534/https://www.haproxy.com/blog/introduction-to-haproxy-stick-tables/)用于跟踪和保存不同的参数。**棒表是 HAProxy** 内的快速内存存储。它们将与客户端会话相关的数据存储在键/值对中。

在这种情况下，我们将跟踪订单请求发出的次数:

```java
frontend haproxy_as_api_gateway
    bind :80
    stick-table type string size 1m expire 10s store http_rate_limiting
    http-request track-sc0 base32+src
    http-request set-var(req.rate_limit) path,map_beg(path_param_rates.map,20)
    http-request set-var(req.request_rate) base32+src,table_http_rate_limiting()
    acl rate_abuse var(req.rate_limit),sub(req.request_rate) lt 0
    http-request deny deny_status 429 if rate_abuse    

    acl consumerapi_host req.hdr(Host) -i -m dom 127.0.0.1
    acl operationapi_host req.hdr(Host) -i -m dom 127.0.0.2
    acl consumerapi_json req.hdr(Content-Type) -i -m dom application/json
    acl consumerapi_xml req.hdr(Content-Type) -i -m dom application/xml    
    acl PATH_order path_beg -i /order
    acl PATH_invoicing path_beg -i /invoicing
    use_backend order_service_json if consumerapi_host consumerapi_json PATH_order
    use_backend order_service_xml if consumerapi_host consumerapi_xml PATH_order
    use_backend invoicing_service if operationapi_host PATH_invoicing
backend order_service_json
    server server1 127.0.0.1:8080
backend order_service_xml    
    server server2 127.0.0.1:8090
backend invoicing_service    
    server server1 127.0.0.1:8081
```

我们已经创建了一个棒表`http_rate_limiting`。它每 10 秒钟清除一次数据。

**在这种配置下，如果订单请求在 10 秒内超过 100 个，客户端会收到一个错误代码 429。**

## 5.结论

在本文中，我们使用 HAProxy 作为 API 网关。我们已经研究了 HTTP 路由和速率限制的情况。在这些用例中，HAProxy 非常容易配置。