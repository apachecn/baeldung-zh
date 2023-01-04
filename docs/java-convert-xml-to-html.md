# 用 Java 将 XML 转换成 HTML

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-convert-xml-to-html>

## 1。简介

在本教程中，我们将描述如何使用常见的 Java 库和模板引擎(JAXP、StAX、Freemarker 和 Mustache)将 XML 转换为 HTML。

## 2。要解组的 XML

让我们从一个简单的 XML 文档开始，在将它转换成 HTML 之前，我们将把它解组成一个合适的 Java 表示。我们将牢记几个关键目标:

1.  为我们所有的示例保持相同的 XML
2.  最后创建一个语法和语义有效的 HTML5 文档
3.  将所有 XML 元素转换成文本

让我们使用一个简单的 Jenkins 通知作为示例 XML:

```
<?xml version="1.0" encoding="UTF-8"?>
<notification>
    <from>[[email protected]](/web/20220826140012/https://www.baeldung.com/cdn-cgi/l/email-protection)</from>
    <heading>Build #7 passed</heading>
    <content>Success: The Jenkins CI build passed</content>
</notification>
```

这很简单。它包括一个根元素和一些嵌套元素。

我们的目标是在创建 HTML 文件时删除所有唯一的 XML 标记并打印出键值对。

## 3。JAXP

Java Architecture for XML Processing([JAXP](https://web.archive.org/web/20220826140012/https://docs.oracle.com/javase/8/docs/technotes/guides/xml/jaxp/index.html))是一个库，旨在通过额外的 DOM 支持来扩展流行的 SAX 解析器的功能。JAXP 提供了使用 SAX 解析器将 XML 定义的对象与 POJOs 进行**编组和解组的能力。我们还将利用内置的 DOM 助手。**

让我们将 JAXP 的 [Maven 依赖项添加到我们的项目中:](https://web.archive.org/web/20220826140012/https://search.maven.org/search?q=a:jaxp-api%20g:javax.xml)

```
<dependency>
    <groupId>javax.xml</groupId>
    <artifactId>jaxp-api</artifactId>
    <version>1.4.2</version>
</dependency> 
```

### 3.1。使用 DOM Builder 解组

让我们首先将 XML 文件解组成一个 Java `Element`对象:

```
DocumentBuilderFactory factory = DocumentBuilderFactory.newInstance();
factory.setFeature(XMLConstants.FEATURE_SECURE_PROCESSING, true);
factory.setFeature("http://apache.org/xml/features/disallow-doctype-decl", true);

Document input = factory
  .newDocumentBuilder()
  .parse(resourcePath);
Element xml = input.getDocumentElement(); 
```

### 3.2。提取地图中的 XML 文件内容

现在，让我们用 XML 文件的相关内容构建一个`Map`:

```
Map<String, String> map = new HashMap<>();
map.put("heading", 
  xml.getElementsByTagName("heading")
    .item(0)
    .getTextContent());
map.put("from", String.format("from: %s",
  xml.getElementsByTagName("from")
    .item(0)
    .getTextContent()));
map.put("content", 
  xml.getElementsByTagName("content")
    .item(0)
    .getTextContent());
```

### 3.3。使用 DOM 构建器进行编组

将 XML 整理成 HTML 文件稍微复杂一些。

让我们准备一个用于写出 HTML 的传输`Document`:

```
Document doc = factory
  .newDocumentBuilder()
  .newDocument(); 
```

接下来，我们将在`map`中用`Elements`填充`Document`:

```
Element html = doc.createElement("html");

Element head = doc.createElement("head");
html.setAttribute("lang", "en");

Element title = doc.createElement("title");
title.setTextContent(map.get("heading"));

head.appendChild(title);
html.appendChild(head);

Element body = doc.createElement("body");

Element from = doc.createElement("p");
from.setTextContent(map.get("from"));

Element success = doc.createElement("p");
success.setTextContent(map.get("content"));

body.appendChild(from);
body.appendChild(success);

html.appendChild(body);
doc.appendChild(html); 
```

最后，让我们的**用一个`TransformerFactory`** 来编组我们的`Document`对象:

```
TransformerFactory transformerFactory = TransformerFactory.newInstance();
transformerFactory.setFeature(XMLConstants.FEATURE_SECURE_PROCESSING, true);
transformerFactory.setAttribute(XMLConstants.ACCESS_EXTERNAL_DTD, "");
transformerFactory.setAttribute(XMLConstants.ACCESS_EXTERNAL_STYLESHEET, "");

try (Writer output = new StringWriter()) {
    Transformer transformer = transformerFactory.newTransformer();
    transformer.transform(new DOMSource(doc), new StreamResult(output));
}
```

如果我们调用`output.toString()`，我们将得到 HTML 表示。

**注意，我们在工厂设置的一些额外特性和属性取自 OWASP 项目的[建议，以避免 XXE 注入](https://web.archive.org/web/20220826140012/https://cheatsheetseries.owasp.org/cheatsheets/XML_External_Entity_Prevention_Cheat_Sheet.html)** 。

## 4。StAX

我们可以使用的另一个库是 XML 流 API([StAX](/web/20220826140012/https://www.baeldung.com/java-stax))。像 JAXP 一样，StAX 已经存在很长时间了——从 2004 年开始。

另外两个库简化了 XML 文件的解析。这对于简单的任务或项目来说很好，但是当我们需要迭代或者对元素解析本身进行明确和细粒度的控制时就不那么好了。这就是 StAX 派上用场的地方。

让我们将 StAX API 的 [Maven 依赖项添加到我们的项目中:](https://web.archive.org/web/20220826140012/https://search.maven.org/search?q=g:javax.xml.stream%20AND%20a:stax-api)

```
<dependency>
    <groupId>javax.xml.stream</groupId>
    <artifactId>stax-api</artifactId>
    <version>1.0-2</version>
</dependency> 
```

### 4.1。使用 StAX 解组

我们将使用一个简单的迭代控制流来将 XML 值存储到我们的`Map` 中:

```
XMLInputFactory factory = XMLInputFactory.newInstance();
factory.setProperty(XMLInputFactory.IS_SUPPORTING_EXTERNAL_ENTITIES, Boolean.FALSE);
factory.setProperty(XMLInputFactory.SUPPORT_DTD, Boolean.FALSE);
XMLStreamReader input = null;
try (FileInputStream file = new FileInputStream(resourcePath)) {
    input = factory.createXMLStreamReader(file);

    Map<String, String> map = new HashMap<>();
    while (input.hasNext()) {
        input.next();
        if (input.isStartElement()) {
            if (input.getLocalName().equals("heading")) {
                map.put("heading", input.getElementText());
            }
            if (input.getLocalName().equals("from")) {
                map.put("from", String.format("from: %s", input.getElementText()));
            }
            if (input.getLocalName().equals("content")) {
                map.put("content", input.getElementText());
            }
        }
    }
} finally {
    if (input != null) {
        input.close();
    }
}
```

### 4.2。使用 StAX 进行编组

现在，让我们用我们的`map`和**写出 HTML** :

```
try (Writer output = new StringWriter()) {
    XMLStreamWriter writer = XMLOutputFactory
      .newInstance()
      .createXMLStreamWriter(output);

    writer.writeDTD("<!DOCTYPE html>");
    writer.writeStartElement("html");
    writer.writeAttribute("lang", "en");
    writer.writeStartElement("head");
    writer.writeDTD("<META http-equiv=\"Content-Type\" content=\"text/html; charset=UTF-8\">");
    writer.writeStartElement("title");
    writer.writeCharacters(map.get("heading"));
    writer.writeEndElement();
    writer.writeEndElement();

    writer.writeStartElement("body");

    writer.writeStartElement("p");
    writer.writeCharacters(map.get("from"));
    writer.writeEndElement();

    writer.writeStartElement("p");
    writer.writeCharacters(map.get("content"));
    writer.writeEndElement();

    writer.writeEndElement();
    writer.writeEndDocument();
    writer.flush();
}
```

就像在 JAXP 的例子中，我们可以调用`output.toString()`来获得 HTML 表示。

## 5。使用模板引擎

作为编写 HTML 表示的替代方法，我们可以使用[模板引擎](/web/20220826140012/https://www.baeldung.com/spring-template-engines)。Java 生态系统中有多种选择。让我们探索其中的一些。

### 5.1。使用 Apache Freemarker

Apache FreeMarker 是一个基于 Java 的模板引擎，用于生成文本输出(HTML 网页、电子邮件、配置文件、源代码等)。)基于模板和变化的数据。

为了使用它，我们需要将 [freemarker](https://web.archive.org/web/20220826140012/https://search.maven.org/search?q=g:org.freemarker%20AND%20a:freemarker) 依赖项添加到我们的 [Maven](/web/20220826140012/https://www.baeldung.com/maven) 项目中:

```
<dependency>
    <groupId>org.freemarker</groupId>
    <artifactId>freemarker</artifactId>
    <version>2.3.29</version>
</dependency>
```

首先，让我们使用 FreeMarker 语法创建一个模板:

```
<!DOCTYPE html>
<html lang="en">
<head>
    <META http-equiv="Content-Type" content="text/html; charset=UTF-8">
    <title>${heading}</title>
</head>
<body>
<p>${from}</p>
<p>${content}</p>
</body>
</html>
```

现在，让我们重用我们的`map`并填充模板中的空白:

```
Configuration cfg = new Configuration(Configuration.VERSION_2_3_29);
cfg.setDirectoryForTemplateLoading(new File(templateDirectory));
cfg.setDefaultEncoding(StandardCharsets.UTF_8.toString());
cfg.setTemplateExceptionHandler(TemplateExceptionHandler.RETHROW_HANDLER);
cfg.setLogTemplateExceptions(false);
cfg.setWrapUncheckedExceptions(true);
cfg.setFallbackOnNullLoopVariable(false);
Template temp = cfg.getTemplate(templateFile);
try (Writer output = new StringWriter()) {
    temp.process(staxTransformer.getMap(), output);
}
```

### 5.2。使用小胡子

Mustache 是一个无逻辑的模板引擎。Mustache 可以用于 HTML、配置文件、源代码——几乎任何东西。它通过使用散列或对象中提供的值扩展模板中的标签来工作。

要使用它，我们需要将 [mustache](https://web.archive.org/web/20220826140012/https://search.maven.org/search?q=g:com.github.spullara.mustache.java%20AND%20a:compiler) 依赖项添加到我们的 [Maven](/web/20220826140012/https://www.baeldung.com/maven) 项目中:

```
<dependency>
    <groupId>com.github.spullara.mustache.java</groupId>
    <artifactId>compiler</artifactId>
    <version>0.9.6</version>
</dependency>
```

让我们开始使用 Mustache 语法创建一个模板:

```
<!DOCTYPE html>
<html lang="en">
<head>
    <META http-equiv="Content-Type" content="text/html; charset=UTF-8">
    <title>{{heading}}</title>
</head>
<body>
<p>{{from}}</p>
<p>{{content}}</p>
</body>
</html>
```

现在，让我们用我们的`map`填充模板:

```
MustacheFactory mf = new DefaultMustacheFactory();
Mustache mustache = mf.compile(templateFile);
try (Writer output = new StringWriter()) {
    mustache.execute(output, staxTransformer.getMap());
    output.flush();
}
```

## 6。产生的 HTML

最后，对于所有的代码示例，我们将得到相同的 HTML 输出:

```
<!DOCTYPE html>
<html lang="en">
<head>
    <META http-equiv="Content-Type" content="text/html; charset=UTF-8">
    <title>Build #7 passed</title>
</head>
<body>
<p>from: [[email protected]](/web/20220826140012/https://www.baeldung.com/cdn-cgi/l/email-protection)</p>
<p>Success: The Jenkins CI build passed</p>
</body>
</html>
```

## 7。结论

在本教程中，我们学习了使用 JAXP、StAX、Freemarker 和 Mustache 将 XML 转换成 HTML 的基础知识。

有关 Java 中 XML 的更多信息，请查看 Baeldung 上的其他优秀资源:

*   [将 XML 反序列化为 XStream 中的对象](/web/20220826140012/https://www.baeldung.com/xstream-deserialize-xml-to-object)
*   [在 XStream 中将对象序列化为 XML](/web/20220826140012/https://www.baeldung.com/xstream-serialize-object-to-xml)
*   [Java XML 库](/web/20220826140012/https://www.baeldung.com/java-xml-libraries)

和往常一样，这里看到的完整代码样本可以在 GitHub 上找到[。](https://web.archive.org/web/20220826140012/https://github.com/eugenp/tutorials/tree/master/xml)