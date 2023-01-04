# SolrCloud 中的提交和 NRT 搜索

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/solr-cloud-nrt-search>

## 1。概述

Solr 是最流行的基于 Lucene 的搜索解决方案之一。它快速、分布式、健壮、灵活，背后有一个活跃的开发者社区。SolrCloud 是 Solr 的新的分布式版本。

**它的一个关键特性是近乎实时的(NRT)搜索**，也就是说，当文档被编入索引时，就可以作为`soon`进行搜索。

## 2。SolrCloud 中的索引

Solr 中的集合由多个碎片组成，每个碎片都有不同的副本。创建集合时，碎片的副本之一被选为该碎片的主碎片:

*   当客户机试图索引一个文档时，首先根据文档的`id` 散列值给文档分配一个碎片
*   客户端从 zookeeper 获取该碎片的领导者的 URL，最后，对该 URL 发出索引请求
*   在将文档发送到副本之前，分片负责人在本地对文档进行索引
*   一旦领导者从所有活动的和正在恢复的复制品接收到确认，它就向索引客户端应用返回确认

**当我们在 Solr 中索引一个文档时，它不会直接进入索引。它被写在所谓的`tlog`(事务日志)中。** Solr 使用事务日志来确保文档在提交之前不会丢失，以防系统崩溃。

如果系统在事务日志中的文档被提交(即，保存到磁盘)之前崩溃，则当系统重新启动时，事务日志被重放，从而导致零文档丢失。

每个索引/更新请求都被记录到事务日志中，事务日志会继续增长，直到我们发出一个 commit。

## 3。在 SolrCloud 提交

一个`commit` 操作意味着完成一个改变，并把这个改变保存在磁盘上。SolrCloud 提供了两种提交操作，即提交和软提交。

### 3.1。提交(硬提交)

提交或硬提交是指 Solr 将事务日志中所有未提交的文档刷新到磁盘。**处理活动事务日志，然后打开新的事务日志文件。**

它还刷新了一个名为 searcher 的组件，以便可以搜索新提交的文档。搜索器可以被视为索引中所有已提交文档的只读视图。

提交操作可以由客户端通过调用`commit` API 专门完成:

```
String zkHostString = "zkServer1:2181,zkServer2:2181,zkServer3:2181/solr";
SolrClient solr = new CloudSolrClient.Builder()
  .withZkHost(zkHostString)
  .build();
SolrInputDocument doc1 = new SolrInputDocument();
doc1.addField("id", "123abc");
doc1.addField("date", "14/10/2017");
doc1.addField("book", "To kill a mockingbird");
doc1.addField("author", "Harper Lee");
solr.add(doc1);
solr.commit();
```

同样，通过在 `solrconfig.xml` 文件中指定它，它可以自动成为`autoCommit` ，参见第 3.4 节。

### 3.2 .软交换〔t1〕

从 Solr 4 开始，增加了 Softcommit，主要是为了支持 SolrCloud 的 NRT 特性。这是一种通过跳过高成本的硬提交来使文档近乎实时可搜索的机制。

在软提交期间，事务日志不会被截断，而是继续增长。然而，一个新的搜索器被打开，这使得自上次软提交以来的文档对于搜索是可见的。此外，Solr 中的一些顶级缓存是无效的，因此它不是一个完全免费的操作。

**当我们将软提交的`maxTime`指定为 1000 时，这意味着从文档被编入索引起不晚于 1 秒的时间内，文档将在查询中可用。**

这一特性赋予 SolrCloud 近乎实时的搜索能力，因为即使不提交新文档也可以进行搜索。软提交只能通过在 s `olrconfig.xml` 文件中指定为`autoSoftCommit`来触发，参见第 3.4 节。

### 3.3。自动提交和自动软件提交

`solrconfig.xml` 文件是 SolrCloud 中最重要的配置文件之一。它在集合创建时生成。要启用`autoCommit`或`autoSoftCommit`，我们需要更新文件中的以下部分:

```
<autoCommit>
  <maxDocs>10000</maxDocs>
  <maxTime>30000</maxTime>
  <openSearcher>true</openSearcher>
</autoCommit>

<autoSoftCommit>
  <maxTime>6000</maxTime>
  <maxDocs>1000</maxDocs>
</autoSoftCommit>
```

`maxTime:`自最早未提交更新后的毫秒数，在此之后应进行下一次提交/软提交。

`maxDocs:`自上次提交以来已经发生的更新次数，在此之后下一次提交/软提交应该发生。

这个属性告诉 Solr 是否在提交操作后打开一个新的搜索器。**如果是`true`，提交后关闭旧的搜索器，打开新的搜索器，使提交的文档可见用于搜索**，如果是`false`，提交后文档将不可用于搜索。

## 4。近实时搜索

使用提交和软提交的组合在 Solr 中实现了接近实时的搜索。如前所述，当一个文档被添加到 Solr 中时，在提交到索引之前，它在搜索结果中是不可见的。

正常提交的成本很高，这就是软提交有用的原因。但是，由于软提交并不持久保存文档，我们确实需要根据我们预期的负载，将自动提交间隔`maxTime`设置为一个合理的值。

### 4.1。实时 G `et` s

Solr 提供的另一个特性实际上是实时的——`get`API。**`get` API 可以返回给我们一个甚至还没有软提交的文档。**

如果在索引中找不到文档，它将直接在事务日志中进行搜索。因此，我们可以在索引调用返回后立即启动一个`get` API 调用，我们仍然能够检索文档。

然而，像所有太好的事情一样，这里有一个陷阱。**我们需要在`get` API 调用中传递文档的`id` 。**当然，我们可以随`id`一起提供其他过滤查询，但是如果没有`id`，调用就不起作用:

```
http://localhost:8985/solr/myCollection/get?id=1234&fq;=name:baeldung
```

## 5。结论

Solr 为我们调整 NRT 功能提供了相当大的灵活性。为了获得服务器的最佳性能，我们需要根据我们的用例以及预期的负载来试验提交和软提交的值。

我们不应该让我们的提交间隔太长，否则我们的事务日志将会变得相当大。但是我们不应该过于频繁地执行软提交。

我们还建议在投入生产之前对我们的系统进行适当的性能测试。我们应该检查文档是否在我们期望的时间间隔内变得可搜索。