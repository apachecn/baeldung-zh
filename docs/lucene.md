# Apache Lucene 简介

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/lucene>

## 1。概述

Apache Lucene 是一个全文搜索引擎，可以在各种编程语言中使用。

在本文中，我们将尝试理解这个库的核心概念，并创建一个简单的应用程序。

## 2。Maven 设置

首先，让我们添加必要的依赖项:

```java
<dependency>        
    <groupId>org.apache.lucene</groupId>          
    <artifactId>lucene-core</artifactId>
    <version>7.1.0</version>
</dependency>
```

最新版本可以在这里找到[。](https://web.archive.org/web/20220928002820/https://search.maven.org/classic/#search%7Cgav%7C1%7Cg%3A%22org.apache.lucene%22%20AND%20a%3A%22lucene-core%22)

此外，为了解析我们的搜索查询，我们需要:

```java
<dependency>
    <groupId>org.apache.lucene</groupId>
    <artifactId>lucene-queryparser</artifactId>
    <version>7.1.0</version>
</dependency>
```

点击查看最新版本[。](https://web.archive.org/web/20220928002820/https://search.maven.org/classic/#search%7Cgav%7C1%7Cg%3A%22org.apache.lucene%22%20AND%20a%3A%22lucene-queryparser%22)

## 3。核心概念

### 3.1。索引

简单地说，Lucene 使用数据的“倒排索引”—**而不是将页面映射到关键词，它将关键词映射到页面**,就像任何一本书末尾的术语表一样。

这允许更快的搜索响应，因为它通过索引搜索，而不是直接通过文本搜索。

### 3.2。文件

在这里，文档是字段的集合，每个字段都有一个与之关联的值。

索引通常由一个或多个文档组成，搜索结果是一组最匹配的文档。

它并不总是一个纯文本文档，它也可能是一个数据库表或集合。

### 3.3。字段

文档可以有字段数据，其中字段通常是保存数据值的键:

```java
title: Goodness of Tea
body: Discussing goodness of drinking herbal tea...
```

注意，这里的`title`和`body`是字段，可以一起或单独搜索。

### 3.4。分析

分析是将给定的文本转换成更小更精确的单元，以便于搜索。

文本经过提取关键字、去除常用单词和标点符号、将单词改为小写等各种操作。

为此，有多个内置分析器:

1.  `StandardAnalyzer`–基于基本语法的分析，删除“a”、“an”等停用词。也转换成小写
2.  `SimpleAnalyzer`–根据非字母字符断开文本并转换成小写
3.  `WhiteSpaceAnalyzer`–根据空格断开文本

还有更多的分析器可供我们使用和定制。

### 3.5。正在搜索

一旦构建了一个索引，我们就可以使用一个`Query`和一个`IndexSearcher.`来搜索该索引。搜索结果通常是一个结果集，包含检索到的数据。

注意，`IndexWritter` 负责创建索引，而`IndexSearcher` 负责搜索索引。

### 3.6。查询语法

Lucene 提供了非常动态且易于编写的查询语法。

要搜索自由文本，我们只需使用文本`String`作为查询。

要搜索特定字段中的文本，我们可以使用:

```java
fieldName:text

eg: title:tea
```

范围搜索:

```java
timestamp:[1509909322,1572981321] 
```

我们还可以使用通配符进行搜索:

```java
dri?nk
```

将搜索单个字符来代替通配符“？”

```java
d*k
```

搜索以“d”开头、以“k”结尾、中间有多个字符的单词。

```java
uni*
```

会找到以“uni”开头的单词。

我们也可以组合这些查询并创建更复杂的查询。并包括逻辑运算符，如 And、NOT 或:

```java
title: "Tea in breakfast" AND "coffee"
```

关于查询语法[的更多信息，请点击](https://web.archive.org/web/20220928002820/https://lucene.apache.org/core/2_9_4/queryparsersyntax.html)。

## 4.一个简单的应用

让我们创建一个简单的应用程序，并索引一些文档。

首先，我们将创建一个内存索引，并向其中添加一些文档:

```java
...
Directory memoryIndex = new RAMDirectory();
StandardAnalyzer analyzer = new StandardAnalyzer();
IndexWriterConfig indexWriterConfig = new IndexWriterConfig(analyzer);
IndexWriter writter = new IndexWriter(memoryIndex, indexWriterConfig);
Document document = new Document();

document.add(new TextField("title", title, Field.Store.YES));
document.add(new TextField("body", body, Field.Store.YES));

writter.addDocument(document);
writter.close(); 
```

在这里，我们用`TextField` 创建一个文档，并使用`IndexWriter.`将它们添加到索引中。`TextField` 构造函数中的第三个参数指示是否也要存储该字段的值。

分析器用于将数据或文本分割成块，然后从中过滤出停用词。停用词是像“a”、“am”、“is”等词。这些完全取决于给定的语言。

接下来，让我们创建一个搜索查询，并在索引中搜索添加的文档:

```java
public List<Document> searchIndex(String inField, String queryString) {
    Query query = new QueryParser(inField, analyzer)
      .parse(queryString);

    IndexReader indexReader = DirectoryReader.open(memoryIndex);
    IndexSearcher searcher = new IndexSearcher(indexReader);
    TopDocs topDocs = searcher.search(query, 10);
    List<Document> documents = new ArrayList<>();
    for (ScoreDoc scoreDoc : topDocs.scoreDocs) {
        documents.add(searcher.doc(scoreDoc.doc));
    }

    return documents;
}
```

在`search()` 方法中，第二个整数参数表示应该返回多少个顶级搜索结果。

现在我们来测试一下:

```java
@Test
public void givenSearchQueryWhenFetchedDocumentThenCorrect() {
    InMemoryLuceneIndex inMemoryLuceneIndex 
      = new InMemoryLuceneIndex(new RAMDirectory(), new StandardAnalyzer());
    inMemoryLuceneIndex.indexDocument("Hello world", "Some hello world");

    List<Document> documents 
      = inMemoryLuceneIndex.searchIndex("body", "world");

    assertEquals(
      "Hello world", 
      documents.get(0).get("title"));
}
```

这里，我们将一个简单的文档添加到索引中，带有两个字段“title”和“body ”,然后尝试使用搜索查询来搜索相同的内容。

## 6。Lucene 查询

现在我们已经熟悉了索引和搜索的基础，让我们再深入一点。

在前面的章节中，我们已经看到了基本的查询语法，以及如何使用`QueryParser.` 将其转换成`Query` 实例

Lucene 也提供了各种具体的实现:

### 6.1。`TermQuery`

A `Term`是搜索的基本单位，包含字段名和要搜索的文本。

`TermQuery` 是由一个词组成的所有查询中最简单的一个:

```java
@Test
public void givenTermQueryWhenFetchedDocumentThenCorrect() {
    InMemoryLuceneIndex inMemoryLuceneIndex 
      = new InMemoryLuceneIndex(new RAMDirectory(), new StandardAnalyzer());
    inMemoryLuceneIndex.indexDocument("activity", "running in track");
    inMemoryLuceneIndex.indexDocument("activity", "Cars are running on road");

    Term term = new Term("body", "running");
    Query query = new TermQuery(term);

    List<Document> documents = inMemoryLuceneIndex.searchIndex(query);
    assertEquals(2, documents.size());
}
```

### 6.2。`PrefixQuery`

若要搜索包含“开头为”单词的文稿:

```java
@Test
public void givenPrefixQueryWhenFetchedDocumentThenCorrect() {
    InMemoryLuceneIndex inMemoryLuceneIndex 
      = new InMemoryLuceneIndex(new RAMDirectory(), new StandardAnalyzer());
    inMemoryLuceneIndex.indexDocument("article", "Lucene introduction");
    inMemoryLuceneIndex.indexDocument("article", "Introduction to Lucene");

    Term term = new Term("body", "intro");
    Query query = new PrefixQuery(term);

    List<Document> documents = inMemoryLuceneIndex.searchIndex(query);
    assertEquals(2, documents.size());
}
```

### 6.3。`WildcardQuery`

顾名思义，我们可以使用通配符“*”或“？”用于搜索:

```java
// ...
Term term = new Term("body", "intro*");
Query query = new WildcardQuery(term);
// ...
```

### 6.4。`PhraseQuery`

它用于搜索文档中的一系列文本:

```java
// ...
inMemoryLuceneIndex.indexDocument(
  "quotes", 
  "A rose by any other name would smell as sweet.");

Query query = new PhraseQuery(
  1, "body", new BytesRef("smell"), new BytesRef("sweet"));

List<Document> documents = inMemoryLuceneIndex.searchIndex(query);
// ...
```

注意,`PhraseQuery` 构造函数的第一个参数叫做`slop,`,它是要匹配的词之间的单词数距离。

### 6.5。`FuzzyQuery`

我们可以在搜索相似但不一定相同的东西时使用这个:

```java
// ...
inMemoryLuceneIndex.indexDocument("article", "Halloween Festival");
inMemoryLuceneIndex.indexDocument("decoration", "Decorations for Halloween");

Term term = new Term("body", "hallowen");
Query query = new FuzzyQuery(term);

List<Document> documents = inMemoryLuceneIndex.searchIndex(query);
// ...
```

我们试着搜索“万圣节”这个词，但结果却是拼写错误的“万圣节”。

### 6.6。`BooleanQuery`

有时我们可能需要执行复杂的搜索，将两种或多种不同类型的查询结合起来:

```java
// ...
inMemoryLuceneIndex.indexDocument("Destination", "Las Vegas singapore car");
inMemoryLuceneIndex.indexDocument("Commutes in singapore", "Bus Car Bikes");

Term term1 = new Term("body", "singapore");
Term term2 = new Term("body", "car");

TermQuery query1 = new TermQuery(term1);
TermQuery query2 = new TermQuery(term2);

BooleanQuery booleanQuery 
  = new BooleanQuery.Builder()
    .add(query1, BooleanClause.Occur.MUST)
    .add(query2, BooleanClause.Occur.MUST)
    .build();
// ...
```

## 7.排序搜索结果

我们还可以根据某些字段对搜索结果文档进行排序:

```java
@Test
public void givenSortFieldWhenSortedThenCorrect() {
    InMemoryLuceneIndex inMemoryLuceneIndex 
      = new InMemoryLuceneIndex(new RAMDirectory(), new StandardAnalyzer());
    inMemoryLuceneIndex.indexDocument("Ganges", "River in India");
    inMemoryLuceneIndex.indexDocument("Mekong", "This river flows in south Asia");
    inMemoryLuceneIndex.indexDocument("Amazon", "Rain forest river");
    inMemoryLuceneIndex.indexDocument("Rhine", "Belongs to Europe");
    inMemoryLuceneIndex.indexDocument("Nile", "Longest River");

    Term term = new Term("body", "river");
    Query query = new WildcardQuery(term);

    SortField sortField 
      = new SortField("title", SortField.Type.STRING_VAL, false);
    Sort sortByTitle = new Sort(sortField);

    List<Document> documents 
      = inMemoryLuceneIndex.searchIndex(query, sortByTitle);
    assertEquals(4, documents.size());
    assertEquals("Amazon", documents.get(0).getField("title").stringValue());
}
```

我们尝试按照标题字段对获取的文档进行排序，标题字段是河流的名称。`SortField` 构造函数的布尔参数用于反转排序顺序。

## 8.从索引中删除文档

让我们试着根据给定的`Term:`从索引中删除一些文档

```java
// ...
IndexWriterConfig indexWriterConfig = new IndexWriterConfig(analyzer);
IndexWriter writer = new IndexWriter(memoryIndex, indexWriterConfig);
writer.deleteDocuments(term);
// ...
```

我们将对此进行测试:

```java
@Test
public void whenDocumentDeletedThenCorrect() {
    InMemoryLuceneIndex inMemoryLuceneIndex 
      = new InMemoryLuceneIndex(new RAMDirectory(), new StandardAnalyzer());
    inMemoryLuceneIndex.indexDocument("Ganges", "River in India");
    inMemoryLuceneIndex.indexDocument("Mekong", "This river flows in south Asia");

    Term term = new Term("title", "ganges");
    inMemoryLuceneIndex.deleteDocument(term);

    Query query = new TermQuery(term);

    List<Document> documents = inMemoryLuceneIndex.searchIndex(query);
    assertEquals(0, documents.size());
}
```

## 9。结论

本文是 Apache Lucene 入门的快速介绍。此外，我们执行了各种查询，并对检索到的文档进行了排序。

和往常一样，例子的代码可以在 Github 上找到[。](https://web.archive.org/web/20220928002820/https://github.com/eugenp/tutorials/tree/master/lucene)