# Kubernetes 部署与状态集

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/ops/kubernetes-deployment-vs-statefulsets>

## 1.概观

[Kubernetes (K8s)](/web/20220930232245/https://www.baeldung.com/ops/kubernetes) 是一个开源的容器编排系统。它允许我们自动部署、扩展和管理容器化的应用程序。

在本教程中，我们将讨论使用不同的 Kubernetes 资源在 Kubernetes 上部署我们的应用程序(pods)的两种不同方式。下面是 Kubernetes 为部署 pod 提供的两种不同的资源:

*   [部署](https://web.archive.org/web/20220930232245/https://kubernetes.io/docs/concepts/workloads/controllers/deployment/)
*   [状态设置](https://web.archive.org/web/20220930232245/https://kubernetes.io/docs/concepts/workloads/controllers/statefulset/)

让我们先来看看有状态应用程序和无状态应用程序之间的区别。

## 2.有状态和无状态应用程序

有状态应用程序和无状态应用程序之间的主要区别在于**无状态应用程序不“存储”数据。另一方面，有状态应用程序需要后备存储**。例如，像 Cassandra、MongoDB 和 MySQL 数据库这样的应用程序需要某种类型的持久存储，以便在服务重启后仍然存在。

保持状态对于运行有状态应用程序至关重要。但是对于无状态服务，任何数据流通常都是短暂的。此外，状态只存储在一个单独的后端服务中，如数据库。任何相关联的存储通常都是短暂的。例如，如果容器重新启动，存储的所有内容都会丢失。当组织采用容器时，他们倾向于从无状态容器开始，因为它们更容易被采用。

**Kubernetes 以管理无状态服务而闻名。**部署工作负载更适合处理无状态应用程序。就部署而言，吊舱是可以互换的。而`StatefulSet`为它管理的每个 pod 保持唯一的身份。每当它需要重新安排那些豆荚的时候，它使用相同的身份。在本文中，我们将进一步讨论这一点。

## 3.`Deployment`

### 3.1.理解`Deployment`:基础知识

一个 [Kubernetes 部署](https://web.archive.org/web/20220930232245/https://kubernetes.io/docs/concepts/workloads/controllers/deployment/)提供了管理一组 pod 的方法。这些可能是一个或多个正在运行的容器或一组重复的容器，称为`ReplicaSets`。`Deployment`允许我们轻松地保持一组相同的 pod 以通用配置运行。

首先，我们定义我们的 Kubernetes `Deployment`，然后部署它。然后，Kubernetes 将努力确保由部署管理的所有 pod 满足我们设定的任何要求。 **`Deployment`是对豆荚的一种监督。它为我们提供了对如何以及何时推出新的 pod 版本的细粒度控制。当我们必须回滚到以前的版本时，它也提供了控制。**

在副本为 1 的 Kubernetes `Deployment`中，控制器将验证当前状态是否等于`ReplicaSet,`的期望状态，即 1。如果当前状态为 0，它将创建一个【The ReplicaSet 将进一步创建 pod。当我们创建一个名为`web-app`的 Kubernetes `Deployment`时，它将创建一个名为`web-app-<replica-set-id>.` 的`ReplicaSet`，这个副本将进一步创建一个名为`web-app-<replica-set->-<pod-id>`的 pod。

Kubernetes `Deployment`通常用于无状态应用程序。**然而，我们可以通过给它附加一个持久卷并使它有状态来保存`Deployment`的状态。**部署的机架将共享相同的卷，并且所有机架上的数据都相同。

### 3.2.`Deployment`库伯内特斯的成分

以下是 Kubernetes `Deployment`的主要组成部分:

*   `Deployment`模板
*   持久卷
*   服务

首先，让我们制作部署模板，并将其保存为'`deployment.yaml'`。在下面的模板中，我们还附加了一个永久卷:

```java
apiVersion: apps/v1
kind: Deployment
metadata:
  name: web-app-deployment
spec:
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 2
      maxUnavailable: 1
  selector:
    matchLabels:
      app: web-app
  replicas: 3
  template:
    metadata:
      labels:
        app: web-app
    spec:
      containers:
        - name: web-app
          image: hello-world:nanoserver-1809
          volumeMounts:
          - name: counter
            mountPath: /app/
      volumes:
       - name: counter
         persistentVolumeClaim:
          claimName: counter
```

在下面的模板中，我们有我们的`PersistentVolumeClaim`:

```java
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: counter
spec:
  accessModes:
  - ReadWriteMany
  resources:
    requests:
      storage: 50Mi
  storageClassName: default 
```

### 3.3.在 Kubernetes 中执行一个`Deployment`

在执行我们的`Deployment`之前，我们需要一个服务来访问上面的`Deployment`。让我们创建一个类型为`NodePort`的服务，并将其保存为'`service.yaml'` :

```java
apiVersion: v1
kind: Service
metadata:
  name: web-app-service
spec:
  ports:
    - name: http
      port: 80
      nodePort: 30080
  selector:
    name: web-app
  type: NodePort
```

首先，我们用下面的 kubectl `apply`命令运行服务模板:

`kubectl apply -f service.yaml`

然后，我们对部署模板运行相同的命令:

```java
kubectl apply -f deployment.yaml
```

此外，为了获得部署的详细描述，让我们运行 kubectl `describe`命令:

```java
kubectl describe deployment web-app-deployment
```

输出如下所示:

```java
Name:                web-app-deployment
 Namespace:          default
 CreationTimestamp:  Tue, 30 Aug 2016 18:11:37 -0700
 Labels:             app=web-app
 Annotations:        deployment.kubernetes.io/revision=1
 Selector:           app=web-app
 Replicas:           3 desired | 3 updated | 3 total | 3 available | 0 unavailable
 StrategyType:       RollingUpdate
 MinReadySeconds:    0
 RollingUpdateStrategy:  1 max unavailable, 2 max surge
 Pod Template:
   Labels:               app=web-app
   Containers:
    web-app:
     Image:              spring-boot-docker-project:1
     Port:               80/TCP
     Environment:        <none>
     Mounts:             <none>
   Volumes:              <none>
 Conditions:
   Type          Status  Reason
   ----          ------  ------
   Available     True    MinimumReplicasAvailable
   Progressing   True    NewReplicaSetAvailable
 OldReplicaSets:   <none>
 NewReplicaSet:    web-app-deployment-1771418926 (3/3 replicas created)
 No events. 
```

在上面的部分中，我们观察到`Deployment`在内部创建了一个`ReplicaSet`。然后，它在那个`ReplicaSet`内部创建`Pods`。将来，当我们更新当前部署时，它将创建一个新的`ReplicaSet`。然后，它会以可控的速度逐渐将旧的`Pods`移动到新的`ReplicaSet`。

如果更新时发生错误，新的`ReplicaSet`将永远不会处于就绪状态。旧的`ReplicaSet`不会再次终止，确保 100%的正常运行时间，以防更新失败。在 Kubernetes `Deployment`中，我们还可以手动回滚到之前的`ReplicaSet` ，以防我们的新功能没有按预期工作。

## 4.`StatefulSet`年代

### 4.1.了解基础知识

**在 Kubernetes `Deployment`，我们把豆荚当成牛，而不是宠物。**如果其中一个成员生病或死亡，我们可以通过购买新的头来轻松替换它。这样的动作并不引人注意。类似地，如果一个吊舱部署失败，它会带来另一个。**在`StatefulSet` s 中，豆荚被赋予名字，并被当作宠物对待。如果你的一只宠物生病了，马上就会被发现。同样的情况也出现在`StatefulSet`的`,`身上，因为它通过名字与豆荚互动。**

s 为其中的每个 pod 提供两个稳定的唯一身份。首先，**网络身份使我们能够为 pod 分配相同的 DNS 名称，而不管重启的次数**。IP 地址可能仍然不同，因此消费者应该依赖于 DNS 名称(或者观察变化并更新内部缓存)。

其次，**存储标识保持不变**。网络身份始终接收相同的存储实例，而不管它在哪个节点上重新计划。

`StatefulSet`也是一个控制器，但是与 Kubernetes `Deployment`不同，它不创建`ReplicaSet`，而是创建具有独特命名约定的`pod`。每个`pod`根据模式接收 DNS 名称:`<statefulset name>-<ordinal index>.`例如，对于名称为`mysql,`的`StatefulSet`，它将是`mysql-0`。

**有状态集合的每个副本都有自己的状态，每个 pod 都创建自己的 PVC(持久卷声明)。**所以有 3 个副本的`StatefulSet`将创建 3 个舱，每个舱都有自己的体积，所以总共有 3 个 PVC。由于使用数据，我们在停止 pod 实例时应该小心，要留出所需的时间将数据从内存保存到磁盘。仍然可能有有效的理由执行强制删除，例如，当 Kubernetes 节点失败时。

### 4.2.无头服务

有状态的应用程序需要具有唯一身份(主机名)的 pod。一个 pod 应该能够通过定义明确的名称访问其他 pod。一个`StatefulSet`需要一个无头服务来工作。**无头服务是具有服务 IP** 的服务。因此，它直接返回我们关联的 pod 的 IP。这允许我们直接与豆荚互动，而不是代理。就像为`.spec.clusterIP.`指定`None`一样简单

无头服务没有 IP 地址。在内部，它创建必要的端点来公开带有 DNS 名称的 pod。`StatefulSet`定义包括对无头服务的引用，但是我们必须单独创建它。

### 4.3.`StatefulSet`库伯内特斯中的硫成分

以下是`StatefulSet`的主要组成部分:

*   状态集
*   持久卷
*   无头服务

首先，我们创建一个`StatefulSet`模板:

```java
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: web
spec:
  selector:
    matchLabels:
      app: nginx
  serviceName: "nginx"
  replicas: 3
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx
        ports:
        - containerPort: 80
          name: web
        volumeMounts:
        - name: www
          mountPath: /usr/share/nginx/html
      volumes:
      - name: www
        persistentVolumeClaim:
          claimName: myclaim
```

其次，我们创建一个在`StatefulSet`模板中提到的持久卷:

```java
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: myclaim
spec:
  accessModes:
    - ReadWriteMany
  resources:
    requests:
      storage: 5Gi
```

最后，我们现在为上面的`StatefulSet`创建一个无头服务:

```java
apiVersion: v1
kind: Service
metadata:
  name: nginx
  labels:
    app: nginx
spec:
  ports:
  - port: 80
    name: web
  clusterIP: None
  selector:
    app: nginx
```

### 4.4.在 Kubernetes 中执行`StatefulSet` s

我们已经为所有三个组件准备好了模板。现在，让我们运行`create` kubectl 命令来创建`StatefulSet`:

```java
kubectl create -f statefulset.yaml
```

它将创建三个名为`web-0,web-1,web-2`的 pod。我们可以用`get pods`验证创建是否正确:

```java
kubectl get pods
NAME      READY     STATUS    RESTARTS   AGE
web-0     1/1       Running   0          1m
web-1     1/1       Running   0          46s
web-2     1/1       Running   0          18s
```

我们还可以验证正在运行的服务:

```java
kubectl get svc nginx
NAME      TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)   AGE
nginx     ClusterIP   None         < none >      80/TCP    2m
```

s 不创建`ReplicaSet`，所以我们不能将 StatefulSet 回滚到以前的版本。**我们只能删除或放大/缩小`Statefulset`。**如果我们更新一个`StatefulSet`，它也执行 RollingUpdate，即一个副本 pod 将关闭，而更新的 pod 将打开。类似地，下一个复制舱将以同样的方式下降。

例如，我们改变上面`StatefulSet`的图像。`web-2`将终止，一旦它完全终止，那么`web-2`将被重新创建，`web-1`将同时终止。下一个复制品，即`web-0`，也会发生同样的情况。如果在更新时出现错误，那么只有`web-2`会关闭，`web-1` & `web-0`仍然会开启，运行在之前的稳定版本上。与`Deployment`不同，我们不能回滚到`StatefulSet`的任何先前版本。

**删除和/或缩小`StatefulSet`不会删除与`StatefulSet`相关的卷。这确保了数据安全，这通常比自动清除所有相关的`StatefulSet`资源更有价值。删除`StatefulSets`后，pod 的终止没有保证。为了在 StatefulSet 中实现 pods 的有序和优雅的终止，可以在删除之前将`StatefulSet`缩小到 0。**

### 4.5.`StatefulSet`的用法

使我们能够部署有状态应用程序和集群应用程序。它们将数据保存到持久性存储中，如计算引擎持久性磁盘。它们适用于部署 Kafka、MySQL、Redis、ZooKeeper 和其他应用程序(需要唯一、持久的身份和稳定的主机名)。

例如，Solr 数据库集群由几个 Zookeeper 实例管理。为了让这样的应用程序正常运行，每个 Solr 实例都必须知道控制它的 Zookeeper 实例。类似地，Zookeeper 实例本身在彼此之间建立连接来选举主节点。由于这种设计，Solr 集群是有状态应用程序的一个例子。其他有状态应用程序的例子包括 MySQL clusters、Redis、Kafka、MongoDB 等等。在这种情况下，使用`StatefulSet`。

## 5.`Deployment` vs. `StatefulSet` s

让我们来看看`Deployment`和`StatefulSet`的主要区别:

 
| **部署** | **状态设置** |
| 部署用于部署无状态应用程序 | 用于部署有状态的应用程序 |
| 豆荚是可以互换的 | pod 不可互换。每个 pod 都有一个永久标识符，它在任何重新调度中都保持这个标识符 |
| Pod 名称是唯一的 | Pod 名称是按顺序排列的 |
| 需要服务来与部署中的单元进行交互 | 无头服务负责 pod 的网络身份 |
| 指定的 PersistentVolumeClaim 由所有 pod 副本共享。换句话说，共享卷 | 指定的 volumeClaimTemplates，以便每个副本 pod 获得一个与之关联的唯一 PersistentVolumeClaim。换句话说，没有共享卷 |

Kubernetes `Deployment`是 Kubernetes 中的一个资源对象，为应用程序提供声明性更新。部署允许我们描述应用程序的生命周期。例如，应用程序使用哪些图像，应该有多少个窗格，以及应该如何更新它们。

A `StatefulSet`更适合有状态的应用。有状态的应用程序要求 pod 具有唯一的身份(例如主机名)。一个 pod 将能够访问具有明确定义的名称的其他 pod。它需要一个无头服务来连接 pod。无头服务没有 IP 地址。在内部，它创建必要的端点来公开带有 DNS 名称的 pod。

`StatefulSet`定义包括对无头服务的引用，但是我们必须单独创建它。`StatefulSet`需要持久存储，以便托管的应用程序在重启后保存其状态和数据。一旦创建了`StatefulSet`和 Headless 服务，pod 就可以通过以服务名为前缀的名称来访问另一个服务。

## 6.结论

在本教程中，我们已经了解了在 Kubernetes 中进行部署的两种方式:`Statefulset`和`Deployment`。我们已经看到了它们的主要特征和组成部分，最后对它们进行了比较。