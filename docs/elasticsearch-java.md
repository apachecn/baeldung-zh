# Java 中的弹性搜索指南

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/elasticsearch-java>

## 1。概述

在本文中，我们将深入探讨与全文搜索引擎相关的一些关键概念，并特别关注 Elasticsearch。

因为这是一篇面向 Java 的文章，所以我们不打算给出如何设置 Elasticsearch 的详细的一步一步的教程，并展示它是如何工作的。相反，我们将针对 Java 客户端，以及如何使用主要特性，如`index`、`delete`、`get`和`search`。

## 2。设置

为了简单起见，我们将为我们的 Elasticsearch 实例使用 docker 映像，尽管**任何侦听端口 9200 的 Elasticsearch 实例都可以使用**。

我们首先启动我们的 Elasticsearch 实例:

```java
docker run -d --name es762 -p 9200:9200 -e "discovery.type=single-node" elasticsearch:7.6.2
```

默认情况下，Elasticsearch 在 9200 端口监听即将到来的 HTTP 查询。我们可以通过在您喜欢的浏览器中打开`http://localhost:9200/` URL 来验证它是否已成功启动:

```java
{
  "name" : "M4ojISw",
  "cluster_name" : "docker-cluster",
  "cluster_uuid" : "CNnjvDZzRqeVP-B04D3CmA",
  "version" : {
    "number" : "7.6.2",
    "build_flavor" : "default",
    "build_type" : "docker",
    "build_hash" : "2f4c224",
    "build_date" : "2020-03-18T23:22:18.622755Z",
    "build_snapshot" : false,
    "lucene_version" : "8.4.0",
    "minimum_wire_compatibility_version" : "6.8.0",
    "minimum_index_compatibility_version" : "6.8.0-beta1"
  },
  "tagline" : "You Know, for Search"
}
```

## 3。Maven 配置

现在我们已经建立并运行了基本的 Elasticsearch 集群，让我们直接跳到 Java 客户端。首先，我们需要在我们的`pom.xml`文件中声明下面的 [Maven 依赖关系](https://web.archive.org/web/20221206051718/https://search.maven.org/classic/#search%7Cga%7C1%7Cg%3A%22org.elasticsearch%22%20AND%20a%3A%22elasticsearch%22):

```java
<dependency>
    <groupId>org.elasticsearch</groupId>
    <artifactId>elasticsearch</artifactId>
    <version>7.6.2</version>
</dependency>
```

您可以通过之前提供的链接查看 Maven Central 托管的最新版本。

## 4。Java API

在我们直接跳到如何使用主要的 Java API 特性之前，我们需要初始化`RestHighLevelClient` `:`

```java
ClientConfiguration clientConfiguration =
    ClientConfiguration.builder().connectedTo("localhost:9200").build();
RestHighLevelClient client = RestClients.create(clientConfiguration).rest();
```

### 4.1。索引文件

`index()`函数允许存储任意的 JSON 文档并使其可搜索:

```java
@Test
public void givenJsonString_whenJavaObject_thenIndexDocument() {
  String jsonObject = "{\"age\":10,\"dateOfBirth\":1471466076564,"
    +"\"fullName\":\"John Doe\"}";
  IndexRequest request = new IndexRequest("people");
  request.source(jsonObject, XContentType.JSON);

  IndexResponse response = client.index(request, RequestOptions.DEFAULT);
  String index = response.getIndex();
  long version = response.getVersion();

  assertEquals(Result.CREATED, response.getResult());
  assertEquals(1, version);
  assertEquals("people", index);
}
```

注意，可以使用 **[任何 JSON Java 库](/web/20221206051718/https://www.baeldung.com/java-json)** 来创建和处理您的文档。**如果你不熟悉这些，你可以使用 Elasticsearch helpers 来生成你自己的 JSON 文档**:

```java
XContentBuilder builder = XContentFactory.jsonBuilder()
  .startObject()
  .field("fullName", "Test")
  .field("dateOfBirth", new Date())
  .field("age", "10")
  .endObject();

  IndexRequest indexRequest = new IndexRequest("people");
  indexRequest.source(builder);

  IndexResponse response = client.index(indexRequest, RequestOptions.DEFAULT);
  assertEquals(Result.CREATED, response.getResult());
```

### 4.2。查询索引文件

现在我们有了一个可搜索的 JSON 文档索引，我们可以继续使用`search()` 方法进行搜索:

```java
SearchRequest searchRequest = new SearchRequest();
SearchResponse response = client.search(searchRequest, RequestOptions.DEFAULT);
SearchHit[] searchHits = response.getHits().getHits();
List<Person> results = 
  Arrays.stream(searchHits)
    .map(hit -> JSON.parseObject(hit.getSourceAsString(), Person.class))
    .collect(Collectors.toList());
```

**由`search()`方法返回的结果被称为`Hits`** ，每个`Hit` 指的是匹配一个搜索请求的 JSON 文档。

在这种情况下，`results`列表包含集群中存储的所有数据。注意，在这个例子中，我们使用了 [FastJson](/web/20221206051718/https://www.baeldung.com/fastjson) 库来将 JSON `Strings`转换成 Java 对象。

我们可以通过添加额外的参数来增强请求，以便使用`QueryBuilders`方法定制查询:

```java
SearchSourceBuilder builder = new SearchSourceBuilder()
  .postFilter(QueryBuilders.rangeQuery("age").from(5).to(15));

SearchRequest searchRequest = new SearchRequest();
searchRequest.searchType(SearchType.DFS_QUERY_THEN_FETCH);
searchRequest.source(builder);

SearchResponse response = client.search(searchRequest, RequestOptions.DEFAULT);
```

### 4.3。检索和删除文件

`get()`和`()`方法允许从集群中获取或删除一个 JSON 文档，使用它的 id:

```java
GetRequest getRequest = new GetRequest("people");
getRequest.id(id);

GetResponse getResponse = client.get(getRequest, RequestOptions.DEFAULT);
// process fields

DeleteRequest deleteRequest = new DeleteRequest("people");
deleteRequest.id(id);

DeleteResponse deleteResponse = client.delete(deleteRequest, RequestOptions.DEFAULT);
```

语法非常简单，您只需要指定对象 id 旁边的索引。

## 5。`QueryBuilders`例子

`QueryBuilders`类提供了各种静态方法，用作动态匹配器来查找集群中的特定条目。当使用`search()`方法在集群中查找特定的 JSON 文档时，我们可以使用查询构建器来定制搜索结果。

这里列出了`QueryBuilders` API 最常见的用法。

`matchAllQuery()`方法返回一个匹配集群中所有文档的`QueryBuilder`对象:

```java
QueryBuilder matchAllQuery = QueryBuilders.matchAllQuery();
```

`rangeQuery()`匹配字段值在一定范围内的文档:

```java
QueryBuilder matchDocumentsWithinRange = QueryBuilders
  .rangeQuery("price").from(15).to(100)
```

通过提供字段名(如`fullName`)和相应的值(如`John Doe`),`matchQuery()`方法将所有文档与这些字段的值进行匹配:

```java
QueryBuilder matchSpecificFieldQuery= QueryBuilders
  .matchQuery("fullName", "John Doe");
```

我们还可以使用`multiMatchQuery()`方法来构建多字段版本的匹配查询:

```java
QueryBuilder matchSpecificFieldQuery= QueryBuilders.matchQuery(
  "Text I am looking for", "field_1", "field_2^3", "*_field_wildcard");
```

我们可以使用插入符号(^)来增强特定的字段。

在我们的例子中,`field_2`的 boost 值设置为 3，这使得它比其他字段更重要。注意，可以使用通配符和正则表达式查询，但是从性能角度来看，在处理通配符时要注意内存消耗和响应时间延迟，因为像* _ apples 这样的东西可能会对性能造成巨大影响。

重要性系数用于对执行 s `earch()`方法后返回的命中结果集进行排序。

如果您更熟悉 Lucene 查询语法，您可以使用 `simpleQueryStringQuery()`方法定制搜索查询:

```java
QueryBuilder simpleStringQuery = QueryBuilders
  .simpleQueryStringQuery("+John -Doe OR Janette");
```

正如您可能猜到的，**我们可以使用 Lucene 的查询解析器语法来构建简单而强大的查询**。这里有一些基本操作符，可以和`AND/OR/NOT`操作符一起使用来构建搜索查询:

*   必需操作符(`+`):要求一段特定的文本存在于文档的某个字段中。
*   禁止运算符(`–`):排除包含在(`–`)符号后声明的关键字的所有文档。

## 6。结论

在这篇简短的文章中，我们看到了如何使用 ElasticSearch 的 Java API 来执行一些与全文搜索引擎相关的常见功能。

您可以在 [GitHub 项目](https://web.archive.org/web/20221206051718/https://github.com/eugenp/tutorials/tree/master/persistence-modules/spring-data-elasticsearch)中查看本文提供的示例。