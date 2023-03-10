# Java couch base SDK 简介

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-couchbase-sdk>

## 1。简介

在这篇介绍 Couchbase SDK for Java 的文章中，我们演示了如何与 Couchbase 文档数据库进行交互，涵盖了一些基本概念，比如创建 Couchbase 环境、连接到集群、打开数据存储桶、使用基本的持久性操作以及使用文档副本。

## 2。Maven 依赖关系

如果您使用的是 Maven，请将以下内容添加到 pom.xml 文件中:

```java
<dependency>
    <groupId>com.couchbase.client</groupId>
    <artifactId>java-client</artifactId>
    <version>2.2.6</version>
</dependency>
```

## 3。入门

SDK 提供了`CouchbaseEnvironment`接口和一个实现类`DefaultCouchbaseEnvironment`,其中包含管理集群和存储桶访问的默认设置。如有必要，可以覆盖默认的环境设置，我们将在 3.2 节中看到。

**重要提示:**官方 Couchbase SDK 文档提醒用户确保 JVM 中只有一个`CouchbaseEnvironment`是活动的，因为使用两个或更多可能会导致不可预知的行为。

### 3.1。连接到具有默认环境的集群

为了让 SDK 使用默认设置自动创建一个`CouchbaseEnvironment`,并将其与我们的集群相关联，我们可以简单地通过提供集群中一个或多个节点的 IP 地址或主机名来连接到集群。

在本例中，我们连接到本地工作站上的单节点集群:

```java
Cluster cluster = CouchbaseCluster.create("localhost");
```

为了连接到多节点集群，我们将指定至少两个节点，以防其中一个节点在应用程序尝试建立连接时不可用:

```java
Cluster cluster = CouchbaseCluster.create("192.168.4.1", "192.168.4.2");
```

**注意:**在创建初始连接时，没有必要指定集群中的每个节点。一旦连接建立，`CouchbaseEnvironment`将查询集群，以便发现剩余的节点(如果有的话)。

### 3.2。使用自定义环境

如果您的应用程序需要微调由`DefaultCouchbaseEnvironment`提供的任何设置，您可以创建一个自定义环境，然后在连接到您的集群时使用该环境。

下面是一个使用自定义`CouchbaseEnvironment`连接到单节点集群的示例，连接超时为 10 秒，键值查找超时为 3 秒:

```java
CouchbaseEnvironment env = DefaultCouchbaseEnvironment.builder()
  .connectTimeout(10000)
  .kvTimeout(3000)
  .build();
Cluster cluster = CouchbaseCluster.create(env, "localhost");
```

并使用自定义环境连接到多节点集群:

```java
Cluster cluster = CouchbaseCluster.create(env,
  "192.168.4.1", "192.168.4.2");
```

### 3.3。打开水桶

一旦连接到 Couchbase 集群，就可以打开一个或多个 buckets。

当您第一次设置 Couchbase 集群时，安装包会自动创建一个名为`“default”`的 bucket，密码为空。

这里有一种方法可以打开密码为空的`“default”`桶:

```java
Bucket bucket = cluster.openBucket();
```

您也可以在打开存储桶时指定其名称:

```java
Bucket bucket = cluster.openBucket("default");
```

对于任何其他密码为空的存储桶，您需要`must`提供存储桶名称:

```java
Bucket myBucket = cluster.openBucket("myBucket");
```

要打开密码非空的存储桶，您必须提供存储桶名称`and`密码:

```java
Bucket bucket = cluster.openBucket("bucketName", "bucketPassword");
```

## 4。持续操作

在本节中，我们将展示如何在 Couchbase 中执行 CRUD 操作。在我们的示例中，我们将使用代表一个人的简单 JSON 文档，如这个示例文档所示:

```java
{
  "name": "John Doe",
  "type": "Person",
  "email": "[[email protected]](/web/20220524120617/https://www.baeldung.com/cdn-cgi/l/email-protection)",
  "homeTown": "Chicago"
}
```

`“type”`属性不是必需的，但是通常的做法是包含一个指定文档类型的属性，以防有人决定在同一个桶中存储多种类型。

### 4.1。文档 id

存储在 Couchbase 中的每个文档都与一个`id`相关联，这个`id`对于存储文档的存储桶来说是唯一的。文档`id`类似于传统关系数据库行中的主键列。

文档`id`值必须是 250 或更少字节的 UTF-8 字符串。

由于 Couchbase 没有提供在插入时自动生成`id`的机制，我们必须提供自己的机制。

生成`ids`的常见策略包括使用自然键的键派生，比如我们的示例文档中显示的`“email”`属性，以及使用`UUID`字符串。

对于我们的例子，我们将生成随机的`UUID`字符串。

### 4.2。插入文件

在我们可以将新文档插入我们的 bucket 之前，我们必须首先创建一个包含文档内容的`JSONObject`实例:

```java
JsonObject content = JsonObject.empty()
  .put("name", "John Doe")
  .put("type", "Person")
  .put("email", "[[email protected]](/web/20220524120617/https://www.baeldung.com/cdn-cgi/l/email-protection)")
  .put("homeTown", "Chicago");
```

接下来，我们创建一个由`id`值和`JSONObject`组成的`JSONDocument`对象:

```java
String id = UUID.randomUUID().toString();
JsonDocument document = JsonDocument.create(id, content);
```

为了向桶中添加新文档，我们使用了`insert`方法:

```java
JsonDocument inserted = bucket.insert(document);
```

返回的`JsonDocument`包含原始文档的所有属性，加上一个名为`“CAS”`(比较和交换)的值，Couchbase 使用这个值进行版本跟踪。

如果 bucket 中已经存在带有所提供的`id`的文档，Couchbase 抛出一个`DocumentAlreadyExistsException`。

我们也可以使用`upsert`方法，它要么插入文档(如果没有找到`id`要么更新文档(如果找到了`id`):

```java
JsonDocument upserted = bucket.upsert(document);
```

### 4.3。检索文档

为了通过文档的`id`来检索文档，我们使用了`get`方法:

```java
JsonDocument retrieved = bucket.get(id);
```

如果不存在给定`id`的文档，该方法返回`null`。

### 4.4。更新或替换文档

我们可以使用`upsert`方法更新现有文档:

```java
JsonObject content = document.content();
content.put("homeTown", "Kansas City");
JsonDocument upserted = bucket.upsert(document);
```

正如我们在 4.2 节中提到的，无论是否找到给定`id`的文档，`upsert`都将成功。

如果在我们最初检索文档和我们尝试向上插入修订文档之间已经过了足够长的时间，那么原始文档可能已经被另一个流程或用户从存储桶中删除了。

如果我们需要在应用程序中防止这种情况，我们可以使用`replace`方法，如果在 Couchbase 中找不到带有给定`id`的文档，该方法将失败并返回一个`DocumentDoesNotExistException`:

```java
JsonDocument replaced = bucket.replace(document);
```

### 4.5。删除文件

要删除 Couchbase 文档，我们使用`remove`方法:

```java
JsonDocument removed = bucket.remove(document);
```

您也可以通过`id`删除:

```java
JsonDocument removed = bucket.remove(id);
```

返回的`JsonDocument`对象只设置了`id`和`CAS`属性；所有其他属性(包括 JSON 内容)都从返回的对象中删除。

如果给定的`id`没有文档存在，Couchbase 抛出一个`DocumentDoesNotExistException`。

## 5。使用副本

本节讨论 Couchbase 的虚拟存储桶和副本架构，并介绍一种在文档的主节点不可用的情况下检索文档副本的机制。

### 5.1。虚拟存储桶和副本

Couchbase 将一个 bucket 的文档分布在 1024 个虚拟 bucket 或`vbuckets`的集合中，对文档`id`使用哈希算法来确定存储每个文档的`vbucket`。

每个床座铲斗也可以配置成保持每个`vbucket`的一个或多个`replicas`。每当一个文档被插入或更新并写入其`vbucket`时，Couchbase 就会启动一个进程，将新的或更新的文档复制到其`replica vbucket`中。

在多节点集群中，Couchbase 将`vbuckets`和`replica vbuckets`分布在集群中的所有数据节点中。一个`vbucket`和它的`replica vbucket`被保存在不同的数据节点上，以实现一定程度的高可用性。

### 5.2。从副本中检索文档

当通过文档的`id`检索文档时，如果文档的主节点关闭或者由于网络错误而无法访问，Couchbase 会抛出一个异常。

您可以让您的应用程序捕捉异常，并尝试使用`getFromReplica`方法检索文档的一个或多个副本。

以下代码将使用找到的第一个副本:

```java
JsonDocument doc;
try{
    doc = bucket.get(id);
}
catch(CouchbaseException e) {
    List<JsonDocument> list = bucket.getFromReplica(id, ReplicaMode.FIRST);
    if(!list.isEmpty()) {
        doc = list.get(0);
     }
}
```

请注意，在编写应用程序时，可能会阻止写操作，直到持久性和复制完成。然而，出于性能原因，更常见的做法是让应用程序在写入文档主节点的内存后立即从写入中返回，因为磁盘写入本来就比内存写入慢。

使用后一种方法时，如果最近更新的文档的主节点在更新完全复制之前失败或离线，则副本读取可能会也可能不会返回文档的最新版本。

同样值得注意的是，Couchbase 异步检索副本(如果找到的话)。因此，如果您的 bucket 配置了多个副本，则 SDK 返回它们的顺序没有保证，您可能希望遍历找到的所有副本，以确保您的应用程序拥有最新的副本版本:

```java
long maxCasValue = -1;
for(JsonDocument replica : bucket.getFromReplica(id, ReplicaMode.ALL)) {
    if(replica.cas() > maxCasValue) {
        doc = replica;
        maxCasValue = replica.cas();
    }
}
```

## 6。结论

为了开始使用 Couchbase SDK，我们已经介绍了一些基本的使用场景。

本教程中的代码片段可以在 [GitHub 项目](https://web.archive.org/web/20220524120617/https://github.com/eugenp/tutorials/tree/master/couchbase)中找到。

你可以在官方的 Couchbase SDK 开发者文档网站了解更多关于 SDK 的信息。