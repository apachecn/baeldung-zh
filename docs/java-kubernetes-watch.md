# 在 Kubernetes API 中使用 Watch

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-kubernetes-watch>

## 1.介绍

在本教程中，我们将继续探索 Java Kubernetes API。**这一次，我们将展示如何使用`Watches`来有效地监控集群事件。**

## 2.什么是 Kubernetes 手表？

在我们之前关于 Kubernetes API 的文章中，我们已经展示了如何恢复关于给定资源或资源集合的信息。如果我们想要的只是获得这些资源在给定时间点的状态，这是没问题的。然而，考虑到 Kubernetes 星团在本质上是高度动态的，这通常是不够的。

**大多数情况下，我们还希望监控这些资源，并在事件发生时进行跟踪**。例如，我们可能对跟踪 pod 生命周期事件或部署状态变化感兴趣。虽然我们可以使用轮询，但是这种方法会受到一些限制。首先，随着要监控的资源数量的增加，它将无法很好地扩展。其次，我们冒着丢失在轮询周期之间发生的事件的风险。

为了解决这些问题，Kubernetes 引入了`Watches, `的概念，通过`watch`查询参数，所有资源收集 API 调用都可以使用这个概念。当它的值为`false`或省略时，`GET`操作像往常一样运行:服务器处理请求并返回符合给定标准的资源实例列表。**然而，经过`watch=true`会戏剧性地改变它的行为:**

*   现在，响应由一系列修改事件组成，包含修改类型和受影响的对象
*   使用一种称为长轮询的技术，在发送第一批事件后，连接将保持打开

## 3.创建一个`Watch`

Java Kubernetes API 通过`Watch`类支持`Watches`，该类有一个静态方法:`createWatch. `该方法有三个参数:

*   一个 [`ApiClient`](/web/20221208143832/https://www.baeldung.com/kubernetes-java-client#1-apiclient-initialization) ，它处理对 Kubernetes API 服务器的实际 REST 调用
*   描述要监视的资源集合的`Call`实例
*   具有预期资源类型的`TypeToken`

我们使用库中任何一个可用的`xxxApi`类的`listXXXCall()` 方法创建一个`Call `实例。例如，要创建一个检测 [Pod](https://web.archive.org/web/20221208143832/https://kubernetes.io/docs/concepts/workloads/pods/) 事件的`Watch`，我们可以使用`listPodForAllNamespacesCall()`:

```java
CoreV1Api api = new CoreV1Api(client);
Call call = api.listPodForAllNamespacesCall(null, null, null, null, null, null, null, null, 10, true, null);
Watch<V1Pod> watch = Watch.createWatch(
  client, 
  call, 
  new TypeToken<Response<V1Pod>>(){}.getType())); 
```

这里，我们对大多数参数使用`null`，意思是“使用默认值”，只有两个例外:`timeout`和`watch. `，后者必须设置为`true`用于观察调用。否则，这将是一个常规的休息呼叫。**在这种情况下，`timeout,`作为手表“生存时间”工作，这意味着一旦超时**，服务器将停止发送事件并终止连接。

为`timeout`参数找到一个好的值(以秒为单位)需要一些反复试验，因为它取决于客户端应用程序的确切需求。此外，检查您的 Kubernetes 集群配置也很重要。通常，手表有一个 5 分钟的硬性限制，所以超过这个时间就不会有想要的效果。

## 4.接收事件

仔细看看`Watch`类，我们可以看到它实现了标准 JRE 中的`Iterator`和`Iterable`，因此我们可以在`for-each`或`hasNext()-next()` 循环中使用从`createWatch()` 返回的值:

```java
for (Response<V1Pod> event : watch) {
    V1Pod pod = event.object;
    V1ObjectMeta meta = pod.getMetadata();
    switch (event.type) {
    case "ADDED":
    case "MODIFIED":
    case "DELETED":
        // ... process pod data
        break;
    default:
        log.warn("Unknown event type: {}", event.type);
    }
} 
```

每个事件的`type`字段告诉我们对象发生了什么样的事件——在我们的例子中是一个 Pod。一旦我们消费了所有事件，我们必须对`Watch.createWatch()`进行新的调用，以再次开始接收事件。在[示例代码](https://web.archive.org/web/20221208143832/https://github.com/eugenp/tutorials/tree/master/kubernetes-modules/k8s-intro)中，我们将`Watch`创建和结果处理放在一个`while`循环中。其他方法也是可能的，例如使用`ExecutorService`或类似的方法在后台接收更新。

## 5.使用资源版本和书签

上面代码的一个问题是，每次我们创建一个新的`Watch, `时，都会有一个初始事件流，其中包含给定类型的所有现有资源实例。**发生这种情况是因为服务器认为我们没有关于它们的任何先前信息，所以它只发送它们所有的**。

然而，这样做违背了有效处理事件的目的，因为我们只需要初始加载后的新事件。为了防止再次接收所有数据，监视机制支持两个额外的概念:资源版本和书签。

### 5.1.资源版本

Kubernetes 中的每个资源在其元数据中都包含一个`resourceVersion`字段，它只是服务器在每次发生变化时设置的一个不透明的字符串。此外，因为资源集合也是一种资源，所以有一个`resourceVersion `与之相关联。随着从集合中添加、移除和/或修改新的资源，该字段将相应地改变。

**当我们进行 API 调用，返回包含`resourceVersion`参数的集合`and`时，服务器将使用它的值作为查询的“起点”。**对于`Watch` API 调用，这意味着只有在创建通知版本之后发生的事件才会被包含在内。

但是，我们如何在电话中加入一个`resourceVersion`?简单:我们只进行一次初始同步调用来检索初始资源列表，其中包括集合的`resourceVersion,`，然后在后续的`Watch`调用中使用它:

```java
String resourceVersion = null;
while (true) {
    if (resourceVersion == null) {
        V1PodList podList = api.listPodForAllNamespaces(null, null, null, null, null, "false",
          resourceVersion, null, 10, null);
        resourceVersion = podList.getMetadata().getResourceVersion();
    }
    try (Watch<V1Pod> watch = Watch.createWatch(
      client,
      api.listPodForAllNamespacesCall(null, null, null, null, null, "false",
        resourceVersion, null, 10, true, null),
      new TypeToken<Response<V1Pod>>(){}.getType())) {

        for (Response<V1Pod> event : watch) {
            // ... process events
        }
    } catch (ApiException ex) {
        if (ex.getCode() == 504 || ex.getCode() == 410) {
            resourceVersion = extractResourceVersionFromException(ex);
        }
        else {
            resourceVersion = null;
        }
    }
} 
```

**在这种情况下，异常处理代码相当重要**。由于某种原因，当请求的`resourceVersion `不存在时，Kubernetes 服务器将返回 504 或 410 错误代码。在这种情况下，返回的消息通常包含当前版本。不幸的是，这些信息不是以结构化的方式出现的，而是作为错误消息本身的一部分。

提取代码(也称为丑陋的黑客)使用正则表达式来实现这一目的，但是由于错误消息往往依赖于实现，代码退回到一个`null`值。通过这样做，主循环回到它的起点，用新的`resourceVersion` 恢复一个新的列表，并继续监视操作。

无论如何，即使有这个警告，关键的一点是，现在事件列表将不会在每只手表上从头开始。

### 5.2.书签

**书签是一个可选特性，可以在从`Watch`调用**返回的事件流上启用一个特殊的`BOOKMARK`事件。该事件在其元数据中包含一个`resourceVersion `值，我们可以在后续的`Watch`调用中使用该值作为新起点。

由于这是一个选择加入的特性，我们必须通过在 API 调用中将`true `传递给`allowWatchBookmarks`来显式启用它。该选项仅在创建`Watch`时有效，否则忽略。此外，服务器可能会完全忽略它，所以客户端不应该依赖于接收这些事件。

当与前面单独使用`resourceVersion`的方法相比时，书签允许我们在很大程度上摆脱昂贵的同步调用:

```java
String resourceVersion = null;

while (true) {
    // Get a fresh list whenever we need to resync
    if (resourceVersion == null) {
        V1PodList podList = api.listPodForAllNamespaces(true, null, null, null, null,
          "false", resourceVersion, null, null, null);
        resourceVersion = podList.getMetadata().getResourceVersion();
    }

    while (true) {
        try (Watch<V1Pod> watch = Watch.createWatch(
          client,
          api.listPodForAllNamespacesCall(true, null, null, null, null, 
            "false", resourceVersion, null, 10, true, null),
          new TypeToken<Response<V1Pod>>(){}.getType())) {
              for (Response<V1Pod> event : watch) {
                  V1Pod pod = event.object;
                  V1ObjectMeta meta = pod.getMetadata();
                  switch (event.type) {
                      case "BOOKMARK":
                          resourceVersion = meta.getResourceVersion();
                          break;
                      case "ADDED":
                      case "MODIFIED":
                      case "DELETED":
                          // ... event processing omitted
                          break;
                      default:
                          log.warn("Unknown event type: {}", event.type);
                  }
              }
          }
        } catch (ApiException ex) {
            resourceVersion = null;
            break;
        }
    }
} 
```

在这里，我们只需要在第一次通过时以及每当我们在内部循环中得到一个 ApiException 时获取完整的列表。注意`BOOKMARK`事件和其他事件有相同的对象类型，所以我们在这里不需要任何特殊的转换。然而，我们唯一关心的字段是`resourceVersion`，我们把它留给下一个`Watch` 调用。

## 6.结论

在本文中，我们介绍了使用 Java API 客户端创建 Kubernetes `Watches`的不同方法。像往常一样，例子的完整源代码可以在 GitHub 上找到[。](https://web.archive.org/web/20221208143832/https://github.com/eugenp/tutorials/tree/master/kubernetes-modules/k8s-intro)