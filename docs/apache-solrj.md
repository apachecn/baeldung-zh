# 使用 Apache SolrJ 的 Java Solr 指南

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/apache-solrj>

## 1。概述

Apache Solr 是一个基于 Lucene 的开源搜索平台。Apache SolrJ 是一个基于 Java 的 Solr 客户机，它为搜索的主要特性提供了接口，如索引、查询和删除文档。

在本文中，我们将探索如何使用 SolrJ 与 Apache Solr 服务器进行交互。

## 2。设置

为了在您的机器上安装 Solr 服务器，请参考 [Solr 快速入门指南](https://web.archive.org/web/20220521220323/https://lucene.apache.org/solr/quickstart.html)。

安装过程很简单——只需下载 zip/tar 包，提取其中的内容，然后从命令行启动服务器。对于本文，我们将创建一个 Solr 服务器，其核心称为“bigboxstore”:

```java
bin/solr start
bin/solr create -c 'bigboxstore'
```

默认情况下，Solr 监听端口 8983 来接收 HTTP 查询。您可以通过在浏览器中打开`http://localhost:8983/solr/#/bigboxstore` URL 并观察 Solr 仪表板来验证它是否成功启动。

## 3。Maven 配置

现在我们已经启动并运行了 Solr 服务器，让我们直接跳到 SolrJ Java 客户端。要在您的项目中使用 SolrJ，您需要在您的`pom.xml`文件中声明以下 Maven 依赖项:

```java
<dependency>
    <groupId>org.apache.solr</groupId>
    <artifactId>solr-solrj</artifactId>
    <version>6.4.0</version>
</dependency>
```

你总能找到 [Maven Central](https://web.archive.org/web/20220521220323/https://search.maven.org/classic/#search%7Cga%7C1%7Cg%3A%22org.apache.solr%22%20a%3A%22solr-solrj%22) 托管的最新版本。

## 4。Apache 求解 Java API

让我们通过连接到我们的 Solr 服务器来启动 SolrJ 客户机:

```java
String urlString = "http://localhost:8983/solr/bigboxstore";
HttpSolrClient solr = new HttpSolrClient.Builder(urlString).build();
solr.setParser(new XMLResponseParser());
```

注意: **SolrJ 使用二进制格式，而不是 XML** 作为它的默认响应格式。为了与 Solr 兼容，需要显式调用 XML 的`setParser()` ,如上所示。更多细节可以在这里找到[。](https://web.archive.org/web/20220521220323/https://cwiki.apache.org/confluence/display/solr/Using+SolrJ)

### 4.1。索引文件

让我们使用`SolrInputDocument`定义要索引的数据，并使用`add()`方法将其添加到我们的索引中:

```java
SolrInputDocument document = new SolrInputDocument();
document.addField("id", "123456");
document.addField("name", "Kenmore Dishwasher");
document.addField("price", "599.99");
solr.add(document);
solr.commit();
```

**注意:任何修改 Solr 数据库的动作都需要在动作之后加上`commit()`。**

### 4.2。用豆子索引

**您还可以使用 beans** 来索引 Solr 文档。让我们定义一个 ProductBean，其属性用@ `Field`注释:

```java
public class ProductBean {

    String id;
    String name;
    String price;

    @Field("id")
    protected void setId(String id) {
        this.id = id;
    }

    @Field("name")
    protected void setName(String name) {
        this.name = name;
    }

    @Field("price")
    protected void setPrice(String price) {
        this.price = price;
    }

    // getters and constructor omitted for space
}
```

然后，让我们将 bean 添加到我们的索引中:

```java
solrClient.addBean( new ProductBean("888", "Apple iPhone 6s", "299.99") );
solrClient.commit();
```

### 4.3。通过字段和 Id 查询索引文档

让我们通过使用`SolrQuery` 查询 Solr 服务器来验证我们的文档是否被添加。

来自服务器的`QueryResponse` 将包含一个与格式为`field:value`的任何查询匹配的 `SolrDocument` 对象列表。在本例中，我们按价格查询:

```java
SolrQuery query = new SolrQuery();
query.set("q", "price:599.99");
QueryResponse response = solr.query(query);

SolrDocumentList docList = response.getResults();
assertEquals(docList.getNumFound(), 1);

for (SolrDocument doc : docList) {
     assertEquals((String) doc.getFieldValue("id"), "123456");
     assertEquals((Double) doc.getFieldValue("price"), (Double) 599.99);
}
```

一个更简单的选择是使用`getById()`通过`Id`进行查询。如果找到匹配，它将只返回一个文档:

```java
SolrDocument doc = solr.getById("123456");
assertEquals((String) doc.getFieldValue("name"), "Kenmore Dishwasher");
assertEquals((Double) doc.getFieldValue("price"), (Double) 599.99);
```

### 4.4。删除文件

当我们想从索引中删除一个文档时，我们可以使用`deleteById()`并验证它已经被删除:

```java
solr.deleteById("123456");
solr.commit();
SolrQuery query = new SolrQuery();
query.set("q", "id:123456");
QueryResponse response = solr.query(query);
SolrDocumentList docList = response.getResults();
assertEquals(docList.getNumFound(), 0);
```

我们还可以选择`deleteByQuery()`，所以让我们尝试删除任何具有特定名称的文档:

```java
solr.deleteByQuery("name:Kenmore Dishwasher");
solr.commit();
SolrQuery query = new SolrQuery();
query.set("q", "id:123456");
QueryResponse response = solr.query(query);
SolrDocumentList docList = response.getResults();
assertEquals(docList.getNumFound(), 0);
```

## 5。结论

在这篇简短的文章中，我们看到了如何使用 SolrJ Java API 来执行与 Apache Solr 全文搜索引擎的一些常见交互。

你可以在 GitHub 上查看本文[中提供的例子。](https://web.archive.org/web/20220521220323/https://github.com/eugenp/tutorials/tree/master/apache-libraries)