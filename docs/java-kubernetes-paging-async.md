# 使用 Kubernetes API 进行分页和异步调用

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-kubernetes-paging-async>

## 1.介绍

在本教程中，我们将继续探索用于 Java 的 Kubernetes API。这一次，我们将关注它的两个特性:**分页和异步调用**。

## 2.分页

**简而言之，[分页](/web/20220628055841/https://www.baeldung.com/spring-data-jpa-pagination-sorting)允许我们在大块**中迭代一个大的结果集，也就是所谓的页面——这就是这种方法的名字。在 Kubernetes Java API 的上下文中，**这个特性在所有返回资源列表的方法中可用**。这些方法总是包含两个可选参数，我们可以用它们来迭代结果:

*   `limit`:单次 API 调用返回的最大项数
*   `continue`:一个`continuation token`，告诉服务器返回结果集的起点

使用这些参数，我们可以迭代任意数量的条目，而不会给服务器带来太大的压力。更好的是，客户端保存结果所需的内存量也是有限的。

现在，让我们看看如何使用这些参数来获得集群中所有可用 pod 的列表，方法如下:

```java
ApiClient client = Config.defaultClient();
CoreV1Api api = new CoreV1Api(client);
String continuationToken = null;
do {
    V1PodList items = api.listPodForAllNamespaces(
      null,
      continuationToken, 
      null,
      null, 
      2, 
      null, 
      null,
      null,
      10,
      false);
    continuationToken = items.getMetadata().getContinue();
    items.getItems()
      .stream()
      .forEach((node) -> System.out.println(node.getMetadata()));
} while (continuationToken != null); 
```

这里，`listPodForAllNamespaces()` API 调用的第二个参数包含延续标记，第五个是`limit `参数。虽然`limit` 通常只是一个固定值，但是`continue`需要一点额外的努力。

对于第一次调用，我们发送一个`null `值，**通知服务器这是分页请求序列**的第一次调用。收到响应后，我们从相应的列表元数据字段中获取下一个`continue`值的新值。

当没有更多的结果可用时，这个值将是`null`，所以我们使用这个事实来定义迭代循环的退出条件。

### 2.1.分页问题

分页机制非常简单，但是我们必须记住一些细节:

*   目前，该 API 不支持服务器端排序。鉴于目前缺乏对排序的存储级支持，这不太可能很快改变
*   除了`continue`之外，所有调用参数在调用之间必须相同
*   必须将`continue`值视为不透明句柄。我们永远不应该对它的价值做任何假设
*   迭代是单向的`.` 我们不能使用之前收到的`continue` 令牌返回结果集
*   尽管返回的列表元数据包含一个`remainingItemCount`字段，但是它的值既不可靠也不被所有实现支持

### 2.2.列表数据一致性

由于 Kubernetes 集群是一个非常动态的环境，**与分页调用序列相关联的结果集有可能在被客户端**读取时被修改。在这种情况下，Kubernetes API 表现如何？

[正如 Kubernetes 文档](https://web.archive.org/web/20220628055841/https://kubernetes.io/docs/reference/using-api/api-concepts/#the-resourceversion-parameter)中所解释的，列表 API 支持一个`resourceVersion`参数，该参数与`resourceVersionMatch`一起定义了如何选择一个特定的版本来包含。然而，对于分页结果集的情况，行为总是相同的:“Continue Token，Exact”。

这意味着返回的资源版本对应于分页列表调用开始时可用的版本。虽然这种方法提供了一致性，但它不包括后来修改的结果。例如，当我们遍历完一个大型集群中的所有 pod 时，其中一些可能已经终止了。

## 3.异步呼叫

到目前为止，我们已经以同步的方式使用了 Kubernetes API，这对于简单的程序来说很好，但是从资源使用的角度来看效率不是很高，因为它会阻塞调用线程，直到我们收到来自集群的响应并对其进行处理。例如，如果我们开始在 GUI 线程中进行这些调用，这种行为会严重损害应用程序的响应能力。

**幸运的是，这个库支持基于回调的异步模式，它会立即将控制权返回给调用者**。

检查`CoreV1Api`类，我们会注意到，对于每个同步`xxx()`方法，还有一个`xxxAsync() `变体。例如，`listPodForAllNamespaces()` 的异步方法是`listPodForAllNamespacesAsync()`。参数是相同的，只是为回调实现增加了一个额外的参数。

### 3.1.回拨详细信息

回调参数对象必须实现通用接口`ApiCallback<T>,` ，它只包含四个方法:

*   `onSuccess:` 当且仅当呼叫成功时被呼叫。第一个参数类型与同步版本返回的类型相同
*   `onFailure: `调用服务器出错或回复包含错误代码
*   `onUploadProgress`:上传时调用。在漫长的操作过程中，我们可以使用这个回调向用户提供反馈
*   `onDownloadProgress`:与`onUploadProgress`相同，但用于下载

异步调用也不返回常规结果。相反，它们返回一个 [OkHttp 的](/web/20220628055841/https://www.baeldung.com/guide-to-okhttp)(Kubernetes API 使用的底层 REST 客户端)`Call`实例，作为正在进行的调用的句柄。我们可以使用这个对象来轮询完成状态，或者，如果我们愿意，在完成之前取消它。

### 3.2.异步调用示例

可以想象，在任何地方实现回调都需要大量的样板代码。为了避免这种情况，我们将使用一个[调用助手](https://web.archive.org/web/20220628055841/https://github.com/eugenp/tutorials/blob/master/kubernetes/k8s-intro/src/main/java/com/baeldung/kubernetes/intro/AsyncHelper.java)来稍微简化这个任务:

```java
// Start async call
CompletableFuture<V1NodeList> p = AsyncHelper.doAsync(api,(capi,cb) ->
  capi.listNodeAsync(null, null, null, null, null, null, null, null, 10, false, cb)
);
p.thenAcceptAsync((nodeList) -> {
    nodeList.getItems()
      .stream()
      .forEach((node) -> System.out.println(node.getMetadata()));
});
// ... do something useful while we wait for results 
```

在这里，助手包装异步调用，并使其适应更标准的`CompletableFuture`。这使得我们可以将它与其他库一起使用，例如来自[反应堆项目](/web/20220628055841/https://www.baeldung.com/reactor-core)的库。在本例中，我们添加了一个完成阶段，将所有元数据打印到标准输出中。

像往常一样，在处理未来时，我们必须意识到可能出现的并发问题。这段代码的在线版本包含一些调试日志，这些日志清楚地表明，即使对于这段简单的代码，也至少使用了三个线程:

*   启动异步调用的`main`线程`,`
*   OkHttp 的线程用来进行实际的 Http 调用
*   处理结果的完成线程

## 4.结论

在本文中，我们看到了如何在 Kubernetes Java API 中使用分页和异步调用。

像往常一样，例子的完整源代码可以在 GitHub 上找到[。](https://web.archive.org/web/20220628055841/https://github.com/eugenp/tutorials/tree/master/kubernetes/k8s-intro)