# 使用 Helm 和 Kubernetes

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/ops/kubernetes-helm>

## 1.概观

**[赫尔姆](https://web.archive.org/web/20220525121249/https://helm.sh/)是 Kubernetes 应用**的包管理者。在本教程中，我们将了解 Helm 的基础知识，以及它们如何形成一个强大的工具来使用 Kubernetes 资源。

在过去的几年里，Kubernetes 取得了巨大的发展，支持它的生态系统也是如此。最近，Helm 被[云本地计算基金会(CNCF)](https://web.archive.org/web/20220525121249/https://www.cncf.io/) 授予毕业资格，这表明它在 Kubernetes 用户中的受欢迎程度越来越高。

## 2.背景

尽管这些术语如今相当常见，尤其是在从事云技术工作的人员中，但让我们为那些不了解的人快速浏览一下:

1.  [容器](https://web.archive.org/web/20220525121249/https://www.docker.com/resources/what-container) : **容器是指操作系统级虚拟化**。多个容器在隔离的用户空间中运行在一个操作系统中。在容器中运行的程序只能访问分配给该容器的资源。
2.  [Docker](https://web.archive.org/web/20220525121249/https://www.docker.com/) : **Docker 是一个创建和运行容器的流行程序**。它附带了 Docker 守护进程，这是管理容器的主要程序。Docker 守护进程通过 Docker 引擎 API 提供对其功能的访问，Docker 引擎 API 由 Docker 命令行界面(CLI)使用。[关于 Docker](/web/20220525121249/https://www.baeldung.com/dockerizing-spring-boot-application) 更详细的描述请参考这篇文章。
3.  [Kubernetes](https://web.archive.org/web/20220525121249/https://kubernetes.io/):**Kubernetes 是一个流行的容器编排程序**。尽管 Docker 被设计成可以与不同的容器一起工作，但它是最常用的。它提供了广泛的功能选择，包括部署自动化、扩展和跨主机集群的操作。[这篇文章对 Kubernetes 有很好的报道，供进一步参考](/web/20220525121249/https://www.baeldung.com/kubernetes)。

## 3.舵建筑

作为圣盔 3 的一部分，圣盔经历了重大的架构升级。与头盔 2 相比，它有一些期待已久的重大变化。除了包装一套新的能力，头盔 3 还具有内部管道的变化。我们将研究其中的一些变化。

Helm 2 主要基于由客户端和集群内服务器组成的客户端-服务器架构:

[![](img/6016cf6030ef5ecd4fff69cfeb2eb0f6.png)](/web/20220525121249/https://www.baeldung.com/wp-content/uploads/2019/03/Helm-2-Architecture.jpg)

*   Tiller 服务器: **Helm 通过安装在 Kubernetes 集群中的 Tiller 服务器**管理 Kubernetes 应用程序。Tiller 与 Kubernetes API 服务器交互来安装、升级、查询和删除 Kubernetes 资源。
*   Helm 客户端: **Helm 为用户提供了一个命令行界面来操作 Helm 图表**。它负责与 Tiller 服务器交互，以执行各种操作，如安装、升级和回滚图表。

Helm 3 已经**迁移到完全客户端架构**，其中集群内服务器已被移除:

[![](img/935efd711d4740dbc2ac67b8777283c9.png)](/web/20220525121249/https://www.baeldung.com/wp-content/uploads/2019/03/Helm-3-Architecture.jpg)

正如我们所见，Helm 3 中的客户端工作方式基本相同，但是**直接与 Kubernetes API 服务器**交互，而不是 Tiller 服务器。这一举措简化了 Helm 的架构，并允许它利用 Kubernetes 用户集群安全性。

## 4.舵图、版本和存储库

Helm 通过图表管理 Kubernetes 资源包。图表基本上是 Helm 的包装格式。与头盔 2 相比，头盔 3 的海图基础设施也有一些变化。

我们将会看到更多的图表和头盔 3 的变化，因为我们很快就会创建它们。但是现在，图表只不过是创建 Kubernetes 应用程序所必需的一组信息，给定一个 Kubernetes 集群:

*   **图表是以特定目录结构组织的文件**的集合
*   在配置中管理与图表相关的配置信息
*   最后，**具有特定配置的图表的运行实例被称为发布**

Helm 3 还引入了库图表的概念。基本上，**库图表支持我们可以用来定义图表原语或定义的通用图表**。这有助于共享我们可以跨图表重用的代码片段。

Helm **使用 releases** 跟踪 Kubernetes 集群中已安装的图表。这允许我们在一个集群中用不同的版本多次安装一个图表。在 Helm 2 之前，版本是作为配置图或机密存储在 Tiller 名称空间下的集群中的。从 Helm 3 开始，默认情况下，版本作为秘密直接存储在版本的名称空间中。

最后，我们可以**通过存储库**将图表作为档案共享。它基本上是一个存储和共享软件包图表的地方。有一个名为[工件中心](https://web.archive.org/web/20220525121249/https://artifacthub.io/)的分布式社区图表存储库，我们可以在那里合作。我们也可以创建自己的私人图表库。我们可以添加任意数量的图表库来使用。

## 5.先决条件

我们需要预先设置一些东西来开发我们的第一张舵图。

首先，要开始使用 Helm，我们需要一个 Kubernetes 集群。对于本教程，我们将使用 **[Minikube](https://web.archive.org/web/20220525121249/https://kubernetes.io/docs/setup/minikube/) ，它提供了一种在本地使用单节点 Kubernetes 集群的极好方式**。在 Windows 上，现在可以使用 Hyper-V 作为本机虚拟机管理程序来运行 Minikube。[参考本文，了解设置 Minikube 的更多细节](/web/20220525121249/https://www.baeldung.com/spring-boot-minikube)。

通常建议安装由赫尔姆支持的最兼容版本的库伯内特作为[。我们还应该安装和配置 Kubernetes 命令行工具`kubectl, enabling`来高效地使用我们的集群。](https://web.archive.org/web/20220525121249/https://helm.sh/docs/topics/version_skew/)

此外，我们需要一个基本的应用程序来管理 Kubernetes 集群。对于本教程，我们将使用一个简单的打包成 Docker 容器的 Spring Boot 应用程序。[关于如何将这样的应用程序打包成 Docker 容器的更详细描述，请参考本文](/web/20220525121249/https://www.baeldung.com/dockerizing-spring-boot-application#Dockerize)。

## 6.安装舵

Helm 的官方安装页面[上简洁地描述了几种安装 Helm 的方法。**在 Windows 上安装 helm 最快的方法是使用**](https://web.archive.org/web/20220525121249/https://helm.sh/docs/intro/install/)**[Chocolaty](https://web.archive.org/web/20220525121249/https://chocolatey.org/)，一个用于 Windows 平台的软件包管理器。**

使用 Chocolaty，安装 Helm 是一个简单的一行命令:

```java
choco install kubernetes-helm
```

这将在本地安装 Helm 客户端。这也为我们提供了 Helm 命令行工具，我们将在本教程中使用 Helm。

在继续之前，我们应该**确保 Kubernetes 集群正在运行**，并且可以使用`kubectl` 命令 **`:`** 进行访问

```java
kubectl cluster-info
```

现在，直到头盔 2，也需要初始化头盔。这有效地安装了 Tiller 服务器，并在 Kubernetes 集群上设置了 Helm 状态。我们可以使用以下命令通过 Helm CLI 初始化 Helm:

```java
helm init
```

但是，从头盔 3 开始，由于不再有蒂勒服务器，**没有必要初始化头盔**。事实上，该命令已被删除。因此，需要时会自动创建舵状态。

## 7.开发我们的第一张图表

现在我们准备开发我们的第一个带模板和值的舵图。我们将使用之前安装的 Helm CLI 来执行一些与图表相关的常见活动。

### 7.1.创建图表

当然，第一步是用给定的名称创建一个新图表:

```java
helm create hello-world
```

请注意，此处提供的图表名称**将是创建和存储图表的目录名称**。

让我们快速查看一下为我们创建的目录结构:

```java
hello-world /
  Chart.yaml
  values.yaml
  templates /
  charts /
  .helmignore
```

让我们了解为我们创建的这些文件和文件夹的相关性:

*   这是包含我们图表描述的主文件
*   这是包含图表默认值的文件
*   `templates`:这是 Kubernetes 资源被定义为模板的目录
*   `charts`:这是一个可选的目录，可能包含子图表
*   `.helmignore`:这是我们可以定义打包时要忽略的模式的地方(在概念上类似于。gitignore)

### 7.2.创建模板

如果我们查看模板目录，我们会注意到已经为我们创建了几个用于常见 Kubernetes 资源的**模板:**

```java
hello-world /
  templates /
    deployment.yaml
    service.yaml
    ingress.yaml
    ......
```

在我们的应用程序中，我们可能需要这些资源中的一些，也可能需要其他资源，我们必须自己创建这些资源作为模板。

对于本教程，我们将创建一个部署和服务来公开该部署。请注意，这里强调的不是详细了解 Kubernetes。因此，我们将尽可能保持这些资源的简单性。

让我们编辑`templates`目录中的文件`deployment.yaml`,如下所示:

```java
apiVersion: apps/v1
kind: Deployment
metadata:
  name: {{ include "hello-world.fullname" . }}
  labels:
    app.kubernetes.io/name: {{ include "hello-world.name" . }}
    helm.sh/chart: {{ include "hello-world.chart" . }}
    app.kubernetes.io/instance: {{ .Release.Name }}
    app.kubernetes.io/managed-by: {{ .Release.Service }}
spec:
  replicas: {{ .Values.replicaCount }}
  selector:
    matchLabels:
      app.kubernetes.io/name: {{ include "hello-world.name" . }}
      app.kubernetes.io/instance: {{ .Release.Name }}
  template:
    metadata:
      labels:
        app.kubernetes.io/name: {{ include "hello-world.name" . }}
        app.kubernetes.io/instance: {{ .Release.Name }}
    spec:
      containers:
        - name: {{ .Chart.Name }}
          image: "{{ .Values.image.repository }}:{{ .Values.image.tag }}"
          imagePullPolicy: {{ .Values.image.pullPolicy }}
          ports:
            - name: http
              containerPort: 8080
              protocol: TCP
```

类似地，让我们编辑文件`service.yaml`,如下所示:

```java
apiVersion: v1
kind: Service
metadata:
  name: {{ include "hello-world.fullname" . }}
  labels:
    app.kubernetes.io/name: {{ include "hello-world.name" . }}
    helm.sh/chart: {{ include "hello-world.chart" . }}
    app.kubernetes.io/instance: {{ .Release.Name }}
    app.kubernetes.io/managed-by: {{ .Release.Service }}
spec:
  type: {{ .Values.service.type }}
  ports:
    - port: {{ .Values.service.port }}
      targetPort: http
      protocol: TCP
      name: http
  selector:
    app.kubernetes.io/name: {{ include "hello-world.name" . }}
    app.kubernetes.io/instance: {{ .Release.Name }}
```

现在，根据我们对 Kubernetes 的了解，除了一些奇怪的地方，这些模板文件看起来很熟悉。请注意双括号{{}}中文本的自由用法。这就是所谓的模板指令。

Helm 使用了 Go 模板语言，并将其扩展为 Helm 模板语言。在评估期间，模板目录中的每个文件都被提交给模板渲染引擎。这是模板指令将实际值注入模板的地方。

### 7.3.提供价值

在前一小节中，我们看到了如何在模板中使用模板指令。现在，让我们了解如何将值传递给模板呈现引擎。我们通常通过 Helm 中的内置对象传递值。

Helm 中有许多这样的对象，如 Release、Values、Chart 和 Files。

我们可以使用图表中的文件`values.yaml`通过内置的对象值将值传递给模板渲染引擎。让我们将`values.yaml`修改成这样:

```java
replicaCount: 1
image:
  repository: "hello-world"
  tag: "1.0"
  pullPolicy: IfNotPresent
service:
  type: NodePort
  port: 80
```

但是，请注意这些值是如何在使用点分隔名称空间的模板中被访问的。我们使用了图像存储库和标签“hello-world”和“1.0”，这必须与我们为 Spring Boot 应用程序创建的 docker 图像标签相匹配。

## 8.管理图表

到目前为止，所有的工作都完成了，我们现在可以开始使用图表了。让我们看看 Helm CLI 中有哪些不同的命令可以让这变得有趣！请注意，我们将只涉及 Helm 中可用的一些命令。

### 8.1.舵绒

首先，这是一个简单的命令，它获取一个图表的路径，并运行一系列测试来确保该图表是格式良好的:

```java
helm lint ./hello-world
==> Linting ./hello-world
1 chart(s) linted, no failures
```

输出显示林挺的结果及其识别的问题。

### 8.2。舵模板

此外，我们使用这个命令在本地呈现模板以获得快速反馈:

```java
helm template ./hello-world
---
# Source: hello-world/templates/service.yaml
apiVersion: v1
kind: Service
metadata:
  name: release-name-hello-world
  labels:
    app.kubernetes.io/name: hello-world
    helm.sh/chart: hello-world-0.1.0
    app.kubernetes.io/instance: release-name
    app.kubernetes.io/managed-by: Tiller
spec:
  type: NodePort
  ports:
    - port: 80
      targetPort: http
      protocol: TCP
      name: http
  selector:
    app.kubernetes.io/name: hello-world
    app.kubernetes.io/instance: release-name

---
# Source: hello-world/templates/deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: release-name-hello-world
  labels:
    app.kubernetes.io/name: hello-world
    helm.sh/chart: hello-world-0.1.0
    app.kubernetes.io/instance: release-name
    app.kubernetes.io/managed-by: Tiller
spec:
  replicas: 1
  selector:
    matchLabels:
      app.kubernetes.io/name: hello-world
      app.kubernetes.io/instance: release-name
  template:
    metadata:
      labels:
        app.kubernetes.io/name: hello-world
        app.kubernetes.io/instance: release-name
    spec:
      containers:
        - name: hello-world
          image: "hello-world:1.0"
          imagePullPolicy: IfNotPresent
          ports:
            - name: http
              containerPort: 8080
              protocol: TCP
```

请注意，该命令伪造了本应在集群中检索的值。

### 8.3.舵安装

一旦我们确认图表没有问题，最后，我们可以运行这个命令将图表安装到 Kubernetes 集群中:

```java
helm install --name hello-world ./hello-world
NAME:   hello-world
LAST DEPLOYED: Mon Feb 25 15:29:59 2019
NAMESPACE: default
STATUS: DEPLOYED

RESOURCES:
==> v1/Service
NAME         TYPE      CLUSTER-IP     EXTERNAL-IP  PORT(S)       AGE
hello-world  NodePort  10.110.63.169  <none>       80:30439/TCP  1s

==> v1/Deployment
NAME         DESIRED  CURRENT  UP-TO-DATE  AVAILABLE  AGE
hello-world  1        0        0           0          1s

==> v1/Pod(related)
NAME                          READY  STATUS   RESTARTS  AGE
hello-world-7758b9cdf8-cs798  0/1    Pending  0         0s
```

该命令还提供了几个选项来覆盖图表中的值。请注意，我们已经将这个图表的发布命名为 flag-name。该命令以在该过程中创建的 Kubernetes 资源摘要作为响应。

### 8.4.舵杆

现在，我们想看看哪个版本安装了哪些图表。这个命令让我们查询命名的版本:

```java
helm ls --all
NAME            REVISION        UPDATED                         STATUS          CHART               APP VERSION NAMESPACE
hello-world     1               Mon Feb 25 15:29:59 2019        DEPLOYED        hello-world-0.1.0   1.0         default
```

该命令有几个子命令可用于获取扩展信息。这些包括 All、Hooks、Manifest、Notes 和值。

### 8.5.头盔升级

如果我们修改了图表，需要安装更新的版本，该怎么办？此命令帮助我们将版本升级到图表或配置的指定或当前版本:

```java
helm upgrade hello-world ./hello-world
Release "hello-world" has been upgraded. Happy Helming!
LAST DEPLOYED: Mon Feb 25 15:36:04 2019
NAMESPACE: default
STATUS: DEPLOYED

RESOURCES:
==> v1/Service
NAME         TYPE      CLUSTER-IP     EXTERNAL-IP  PORT(S)       AGE
hello-world  NodePort  10.110.63.169  <none>       80:30439/TCP  6m5s

==> v1/Deployment
NAME         DESIRED  CURRENT  UP-TO-DATE  AVAILABLE  AGE
hello-world  1        1        1           1          6m5s

==> v1/Pod(related)
NAME                          READY  STATUS   RESTARTS  AGE
hello-world-7758b9cdf8-cs798  1/1    Running  0         6m4s
```

请注意，在 Helm 3 中，版本升级使用了一个三路战略合并补丁。这里，它在生成补丁时考虑旧清单、集群活动状态和新清单。Helm 2 使用了一个双向战略合并补丁，丢弃了应用于 Helm 之外的集群的更改。

### 8.6.舵回滚

发布出错并需要被收回的情况总是会发生的。这是将版本回滚到以前版本的命令:

```java
helm rollback hello-world 1
Rollback was a success! Happy Helming!
```

我们可以指定回滚到特定的版本，或者将该参数保留为黑色，在这种情况下，它将回滚到前一个版本。

### 8.7.头盔卸载

虽然可能性较小，但我们可能希望完全卸载某个版本。我们可以使用这个命令从 Kubernetes 卸载一个版本:

```java
helm uninstall hello-world
release "hello-world" deleted
```

它会删除与图表的上一个版本和版本历史相关联的所有资源。

## 9.分发图表

虽然模板是 Helm 为管理 Kubernetes 资源带来的一个强大工具，但这并不是使用 Helm 的唯一好处。正如我们在上一节中看到的，Helm 充当 Kubernetes 应用程序的包管理器，使得安装、查询、升级和删除版本变得非常无缝。

除此之外，我们还可以使用 Helm 打包、发布和获取 Kubernetes 应用程序作为图表存档。我们也可以使用 Helm CLI 来实现这一点，因为它提供了几个命令来执行这些活动。和以前一样，我们不会涵盖所有可用的命令。

### 9.1.头盔组件

首先，我们需要将我们创建的图表打包，以便能够分发它们。这是创建图表的版本化归档文件的命令:

```java
helm package ./hello-world
Successfully packaged chart and saved it to: \hello-world\hello-world-0.1.0.tgz
```

请注意，它在我们的机器上生成一个归档文件，我们可以手动或通过公共或私有图表存储库来分发它。我们还可以选择对图表存档进行签名。

### 9.2\. Helm Repo

最后，我们需要一种机制来与共享存储库协作。这个命令中有几个子命令，我们可以使用它们来添加、删除、更新、列出或索引图表存储库。让我们看看如何使用它们。

**我们可以创建一个 git 存储库，并使用它作为我们的图表存储库。**唯一的要求是它应该有一个`index.yaml`文件。

我们可以为图表报告创建`index.yaml`:

```java
helm repo index my-repo/ --url https://<username>.github.io/my-repo
```

这将生成`index.yaml`文件，我们应该将它与图表档案一起推送到存储库。

成功创建图表存储库之后，我们可以远程添加这个存储库:

```java
helm repo add my-repo https://my-pages.github.io/my-repo
```

现在，我们应该能够直接从我们的回购安装图表:

```java
helm install my-repo/hello-world --name=hello-world
```

有相当多的命令可用于图表存储库。

### 9.3.掌舵搜索

最后，我们应该在图表中搜索一个可以出现在任何公共或私有图表存储库中的关键字。

```java
helm search repo <KEYWORD>
```

此命令有子命令，允许我们搜索图表的不同位置。例如，我们可以在工件中心或者我们自己的存储库中搜索图表。此外，我们可以在我们配置的所有存储库中的可用图表中搜索关键字。

## 10.从头盔 2 迁移到头盔 3

由于头盔已经使用了一段时间，很明显，头盔 2 的未来会随着头盔 3 的显著变化而改变。如果我们重新开始，建议从头盔 3 开始，在不久的将来，头盔 3 将继续支持头盔 2。虽然，有警告，因此将不得不作出必要的住宿。

值得注意的一些重要变化包括头盔 3 不再自动生成版本名称。然而，我们已经得到了必要的标志，可以用来生成发布名称。此外，当创建一个版本时，不再创建名称空间。我们应该提前创建名称空间。

但是对于一个使用头盔 2 并希望迁移到头盔 3 的项目来说，有几个选择。首先，我们可以使用头盔 2 和头盔 3 来管理同一个集群，慢慢耗尽头盔 2 的版本，同时使用头盔 3 来管理新版本。或者，我们可以决定使用头盔 3 来管理头盔 2 的发布。虽然这可能很棘手，但 Helm 提供了一个插件来处理这种类型的迁移。

## 11.结论

总之，在本教程中，我们讨论了 Helm 的核心组件，Helm 是 Kubernetes 应用程序的包管理器。我们了解安装头盔的选项。此外，我们还创建了一个示例图表和带有值的模板。

然后，我们通过 Helm CLI 中的多个命令来管理 Kubernetes 应用程序。最后，我们讨论了通过存储库分发 Helm 包的选项。在这个过程中，我们看到了相对于头盔 2，头盔 3 所做的改变。