# Hibernate 搜索简介

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/hibernate-search>

## 1。概述

在本文中，我们将讨论 Hibernate 搜索的基础知识，如何配置它，并且我们将实现一些简单的查询。

## 2。休眠搜索的基础知识

每当我们必须实现全文搜索功能时，使用我们已经非常熟悉的工具总是有利的。

如果我们已经在为 ORM 使用 Hibernate 和 JPA，我们离 Hibernate Search 只有一步之遥。

Hibernate Search 集成了 Apache Lucene，这是一个用 Java 编写的高性能、可扩展的全文搜索引擎库。这结合了 Lucene 的强大和 Hibernate 和 JPA 的简单。

简单地说，我们只需要给我们的领域类添加一些额外的注释，这个工具会处理像数据库/索引同步这样的事情。

Hibernate Search 还提供了一个 Elasticsearch 集成；然而，由于它仍处于实验阶段，我们在这里将重点关注 Lucene。

## 3。配置

### 3.1。Maven 依赖关系

在开始之前，我们首先需要将必要的[依赖项](https://web.archive.org/web/20220526043218/https://search.maven.org/classic/#search%7Cgav%7C1%7Cg%3A%22org.hibernate%22%20AND%20a%3A%22hibernate-search-orm%22)添加到我们的`pom.xml`中:

```java
<dependency>
    <groupId>org.hibernate</groupId>
    <artifactId>hibernate-search-orm</artifactId>
    <version>5.8.2.Final</version>
</dependency>
```

为了简单起见，我们将使用 [H2](https://web.archive.org/web/20220526043218/https://search.maven.org/classic/#search%7Cga%7C1%7Cg%3A%22com.h2database%22%20AND%20a%3A%22h2%22) 作为数据库:

```java
<dependency>
    <groupId>com.h2database</groupId> 
    <artifactId>h2</artifactId>
    <version>1.4.196</version>
</dependency>
```

### 3.2。配置

我们还必须指定 Lucene 应该在哪里存储索引。

这可以通过属性`hibernate.search.default.directory_provider`来完成。

我们将选择`filesystem`，这是我们用例中最简单的选项。更多选项在[官方文档](https://web.archive.org/web/20220526043218/https://docs.jboss.org/hibernate/stable/search/reference/en-US/html_single/#search-configuration-directory)中列出。对于集群应用程序来说，`Filesystem-master` / `filesystem-slave`和`infinispan`是值得注意的，在集群应用程序中，索引必须在节点之间同步。

我们还必须定义一个存储索引的默认基本目录:

```java
hibernate.search.default.directory_provider = filesystem
hibernate.search.default.indexBase = /data/index/default
```

## 4。模型类

配置完成后，我们现在可以指定我们的模型了。

**在 JPA 注释`@Entity`和`@Table`之上，我们必须添加一个`@Indexed`注释。**它告诉 Hibernate Search 实体`Product`应该被索引。

**之后，我们必须通过添加一个`@Field`注释**将所需的属性定义为可搜索的:

```java
@Entity
@Indexed
@Table(name = "product")
public class Product {

    @Id
    private int id;

    @Field(termVector = TermVector.YES)
    private String productName;

    @Field(termVector = TermVector.YES)
    private String description;

    @Field
    private int memory;

    // getters, setters, and constructors
}
```

稍后的“更像这样”查询将需要`termVector = TermVector.YES`属性。

## 5。构建 Lucene 索引

在开始实际的查询之前，**我们必须首先触发 Lucene 构建索引**:

```java
FullTextEntityManager fullTextEntityManager 
  = Search.getFullTextEntityManager(entityManager);
fullTextEntityManager.createIndexer().startAndWait();
```

在这个初始构建之后，Hibernate Search 将负责保持索引最新。也就是说，我们可以像往常一样通过`EntityManager`创建、操作和删除实体。

注意:**我们必须确保实体在被 Lucene** 发现和索引之前完全提交给数据库(顺便说一下，这也是为什么我们的[示例代码测试用例](https://web.archive.org/web/20220526043218/https://github.com/eugenp/tutorials/tree/master/persistence-modules/spring-hibernate-5)中的初始测试数据导入到一个专用的 JUnit 测试用例中的原因，用`@Commit`进行了注释)。

## 6。构建和执行查询

现在，我们准备好创建我们的第一个查询。

在下一节中，**我们将展示准备和执行查询的一般工作流程。**

之后，我们将为最重要的查询类型创建一些示例查询。

### 6.1。创建和执行查询的一般工作流程

**准备和执行查询通常包括四个步骤**:

在步骤 1 中，我们必须获得一个 JPA `FullTextEntityManager`并从中获得一个`QueryBuilder`:

```java
FullTextEntityManager fullTextEntityManager 
  = Search.getFullTextEntityManager(entityManager);

QueryBuilder queryBuilder = fullTextEntityManager.getSearchFactory() 
  .buildQueryBuilder()
  .forEntity(Product.class)
  .get();
```

在第 2 步中，我们将通过 Hibernate query DSL 创建一个 Lucene 查询:

```java
org.apache.lucene.search.Query query = queryBuilder
  .keyword()
  .onField("productName")
  .matching("iphone")
  .createQuery();
```

在第 3 步中，我们将把 Lucene 查询包装成 Hibernate 查询:

```java
org.hibernate.search.jpa.FullTextQuery jpaQuery
  = fullTextEntityManager.createFullTextQuery(query, Product.class);
```

最后，在步骤 4 中，我们将执行查询:

```java
List<Product> results = jpaQuery.getResultList();
```

默认情况下，Lucene 根据相关性对结果进行排序。

对于所有查询类型，步骤 1、3 和 4 都是相同的。

在下文中，我们将关注第 2 步，即如何创建不同类型的查询。

### 6.2。关键词查询

最基本的用例是**搜索特定的单词**。

这就是我们在上一节中实际已经做的事情:

```java
Query keywordQuery = queryBuilder
  .keyword()
  .onField("productName")
  .matching("iphone")
  .createQuery();
```

这里，`keyword()`指定我们在寻找一个特定的单词，`onField()`告诉 Lucene 去哪里找，`matching()`告诉我们要找什么。

### 6.3。模糊查询

模糊查询的工作方式类似于关键字查询，除了**我们可以定义一个“模糊性”的界限**，超过这个界限 Lucene 将接受这两个词作为匹配。

通过`withEditDistanceUpTo()`、**我们可以定义一个项可以偏离另一个**多少。它可以设置为 0、1 和 2，缺省值为 2 ( `note`:这个限制来自 Lucene 的实现)。

通过`withPrefixLength()`，我们可以定义被模糊性忽略的前缀的长度:

```java
Query fuzzyQuery = queryBuilder
  .keyword()
  .fuzzy()
  .withEditDistanceUpTo(2)
  .withPrefixLength(0)
  .onField("productName")
  .matching("iPhaen")
  .createQuery();
```

### 6.4。通配符查询

Hibernate Search 还使我们能够执行通配符查询，即部分单词未知的查询。

为此，我们可以使用“`?”`表示单个字符，使用“`*”`表示任何字符序列:

```java
Query wildcardQuery = queryBuilder
  .keyword()
  .wildcard()
  .onField("productName")
  .matching("Z*")
  .createQuery();
```

### 6.5。短语查询

如果我们想要搜索一个以上的单词，我们可以使用短语查询。如果有必要的话，我们可以使用`phrase()`和`withSlop()`在**中寻找精确或近似的句子**。斜率定义了句子中允许的其他单词的数量:

```java
Query phraseQuery = queryBuilder
  .phrase()
  .withSlop(1)
  .onField("description")
  .sentence("with wireless charging")
  .createQuery();
```

### 6.6。简单查询字符串查询

对于以前的查询类型，我们必须显式指定查询类型。

如果我们想给用户更多的权力，我们可以使用简单的查询字符串 query】这样，他可以在运行时定义自己的查询。

支持以下查询类型:

*   布尔型(并且使用“+”，或者使用“|”，而不使用“-”)
*   前缀(前缀*)
*   短语(“某个短语”)
*   优先级(使用括号)
*   模糊(模糊～2)
*   短语查询的 near 运算符(" some phrase"~3)

以下示例将结合模糊、短语和布尔查询:

```java
Query simpleQueryStringQuery = queryBuilder
  .simpleQueryString()
  .onFields("productName", "description")
  .matching("Aple~2 + \"iPhone X\" + (256 | 128)")
  .createQuery();
```

### 6.7。范围查询

**范围查询搜索给定边界**之间的 **值。这可以应用于数字、日期、时间戳和字符串:**

```java
Query rangeQuery = queryBuilder
  .range()
  .onField("memory")
  .from(64).to(256)
  .createQuery();
```

### 6.8。更像这样的查询

我们最后一个查询类型是"`More Like This`"–查询。为此，我们提供一个实体， **Hibernate Search 返回一个包含相似实体**的列表，每个实体都有一个相似性得分。

如前所述，我们的模型类中的`termVector = TermVector.YES`属性对于这种情况是必需的:它告诉 Lucene 在索引期间存储每个术语的频率。

基于此，将在查询执行时计算相似性:

```java
Query moreLikeThisQuery = queryBuilder
  .moreLikeThis()
  .comparingField("productName").boostedTo(10f)
  .andField("description").boostedTo(1f)
  .toEntity(entity)
  .createQuery();
List<Object[]> results = (List<Object[]>) fullTextEntityManager
  .createFullTextQuery(moreLikeThisQuery, Product.class)
  .setProjection(ProjectionConstants.THIS, ProjectionConstants.SCORE)
  .getResultList();
```

### 6.9。搜索多个字段

到目前为止，我们只使用`onField()`为搜索一个属性创建了查询。

根据用例的不同，**我们还可以搜索两个或更多属性**:

```java
Query luceneQuery = queryBuilder
  .keyword()
  .onFields("productName", "description")
  .matching(text)
  .createQuery();
```

此外，**我们可以分别指定要搜索的每个属性**，例如，如果我们想要为一个属性定义一个提升:

```java
Query moreLikeThisQuery = queryBuilder
  .moreLikeThis()
  .comparingField("productName").boostedTo(10f)
  .andField("description").boostedTo(1f)
  .toEntity(entity)
  .createQuery();
```

### 6.10。组合查询

最后，Hibernate Search 还支持使用各种策略组合查询:

*   `SHOULD:`查询应该包含子查询的匹配元素
*   `MUST:`查询必须包含子查询的匹配元素
*   `MUST NOT`:查询不能包含子查询的匹配元素

聚合**类似于布尔聚合`AND, OR`和布尔聚合`NOT`** `.`，但是名称不同，以强调它们也对相关性有影响。

例如，两个查询之间的一个`SHOULD`类似于布尔`OR:`如果两个查询中有一个匹配，那么这个匹配将被返回。

但是，如果两个查询都匹配，则与只有一个查询匹配相比，匹配将具有更高的相关性:

```java
Query combinedQuery = queryBuilder
  .bool()
  .must(queryBuilder.keyword()
    .onField("productName").matching("apple")
    .createQuery())
  .must(queryBuilder.range()
    .onField("memory").from(64).to(256)
    .createQuery())
  .should(queryBuilder.phrase()
    .onField("description").sentence("face id")
    .createQuery())
  .must(queryBuilder.keyword()
    .onField("productName").matching("samsung")
    .createQuery())
  .not()
  .createQuery();
```

## 7。结论

在本文中，我们讨论了 Hibernate 搜索的基础，并展示了如何实现最重要的查询类型。更多高级主题可以在官方文档中找到。

与往常一样，GitHub 上的[提供了示例的完整源代码。](https://web.archive.org/web/20220526043218/https://github.com/eugenp/tutorials/tree/master/persistence-modules/spring-hibernate-5)