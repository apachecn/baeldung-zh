# 通过 Minikube 使用本地 Docker 图像

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/docker-local-images-minikube>

## 1。概述

在本教程中，我们将把 Docker 容器部署到 Kubernetes，并看看如何为这些容器使用本地映像。我们将使用 [Minikube](https://web.archive.org/web/20221025150553/https://minikube.sigs.k8s.io/docs/) 来运行 Kubernetes 集群。

## 2。`Dockerfile`

**首先，我们需要一个`[Dockerfile](https://web.archive.org/web/20221025150553/https://docs.docker.com/engine/reference/builder/)`来创建本地 Docker 图像。**这应该很简单，因为我们将重点关注 Minikube 命令。

让我们创建一个`Dockerfile`,只使用一个`echo`命令来打印一条消息:

```java
FROM alpine 

CMD ["echo", "Hello World"]
```

## 3。`docker-env`命令

对于第一种方法，我们需要确保安装了 [Docker CLI](https://web.archive.org/web/20221025150553/https://docs.docker.com/engine/reference/commandline/cli/) 。这是一个管理 Docker 资源的工具，比如图片和容器。

默认情况下，它使用我们机器上的 Docker 引擎，但是我们可以很容易地改变它。我们将使用它并将我们的 Docker CLI 指向 Minikube 内部的 Docker 引擎。

让我们检查一下这个先决条件，看看 Docker CLI 是否正常工作:

```java
$ docker version
```

输出应该如下所示:

```java
Client: Docker Engine - Community
 Version:           19.03.12
 ...

Server: Docker Engine - Community
 Engine:
  Version:          19.03.12
  ...
```

让我们继续下一步。我们可以配置这个 CLI 来使用 Minikube 内部的 Docker 引擎。这样，我们将能够列出 Minikube 中可用的图像，甚至可以在其中构建图像。

让我们看看配置 Docker CLI 所需的[步骤:](https://web.archive.org/web/20221025150553/https://minikube.sigs.k8s.io/docs/commands/docker-env/)

```java
$ minikube docker-env
```

我们可以在这里看到命令列表:

```java
export DOCKER_TLS_VERIFY="1"
export DOCKER_HOST="tcp://172.22.238.61:2376"
export DOCKER_CERT_PATH="C:\Users\Baeldung\.minikube\certs"
export MINIKUBE_ACTIVE_DOCKERD="minikube"

# To point your shell to minikube's docker-daemon, run:
# eval $(minikube -p minikube docker-env)
```

**让我们执行最后一行的命令，因为它会为我们进行配置:**

```java
$ eval $(minikube -p minikube docker-env)
```

现在，我们可以使用 Docker CLI 来研究 Minikube 内部的 Docker 环境。

**让我们用 [`minikube image ls`](https://web.archive.org/web/20221025150553/https://minikube.sigs.k8s.io/docs/commands/image/#minikube-image-ls) 命令列出可用的图像:**

```java
$ minikube image ls --format table
```

这将打印一个包含图像的表格:

```java
|-----------------------------------------|---------|---------------|--------|
|                  Image                  |   Tag   |   Image ID    |  Size  |
|-----------------------------------------|---------|---------------|--------|
| docker.io/kubernetesui/dashboard        | <none>  | 1042d9e0d8fcc | 246MB  |
| docker.io/kubernetesui/metrics-scraper  | <none>  | 115053965e86b | 43.8MB |
| k8s.gcr.io/etcd                         | 3.5.3-0 | aebe758cef4cd | 299MB  |
| k8s.gcr.io/pause                        | 3.7     | 221177c6082a8 | 711kB  |
| k8s.gcr.io/coredns/coredns              | v1.8.6  | a4ca41631cc7a | 46.8MB |
| k8s.gcr.io/kube-controller-manager      | v1.24.3 | 586c112956dfc | 119MB  |
| k8s.gcr.io/kube-scheduler               | v1.24.3 | 3a5aa3a515f5d | 51MB   |
| k8s.gcr.io/kube-proxy                   | v1.24.3 | 2ae1ba6417cbc | 110MB  |
| k8s.gcr.io/pause                        | 3.6     | 6270bb605e12e | 683kB  |
| gcr.io/k8s-minikube/storage-provisioner | v5      | 6e38f40d628db | 31.5MB |
| k8s.gcr.io/echoserver                   | 1.4     | a90209bb39e3d | 140MB  |
| k8s.gcr.io/kube-apiserver               | v1.24.3 | d521dd763e2e3 | 130MB  |
|-----------------------------------------|---------|---------------|--------|
```

如果我们将其与 [`docker image ls`](https://web.archive.org/web/20221025150553/https://docs.docker.com/engine/reference/commandline/image_ls/) 命令的输出进行比较，我们会看到两者显示的是相同的列表。这意味着我们的 Docker CLI 配置正确。

让我们用我们的`Dockerfile`和[建立一个图像](https://web.archive.org/web/20221025150553/https://docs.docker.com/engine/reference/commandline/build/):

```java
$ docker build -t first-image -f ./Dockerfile .
```

现在它在 Minikube 中可用，我们可以创建一个使用此图像的 pod:

```java
$ kubectl run first-container --image=first-image --image-pull-policy=Never --restart=Never
```

让我们检查一下这个 pod 的日志:

```java
$ kubectl logs first-container
```

我们可以看到预期的“Hello World”消息。一切正常。在下一个示例中，让我们关闭终端，确保我们的 Docker CLI 没有连接到 Minikube。

## 4。Minikube 图像加载命令

让我们看看使用本地图像的另一种方法。这一次，我们将在机器上的 Minikube 外部构建 Docker 映像，并将其加载到 Minikube 中。让我们建立形象:

```java
$ docker build -t second-image -f ./Dockerfile .
```

现在图像存在了，但在 Minikube 中还不可用。让我们[加载它](https://web.archive.org/web/20221025150553/https://minikube.sigs.k8s.io/docs/commands/image/#minikube-image-load):

```java
$ minikube image load second-image
```

让我们列出图片并检查是否可用:

```java
$ minikube image ls --format table
```

我们可以在列表中看到新的图像。这意味着我们可以创建 pod:

```java
$ kubectl run second-container --image=second-image --image-pull-policy=Never --restart=Never
```

容器成功启动。让我们检查日志:

```java
$ kubectl logs second-container
```

我们可以看到它打印了正确的信息。

## 5。Minikube 映像构建命令

在前面的例子中，我们将一个预构建的 Docker 映像加载到 Minikube。然而，我们也可以在 Minikube 中构建我们的图像。

让我们使用相同的`Dockerfile`并构建一个新的 Docker 映像:

```java
$ minikube image build -t third-image -f ./Dockerfile .
```

现在图像在 Minikube 中可用了，我们可以用它启动一个容器:

```java
$ kubectl run third-container --image=third-image --image-pull-policy=Never --restart=Never
```

让我们检查日志以确保它在工作:

```java
$ kubectl logs third-container
```

它按预期打印“Hello World”消息。

## 6。结论

在本文中，我们使用了三种不同的方式在 Minikube 中运行本地 Docker 映像。

首先，我们将 Docker CLI 配置为连接 Minikube 内部的 Docker 引擎。然后，我们看到了加载一个预构建的映像和直接在 Minikube 中构建一个映像的两个命令。