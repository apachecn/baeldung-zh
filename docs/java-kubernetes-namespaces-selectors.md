# 在 Kubernetes Java API 中使用名称空间和选择器

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-kubernetes-namespaces-selectors>

## 1.介绍

在本教程中，我们将探索使用 Kubernetes Java API 过滤资源的不同方法。

在我们以前的关于 Kubernetes Java API 的文章中，我们重点介绍了查询、操作和监控集群资源的可用方法。

这些例子假设我们要么寻找特定种类的资源，要么以单一资源为目标。然而，在实践中，大多数应用程序都需要一种基于某种标准来定位资源的方法。

Kubernetes 的 API 支持三种方式来限制这些搜索的范围:

*   名称空间:范围限于给定的 Kubernetes [名称空间](https://web.archive.org/web/20220626072122/https://kubernetes.io/docs/concepts/overview/working-with-objects/namespaces/)
*   字段选择器:范围限于具有匹配字段值的资源
*   标签选择器:范围限于具有匹配标签的资源

此外，我们可以在一个查询中组合这些方法。这为我们解决复杂的需求提供了很大的灵活性。

现在，让我们更详细地看看每种方法。

## 2.使用名称空间

使用名称空间是限制查询范围的最基本方式。顾名思义，命名空间查询只返回指定命名空间内的项目。

在 Java API 中，命名空间查询方法遵循模式`listNamespacedXXX().`例如，要列出特定命名空间中的[窗格](https://web.archive.org/web/20220626072122/https://kubernetes.io/docs/concepts/workloads/pods/)，我们可以使用`listNamespacedPod()`:

```java
ApiClient client  = Config.defaultClient();
CoreV1Api api = new CoreV1Api(client);
String ns = "ns1";
V1PodList items = api.listNamespacedPod(ns,null, null, null, null, null, null, null, null, 10, false);
items.getItems()
  .stream()
  .map((pod) -> pod.getMetadata().getName() )
  .forEach((name) -> System.out.println("name=" + name)); 
```

这里， *[ApliClient](/web/20220626072122/https://www.baeldung.com/kubernetes-java-client#1-apiclient-initialization)* 和`CoreV1Api`用于执行对 Kubernetes API 服务器的实际访问。我们使用`ns1`作为名称空间来过滤资源。我们还使用类似于非命名空间方法中的其余参数。

正如所料，命名空间查询也有`call`变体，因此允许我们使用与前面描述的[相同的技术来创建`Watches`。](/web/20220626072122/https://www.baeldung.com/java-kubernetes-watch)[异步调用和分页](/web/20220626072122/https://www.baeldung.com/java-kubernetes-paging-async)也以与它们的无命名空间版本相同的方式工作。

## 3.使用字段选择器

命名空间 API 调用使用起来很简单，但是有一些限制:

*   这是全有或全无，意味着我们不能选择一个以上(但不是全部)的名称空间
*   无法根据资源属性进行筛选
*   对每个场景使用不同的方法会导致更复杂/冗长的客户端代码

**[字段选择器](https://web.archive.org/web/20220626072122/https://kubernetes.io/docs/concepts/overview/working-with-objects/field-selectors/)提供了一种基于其`fields`** 之一的值来选择资源的方法。按照 Kubernetes 的说法，A `field`就是与资源的 YAML 或 JSON 文档中的给定值相关联的 JSON 路径。例如，这是一个运行 Apache HTTP 服务器的 pod 的典型 Kubernetes YAML:

```java
apiVersion: v1
kind: Pod
metadata:
  labels:
    app: httpd
  name: httpd-6976bbc66c-4lbdp
  namespace: ns1
spec:
  ... fields omitted
status:
  ... fields omitted
  phase: Running 
```

字段`status.phase`包含现有`Pod. `的状态。相应的`field selector`表达式只是字段名称，后跟一个运算符和值。现在，让我们编写一个查询，返回所有名称空间中所有正在运行的窗格:

```java
String fs = "status.phase=Running";        
V1PodList items = api.listPodForAllNamespaces(null, null, fs, null, null, null, null, null, 10, false);
// ... process items
```

字段选择器表达式仅支持等式(' = '或' == ')和不等式('！= ')运算符。此外，我们可以在同一个调用中传递多个逗号分隔的表达式。在这种情况下，最终结果是它们将被“与”在一起以产生最终结果:

```java
String fs = "metadata.namespace=ns1,status.phase=Running";        
V1PodList items = api.listPodForAllNamespaces(null, null, fs, null, null, null, null, null, 10, false);
// ... process items 
```

请注意:字段值区分大小写！在前面的查询中，使用“running”而不是“Running”(大写“R”)将产生一个空的结果集。

字段选择器的一个重要限制是它们依赖于资源。所有资源类型都只支持`metadata.name`和`metadata.namespace`字段。

然而，当与动态字段一起使用时，字段选择器尤其有用。前一个例子中的`status.phase `就是一个例子。使用一个字段选择器和一个`Watch, `我们可以很容易地创建一个监控应用程序，当 pod 终止时会得到通知。

## 4.使用标签选择器

标签是包含任意键/值对的特殊字段，我们可以将其添加到任何 Kubernetes 资源中，作为其创建的一部分。标签选择器类似于字段选择器，因为它们本质上允许基于值过滤资源列表，但是提供了更大的灵活性:

*   对附加运算符的支持:`in/notin/exists/not exists`
*   与字段选择器相比，跨资源类型的使用一致

回到 Java API，我们使用标签选择器，方法是用期望的标准构造一个字符串，并将其作为参数传递给期望的资源 API `listXXX`调用。使用等式和/或不等式对特定标注值进行过滤使用了与字段选择器相同的语法。

让我们看看查找所有标签为“app”且值为“httpd”的 pod 的代码:

```java
String ls = "app=httpd";        
V1PodList items = api.listPodForAllNamespaces(null, null, null, ls, null, null, null, null, 10, false);
// ... process items
```

`in`操作符类似于它的 SQL 对应物，允许我们在查询中创建一些 or 逻辑:

```java
String ls = "app in ( httpd, test )";        
V1PodList items = api.listPodForAllNamespaces(null, null, null, ls, null, null, null, null, 10, false);
```

此外，我们可以使用`labelname`或`!` labelname 语法检查字段是否存在:

```java
String ls = "app";
V1PodList items = api.listPodForAllNamespaces(null, null, null, ls, null, null, null, null, 10, false);
```

最后，我们可以在一个 API 调用中链接多个表达式。结果项列表仅包含满足所有表达式的资源:

```java
String ls = "app in ( httpd, test ),version=1,foo";
V1PodList items = api.listPodForAllNamespaces(null, null, null, ls, null, null, null, null, 10, false);
```

## 5.结论

在本文中，我们介绍了使用 Java Kubernetes API 客户机过滤资源的不同方法。像往常一样，例子的完整源代码可以在 GitHub 上找到[。](https://web.archive.org/web/20220626072122/https://github.com/eugenp/tutorials/tree/master/kubernetes/k8s-intro)