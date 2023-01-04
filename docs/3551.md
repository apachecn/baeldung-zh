# Lucene 分析器指南

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/lucene-analyzers>

## 1。概述

Lucene 分析器用于在索引和搜索文档时分析文本。

我们在[入门教程](/web/20220928005740/https://www.baeldung.com/lucene)中简单提到了分析器。

在本教程中，**我们将讨论常用的分析器，如何构建我们的自定义分析器，以及如何为不同的文档字段分配不同的分析器**。

## 2。Maven 依赖关系

首先，我们需要将这些依赖项添加到我们的`pom.xml`:

```
<dependency>
    <groupId>org.apache.lucene</groupId>
    <artifactId>lucene-core</artifactId>
    <version>7.4.0</version>
</dependency>
<dependency>
    <groupId>org.apache.lucene</groupId>
    <artifactId>lucene-queryparser</artifactId>
    <version>7.4.0</version>
</dependency>
<dependency>
    <groupId>org.apache.lucene</groupId>
    <artifactId>lucene-analyzers-common</artifactId>
    <version>7.4.0</version>
</dependency>
```

最新的 Lucene 版本可以在[这里](https://web.archive.org/web/20220928005740/https://search.maven.org/classic/#search%7Cgav%7C1%7Cg%3A%22org.apache.lucene%22%20AND%20a%3A%22lucene-core%22)找到。

## 3。Lucene 分析器

Lucene 分析器将文本分割成标记。

分析器主要由记号赋予器和过滤器组成。不同的分析器由标记器和过滤器的不同组合组成。

为了展示常用分析仪之间的差异，我们将使用以下方法:

```
public List<String> analyze(String text, Analyzer analyzer) throws IOException{
    List<String> result = new ArrayList<String>();
    TokenStream tokenStream = analyzer.tokenStream(FIELD_NAME, text);
    CharTermAttribute attr = tokenStream.addAttribute(CharTermAttribute.class);
    tokenStream.reset();
    while(tokenStream.incrementToken()) {
       result.add(attr.toString());
    }       
    return result;
}
```

这个方法使用给定的分析器将给定的文本转换成一个标记列表。

## 4。常见的 Lucene 分析器

现在，我们来看看一些常用的 Lucene 分析器。

### 4.1。`StandardAnalyzer`

我们从最常用的分析器`StandardAnalyzer` 开始:

```
private static final String SAMPLE_TEXT
  = "This is baeldung.com Lucene Analyzers test";

@Test
public void whenUseStandardAnalyzer_thenAnalyzed() throws IOException {
    List<String> result = analyze(SAMPLE_TEXT, new StandardAnalyzer());

    assertThat(result, 
      contains("baeldung.com", "lucene", "analyzers","test"));
}
```

请注意，`StandardAnalyzer`可以识别 URL 和电子邮件。

此外，它删除了停用词，并降低了生成的令牌的大小写。

### 4.2。`StopAnalyzer`

`StopAnalyzer`由`LetterTokenizer, LowerCaseFilter`和 `StopFilter:`组成

```
@Test
public void whenUseStopAnalyzer_thenAnalyzed() throws IOException {
    List<String> result = analyze(SAMPLE_TEXT, new StopAnalyzer());

    assertThat(result, 
      contains("baeldung", "com", "lucene", "analyzers", "test"));
}
```

在这个例子中，`LetterTokenizer `通过非字母字符分割文本，而`StopFilter`从标记列表中删除停用词。

然而，与`StandardAnalyzer`不同的是，`StopAnalyzer`不能识别 URL。

### 4.3。`SimpleAnalyzer`

`SimpleAnalyzer`由`LetterTokenizer`和一个`LowerCaseFilter`组成:

```
@Test
public void whenUseSimpleAnalyzer_thenAnalyzed() throws IOException {
    List<String> result = analyze(SAMPLE_TEXT, new SimpleAnalyzer());

    assertThat(result, 
      contains("this", "is", "baeldung", "com", "lucene", "analyzers", "test"));
}
```

在这里，`SimpleAnalyzer`没有删除停止词。它也不识别网址。

### 4.4。`WhitespaceAnalyzer`

`WhitespaceAnalyzer`只使用了一个`WhitespaceTokenizer`,它通过空白字符分割文本:

```
@Test
public void whenUseWhiteSpaceAnalyzer_thenAnalyzed() throws IOException {
    List<String> result = analyze(SAMPLE_TEXT, new WhitespaceAnalyzer());

    assertThat(result, 
      contains("This", "is", "baeldung.com", "Lucene", "Analyzers", "test"));
}
```

### 4.5。`KeywordAnalyzer`

`KeywordAnalyzer`将输入标记为一个单独的标记:

```
@Test
public void whenUseKeywordAnalyzer_thenAnalyzed() throws IOException {
    List<String> result = analyze(SAMPLE_TEXT, new KeywordAnalyzer());

    assertThat(result, contains("This is baeldung.com Lucene Analyzers test"));
}
```

`KeywordAnalyzer `对于 id 和邮政编码这样的字段很有用。

### 4.6。语言分析器

还有针对不同语言的特殊分析器，如`EnglishAnalyzer`、`FrenchAnalyzer`和`SpanishAnalyzer`:

```
@Test
public void whenUseEnglishAnalyzer_thenAnalyzed() throws IOException {
    List<String> result = analyze(SAMPLE_TEXT, new EnglishAnalyzer());

    assertThat(result, contains("baeldung.com", "lucen", "analyz", "test"));
}
```

这里，我们使用的是由`StandardTokenizer`、`StandardFilter`、`EnglishPossessiveFilter`、`LowerCaseFilter`、`StopFilter`和`PorterStemFilter`组成的`EnglishAnalyzer`。

## 5。定制分析仪

接下来，让我们看看如何构建我们的定制分析器。我们将以两种不同的方式构建同一个定制分析器。

在第一个例子中，**我们将使用`CustomAnalyzer`构建器从预定义的记号赋予器和过滤器**构建我们的分析器:

```
@Test
public void whenUseCustomAnalyzerBuilder_thenAnalyzed() throws IOException {
    Analyzer analyzer = CustomAnalyzer.builder()
      .withTokenizer("standard")
      .addTokenFilter("lowercase")
      .addTokenFilter("stop")
      .addTokenFilter("porterstem")
      .addTokenFilter("capitalization")
      .build();
    List<String> result = analyze(SAMPLE_TEXT, analyzer);

    assertThat(result, contains("Baeldung.com", "Lucen", "Analyz", "Test"));
}
```

我们的分析器与`EnglishAnalyzer`非常相似，但是它将标记大写。

在第二个例子中，**我们将通过扩展`Analyzer`抽象类并覆盖`createComponents()`方法**来构建相同的分析器:

```
public class MyCustomAnalyzer extends Analyzer {

    @Override
    protected TokenStreamComponents createComponents(String fieldName) {
        StandardTokenizer src = new StandardTokenizer();
        TokenStream result = new StandardFilter(src);
        result = new LowerCaseFilter(result);
        result = new StopFilter(result,  StandardAnalyzer.STOP_WORDS_SET);
        result = new PorterStemFilter(result);
        result = new CapitalizationFilter(result);
        return new TokenStreamComponents(src, result);
    }
}
```

如果需要，我们还可以创建我们的自定义标记器或过滤器，并将其添加到我们的自定义分析器中。

现在，让我们看看我们的自定义分析器的运行情况——在本例中，我们将使用 [`InMemoryLuceneIndex`](/web/20220928005740/https://www.baeldung.com/lucene) :

```
@Test
public void givenTermQuery_whenUseCustomAnalyzer_thenCorrect() {
    InMemoryLuceneIndex luceneIndex = new InMemoryLuceneIndex(
      new RAMDirectory(), new MyCustomAnalyzer());
    luceneIndex.indexDocument("introduction", "introduction to lucene");
    luceneIndex.indexDocument("analyzers", "guide to lucene analyzers");
    Query query = new TermQuery(new Term("body", "Introduct"));

    List<Document> documents = luceneIndex.searchIndex(query);
    assertEquals(1, documents.size());
}
```

## 6。`PerFieldAnalyzerWrapper`

最后，**我们可以使用`PerFieldAnalyzerWrapper`** 将不同的分析仪分配到不同的字段。

首先，我们需要定义我们的`analyzerMap`来将每个分析器映射到一个特定的字段:

```
Map<String,Analyzer> analyzerMap = new HashMap<>();
analyzerMap.put("title", new MyCustomAnalyzer());
analyzerMap.put("body", new EnglishAnalyzer());
```

我们将“标题”映射到自定义分析器，将“正文”映射到英语分析器。

接下来，让我们通过提供`analyzerMap`和一个默认的`Analyzer`来创建我们的`PerFieldAnalyzerWrapper`:

```
PerFieldAnalyzerWrapper wrapper = new PerFieldAnalyzerWrapper(
  new StandardAnalyzer(), analyzerMap);
```

现在，让我们来测试一下:

```
@Test
public void givenTermQuery_whenUsePerFieldAnalyzerWrapper_thenCorrect() {
    InMemoryLuceneIndex luceneIndex = new InMemoryLuceneIndex(new RAMDirectory(), wrapper);
    luceneIndex.indexDocument("introduction", "introduction to lucene");
    luceneIndex.indexDocument("analyzers", "guide to lucene analyzers");

    Query query = new TermQuery(new Term("body", "introduct"));
    List<Document> documents = luceneIndex.searchIndex(query);
    assertEquals(1, documents.size());

    query = new TermQuery(new Term("title", "Introduct"));
    documents = luceneIndex.searchIndex(query);
    assertEquals(1, documents.size());
}
```

## 7。结论

我们讨论了流行的 Lucene 分析器，如何构建一个定制的分析器，以及如何为每个字段使用不同的分析器。

完整的源代码可以在 GitHub 上找到[。](https://web.archive.org/web/20220928005740/https://github.com/eugenp/tutorials/tree/master/lucene)