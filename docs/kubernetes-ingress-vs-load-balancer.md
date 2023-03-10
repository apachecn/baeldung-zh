# Kubernetes 中的入口与负载平衡器

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/ops/kubernetes-ingress-vs-load-balancer>

## 1.介绍

对于许多软件应用程序来说，与外部服务通信是完成其任务所必需的。无论是发送消息还是使用 API，**大多数应用程序都依赖其他系统才能正常运行**。

然而，随着越来越多的用户将他们的应用程序转移到 Kubernetes，提供安全可靠的访问变得越来越具有挑战性。浏览各种部署和服务会使网络流量难以到达正确的位置。

幸运的是，有许多不同的机制可以帮助管理网络流量，并确保请求到达集群中的预期目的地。在本教程中，我们将研究其中的两种机制:入口和负载平衡器。

## 2.工作负载和服务

在我们讨论这些之前，我们必须先回过头来看看应用程序是如何在 Kubernetes 中部署和管理的。

### 2.1.工作负载和 pod

我们首先将应用程序打包成 docker 映像。这些 docker 映像随后用于创建预定义的[工作负载](https://web.archive.org/web/20220928104813/https://kubernetes.io/docs/concepts/workloads/)类型之一，例如:

*   `ReplicaSet`:确保最少数量的`pods`随时可用
*   `StatefulSet`:当增加或减少`pods`的数量时，提供可预测的和唯一的排序
*   `DameonSet`:确保特定数量的`pods`始终在部分或全部`nodes`上运行

**所有这些工作负载都会在集群**中部署一个或多个`pods`。A `pod`是我们可以在 Kubernetes 中使用的最小可部署单元。它本质上代表了在集群中某处运行的应用程序。

默认情况下，每个`pod`都有一个唯一的 IP，可供集群的其他成员访问。然而，**使用 IP 地址访问`pod`并不是一个好的做法**，原因有很多。

首先，提前知道哪个 IP 将被分配给一个`pod`并不容易。这使得在其他应用程序可以访问的配置中存储 IP 信息几乎是不可能的。其次，许多工作负载会创建多个 pod，在某些情况下是动态的。这意味着，在任何时间点，我们可能都不知道给定的应用程序有多少 pods 在运行。

最后，`pods`是非永久性资源。它们会随着时间的推移而开始和停止，每次发生这种情况时，它们很可能会获得一个新的 IP 地址。

出于所有这些原因，使用它们的 IP 地址与 pod 通信是一个坏主意。而是要用`services`。

### 2.2.服务

Kubernetes 是一个抽象概念，它将一组 pod 作为网络服务公开。`service`处理识别正在运行的 pod 及其 IP 地址的所有复杂性。每个`service`都有一个唯一的 URL，可以跨集群访问。因此，pods 只需要使用提供的服务 URL，而不是使用 IPs 进行通信。

让我们看一个示例服务定义:

```java
apiVersion: v1
kind: Service
metadata:
  name: api-service
spec:
  selector:
    app: api-app
  ports:
    - protocol: TCP
      port: 80
      targetPort: 8080
```

这将在集群中创建一个名为`api-service`的`service`。这个`service`被绑定到任何运行应用程序`api-app`的 pod，不管有多少个这样的`pods`存在或者它们在哪里运行。并且，随着新的`pods`这种类型的启动，`service`会自动发现它们。

**使用`service`是将 pod 从使用它们的应用程序中分离出来的良好开端**。但是，就其本身而言，它们并不总能达到我们的预期目标。这就是入口和负载平衡器的用武之地。

## 3.进入

默认情况下，Kubernetes `service`是集群私有的。这意味着只有群集中的应用程序可以访问它们。有很多方法可以解决这个问题，最好的方法之一是使用`ingress`。

在 Kubernetes 中， **an `ingress`让我们将集群外的流量路由到集群内的一个或多个`services`**。通常，入口作为所有传入流量的单一入口点。

入口接收公共 IP，这意味着它可以在群集外部访问。然后，使用一组规则，它将所有流量转发到适当的`service`。反过来，`service`将把请求发送给能够实际处理请求的`pod`。

创建`ingress.`、**时要记住几件事，它们被设计用来处理网络流量(HTTP 或 HTTPS)** 。虽然可以将`ingress`用于其他类型的协议，但通常需要额外的配置。

其次，一个`ingress`可以做的不仅仅是路由。其他一些用例包括负载平衡和 SSL 终止。

最重要的是，**对象本身实际上并不做任何事情**。因此，为了让一个`ingress`真正做任何事情，我们需要有一个`ingress controller`可用。

### 3.1.入口控制器

和大多数 Kubernetes 对象一样，`ingress`需要一个关联的控制器来管理它。然而，虽然 Kubernetes 为大多数对象提供了控制器，如`deployments`和`services`、**，但默认情况下它不包括`ingress` 、`con`、T5**。因此，由集群管理员来确保有合适的控制器可用。

大多数云平台都提供自己的`ingress controllers`，但也有大量开源选项可供选择。也许最受欢迎的是 [nginx 入口控制器](https://web.archive.org/web/20220928104813/https://www.nginx.com/products/nginx-ingress-controller/)，它建立在同名的流行网络服务器之上。

让我们使用 nginx `ingress controller`检查一个示例配置:

```java
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: ingress-example
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
spec:
  ingressClassName: nginx-example
  rules:
  - http:
      paths:
      - path: /api
        pathType: Prefix
        backend:
          service:
            name: api-service
            port:
              number: 80
```

在这个例子中，我们创建了一个`ingress`，它将任何以`/api`开头的请求路由到一个名为`api-service`的 Kubernetes `service`。

注意,`annotations`字段包含特定于 nginx 的值。因为所有的`ingress controllers`使用相同的 API 对象，我们通常使用注释字段将特定的配置传递给`ingress controller`。

Kubernetes 生态系统中有几十个可用的[`ingress controllers`](https://web.archive.org/web/20220928104813/https://kubernetes.io/docs/concepts/services-networking/ingress-controllers/)，涵盖它们超出了本文的范围。然而，**因为它们使用相同的 API 对象，所以它们共享一些共同的特性**:

*   入口规则:定义如何将流量路由到特定服务的一组规则(通常基于 URL 或主机名)
*   默认后端:处理不匹配任何规则的流量的默认资源
*   TLS:定义私钥和证书以允许 TLS 终止的秘密

不同的入口控制器建立在这些概念之上，并添加了它们自己的功能和特性。

## 4.负载平衡器

Kubernetes 中的负载平衡器与`ingresses`有很多重叠。这是因为它们主要用于将`services`暴露给互联网，正如我们在上面看到的，这也是`ingresses`的一个特征。

**然而，负载平衡器具有与`ingresses`** 不同的特性。负载平衡器不是像`ingress`那样的独立对象，而是`service`的扩展。

让我们看一个带有负载平衡器的服务的简单示例:

```java
apiVersion: v1
kind: Service
metadata:
  name: api-service
spec:
  selector:
    app: api-app
  ports:
    - protocol: TCP
      port: 80
      targetPort: 8080
  type: LoadBalancer
```

就像我们之前的服务示例一样，在这里，我们创建了一个`service`，它将流量路由到任何运行我们的 API 应用程序的`pod`。在这种情况下，我们包括了一个`LoadBalancer`配置。

为此，**集群必须运行在支持外部负载平衡器的提供者上**。所有主要的云提供商都支持使用自己的资源类型的外部负载平衡器:

*   AWS 使用一个[网络负载平衡器](https://web.archive.org/web/20220928104813/https://docs.aws.amazon.com/eks/latest/userguide/aws-load-balancer-controller.html)
*   GKE 还使用了一个[网络负载平衡器](https://web.archive.org/web/20220928104813/https://cloud.google.com/load-balancing/docs/network)
*   Azure 使用一个[公共负载平衡器](https://web.archive.org/web/20220928104813/https://docs.microsoft.com/en-us/azure/aks/load-balancer-standard)

就像我们看到的不同的`ingress controllers`一样，不同的负载均衡器提供商有他们自己的设置。通常，我们使用 CLI 或特定于基础架构的工具直接管理这些设置，而不是使用 YAML。不同的负载平衡器实现也将提供额外的功能，如 SSL 终端。

因为负载平衡器是根据`service`、**定义的，所以它们只能路由到单个`service`、**。这与`ingress`不同，后者能够路由到集群内的多个`services`。

此外，请记住，无论是哪家提供商，**使用外部负载均衡器通常会带来额外的成本**。这是因为，与`ingresses`及其控制器不同，外部负载平衡器**存在于 Kubernetes 集群**之外。因此，大多数云提供商将对超出群集本身的额外资源使用收费。

## 5.结论

在本文中，我们已经了解了 Kubernetes 的一些核心概念，包括`deployments`、`services`和`ingresses`。这些对象中的每一个都在不同`pods.`之间的网络流量路由中起着关键作用

虽然`ingresses`和负载平衡器在功能上有很多重叠，但它们的行为却不同。**主要区别在于`ingresses`是集群内部的本机对象，可以路由到多个`services`，而负载平衡器在集群外部，只能路由到单个`service.`**