# 使用 Solr 进行全文搜索

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/full-text-search-with-solr>

## 1。概述

在本文中，我们将探索 [Apache Solr](https://web.archive.org/web/20221101164507/https://lucene.apache.org/solr/) 搜索引擎中的一个基本概念——全文搜索。

Apache Solr 是一个开源框架，旨在处理数百万份文档。我们将通过使用 Java 库的示例来了解 it 的核心功能—[SolrJ](https://web.archive.org/web/20221101164507/https://cwiki.apache.org/confluence/display/solr/Solrj)。

## 2。Maven 配置

鉴于 Solr 是开源的——我们可以简单地下载二进制文件并从我们的应用程序中单独启动服务器。

为了与服务器通信，我们将为 SolrJ 客户机定义 Maven 依赖关系:

```java
<dependency>
    <groupId>org.apache.solr</groupId>
    <artifactId>solr-solrj</artifactId>
    <version>6.4.2</version>
</dependency>
```

你可以在这里找到最新的依赖[。](https://web.archive.org/web/20221101164507/https://search.maven.org/classic/#search%7Cga%7C1%7Cg%3A%22org.apache.solr%22%20AND%20a%3A%22solr-solrj%22)

## 3。索引数据

为了索引和搜索数据，我们需要创建一个`core`；我们将创建一个名为`item`的来索引我们的数据。

在此之前，我们需要在服务器上对数据进行索引，以便可以搜索。

我们有许多不同的方法来索引数据。我们可以使用数据导入处理程序直接从关系数据库导入数据，使用 Apache Tika 通过 Solr Cell 上传数据，或者使用索引处理程序上传 XML/XSLT、JSON 和 CSV 数据。

### 3.1。索引 Solr 文档

我们可以通过创建`SolrInputDocument`将数据索引到`core`中。首先，我们需要用我们的数据填充文档，然后只调用 SolrJ 的 API 来索引文档:

```java
SolrInputDocument doc = new SolrInputDocument();
doc.addField("id", id);
doc.addField("description", description);
doc.addField("category", category);
doc.addField("price", price);
solrClient.add(doc);
solrClient.commit();
```

注意，`id`对于不同的`items`自然应该是唯一的。拥有一个已经索引的文档的`id` 将会更新那个文档。

### 3.2。索引豆

SolrJ 提供了用于索引 Java beans 的 API。为了索引一个 bean，我们需要用`@Field` 注释对它进行注释:

```java
public class Item {

    @Field
    private String id;

    @Field
    private String description;

    @Field
    private String category;

    @Field
    private float price;
}
```

一旦我们有了 bean，索引就简单了:

```java
solrClient.addBean(item); 
solrClient.commit();
```

## 4。Solr 查询

搜索是 Solr 最强大的功能。一旦我们在存储库中建立了文档索引，我们就可以搜索关键字、短语、日期范围等。结果按相关性(分数)排序。

### 4.1。基本查询

服务器公开了用于搜索操作的 API。我们可以调用`/select` 或`/query` 请求处理程序。

让我们做一个简单的搜索:

```java
SolrQuery query = new SolrQuery();
query.setQuery("brand1");
query.setStart(0);
query.setRows(10);

QueryResponse response = solrClient.query(query);
List<Item> items = response.getBeans(Item.class);
```

SolrJ 将在给服务器的请求中内部使用主查询参数`q` 。当没有指定`start` 和`rows` 时，返回的记录数将是 10，从零开始索引。

上面的搜索查询将查找在其任何索引字段中包含完整单词`“brand1”` 的任何文档。注意**简单搜索不区分大小写。**

**我们来看另一个例子。**我们想要搜索任何包含`“rand”`的单词，以任意数量的字符开始，以一个字符结束。我们可以在查询中使用通配符`*` 和`?` :

```java
query.setQuery("*rand?");
```

Solr 查询也支持类似 SQL 中的布尔运算符:

```java
query.setQuery("brand1 AND (Washing OR Refrigerator)");
```

所有布尔运算符必须全部大写；查询解析器支持的有`AND`、`OR, NOT`、 `+` 和-。

此外，如果我们想要搜索特定字段而不是所有索引字段，我们可以在查询中指定这些字段:

```java
query.setQuery("description:Brand* AND category:*Washing*");
```

### 4.2。短语查询

到目前为止，我们的代码在索引字段中寻找关键字。我们还可以对索引字段进行短语搜索:

```java
query.setQuery("Washing Machine");
```

当我们有一个类似于“`Washing Machine`”的短语时，Solr 的标准查询解析器会将其解析为“`Washing OR Machine`”。要搜索整个短语，我们只能在双引号内添加表达式:

```java
query.setQuery("\"Washing Machine\"");
```

我们可以使用邻近搜索来查找特定距离内的单词。如果我们想找到至少相隔两个单词的单词，我们可以使用下面的查询:

```java
query.setQuery("\"Washing equipment\"~2");
```

### 4.3。范围查询

范围查询允许获取字段在特定范围之间的文档。

假设我们想要查找价格范围在 100 到 300 之间的商品:

```java
query.setQuery("price:[100 TO 300]");
```

上面的查询将查找价格在 100 到 300 之间的所有元素。我们可以使用“`}`”和“`{`”来排除端点:

```java
query.setQuery("price:{100 TO 300]");
```

### 4.4。过滤查询

筛选查询可用于限制可返回的结果超集。过滤查询不影响分数:

```java
SolrQuery query = new SolrQuery();
query.setQuery("price:[100 TO 300]");
query.addFilterQuery("description:Brand1","category:Home Appliances");
```

通常，筛选查询将包含常用的查询。因为它们通常是可重用的，所以它们被缓存以使搜索更有效。

## 5。分面搜索

分面有助于将搜索结果排列成组计数。我们可以分面字段、查询或范围。

### 5.1。现场刻面

例如，我们希望获得搜索结果中类别的总数。我们可以在查询中添加`category` 字段:

```java
query.addFacetField("category");

QueryResponse response = solrClient.query(query);
List<Count> facetResults = response.getFacetField("category").getValues();
```

`facetResults` 将包含结果中每个类别的计数。

### 5.2。查询分面

当我们想要返回子查询的计数时，查询分面非常有用:

```java
query.addFacetQuery("Washing OR Refrigerator");
query.addFacetQuery("Brand2");

QueryResponse response = solrClient.query(query);
Map<String,Integer> facetQueryMap = response.getFacetQuery();
```

因此， `facetQueryMap` 将会有 facet 查询的计数。

### 5.3。靶场刻面

范围分面用于获取搜索结果中的范围计数。以下查询将返回介于 100 和 251 之间的价格范围的计数，间隔为 25:

```java
query.addNumericRangeFacet("price", 100, 275, 25);

QueryResponse response = solrClient.query(query);
List<RangeFacet> rangeFacets =  response.getFacetRanges().get(0).getCounts();
```

除了数字范围，Solr 还支持日期范围、间隔分面和透视分面。

## 6。点击高亮显示

我们可能希望搜索查询中的关键字在结果中突出显示。这将非常有助于更好地了解结果。让我们索引一些文档并定义要突出显示的关键字:

```java
itemSearchService.index("hm0001", "Brand1 Washing Machine", "Home Appliances", 100f);
itemSearchService.index("hm0002", "Brand1 Refrigerator", "Home Appliances", 300f);
itemSearchService.index("hm0003", "Brand2 Ceiling Fan", "Home Appliances", 200f);
itemSearchService.index("hm0004", "Brand2 Dishwasher", "Washing equipments", 250f);

SolrQuery query = new SolrQuery();
query.setQuery("Appliances");
query.setHighlight(true);
query.addHighlightField("category");
QueryResponse response = solrClient.query(query);

Map<String, Map<String, List<String>>> hitHighlightedMap = response.getHighlighting();
Map<String, List<String>> highlightedFieldMap = hitHighlightedMap.get("hm0001");
List<String> highlightedList = highlightedFieldMap.get("category");
String highLightedText = highlightedList.get(0);
```

我们将把`highLightedText`作为`Home <em>Appliances</em>`。请注意，搜索关键字`Appliances` 带有`<em>`标记。Solr 使用的默认高亮标签是`<em>`，但是我们可以通过设置`pre` 和`post` 标签来改变它:

```java
query.setHighlightSimplePre("<strong>");
query.setHighlightSimplePost("</strong>");
```

## 7 .**。搜索建议**

Solr 支持的一个重要特性是建议。如果查询中的关键字包含拼写错误，或者如果我们希望建议自动完成搜索关键字，我们可以使用建议功能。

### 7.1。拼写检查

标准搜索处理程序不包括拼写检查组件；必须手动配置。有三种方法可以做到。你可以在官方的[维基页面](https://web.archive.org/web/20221101164507/https://cwiki.apache.org/confluence/display/solr/Spell+Checking)中找到配置细节。在我们的例子中，我们将使用`IndexBasedSpellChecker`，它使用索引数据进行关键字拼写检查。

让我们搜索一个有拼写错误的关键词:

```java
query.setQuery("hme");
query.set("spellcheck", "on");
QueryResponse response = solrClient.query(query);

SpellCheckResponse spellCheckResponse = response.getSpellCheckResponse();
Suggestion suggestion = spellCheckResponse.getSuggestions().get(0);
List<String> alternatives = suggestion.getAlternatives();
String alternative = alternatives.get(0);
```

我们的关键字`“hme”` 的预期替代应该是`“home”` ,因为我们的索引包含术语`“home”.` 。注意在执行搜索之前必须激活`spellcheck` 。

### 7.2。自动建议术语

我们可能希望得到不完整关键字的建议来帮助搜索。Solr 建议的组件必须手动配置。你可以在它的官方 [wiki 页面](https://web.archive.org/web/20221101164507/https://cwiki.apache.org/confluence/display/solr/Suggester)中找到配置细节。

我们已经配置了一个名为`/suggest` 的请求处理器来处理建议。让我们为关键词`“Hom”`获得建议:

```java
SolrQuery query = new SolrQuery();
query.setRequestHandler("/suggest");
query.set("suggest", "true");
query.set("suggest.build", "true");
query.set("suggest.dictionary", "mySuggester");
query.set("suggest.q", "Hom");
QueryResponse response = solrClient.query(query);

SuggesterResponse suggesterResponse = response.getSuggesterResponse();
Map<String,List<String>> suggestedTerms = suggesterResponse.getSuggestedTerms();
List<String> suggestions = suggestedTerms.get("mySuggester");
```

列表`suggestions` 应该包含所有单词和短语。注意，我们在配置中配置了一个名为`mySuggester`的建议器。

## 8。结论

本文快速介绍了搜索引擎 Solr 的功能和特性。

我们提到了许多特性，但这些当然只是我们可以用 Solr 这样的先进和成熟的搜索服务器做的事情的皮毛。

这里使用的例子可以在 GitHub 上找到。