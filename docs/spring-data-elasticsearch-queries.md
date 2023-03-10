# 使用 Spring 数据的弹性搜索查询

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-data-elasticsearch-queries>

## 1。简介

在[之前的一篇文章](/web/20220625165452/https://www.baeldung.com/spring-data-elasticsearch-tutorial)中，我们展示了如何为一个项目配置和使用 Spring Data Elasticsearch。在本文中，我们将研究 Elasticsearch 提供的几种查询类型，我们还将讨论字段分析器及其对搜索结果的影响。

## 2。分析仪

默认情况下，所有存储的字符串字段都由分析器处理。分析器由一个记号赋予器和几个记号过滤器组成，通常前面有一个或多个字符过滤器。

默认的分析器通过常用的单词分隔符(如空格或标点)来分割字符串，并将每个标记都用小写字母表示。它还忽略了常见的英语单词。

Elasticsearch 也可以配置为同时将一个字段视为已分析和未分析。

例如，在一个`Article`类中，假设我们将标题字段存储为一个标准的分析字段。带有后缀`verbatim`的相同字段将被存储为未分析字段:

```java
@MultiField(
  mainField = @Field(type = Text, fielddata = true),
  otherFields = {
      @InnerField(suffix = "verbatim", type = Keyword)
  }
)
private String title;
```

这里，我们应用了`@MultiField`注释来告诉 Spring 数据，我们希望这个字段以多种方式进行索引。主字段将使用名称`title`,并将根据上述规则进行分析。

但是我们还提供了第二个注释，`@InnerField`，它描述了`title` 字段的附加索引。我们使用`FieldType.keyword`来表示，在执行字段的额外索引时，我们不想使用分析器，并且应该使用带有后缀`verbatim`的嵌套字段来存储这个值。

### 2.1。分析字段

让我们看一个例子。假设一篇标题为“Spring Data Elasticsearch”的文章被添加到我们的索引中。默认分析器将在空格字符处分解字符串，并生成小写标记:“`spring`”、“`data”,`和“`elasticsearch`”。

现在，我们可以使用这些术语的任意组合来匹配文档:

```java
NativeSearchQuery searchQuery = new NativeSearchQueryBuilder()
  .withQuery(matchQuery("title", "elasticsearch data"))
  .build();
```

### 2.2。未分析的字段

未分析的字段没有标记化，因此在使用匹配或术语查询时只能作为一个整体进行匹配:

```java
NativeSearchQuery searchQuery = new NativeSearchQueryBuilder()
  .withQuery(matchQuery("title.verbatim", "Second Article About Elasticsearch"))
  .build();
```

使用匹配查询，我们可能只能通过完整的标题进行搜索，这也是区分大小写的。

## 3。匹配查询

一个**匹配查询**接受文本、数字和日期。

有三种类型“匹配”查询:

*   `**boolean**`
*   `**phrase**`和
*   `**phrase_prefix**`

在本节中，我们将探索`boolean`匹配查询。

### 3.1。用布尔运算符匹配

`boolean`是匹配查询的默认类型；您可以指定使用哪个布尔运算符(`or`是默认的):

```java
NativeSearchQuery searchQuery = new NativeSearchQueryBuilder()
  .withQuery(matchQuery("title","Search engines").operator(Operator.AND))
  .build();
SearchHits<Article> articles = elasticsearchTemplate()
  .search(searchQuery, Article.class, IndexCoordinates.of("blog"));
```

该查询将返回一篇标题为“Search engines”的文章，方法是使用`and`操作符指定标题中的两个术语。但是如果我们使用默认的(`or`)操作符进行搜索，当只有一个词匹配时会发生什么呢？

```java
NativeSearchQuery searchQuery = new NativeSearchQueryBuilder()
  .withQuery(matchQuery("title", "Engines Solutions"))
  .build();
SearchHits<Article> articles = elasticsearchTemplate()
  .search(searchQuery, Article.class, IndexCoordinates.of("blog"));
assertEquals(1, articles.getTotalHits());
assertEquals("Search engines", articles.getSearchHit(0).getContent().getTitle());
```

“`Search engines`”文章仍然匹配，但是它将具有较低的分数，因为不是所有的术语都匹配。

每个匹配项的得分总和加起来就是每个结果文档的总得分。

可能存在这样的情况，其中包含查询中输入的罕见术语的文档将比包含几个常见术语的文档具有更高的排名。

### 3.2。模糊性

当用户在一个单词中输入错误时，仍然可以通过指定一个`fuzziness` 参数来匹配它，这允许不精确的匹配。

对于字符串字段，`fuzziness`表示编辑距离:为了使一个字符串与另一个字符串相同，需要对该字符串进行的单字符更改的数量。

```java
NativeSearchQuery searchQuery = new NativeSearchQueryBuilder()
  .withQuery(matchQuery("title", "spring date elasticsearch")
  .operator(Operator.AND)
  .fuzziness(Fuzziness.ONE)
  .prefixLength(3))
  .build();
```

`prefix_length`参数用于提高性能。在这种情况下，我们要求前三个字符应该完全匹配，这减少了可能的组合数量。

## 5。短语搜索

相位搜索更严格，尽管您可以用`slop`参数来控制它。该参数告诉短语 query，在仍然认为文档匹配的情况下，术语之间允许有多远的距离。

换句话说，它表示为了使查询和文档匹配，需要移动术语的次数:

```java
NativeSearchQuery searchQuery = new NativeSearchQueryBuilder()
  .withQuery(matchPhraseQuery("title", "spring elasticsearch").slop(1))
  .build();
```

这里，查询将匹配标题为“`Spring Data Elasticsearch`”的文档，因为我们将 slop 设置为 1。

## 6。多匹配查询

当您想要在多个字段中搜索时，您可以使用`QueryBuilders#multiMatchQuery()`来指定所有要匹配的字段:

```java
NativeSearchQuery searchQuery = new NativeSearchQueryBuilder()
  .withQuery(multiMatchQuery("tutorial")
    .field("title")
    .field("tags")
    .type(MultiMatchQueryBuilder.Type.BEST_FIELDS))
  .build();
```

这里我们搜索`title`和`tags`字段进行匹配。

请注意，这里我们使用“最佳字段”评分策略。它会将字段中的最大分数作为文档分数。

## 7 .**。聚合**

在我们的 `Article`类中，我们还定义了一个`tags`字段，它是未分析的。我们可以通过使用聚合来轻松创建标签云。

请记住，因为字段未经分析，所以标记不会被标记化:

```java
TermsAggregationBuilder aggregation = AggregationBuilders.terms("top_tags")
  .field("tags")
  .order(Terms.Order.count(false));
SearchSourceBuilder builder = new SearchSourceBuilder().aggregation(aggregation);

SearchRequest searchRequest = 
  new SearchRequest().indices("blog").types("article").source(builder);
SearchResponse response = client.search(searchRequest, RequestOptions.DEFAULT);

Map<String, Aggregation> results = response.getAggregations().asMap();
StringTerms topTags = (StringTerms) results.get("top_tags");

List<String> keys = topTags.getBuckets()
  .stream()
  .map(b -> b.getKeyAsString())
  .collect(toList());
assertEquals(asList("elasticsearch", "spring data", "search engines", "tutorial"), keys);
```

## 8。总结

在本文中，我们讨论了已分析和未分析字段之间的区别，以及这种区别如何影响搜索。

我们还学习了 Elasticsearch 提供的几种类型的查询，比如匹配查询、短语匹配查询、全文搜索查询和布尔查询。

Elasticsearch 提供了许多其他类型的查询，如地理查询、脚本查询和复合查询。您可以在 [Elasticsearch 文档](https://web.archive.org/web/20220625165452/https://www.elastic.co/guide/en/elasticsearch/reference/current/query-dsl.html)中阅读它们，并探索 Spring Data Elasticsearch API，以便在您的代码中使用这些查询。

您可以在 GitHub 库的[中找到包含本文所用示例的项目。](https://web.archive.org/web/20220625165452/https://github.com/eugenp/tutorials/tree/master/persistence-modules/spring-data-elasticsearch)