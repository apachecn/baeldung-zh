# Kubernetes Java 客户端的快速介绍

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/kubernetes-java-client>

## 1.介绍

在本教程中，我们将展示如何使用官方客户端库从 Java 应用程序中使用 [Kubernetes](/web/20220627141113/https://www.baeldung.com/kubernetes) API。

## 2.为什么使用 Kubernetes API？

**如今，可以肯定地说，Kubernetes 已经成为管理容器化应用**的`de facto`标准。它提供了丰富的 API，允许我们部署、扩展和监控应用程序和相关资源，如存储、机密和环境变量。事实上，考虑这个 API 的一种方式是常规操作系统中可用的系统调用的分布式模拟。

大多数时候，我们的应用程序可以忽略它们运行在 Kubernetes 下的事实。这是一件好事，因为它允许我们在本地开发它们，只需几个命令和 YAML 咒语，只需稍作修改，就可以将它们快速部署到多个云提供商。

然而，在一些有趣的用例中，我们需要与 Kubernetes API 进行对话来实现特定的功能:

*   启动一个外部程序来执行某项任务，并在稍后检索其完成状态
*   动态创建/修改一些服务以响应一些客户请求
*   为跨多个 Kubernetes 集群甚至跨云提供商运行的解决方案创建一个定制的监控仪表板

当然，这些用例并不常见，但是由于它的 API，我们将看到它们很容易实现。

**此外，由于 Kubernetes API 是一个开放的规范，我们可以非常自信地说，我们的代码无需任何修改就可以在任何经过认证的实现上运行**。

## 3.当地发展环境

在继续创建应用程序之前，我们需要做的第一件事就是访问一个正常运行的 Kubernetes 集群。虽然我们可以使用公共云提供商来实现这一点，但本地环境通常会对其设置的所有方面提供更多控制。

有几个轻量级发行版适合这个任务:

*   [K3S](https://web.archive.org/web/20220627141113/https://k3s.io/)
*   [Minikube](/web/20220627141113/https://www.baeldung.com/spring-boot-minikube)
*   [种类](https://web.archive.org/web/20220627141113/https://kind.sigs.k8s.io/docs/user/quick-start/)

实际的设置步骤超出了本文的范围，但是，无论您选择哪个选项，只要在开始任何开发之前确保 **`[kubectl](https://web.archive.org/web/20220627141113/https://kubernetes.io/docs/reference/kubectl/overview/)`** 运行良好即可。

## 4.Maven 依赖性

首先，让我们将 Kubernetes Java API 依赖项添加到我们项目的`pom.xml`:

```
<dependency>
    <groupId>io.kubernetes</groupId>
    <artifactId>client-java</artifactId>
    <version>11.0.0</version>
</dependency> 
```

最新版本的`[client-java](https://web.archive.org/web/20220627141113/https://search.maven.org/search?q=g:io.kubernetes%20a:client-java)`可以从 Maven Central 下载。

## 5.你好，库伯内特

现在，让我们创建一个非常简单的 Kubernetes 应用程序，它将列出可用的节点，以及关于它们的一些信息。

尽管它很简单，但是这个应用程序说明了我们必须通过哪些必要的步骤来连接到一个正在运行的集群并执行一个 API 调用。不管我们在实际应用中使用哪种 API，这些步骤总是相同的。

### 5.1.`ApiClient`初始化

**`ApiClient`类是 API 中最重要的类之一，因为它包含了调用 Kubernetes API 服务器**的所有逻辑。创建该类实例的推荐方法是使用`Config`类中的一个可用静态方法。特别是，最简单的方法是使用`defaultClient()`方法:

```
ApiClient client = Config.defaultClient();
```

使用这种方法可以确保我们的代码在远程和集群场景中都可以工作。此外，它将自动遵循`**kubectl**` 实用程序使用的相同步骤来定位配置文件

*   由`KUBECONFIG`环境变量定义的配置文件
*   `$HOME/.kube/config`文件
*   `/var/run/secrets/kubernetes.io/serviceaccount`下的服务账户令牌
*   直接访问 `http://localhost:8080`

**第三步是使我们的应用程序能够作为任何`pod`的一部分在集群内运行，只要适当的服务帐户可供其使用。**

另外，请注意，如果我们在配置文件中定义了多个上下文，该过程将选择“当前”上下文，如使用`kubectl config` `set-context`命令所定义的。

### 5.2.创建 API 存根

一旦我们获得了一个`ApiClient `实例，我们就可以用它为任何可用的 API 创建一个存根。在我们的例子中，我们将使用`CoreV1Api`类，它包含列出可用节点所需的方法:

```
CoreV1Api api = new CoreV1Api(client);
```

这里，我们使用已经存在的`ApiClient`来创建 API 存根。

**注意，这里也有一个无参数的构造函数，但是一般来说，我们应该避免使用它**。使用它的理由是，在内部，它将使用一个必须通过`Configuration.setDefaultApiClient()`预先设置的全局`ApiClient`。这就创建了对在使用存根之前调用该方法的人的隐式依赖，从而可能导致运行时错误和维护问题。

更好的方法是使用任何依赖注入框架来完成这个初始连接，在需要的地方注入结果存根。

### 5.3.调用 Kubernetes API

最后，让我们进入返回可用节点的实际 API 调用。`CoreApiV1`存根有一个方法可以做到这一点，所以这变得微不足道:

```
V1NodeList nodeList = api.listNode(null, null, null, null, null, null, null, null, 10, false);
nodeList.getItems()
  .stream()
  .forEach((node) -> System.out.println(node)); 
```

**在我们的例子中，我们为方法的大多数参数传递`null`，因为它们是可选的。**最后两个参数与所有`listXXX `呼叫相关，因为它们指定了呼叫超时以及这是否是一个`Watch`呼叫。检查方法的签名揭示了剩余的参数:

```
public V1NodeList listNode(
  String pretty,
  Boolean allowWatchBookmarks,
  String _continue,
  String fieldSelector,
  String labelSelector,
  Integer limit,
  String resourceVersion,
  String resourceVersionMatch,
  Integer timeoutSeconds,
  Boolean watch) {
    // ... method implementation
} 
```

对于这个快速介绍，我们将忽略分页、观察和过滤参数。**在本例中，返回值是一个 POJO，用 Java 表示返回的文档**。对于这个 API 调用，文档包含一个`V1Node `对象列表，其中包含关于每个节点的几条信息。以下是这段代码在控制台上生成的典型输出:

```
class V1Node {
    metadata: class V1ObjectMeta {
        labels: {
            beta.kubernetes.io/arch=amd64,
            beta.kubernetes.io/instance-type=k3s,
            // ... other labels omitted
        }
        name: rancher-template
        resourceVersion: 29218
        selfLink: null
        uid: ac21e09b-e3be-49c3-9e3a-a9567b5c2836
    }
    // ... many fields omitted
    status: class V1NodeStatus {
        addresses: [class V1NodeAddress {
            address: 192.168.71.134
            type: InternalIP
        }, class V1NodeAddress {
            address: rancher-template
            type: Hostname
        }]
        allocatable: {
            cpu=Quantity{number=1, format=DECIMAL_SI},
            ephemeral-storage=Quantity{number=18945365592, format=DECIMAL_SI},
            hugepages-1Gi=Quantity{number=0, format=DECIMAL_SI},
            hugepages-2Mi=Quantity{number=0, format=DECIMAL_SI},
            memory=Quantity{number=8340054016, format=BINARY_SI}, 
            pods=Quantity{number=110, format=DECIMAL_SI}
        }
        capacity: {
            cpu=Quantity{number=1, format=DECIMAL_SI},
            ephemeral-storage=Quantity{number=19942490112, format=BINARY_SI}, 
            hugepages-1Gi=Quantity{number=0, format=DECIMAL_SI}, 
            hugepages-2Mi=Quantity{number=0, format=DECIMAL_SI}, 
            memory=Quantity{number=8340054016, format=BINARY_SI}, 
            pods=Quantity{number=110, format=DECIMAL_SI}}
        conditions: [
            // ... node conditions omitted
        ]
        nodeInfo: class V1NodeSystemInfo {
            architecture: amd64
            kernelVersion: 4.15.0-135-generic
            kubeProxyVersion: v1.20.2+k3s1
            kubeletVersion: v1.20.2+k3s1
            operatingSystem: linux
            osImage: Ubuntu 18.04.5 LTS
            // ... more fields omitted
        }
    }
}
```

正如我们所看到的，有相当多的信息可用。作为比较，这是具有默认设置的等效`kubectl`输出:

```
[[email protected]](/web/20220627141113/https://www.baeldung.com/cdn-cgi/l/email-protection):~# kubectl get nodes
NAME               STATUS   ROLES                  AGE   VERSION
rancher-template   Ready    control-plane,master   24h   v1.20.2+k3s1 
```

## 6.结论

在本文中，我们简要介绍了用于 Java 的 Kubernetes API。在以后的文章中，我们将更深入地研究这个 API，并探索它的一些附加特性:

*   解释可用的 API 调用变体之间的区别
*   使用`Watch`实时监控集群事件
*   如何使用分页从集群中高效地检索大量数据

像往常一样，例子的完整源代码可以在 GitHub 上找到[。](https://web.archive.org/web/20220627141113/https://github.com/eugenp/tutorials/tree/master/kubernetes/k8s-intro)