# 使用 Java Kubernetes API 创建、更新和删除资源

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-kubernetes-api-crud>

## 1.介绍

在本教程中，我们将使用 Kubernetes 的官方 Java API 来介绍 Kubernetes 资源上的 CRUD 操作。

我们已经在以前的文章中介绍了这个 API 的基本用法，包括[基本项目设置](/web/20220630021540/https://www.baeldung.com/kubernetes-java-client)和[各种方式](/web/20220630021540/https://www.baeldung.com/java-kubernetes-watch)，我们可以用它来获取关于正在运行的集群的信息。

一般来说，Kubernetes 部署大多是静态的。我们创建一些工件(例如 YAML 文件)来描述我们想要创建的东西，并将它们提交给 DevOps 管道。我们系统的各个部分保持不变，直到我们添加新的组件或升级现有的组件。

但是，有些情况下我们需要动态添加资源。一个常见的例子是运行[作业](https://web.archive.org/web/20220630021540/https://kubernetes.io/docs/reference/generated/kubernetes-api/v1.19/#job-v1-batch)来响应用户发起的请求。作为响应，应用程序将启动一个后台作业来处理报告，并使其可用于以后的检索。

这里的关键点是，通过使用这些 API，我们可以更好地利用可用的基础设施，因为我们可以只在需要时消耗资源，然后释放它们。

## 2.创建新资源

在本例中，我们将在 Kubernetes 集群中创建一个作业资源。作业是 Kubernetes 的一种工作负载，不同于其他类型的工作负载，它会一直运行到完成。也就是说，一旦在其 pod 中运行的程序终止，作业本身也会终止。其在 YAML 的代表性与其他资源并无不同:

```java
apiVersion: batch/v1
kind: Job
metadata:
  namespace: jobs
  name: report-job
  labels:
    app: reports
spec:
  template:
    metadata:
      name: payroll-report
    spec:
      containers:
      - name: main
        image: report-runner
        command:
        - payroll
        args:
        - --date
        - 2021-05-01
      restartPolicy: Never
```

Kubernetes API 提供了两种创建等效 Java 对象的方法:

*   用`new`创建 POJOS 并通过 setters 填充所有必需的属性
*   使用 fluent API 构建 Java 资源表示

使用哪种方法主要是个人偏好。在这里，我们将使用 fluent 方法创建`V1Job`对象，因为构建过程看起来非常类似于它的 YAML 对应物:

```java
ApiClient client  = Config.defaultClient();
BatchV1Api api = new BatchV1Api(client);
V1Job body = new V1JobBuilder()
  .withNewMetadata()
    .withNamespace("report-jobs")
    .withName("payroll-report-job")
    .endMetadata()
  .withNewSpec()
    .withNewTemplate()
      .withNewMetadata()
        .addToLabels("name", "payroll-report")
        .endMetadata()
      .editOrNewSpec()
        .addNewContainer()
          .withName("main")
          .withImage("report-runner")
          .addNewCommand("payroll")
          .addNewArg("--date")
          .addNewArg("2021-05-01")
          .endContainer()
        .withRestartPolicy("Never")
        .endSpec()
      .endTemplate()
    .endSpec()
  .build(); 
V1Job createdJob = api.createNamespacedJob("report-jobs", body, null, null, null);
```

我们从创建`ApiClient`开始，然后创建 API 存根实例。`Job`资源是`Batch API, `的一部分，因此我们创建一个`BatchV1Api `实例，我们将使用它来调用集群的 API 服务器。

接下来，我们实例化一个`V1JobBuilder`实例，它引导我们完成填充所有属性的过程。注意嵌套构建器的使用:要“关闭”嵌套构建器，我们必须调用它的`endXXX()`方法，这将我们带回它的父构建器。

或者，也可以使用`withXXX`方法直接注入嵌套对象。当我们想要重用一组公共属性时，例如元数据、标签和注释，这是很有用的。

最后一步只是调用 API 存根。这将序列化我们的资源对象并将请求发送到服务器。正如所料，API 有同步(如上所述)和异步版本。

返回的对象将包含与创建的作业相关的元数据和状态字段。在`Job`的情况下，我们可以使用它的状态字段来检查它何时完成。我们还可以使用本文中介绍的关于监控资源的技术之一来接收这个通知。

## 3.更新现有资源

更新现有资源包括向 Kubernetes API 服务器发送一个补丁请求，其中包含我们想要修改的字段。从 Kubernetes 版本 1.16 开始，有四种方法可以指定这些字段:

*   JSON 补丁(RFC 6092)
*   JSON 合并补丁(RFC 7396)
*   战略合并补丁
*   应用 YAML

其中，最后一个是最容易使用的，因为它将所有的合并和冲突解决留给了服务器:我们所要做的就是发送一个带有我们想要修改的字段的 YAML 文档。

不幸的是，Java API 没有提供构建这个部分 YAML 文档的简单方法。相反，我们必须求助于`PatchUtil `助手类来发送原始的 YAML 或 JSON 字符串。然而，我们可以使用内置的 JSON 序列化器，通过`ApiClient`对象获得它:

```java
V1Job patchedJob = new V1JobBuilder(createdJob)
  .withNewMetadata()
    .withName(createdJob.getMetadata().getName())
    .withNamespace(createdJob.getMetadata().getNamespace())
    .endMetadata()
  .editSpec()
    .withParallelism(2)
  .endSpec()
  .build();

String patchedJobJSON = client.getJSON().serialize(patchedJob);

PatchUtils.patch(
  V1Job.class, 
  () -> api.patchNamespacedJobCall(
    createdJob.getMetadata().getName(), 
    createdJob.getMetadata().getNamespace(), 
    new V1Patch(patchedJobJSON), 
    null, 
    null, 
    "baeldung", 
    true, 
    null),
  V1Patch.PATCH_FORMAT_APPLY_YAML,
  api.getApiClient()); 
```

在这里，我们使用从`createNamespacedJob()`返回的对象作为模板，从这个模板我们将构建补丁版本。在这种情况下，我们只是将`parallelism`的值从 1 增加到 2，其他字段保持不变。这里很重要的一点是，当我们构建修改后的资源时，我们必须使用`withNewMetadata().` ,这确保我们不会构建包含托管字段的对象，托管字段存在于创建资源后返回的对象中。关于托管字段以及如何在 Kubernetes 中使用它们的完整描述，请参考[文档](https://web.archive.org/web/20220630021540/https://kubernetes.io/docs/reference/using-api/server-side-apply/#field-management)。

一旦我们用修改过的字段构建了一个对象，我们就使用`serialize `方法将它转换成 JSON 表示。然后，我们使用这个序列化版本构建一个`V1Patch`对象，用作补丁调用的有效负载。`patch`方法还采用了一个额外的参数，我们在这个参数中告知请求中出现的数据类型。在我们的例子中，这是`PATCH_FORMAT_APPLY_YAML`，库将它用作 HTTP 请求中包含的`Content-Type`头。

传递给`fieldManager`参数的`“baeldung”`值定义了操作资源字段的参与者名称。当两个或多个客户机试图修改同一个资源时，Kubernetes 在内部使用这个值来解决最终的冲突。我们还在`force`参数中传递了`true`，这意味着我们将获得任何修改过的字段的所有权。

## 4.删除资源

与前面的操作相比，删除资源非常简单:

```java
V1Status response = api.deleteNamespacedJob(
  createdJob.getMetadata().getName(), 
  createdJob.getMetadata().getNamespace(), 
  null, 
  null, 
  null, 
  null, 
  null, 
  null ) ; 
```

在这里，我们只是使用`deleteNamespacedJob `方法，使用默认选项为这种特定类型的资源删除作业。如果需要，我们可以使用最后一个参数来控制删除过程的细节。这采用了一个`V1DeleteOptions`对象的形式，我们可以用它来指定任何依赖资源的宽限期和级联行为。

## 5.结论

在本文中，我们介绍了如何使用 Java Kubernetes API 库操作 Kubernetes 资源。像往常一样，例子的完整源代码可以在 GitHub 上找到[。](https://web.archive.org/web/20220630021540/https://github.com/eugenp/tutorials/tree/master/kubernetes/k8s-intro)