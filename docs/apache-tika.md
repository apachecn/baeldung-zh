# 使用 Apache Tika 进行内容分析

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/apache-tika>

## 1。概述

**[Apache Tika](https://web.archive.org/web/20220524034333/https://tika.apache.org/index.html) 是一个工具包，用于从各种类型的文档**中提取内容和元数据，例如 Word、Excel 和 PDF，甚至是 JPEG 和 MP4 这样的多媒体文件。

所有基于文本的文件和多媒体文件都可以使用一个通用的界面进行解析，这使得 Tika 成为一个强大而通用的内容分析库。

在本文中，我们将介绍 Apache Tika，包括它的解析 API 以及它如何自动检测文档的内容类型。还将提供工作示例来说明该库的操作。

## 2。入门

为了使用 Apache Tika 解析文档，我们只需要一个 Maven 依赖项:

```java
<dependency>
    <groupId>org.apache.tika</groupId>
    <artifactId>tika-parsers</artifactId>
    <version>1.17</version>
</dependency>
```

这个产品的最新版本可以在[这里](https://web.archive.org/web/20220524034333/https://search.maven.org/classic/#search%7Cgav%7C1%7Cg%3A%22org.apache.tika%22%20AND%20a%3A%22tika-parsers%22)找到。

## 3。`Parser` API

**`Parser`API 是 Apache Tika 的核心，抽象出解析操作的复杂性**。这个 API 依赖于一个方法:

```java
void parse(
  InputStream stream, 
  ContentHandler handler, 
  Metadata metadata, 
  ParseContext context) 
  throws IOException, SAXException, TikaException
```

该方法参数的含义如下:

*   `stream`**–**从要解析的文档中创建的`InputStream`实例
*   `handler`**–**一个`ContentHandler`对象，接收从输入文档中解析出的一系列 XHTML SAX 事件；然后，该处理程序将处理事件，并以特定的形式导出结果
*   `metadata`**–**一个`Metadata`对象，用于将元数据属性传入和传出解析器
*   `context`**–**一个携带上下文相关信息的`ParseContext`实例，用于定制解析过程

如果从输入流中读取失败，`parse`方法抛出一个`IOException`，如果从流中获取的文档不能被解析，抛出一个`TikaException`，如果处理程序不能处理一个事件，抛出一个`SAXException`。

在解析文档时，Tika 试图尽可能重用现有的解析器库，如 Apache POI 或 PDFBox。因此，大多数`Parser`实现类只是这种外部库的适配器。

在第 5 节中，我们将看到如何使用`handler`和`metadata`参数来提取文档的内容和元数据。

为了方便起见，我们可以使用 facade 类`Tika`来访问`Parser` API 的功能。

## 4。自动检测

Apache Tika 可以根据文档本身而不是附加信息自动检测文档的类型和语言。

### 4.1。文件类型检测

**文档类型的检测可以使用`Detector`接口**的实现类来完成，它有一个方法:

```java
MediaType detect(java.io.InputStream input, Metadata metadata) 
  throws IOException
```

该方法获取一个文档及其相关元数据，然后返回一个描述文档类型最佳猜测的`MediaType`对象。

元数据不是探测器依赖的唯一信息来源。检测器还可以利用幻字节，幻字节是文件开头附近的一种特殊模式，或者将检测过程委托给更合适的检测器。

事实上，检测器使用的算法是依赖于实现的。

例如，默认检测器首先处理幻字节，然后处理元数据属性。如果此时还没有找到内容类型，它将使用服务加载器来发现所有可用的检测器，并依次尝试它们。

### 4.2。语言检测

除了文档的类型，Tika 还可以识别其语言，即使没有元数据信息的帮助。

在 Tika 的早期版本中，使用一个`LanguageIdentifier`实例来检测文档的语言。

然而，`LanguageIdentifier`已经被弃用，取而代之的是 web 服务，这在[入门](https://web.archive.org/web/20220524034333/https://tika.apache.org/1.17/detection.html#Language_Detection)文档中没有明确说明。

语言检测服务现在通过抽象类`LanguageDetector`的子类型提供。使用 web 服务，您还可以访问成熟的在线翻译服务，如 Google Translate 或 Microsoft Translator。

为了简洁起见，我们不会详细讨论这些服务。

## 5。Tika 在行动

本节使用工作示例说明了 Apache Tika 的特性。

插图方法将被包装在一个类中:

```java
public class TikaAnalysis {
    // illustration methods
}
```

### 5.1。检测文件类型

下面是我们可以用来检测从`InputStream`中读取的文档类型的代码:

```java
public static String detectDocTypeUsingDetector(InputStream stream) 
  throws IOException {
    Detector detector = new DefaultDetector();
    Metadata metadata = new Metadata();

    MediaType mediaType = detector.detect(stream, metadata);
    return mediaType.toString();
}
```

假设我们在类路径中有一个名为`tika.txt`的 PDF 文件。这个文件的扩展名已经被改变，试图欺骗我们的分析工具。文档的真实类型仍然可以通过测试来发现和确认:

```java
@Test
public void whenUsingDetector_thenDocumentTypeIsReturned() 
  throws IOException {
    InputStream stream = this.getClass().getClassLoader()
      .getResourceAsStream("tika.txt");
    String mediaType = TikaAnalysis.detectDocTypeUsingDetector(stream);

    assertEquals("application/pdf", mediaType);

    stream.close();
}
```

很明显，错误的文件扩展名不能阻止 Tika 找到正确的媒体类型，这要感谢文件开头的神奇字节`%PDF`。

为了方便起见，我们可以使用`Tika` facade 类重写检测代码，得到相同的结果:

```java
public static String detectDocTypeUsingFacade(InputStream stream) 
  throws IOException {

    Tika tika = new Tika();
    String mediaType = tika.detect(stream);
    return mediaType;
}
```

### 5.2。提取内容

现在让我们提取一个文件的内容，并使用`Parser` API 将结果作为`String`返回:

```java
public static String extractContentUsingParser(InputStream stream) 
  throws IOException, TikaException, SAXException {

    Parser parser = new AutoDetectParser();
    ContentHandler handler = new BodyContentHandler();
    Metadata metadata = new Metadata();
    ParseContext context = new ParseContext();

    parser.parse(stream, handler, metadata, context);
    return handler.toString();
}
```

给定类路径中的 Microsoft Word 文件，其内容如下:

```java
Apache Tika - a content analysis toolkit
The Apache Tika™ toolkit detects and extracts metadata and text ...
```

可以提取并验证内容:

```java
@Test
public void whenUsingParser_thenContentIsReturned() 
  throws IOException, TikaException, SAXException {
    InputStream stream = this.getClass().getClassLoader()
      .getResourceAsStream("tika.docx");
    String content = TikaAnalysis.extractContentUsingParser(stream);

    assertThat(content, 
      containsString("Apache Tika - a content analysis toolkit"));
    assertThat(content, 
      containsString("detects and extracts metadata and text"));

    stream.close();
}
```

同样，`Tika`类可以用来更方便地编写代码:

```java
public static String extractContentUsingFacade(InputStream stream) 
  throws IOException, TikaException {

    Tika tika = new Tika();
    String content = tika.parseToString(stream);
    return content;
}
```

### 5.3。提取元数据

除了文档的内容之外，`Parser` API 还可以提取元数据:

```java
public static Metadata extractMetadatatUsingParser(InputStream stream) 
  throws IOException, SAXException, TikaException {

    Parser parser = new AutoDetectParser();
    ContentHandler handler = new BodyContentHandler();
    Metadata metadata = new Metadata();
    ParseContext context = new ParseContext();

    parser.parse(stream, handler, metadata, context);
    return metadata;
}
```

当类路径中存在 Microsoft Excel 文件时，这个测试用例确认提取的元数据是正确的:

```java
@Test
public void whenUsingParser_thenMetadataIsReturned() 
  throws IOException, TikaException, SAXException {
    InputStream stream = this.getClass().getClassLoader()
      .getResourceAsStream("tika.xlsx");
    Metadata metadata = TikaAnalysis.extractMetadatatUsingParser(stream);

    assertEquals("org.apache.tika.parser.DefaultParser", 
      metadata.get("X-Parsed-By"));
    assertEquals("Microsoft Office User", metadata.get("Author"));

    stream.close();
}
```

最后，这里是使用`Tika` facade 类的提取方法的另一个版本:

```java
public static Metadata extractMetadatatUsingFacade(InputStream stream) 
  throws IOException, TikaException {
    Tika tika = new Tika();
    Metadata metadata = new Metadata();

    tika.parse(stream, metadata);
    return metadata;
}
```

## 6。结论

本教程主要讨论使用 Apache Tika 进行内容分析。**使用`Parser`和`Detector`API，我们可以自动检测文档的类型，并提取其内容和元数据**。

对于高级用例，我们可以创建定制的`Parser`和`Detector`类来对解析过程进行更多的控制。

本教程的完整源代码可以在 GitHub 上找到[。](https://web.archive.org/web/20220524034333/https://github.com/eugenp/tutorials/tree/master/apache-tika)