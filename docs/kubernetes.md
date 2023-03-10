# 忽必烈的概论

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/ops/kubernetes>

## 1。概述

在本教程中，我们将对 Kubernetes 进行简单的理论介绍。特别是，我们将讨论以下主题:

*   需要一个容器编排工具
*   库伯内特的特点
*   不可思议的建筑
*   Kubernetes API

为了更深入的了解，我们还可以看一下[官方文档](https://web.archive.org/web/20221101090428/https://kubernetes.io/docs/home/)。

## 2。容器编排

在这篇[前一篇文章](/web/20221101090428/https://www.baeldung.com/dockerizing-spring-boot-application)中，我们已经讨论了一些 Docker 基础知识，以及如何打包和部署定制应用程序。

**简而言之，Docker 是一个容器运行时:它提供了以标准化的方式打包、运输和运行一个应用程序的单个实例的功能，也称为容器。**

然而，随着复杂性的增加，新的需求出现了；自动化部署、容器的编排、调度应用程序、授予高可用性、管理几个应用程序实例的集群，等等。

市场上有很多可用的工具。然而，Kubernetes 越来越像一个强大的竞争对手。

## 3 .立方特征〔t1〕

简而言之，Kubernetes 是一个系统**，用于在一个节点集群上编排容器化的应用，包括网络和存储基础设施**。一些最重要的特征是:

*   资源调度:它确保`Pods`在所有可用节点上得到最优分配
*   自动扩展:随着负载的增加，集群可以动态分配额外的节点，并在其上部署新的`Pods`
*   自我修复:集群监控容器，并根据定义的策略在需要时重启它们
*   服务发现:`Pods`和`Services`通过 DNS 注册和发布
*   滚动更新/回滚:支持基于 pod 和容器的顺序重新部署的滚动更新
*   机密/配置管理:支持对敏感数据(如密码或 API 密钥)的安全处理
*   存储协调:支持多种第三方存储解决方案，可用作外部卷来保存数据

## 4。了解 Kubernetes

`**Master **`保持集群的期望状态。当我们与我们的集群交互时，例如通过使用`kubectl`命令行界面，我们总是与我们集群的主节点通信。

集群中的机器(虚拟机、物理服务器等。)来运行我们的应用程序。主节点控制每个节点。

一个节点需要一个`**container runtime**`。Docker 是 Kubernetes 最常用的运行时。

`**Minikube**` 是 Kubernetes 发行版，它使我们能够在工作站上的虚拟机中运行单节点集群，以进行开发和测试。

`**Kubernetes API**`通过将 Kubernetes 概念包装到对象中，提供了这些概念的抽象(我们将在下一节中查看)。

`**kubectl **`是一个命令行工具，我们可以用它来创建、更新、删除和检查这些 API 对象。

## 5。Kubernetes API 对象

**API 对象是一个“意图记录”**–一旦我们创建了对象，集群系统将持续工作以确保对象存在。

每个对象都由两部分组成:对象规格和对象状态。规范描述了对象的期望状态。状态描述对象的实际状态，由集群提供和更新。

在下一节中，我们将讨论最重要的对象。在那之后，我们将看一个例子，在现实中规范和状态是什么样子的。

### 5.1。基本对象

一个 **`Pod`** 是 Kubernetes 处理的基本单位。它封装了一个或多个密切相关的容器、存储资源、唯一的网络 IP 以及关于容器应该如何运行的配置，从而表示应用程序的单个实例。

**`Service `** 是一个抽象概念，它将 pod 的逻辑集合组合在一起，并定义如何访问它们。服务是一组容器的接口，因此消费者不必担心单一访问位置之外的任何事情。

使用`**Volumes**`，容器可以访问外部存储资源(因为它们的文件系统是短暂的)，它们可以读取文件或永久存储文件。卷还支持容器之间的文件共享。支持一长串的[卷类型](https://web.archive.org/web/20221101090428/https://kubernetes.io/docs/concepts/storage/volumes/#types-of-volumes)。

通过`**Namespaces**`，Kubernetes 提供了在一个物理集群上运行多个虚拟集群的可能性。命名空间为资源名称提供了范围，资源名称在命名空间中必须是唯一的。

### 5.2。控制器

此外，还有一些更高层次的抽象，称为控制器。控制器建立在基本对象之上，并提供附加功能:

一个`**Deployment**`控制器为 pod 和副本集提供声明性更新。我们在部署对象中描述期望的状态，部署控制器将实际状态更改为期望的状态。

`**ReplicaSet**`确保指定数量的 Pod 副本在任何给定时间运行。

有了`**StatefulSet**,`,我们可以运行有状态的应用程序:与部署不同，Pods 将有一个唯一的和持久的身份。使用 StatefulSet，我们可以实现具有唯一网络标识符或持久存储的应用程序，并可以保证有序、优雅的部署、扩展、删除和终止，以及有序和自动的滚动更新。

使用`**DaemonSet**,`,我们可以确保集群中的所有或一组特定节点运行特定 Pod 的一个副本。如果我们需要在每个节点上运行一个守护进程，例如用于应用程序监控或用于收集日志，这可能会很有帮助。

一个`**GarbageCollection**`确保某些对象被删除，这些对象曾经有一个所有者，但现在不再有了。这有助于通过删除不再需要的对象来节省资源。

一个 **`Job`** 创建一个或多个 pod，确保其中特定数量的 pod 成功终止，并跟踪成功的完成。作业有助于并行处理一组独立但相关的工作项目，如发送电子邮件、渲染帧、转码文件等。

### 5.3。对象元数据

元数据是属性，它提供关于对象的附加信息。

强制属性包括:

*   每个对象必须有一个`**Namespace**`(我们之前已经讨论过了)。如果没有明确指定，对象属于`default`名称空间。
*   `**Name**`是对象在其名称空间中的唯一标识符。
*   A `**Uid**`是在时间和空间中唯一的值。它有助于区分已被删除和重新创建的对象。

还有可选的元数据属性。一些最重要的是:

*   **标签**是一个键/值对，可以附加到对象上对它们进行分类。它帮助我们识别满足特定条件的对象集合。它们帮助我们以松散耦合的方式将组织结构映射到对象上。
*   标签选择器帮助我们通过标签来识别一组对象。
*   **注释**也是键/值对。与标签不同，它们不用于识别对象。相反，它们可以保存关于各自对象的信息，比如构建、发布或图像信息。

### 5.4。示例

在理论上讨论了 Kubernetes API 之后，我们现在来看一个例子。

API 对象可以被指定为 JSON 或 YAML 文件。然而，文档建议手动配置 YAML。

在下文中，我们将为无状态应用程序的部署定义规范部分。之后，我们将看看从集群返回的状态可能是什么样子。

名为`demo-backend`的应用程序的规范可能如下所示:

```java
apiVersion: apps/v1
kind: Deployment
metadata:
  name: demo-backend
spec:
  selector:
      matchLabels:
        app: demo-backend
        tier: backend
  replicas: 3
  template:
    metadata:
      labels:
        app: demo-backend
        tier: backend
    spec:
      containers:
        - name: demo-backend
          image: demo-backend:latest
          ports:
            - containerPort: 8080
```

正如我们所看到的，我们指定了一个名为`demo-backend`的`Deployment`对象。下面的`spec:`部分实际上是一个嵌套结构，包含前面章节中讨论的 API 对象:

*   `replicas: 3`指定一个复制因子为 3 的`ReplicationSet`(也就是说，我们将有三个`Deployment`的实例)
*   `template:`指定一个`Pod`
*   在这个`Pod,`中，我们可以使用`spec: containers:`将一个或多个容器分配给我们的`Pod.`，在这个例子中，我们有一个名为`demo-backend`的容器，它是从一个名为`demo-backend`，版本为`latest`的映像实例化的，它监听端口 8080
*   我们还将`labels`连接到我们的 pod: `app: demo-backend`和`tier: backend`
*   通过`selector: matchLabels:`，我们将`Pod`链接到`Deployment`控制器(映射到标签`app: demo-backend`和`tier: backend`

如果我们从集群中查询我们的`Deployment`的状态，响应将类似如下:

```java
Name:                   demo-backend
Namespace:              default
CreationTimestamp:      Thu, 22 Mar 2018 18:58:32 +0100
Labels:                 app=demo-backend
Annotations:            deployment.kubernetes.io/revision=1
Selector:               app=demo-backend
Replicas:               3 desired | 3 updated | 3 total | 3 available | 0 unavailable
StrategyType:           RollingUpdate
MinReadySeconds:        0
RollingUpdateStrategy:  25% max unavailable, 25% max surge
Pod Template:
  Labels:  app=demo-backend
  Containers:
   demo-backend:
    Image:        demo-backend:latest
    Port:         8080/TCP
    Environment:  <none>
    Mounts:       <none>
  Volumes:        <none>
Conditions:
  Type           Status  Reason
  ----           ------  ------
  Progressing    True    NewReplicaSetAvailable
  Available      True    MinimumReplicasAvailable
OldReplicaSets:  <none>
NewReplicaSet:   demo-backend-54d955ccf (3/3 replicas created)
Events:          <none>
```

正如我们所看到的，部署似乎已经启动并运行，我们可以从我们的规范中识别出大多数元素。

我们有一个复制因子为 3 的部署，其中一个 pod 包含一个容器，从映像`demo-backend:latest`实例化。

所有出现在响应中但没有在我们的规范中定义的属性都是默认值。

## 6。Kubernetes 入门

我们可以在各种平台上运行 Kubernetes:从我们的笔记本电脑到云提供商的虚拟机，或者一架裸机服务器。

**首先， [Minikube](https://web.archive.org/web/20221101090428/https://kubernetes.io/docs/getting-started-guides/minikube/) 可能是最简单的选择:它使我们能够在本地工作站上运行单节点集群进行开发和测试。**

请看一下官方文档,了解更多本地机器解决方案、托管解决方案、在 IaaS 云上运行的发行版等等。

## 7。结论

在本文中，我们快速浏览了一些 Kubernetes 的基础知识。

简单地说，我们涵盖了以下几个方面:

*   为什么我们可能需要一个容器编排工具
*   库伯内特最重要的一些特征
*   Kubernetes 建筑及其最重要的组成部分
*   Kubernetes API 以及我们如何使用它来指定我们的集群的期望状态