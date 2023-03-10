# 使用 SAX 解析器解析 XML 文件

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-sax-parser>

## 1。概述

SAX，也称为 XML 的简单 API，用于解析 XML 文档。

在本教程中，我们将学习什么是 SAX，以及为什么、何时和如何使用它。

## 2.XML 的简单应用编程接口

SAX 是一个用于解析 XML 文档的 API。它**基于通读文档时生成的事件**。回调方法接收这些事件。自定义处理程序包含这些回调方法。

API 是高效的，因为它在回调收到事件后立即丢弃事件。因此， **SAX 拥有高效的内存管理**，比如不像 DOM。

## 3.SAX 与 DOM

DOM 代表文档对象模型。 **DOM 解析器不依赖于事件**。此外，它将整个 XML 文档加载到内存中进行解析。SAX 比 DOM 更节省内存。

DOM 也有它的好处。例如，DOM 支持 XPath。由于**文档被加载到内存**中，这使得一次操作整个文档树变得容易。

## 4.SAX 与 StAX

StAX 比 SAX 和 DOM 更新。它代表 XML 的流 API。

与 SAX 的主要区别在于 **StAX 使用拉机制**而不是 SAX 的推机制(使用回调)。
这意味着控制权交给了客户端来决定何时需要拉取事件。因此，如果只需要文档的一部分，就没有义务拉整个文档。

它提供了一个简单的 API，以内存高效的方式解析 XML。

与 SAX 不同，它不提供模式验证作为其特性之一。

## 5.使用自定义处理程序解析 XML 文件

现在让我们使用下面的 XML 来表示 Baeldung 网站及其文章:

```java
<baeldung>
    <articles>
        <article>
            <title>Parsing an XML File Using SAX Parser</title>
            <content>SAX Parser's Lorem ipsum...</content>
        </article>
        <article>
            <title>Parsing an XML File Using DOM Parser</title>
            <content>DOM Parser's Lorem ipsum...</content>
        </article>
        <article>
            <title>Parsing an XML File Using StAX Parser</title>
            <content>StAX's Lorem ipsum...</content>
        </article>
    </articles>
</baeldung>
```

我们将从为我们的`Baeldung`根元素及其子元素创建 POJOs 开始:

```java
public class Baeldung {
    private List<BaeldungArticle> articleList;
    // usual getters and setters
} 
```

```java
public class BaeldungArticle {
    private String title;
    private String content;
    // usual getters and setters
} 
```

我们将继续创建`BaeldungHandler`。这个类将实现捕获事件所必需的回调方法。

我们将覆盖超类`DefaultHandler, `中的四个方法，每个方法描述一个事件:

定义了所有回调之后，我们现在可以编写`BaeldungHandler`类了:

```java
public class BaeldungHandler extends DefaultHandler {
    private static final String ARTICLES = "articles";
    private static final String ARTICLE = "article";
    private static final String TITLE = "title";
    private static final String CONTENT = "content";

    private Baeldung website;
    private StringBuilder elementValue;

    @Override
    public void characters(char[] ch, int start, int length) throws SAXException {
        if (elementValue == null) {
            elementValue = new StringBuilder();
        } else {
            elementValue.append(ch, start, length);
        }
    }

    @Override
    public void startDocument() throws SAXException {
        website = new Baeldung();
    }

    @Override
    public void startElement(String uri, String lName, String qName, Attributes attr) throws SAXException {
        switch (qName) {
            case ARTICLES:
                website.articleList = new ArrayList<>();
                break;
            case ARTICLE:
                website.articleList.add(new BaeldungArticle());
                break;
            case TITLE:
                elementValue = new StringBuilder();
                break;
            case CONTENT:
                elementValue = new StringBuilder();
                break;
        }
    }

    @Override
    public void endElement(String uri, String localName, String qName) throws SAXException {
        switch (qName) {
            case TITLE:
                latestArticle().setTitle(elementValue.toString());
                break;
            case CONTENT:
                latestArticle().setContent(elementValue.toString());
                break;
        }
    }

    private BaeldungArticle latestArticle() {
        List<BaeldungArticle> articleList = website.articleList;
        int latestArticleIndex = articleList.size() - 1;
        return articleList.get(latestArticleIndex);
    }

    public Baeldung getWebsite() {
        return website;
    }
} 
```

还添加了字符串常量以增加可读性。检索最近遇到的文章的方法也很方便。最后，我们需要为`Baeldung`对象获取一个 getter。

注意，上面的代码不是线程安全的，因为我们在方法调用之间保持状态。

## 6.测试解析器

为了测试解析器，我们将实例化`SaxFactory`、`SaxParser`以及`BaeldungHandler`:

```java
SAXParserFactory factory = SAXParserFactory.newInstance();
SAXParser saxParser = factory.newSAXParser();
SaxParserMain.BaeldungHandler baeldungHandler = new SaxParserMain.BaeldungHandler(); 
```

之后，我们将解析 XML 文件，并断言该对象包含所有解析的预期元素:

```java
saxParser.parse("src/test/resources/sax/baeldung.xml", baeldungHandler);

SaxParserMain.Baeldung result = baeldungHandler.getWebsite();

assertNotNull(result);
List<SaxParserMain.BaeldungArticle> articles = result.getArticleList();

assertNotNull(articles);
assertEquals(3, articles.size());

SaxParserMain.BaeldungArticle articleOne = articles.get(0);
assertEquals("Parsing an XML File Using SAX Parser", articleOne.getTitle());
assertEquals("SAX Parser's Lorem ipsum...", articleOne.getContent());

SaxParserMain.BaeldungArticle articleTwo = articles.get(1);
assertEquals("Parsing an XML File Using DOM Parser", articleTwo.getTitle());
assertEquals("DOM Parser's Lorem ipsum...", articleTwo.getContent());

SaxParserMain.BaeldungArticle articleThree = articles.get(2);
assertEquals("Parsing an XML File Using StAX Parser", articleThree.getTitle());
assertEquals("StAX Parser's Lorem ipsum...", articleThree.getContent()); 
```

正如所料，`baeldung`已经被正确解析并包含了等待的子对象。

## 7.结论

我们刚刚发现如何使用 SAX 解析 XML 文件。这是一个强大的 API，它在我们的应用程序中生成了一个轻量级的内存足迹。

和往常一样，这篇文章的代码可以在 GitHub 的[上找到。](https://web.archive.org/web/20220626200405/https://github.com/eugenp/tutorials/tree/master/xml)