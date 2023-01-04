# Java XPath 简介

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-xpath>

## 1。概述

在本文中，我们将回顾标准 Java JDK 对 XPath 的支持。

我们将使用一个简单的 XML 文档，对它进行处理，看看如何检查文档，从中提取我们需要的信息。

XPath 是 W3C 推荐的标准语法，它是一组导航 XML 文档的表达式。您可以在这里找到完整的 XPath 引用。

## 2。一个简单的 XPath 解析器

```java
import javax.xml.namespace.NamespaceContext;
import javax.xml.parsers.DocumentBuilder;
import javax.xml.parsers.DocumentBuilderFactory;
import javax.xml.parsers.ParserConfigurationException;
import javax.xml.xpath.XPath;
import javax.xml.xpath.XPathConstants;
import javax.xml.xpath.XPathExpressionException;
import javax.xml.xpath.XPathFactory;

import org.w3c.dom.Document;

public class DefaultParser {

    private File file;

    public DefaultParser(File file) {
        this.file = file;
    }
} 
```

现在让我们仔细看看您将在`DefaultParser`中找到的元素:

```java
FileInputStream fileIS = new FileInputStream(this.getFile());
DocumentBuilderFactory builderFactory = DocumentBuilderFactory.newInstance();
DocumentBuilder builder = builderFactory.newDocumentBuilder();
Document xmlDocument = builder.parse(fileIS);
XPath xPath = XPathFactory.newInstance().newXPath();
String expression = "/Tutorials/Tutorial";
nodeList = (NodeList) xPath.compile(expression).evaluate(xmlDocument, XPathConstants.NODESET);
```

让我们来分解一下:

```java
DocumentBuilderFactory builderFactory = DocumentBuilderFactory.newInstance();
```

我们将使用这个对象从 xml 文档中生成一个 DOM 对象树:

```java
DocumentBuilder builder = builderFactory.newDocumentBuilder();
```

有了这个类的实例，我们可以解析来自许多不同输入源的 XML 文档，比如`InputStream`、`File`、`URL`和`SAX`:

```java
Document xmlDocument = builder.parse(fileIS);
```

一个`Document` ( `org.w3c.dom.Document`)代表了整个 XML 文档，是文档树的根，提供了我们对数据的第一次访问:

```java
XPath xPath = XPathFactory.newInstance().newXPath();
```

从 XPath 对象中，我们将访问表达式并在我们的文档中执行它们，以从中提取我们需要的内容:

```java
xPath.compile(expression).evaluate(xmlDocument, XPathConstants.NODESET);
```

我们可以编译一个作为字符串传递的 XPath 表达式，并定义我们期望接收哪种数据，例如`NODESET`、`NODE`或`String`。

## 3。让我们开始

现在，我们已经了解了我们将使用的基本组件，让我们从使用一些简单的 XML 进行测试的代码开始:

```java
<?xml version="1.0"?>
<Tutorials>
    <Tutorial tutId="01" type="java">
        <title>Guava</title>
  <description>Introduction to Guava</description>
  <date>04/04/2016</date>
  <author>GuavaAuthor</author>
    </Tutorial>
    <Tutorial tutId="02" type="java">
        <title>XML</title>
  <description>Introduction to XPath</description>
  <date>04/05/2016</date>
  <author>XMLAuthor</author>
    </Tutorial>
</Tutorials>
```

### 3.1。检索元素的基本列表

第一种方法是简单地使用 XPath 表达式从 XML 中检索节点列表:

```java
FileInputStream fileIS = new FileInputStream(this.getFile());
DocumentBuilderFactory builderFactory = DocumentBuilderFactory.newInstance();
DocumentBuilder builder = builderFactory.newDocumentBuilder();
Document xmlDocument = builder.parse(fileIS);
XPath xPath = XPathFactory.newInstance().newXPath();
String expression = "/Tutorials/Tutorial";
nodeList = (NodeList) xPath.compile(expression).evaluate(xmlDocument, XPathConstants.NODESET); 
```

我们可以使用上面的表达式或者使用表达式“`//Tutorial`”来检索根节点中包含的教程列表，但是这个将从当前节点检索文档中的所有`<Tutorial>`节点，不管它们位于文档中的什么位置，这意味着从当前节点开始在树的任何级别。

它通过将`NODESET`指定给 compile 指令作为返回类型而返回的`NodeList`,是一个有序的节点集合，可以通过将一个索引作为参数来访问它。

### 3.2.按 ID 检索特定节点

我们可以根据任何给定的 id 查找元素，只需通过过滤:

```java
DocumentBuilderFactory builderFactory = DocumentBuilderFactory.newInstance();
DocumentBuilder builder = builderFactory.newDocumentBuilder();
Document xmlDocument = builder.parse(this.getFile());
XPath xPath = XPathFactory.newInstance().newXPath();
String expression = "/Tutorials/Tutorial[@tutId=" + "'" + id + "'" + "]";
node = (Node) xPath.compile(expression).evaluate(xmlDocument, XPathConstants.NODE); 
```

通过使用这种表达式，我们可以通过使用正确的语法来过滤我们需要寻找的任何元素。这种表达式称为谓词，是在文档中定位特定数据的一种简单方法，例如:

`/Tutorials/Tutorial[1]`

`/Tutorials/Tutorial[first()]`

`/Tutorials/Tutorial[position()<4]`

您可以在这里找到谓词[的完整引用](https://web.archive.org/web/20220529021418/https://www.w3.org/TR/xpath/#predicates)

### 3.3。通过特定的标签名检索节点

现在我们将进一步引入轴，让我们通过在 XPath 表达式中使用轴来看看它是如何工作的:

```java
Document xmlDocument = builder.parse(this.getFile());
this.clean(xmlDocument);
XPath xPath = XPathFactory.newInstance().newXPath();
String expression = "//Tutorial[descendant::title[text()=" + "'" + name + "'" + "]]";
nodeList = (NodeList) xPath.compile(expression).evaluate(xmlDocument, XPathConstants.NODESET); 
```

使用上面使用的表达式，我们寻找每个`<Tutorial>`元素，其后代`<title>`的文本作为参数传递到“name”变量中。

按照本文提供的示例 xml，我们可以查找包含文本“Guava”或“XML”的`<title>`,我们将检索整个`<Tutorial>`元素及其所有数据。

Axes 提供了一种非常灵活的方式来导航 XML 文档，你可以在官方网站上找到完整的文档。

### 3.4。在表达式中操作数据

如果需要，XPath 还允许我们在表达式中操作数据。

```java
XPath xPath = XPathFactory.newInstance().newXPath();
String expression = "//Tutorial[number(translate(date, '/', '')) > " + date + "]";
nodeList = (NodeList) xPath.compile(expression).evaluate(xmlDocument, XPathConstants.NODESET); 
```

在这个表达式中，我们将一个简单的字符串作为日期传递给我们的方法，看起来像“ddmmyyyy”，但是 XML 以格式“`dd/mm/yyyy`”存储这个数据，所以为了匹配一个结果，我们操作这个字符串，将它转换成我们的文档所使用的正确的数据格式，我们通过使用由 [XPath](https://web.archive.org/web/20220529021418/https://www.w3.org/TR/xpath/#corelib) 提供的一个函数来完成这个操作

### 3.5。从定义了名称空间的文档中检索元素

如果我们的 xml 文档定义了一个名称空间，如这里使用的 example_namespace.xml，那么检索我们需要的数据的规则将会改变，因为我们的 xml 是这样开始的:

```java
<?xml version="1.0"?>
<Tutorials >

</Tutorials>
```

现在，当我们使用类似于"`//Tutoria` l "的表达式时，我们不会得到任何结果。XPath 表达式将返回不在任何名称空间下的所有`<Tutorial>`元素，在我们新的 example_namespace.xml 中，所有的`<Tutorial>`元素都在名称空间`/full_archive`中定义。

让我们看看如何处理名称空间。

首先，我们需要设置名称空间上下文，以便 XPath 能够知道我们在哪里寻找数据:

```java
xPath.setNamespaceContext(new NamespaceContext() {
    @Override
    public Iterator getPrefixes(String arg0) {
        return null;
    }
    @Override
    public String getPrefix(String arg0) {
        return null;
    }
    @Override
    public String getNamespaceURI(String arg0) {
        if ("bdn".equals(arg0)) {
            return "/full_archive";
        }
        return null;
    }
}); 
```

在上面的方法中，我们将"`bdn`"定义为名称空间"`/full_archive`"的名称，从现在开始，我们需要将"`bdn`"添加到用于定位元素的 XPath 表达式中:

```java
String expression = "/bdn:Tutorials/bdn:Tutorial";
nodeList = (NodeList) xPath.compile(expression).evaluate(xmlDocument, XPathConstants.NODESET); 
```

使用上面的表达式，我们能够检索“`bdn`”名称空间下的所有`<Tutorial>`元素。

### 3.6。避免空文本节点的麻烦

正如您所注意到的，在本文 3.3 节的代码中，在将 XML 解析为文档对象之后，就调用了一个新函数，`this.clean(xmlDocument);`

有时，当我们遍历元素、子节点等时，如果我们的文档有空的文本节点，我们会在想要得到的结果中发现一个意外的行为。

当我们遍历所有的`<Tutorial>`元素寻找`<title>` 信息时，我们调用了`node.getFirstChild()`,但是我们没有得到我们想要的信息，我们只有一个空节点“#Text”。

要解决这个问题，我们可以浏览我们的文档并删除那些空节点，如下所示:

```java
NodeList childs = node.getChildNodes();
for (int n = childs.getLength() - 1; n >= 0; n--) {
    Node child = childs.item(n);
    short nodeType = child.getNodeType();
    if (nodeType == Node.ELEMENT_NODE) {
        clean(child);
    }
    else if (nodeType == Node.TEXT_NODE) {
        String trimmedNodeVal = child.getNodeValue().trim();
        if (trimmedNodeVal.length() == 0){
            node.removeChild(child);
        }
        else {
            child.setNodeValue(trimmedNodeVal);
        }
    } else if (nodeType == Node.COMMENT_NODE) {
        node.removeChild(child);
    }
}
```

通过这样做，我们可以检查我们找到的每种类型的节点，并删除那些我们不需要的节点。

## 4。结论

这里我们只介绍了默认 XPath 提供的支持，但是现在有很多流行的库，如 JDOM、Saxon、XQuery、JAXP、Jaxen 甚至 Jackson。也有专门用于 HTML 解析的库，比如 JSoup。

不限于 java，XSLT 语言可以使用 XPath 表达式来导航 XML 文档。

如您所见，如何处理这类文件有很多种可能性。

默认情况下，对 XML/HTML 文档的解析、读取和处理有很好的标准支持。你可以在这里找到完整的工作样本[。](https://web.archive.org/web/20220529021418/https://github.com/eugenp/tutorials/tree/master/xml)