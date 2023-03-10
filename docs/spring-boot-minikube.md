# 使用 Minikube 运行 Spring Boot 应用程序

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-boot-minikube>

## 1。概述

在这篇[上一篇文章](/web/20221206041317/http://www.baeldung.com/kubernetes)中，我们介绍了 Kubernetes 的理论。

在本教程中，**我们将讨论如何在本地 Kubernetes 环境(也称为 Minikube)上部署 Spring Boot 应用程序。**

作为本文的一部分，我们将:

*   在本地机器上安装 minitube
*   开发一个包含两个 Spring Boot 服务的示例应用程序
*   使用 Minikube 在单节点集群上设置应用程序
*   使用配置文件部署应用程序

## 2。安装 Minikube

Minikube 的安装基本上由三个步骤组成:安装一个 Hypervisor(像 VirtualBox)，CLI `kubectl`，以及 Minikube 本身。

官方文档提供了每个步骤以及所有流行操作系统的详细说明。

完成安装后，我们可以启动 Minikube，将 VirtualBox 设置为 Hypervisor，并配置`kubectl`与名为`minikube`的集群对话:

```java
$> minikube start
$> minikube config set vm-driver virtualbox
$> kubectl config use-context minikube
```

之后，我们可以验证`kubectl`是否与我们的集群正确通信:

```java
$> kubectl cluster-info
```

输出应该如下所示:

```java
Kubernetes master is running at https://192.168.99.100:8443
To further debug and diagnose cluster problems, use 'kubectl cluster-info dump'.
```

在这个阶段，我们将保持响应中的 IP 关闭(在我们的例子中为`192.168.99.100`)。我们稍后称之为`NodeIP`，它需要从集群外部调用资源，例如从我们的浏览器。

最后，我们可以检查集群的状态:

```java
$> minikube dashboard
```

该命令在我们的默认浏览器中打开一个站点，它提供了关于我们的集群状态的全面概述。

## 4。演示应用程序

由于我们的集群现在正在运行并准备部署，我们需要一个演示应用程序。

为此，我们将创建一个简单的“Hello world”应用程序，由两个 Spring Boot 服务组成，我们称之为`frontend`和`backend`。

后端在端口 8080 上提供一个 REST 端点，返回一个包含其主机名的`String`。前端在端口 8081 上可用，它将简单地调用后端端点并返回其响应。

之后，我们必须从每个应用程序建立一个 Docker 映像。GitHub 上的[也提供了所有必要的文件。](https://web.archive.org/web/20221206041317/https://github.com/eugenp/tutorials/tree/master/spring-cloud-modules/spring-cloud-kubernetes)

关于如何构建 Docker 映像的详细说明，请看[dockering a Spring Boot 应用](/web/20221206041317/http://www.baeldung.com/dockerizing-spring-boot-application#Dockerize)。

**我们必须确保在 Minikube 集群**的 Docker 主机上触发构建过程，否则 Minikube 将无法在部署过程中找到映像。此外，我们主机上的工作区必须装载到 Minikube 虚拟机中:

```java
$> minikube ssh
$> cd /c/workspace/tutorials/spring-cloud/spring-cloud-kubernetes/demo-backend
$> docker build --file=Dockerfile \
  --tag=demo-backend:latest --rm=true .
```

之后，我们可以从 Minikube VM 注销，所有进一步的步骤将在我们的主机上使用`kubectl`和`minikube`命令行工具执行。

## 5。使用命令式命令的简单部署

第一步，我们将为我们的`demo-backend`应用程序创建一个部署，只包含一个 Pod。在此基础上，我们将讨论一些命令，以便我们可以验证部署、检查日志，并在最后清理它。

### 5.1。创建部署

我们将使用`kubectl`，将所有需要的命令作为参数传递:

```java
$> kubectl run demo-backend --image=demo-backend:latest \
  --port=8080 --image-pull-policy Never
```

如我们所见，我们创建了一个名为`demo-backend, which`的部署，它是从一个名为`demo-backend`的映像实例化而来，版本为`latest`。

使用`–port`，我们指定部署为其 pod 打开端口 8080(因为我们的`demo-backend`应用程序监听端口 8080)。

标志`–image-pull-policy Never`确保 Minikube 不会试图从注册表中获取图像，而是从本地 Docker 主机中获取图像。

### 5.2。验证部署

现在，我们可以检查部署是否成功:

```java
$> kubectl get deployments
```

输出如下所示:

```java
NAME           DESIRED   CURRENT   UP-TO-DATE   AVAILABLE   AGE
demo-backend   1         1         1            1           19s
```

如果我们想要查看应用程序日志，我们首先需要 Pod ID:

```java
$> kubectl get pods
$> kubectl logs <pod id>
```

### 5.3。为部署创建服务

**为了使我们后端应用程序的 REST 端点可用，我们需要创建一个服务:**

```java
$> kubectl expose deployment demo-backend --type=NodePort
```

`–type=NodePort`使服务在集群外部可用。它将在`<NodeIP>:<NodePort>`可用，即服务将在`<NodePort>`传入的任何请求映射到其分配的 pod 的端口 8080。

我们使用 expose 命令，所以`NodePort`会被集群自动设置(这是技术限制)，默认范围是 30000-32767。要获得我们选择的端口，我们可以使用一个配置文件，我们将在下一节看到。

我们可以验证服务是否已成功创建:

```java
$> kubectl get services
```

输出如下所示:

```java
NAME           TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)          AGE
demo-backend   NodePort    10.106.11.133   <none>        8080:30117/TCP   11m
```

正如我们所看到的，我们有一个名为`demo-backend`的服务，类型为`NodePort`，它在集群内部 IP 10.106.11.133 中可用。

我们必须仔细查看列端口:由于端口 8080 是在部署中定义的，因此服务将流量转发到此端口。然而，如果我们想从我们的浏览器调用`demo-backend `，我们必须使用端口 30117，它可以从集群外部到达。

### 5.4。调用服务

现在，我们可以第一次调用我们的后端服务:

```java
$> minikube service demo-backend
```

这个命令将启动我们的默认浏览器，在我们的例子中打开`<NodeIP>:<NodePort>.`，也就是`http://192.168.99.100:30117`。

### 5.5。清理服务和部署

之后，我们可以删除服务和部署:

```java
$> kubectl delete service demo-backend
$> kubectl delete deployment demo-backend
```

## 6。使用配置文件的复杂部署

对于更复杂的设置，配置文件是更好的选择，而不是通过命令行参数传递所有参数。

配置文件是记录我们的部署的好方法，并且它们可以被版本控制。

### 6.1。我们后端应用的服务定义

让我们使用一个配置文件为后端重新定义我们的服务:

```java
kind: Service
apiVersion: v1
metadata:
  name: demo-backend
spec:
  selector:
    app: demo-backend
  ports:
  - protocol: TCP
    port: 8080
  type: ClusterIP
```

我们创建一个名为`demo-backend`的`Service`，由`metadata: name`字段指示。

它以任何带有`app=demo-backend`标签的 Pod 上的 TCP 端口 8080 为目标。

最后，`type: ClusterIP`表示它只在集群内部可用(因为我们这次想从我们的`demo-frontend`应用程序调用端点，而不再像前面的例子那样直接从浏览器调用)。

### 6.2。后端应用的部署定义

接下来，我们可以定义实际的部署:

```java
apiVersion: apps/v1
kind: Deployment
metadata:
  name: demo-backend
spec:
  selector:
      matchLabels:
        app: demo-backend
  replicas: 3
  template:
    metadata:
      labels:
        app: demo-backend
    spec:
      containers:
        - name: demo-backend
          image: demo-backend:latest
          imagePullPolicy: Never
          ports:
            - containerPort: 8080
```

我们创建一个名为`demo-backend`的`Deployment`，由`metadata: name`字段指示。

`spec: selector`字段定义了部署如何找到要管理的 pod。在这种情况下，我们只选择 Pod 模板中定义的一个标签(`app: demo-backend`)。

我们想要三个复制的 pod，我们用`replicas`字段表示。

模板字段定义了实际的 Pod:

*   吊舱被标记为`app: demo-backend`
*   `template: spec`字段指示每个 Pod 复制运行一个版本为`latest`的容器`demo-backend`
*   分离舱打开端口 8080

### 6.3。后端应用部署

我们现在可以触发部署:

```java
$> kubectl create -f backend-deployment.yaml
```

让我们验证部署是否成功:

```java
$> kubectl get deployments
```

输出如下所示:

```java
NAME           DESIRED   CURRENT   UP-TO-DATE   AVAILABLE   AGE
demo-backend   3         3         3            3           25s
```

我们还可以检查服务是否可用:

```java
$> kubectl get services
```

输出如下所示:

```java
NAME            TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)          AGE
demo-backend    ClusterIP   10.102.17.114   <none>        8080/TCP         30s
```

正如我们所看到的，服务的类型是`ClusterIP`，它不提供 30000-32767 范围内的外部端口，这与第 5 节中的例子不同。

### 6.4。我们前端应用的部署和服务定义

之后，我们可以为前端定义服务和部署:

```java
kind: Service
apiVersion: v1
metadata:
  name: demo-frontend
spec:
  selector:
    app: demo-frontend
  ports:
  - protocol: TCP
    port: 8081
    nodePort: 30001
  type: NodePort
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: demo-frontend
spec:
  selector:
      matchLabels:
        app: demo-frontend
  replicas: 3
  template:
    metadata:
      labels:
        app: demo-frontend
    spec:
      containers:
        - name: demo-frontend
          image: demo-frontend:latest
          imagePullPolicy: Never
          ports:
            - containerPort: 8081
```

前端和后端几乎相同，**后端和前端的唯一区别是服务**的规范:

对于前端，我们将类型定义为*节点端口*(因为我们希望前端对集群外部可用)。后端只需要可以从集群内部访问，因此，`type`就是`ClusterIP`。

如前所述，我们也使用`nodePort`字段手动指定`NodePort`。

### 6.5。前端应用部署

我们现在可以用同样的方式触发此部署:

```java
$> kubectl create -f frontend-deployment.yaml
```

让我们快速验证一下部署是否成功以及服务是否可用:

```java
$> kubectl get deployments
$> kubectl get services
```

之后，我们终于可以调用前端应用程序的 REST 端点了:

```java
$> minikube service demo-frontend
```

这个命令将再次启动我们的默认浏览器，打开`<NodeIP>:<NodePort>`，在这个例子中是`http://192.168.99.100:30001`。

### 6.6。 **清理服务和部署**

最后，我们可以通过移除服务和部署来进行清理:

```java
$> kubectl delete service demo-frontend
$> kubectl delete deployment demo-frontend
$> kubectl delete service demo-backend
$> kubectl delete deployment demo-backend
```

## 7。结论

在本文中，我们快速了解了如何使用 Minikube 在本地 Kubernetes 集群上部署 Spring Boot 的“Hello world”应用程序。

我们详细讨论了如何:

*   在我们的本地机器上安装 Minikube
*   开发并构建一个包含两个 Spring Boot 应用程序的示例
*   在单节点集群上部署服务，使用带有`kubectl`的命令以及配置文件

与往常一样，GitHub 上的[提供了示例的完整源代码。](https://web.archive.org/web/20221206041317/https://github.com/eugenp/tutorials/tree/master/spring-cloud-modules/spring-cloud-kubernetes)