# 用 Lucene 进行简单的文件搜索

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/lucene-file-search>

## 1。概述

Apache Lucene 是一个全文搜索引擎，可以被各种编程语言使用。要开始使用 Lucene，请参考我们的介绍性文章。

在这篇简短的文章中，我们将索引一个文本文件，并在该文件中搜索示例`Strings`和文本片段。

## 2。Maven 设置

让我们首先添加必要的依赖项:

```java
<dependency>        
    <groupId>org.apache.lucene</groupId>          
    <artifactId>lucene-core</artifactId>
    <version>7.1.0</version>
</dependency>
```

最新版本可以在这里找到[。](https://web.archive.org/web/20220928005747/https://search.maven.org/classic/#search%7Cgav%7C1%7Cg%3A%22org.apache.lucene%22%20AND%20a%3A%22lucene-core%22)

此外，为了解析我们的搜索查询，我们需要:

```java
<dependency>
    <groupId>org.apache.lucene</groupId>
    <artifactId>lucene-queryparser</artifactId>
    <version>7.1.0</version>
</dependency>
```

记得在这里查看最新版本[。](https://web.archive.org/web/20220928005747/https://search.maven.org/classic/#search%7Cgav%7C1%7Cg%3A%22org.apache.lucene%22%20AND%20a%3A%22lucene-queryparser%22)

## 3。文件系统目录

为了索引文件，我们首先需要创建一个文件系统索引。

Lucene 提供了`FSDirectory` 类来创建文件系统索引:

```java
Directory directory = FSDirectory.open(Paths.get(indexPath));
```

这里的`indexPath` 是目录的位置。**如果目录不存在，Lucene 会创建它。**

Lucene 提供了抽象的`FSDirectory`类的三个具体实现:`SimpleFSDirectory, NIOFSDirectory, and MMapDirectory.` 对于给定的环境，它们中的每一个都可能有特殊的问题。

例如， **`SimpleFSDirectory` 的并发性能很差，因为当多个线程读取同一个文件时，它会阻塞。**

类似地， **`NIOFSDirectory and MMapDirectory` 实现分别面临 Windows 中的文件通道问题和内存释放问题。**

为了克服这样的环境特性，Lucene 提供了`FSDirectory.open()` 方法。当被调用时，它试图根据环境选择最佳的实现。

## 4。索引文本文件

一旦我们创建了索引目录，让我们继续向索引添加一个文件:

```java
public void addFileToIndex(String filepath) {

    Path path = Paths.get(filepath);
    File file = path.toFile();
    IndexWriterConfig indexWriterConfig
     = new IndexWriterConfig(analyzer);
    Directory indexDirectory = FSDirectory
      .open(Paths.get(indexPath));
    IndexWriter indexWriter = new IndexWriter(
      indexDirectory, indexWriterConfig);
    Document document = new Document();

    FileReader fileReader = new FileReader(file);
    document.add(
      new TextField("contents", fileReader));
    document.add(
      new StringField("path", file.getPath(), Field.Store.YES));
    document.add(
      new StringField("filename", file.getName(), Field.Store.YES));

    indexWriter.addDocument(document);
    indexWriter.close();
}
```

这里，我们创建了一个文档，它有两个名为“path”和“filename”的`StringFields`和一个名为“contents”的`TextField` 。

注意，我们将`fileReader` 实例作为第二个参数传递给`TextField`。使用`IndexWriter.`将文档添加到索引中

`TextField`或`StringField`构造函数中的第三个参数表示字段的值是否也将被存储。

**最后，我们调用`IndexWriter` 的`close()` 来优雅地关闭并释放索引文件的锁。**

## 5。搜索索引文件

现在，让我们搜索我们已编制索引的文件:

```java
public List<Document> searchFiles(String inField, String queryString) {
    Query query = new QueryParser(inField, analyzer)
      .parse(queryString);
    Directory indexDirectory = FSDirectory
      .open(Paths.get(indexPath));
    IndexReader indexReader = DirectoryReader
      .open(indexDirectory);
    IndexSearcher searcher = new IndexSearcher(indexReader);
    TopDocs topDocs = searcher.search(query, 10);

    return topDocs.scoreDocs.stream()
      .map(scoreDoc -> searcher.doc(scoreDoc.doc))
      .collect(Collectors.toList());
}
```

现在让我们测试功能:

```java
@Test
public void givenSearchQueryWhenFetchedFileNamehenCorrect(){
    String indexPath = "/tmp/index";
    String dataPath = "/tmp/data/file1.txt";

    Directory directory = FSDirectory
      .open(Paths.get(indexPath));
    LuceneFileSearch luceneFileSearch 
      = new LuceneFileSearch(directory, new StandardAnalyzer());

    luceneFileSearch.addFileToIndex(dataPath);

    List<Document> docs = luceneFileSearch
      .searchFiles("contents", "consectetur");

    assertEquals("file1.txt", docs.get(0).get("filename"));
}
```

注意我们是如何在位置`indexPath`创建文件系统索引并索引`file1.txt.`的

然后，我们只需在`“contents”`字段中搜索`String``consectetur`。

## 6.结论

本文快速演示了如何使用 Apache Lucene 索引和搜索文本。要了解更多关于 Lucene 的索引、搜索和查询，请参考我们的[Lucene 简介文章](/web/20220928005747/https://www.baeldung.com/lucene)。

和往常一样，例子的代码可以在 Github 上找到[。](https://web.archive.org/web/20220928005747/https://github.com/eugenp/tutorials/tree/master/lucene)