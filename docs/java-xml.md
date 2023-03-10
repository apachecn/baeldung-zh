# Java 中的 XML 指南

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-xml>

## 1。概述

这是在 Java 中使用 XML 的指南。

我们将讨论最常见的 [Java XML 处理库](/web/20221008004625/https://www.baeldung.com/java-xml-libraries)——用于解析和绑定。

## 2。DOM 解析器

简而言之，DOM 解析器处理整个 XML 文档，将其加载到内存中，并构造文档的树表示。

### 2.1。有用的资源

*   [使用 DOM 解析在 Java 中处理 XML 文件](/web/20221008004625/https://www.baeldung.com/java-xerces-dom-parsing)
*   [将 org.w3.dom.Document 写入文件](/web/20221008004625/https://www.baeldung.com/java-write-xml-document-file)
*   [用 Java 漂亮地打印 XML](/web/20221008004625/https://www.baeldung.com/java-pretty-print-xml)
*   [Java XPath 简介](/web/20221008004625/https://www.baeldung.com/java-xpath)
*   [使用 dom4j 在 Java 中修改 XML 属性](/web/20221008004625/https://www.baeldung.com/java-modify-xml-attribute#using-dom4j)

## 3。SAX 解析器

SAX 解析器是基于事件的解析器——它使用回调来解析 XML 文档，而无需将整个文档加载到内存中。

### 3.1。有用的资源

*   [使用 SAX 解析器解析 XML 文件](/web/20221008004625/https://www.baeldung.com/java-sax-parser)

## 4。StAX 解析器

StAX 解析器介于 DOM 和 SAX 解析器之间。

### 4.1。有用的资源

*   [使用 StAX 解析 XML 文件](/web/20221008004625/https://www.baeldung.com/java-stax)
*   [使用 StAX 将 XML 转换成 HTML】](/web/20221008004625/https://www.baeldung.com/java-convert-xml-to-html#stax)

## 5 号。JAXB【% 1】

JAXB——用于 XML 绑定的 Java 架构——用于将对象与 XML 相互转换。

JAXB 是 Java SE 平台的一部分，也是 Jakarta EE 中的 API 之一。

### 5.1。有用的资源

*   [JAXB 指南](/web/20221008004625/https://www.baeldung.com/jaxb)
*   [使用 JAXB 解组日期](/web/20221008004625/https://www.baeldung.com/jaxb-unmarshalling-dates)
*   [Oracle JAXB 教程](https://web.archive.org/web/20221008004625/https://docs.oracle.com/javase/tutorial/jaxb/intro/index.html)

## 6\. XStream

XStream 是一个简单的库，可以将对象序列化为 XML 或从 XML 序列化。

### 6.1。有用的资源

*   [XStream 用户指南:将 XML 转换为对象](/web/20221008004625/https://www.baeldung.com/xstream-deserialize-xml-to-object)
*   [XStream 用户指南:将对象转换为 XML](/web/20221008004625/https://www.baeldung.com/xstream-serialize-object-to-xml)
*   [使用 XStream 远程执行代码](/web/20221008004625/https://www.baeldung.com/java-xstream-remote-code-execution)

## 7 .**。杰克逊 XML**

Jackson XML 是 Jackson JSON 处理器的扩展，用于读写 XML 编码的数据。

### 7.1。有用的资源

*   [官网](https://web.archive.org/web/20221008004625/https://github.com/FasterXML/jackson)
*   [Github](https://web.archive.org/web/20221008004625/https://github.com/FasterXML/jackson-dataformat-xml)
*   [Jackson XML 数据绑定 Wiki](https://web.archive.org/web/20221008004625/https://github.com/FasterXML/jackson-dataformat-xml/wiki)
*   [杰克逊 XML 注释](https://web.archive.org/web/20221008004625/https://github.com/FasterXML/jackson-dataformat-xml/wiki/Jackson-XML-annotations)
*   [使用 Jackson 进行 XML 序列化和反序列化](/web/20221008004625/https://www.baeldung.com/jackson-xml-serialization-and-deserialization)
*   [使用 Jackson 将 XML 转换成 JSON】](/web/20221008004625/https://www.baeldung.com/jackson-convert-xml-json)

## 8。阿帕奇 CXF 神盾

Aegis 是一个数据绑定或子系统，可以在 Java 对象和 XML 模式描述的 XML 文档之间进行映射。

### 8.1。有用的资源

*   [官网](https://web.archive.org/web/20221008004625/https://cxf.apache.org/docs/aegis-21.html)
*   [Apache CXF Aegis 数据绑定简介](/web/20221008004625/https://www.baeldung.com/aegis-data-binding-in-apache-cxf)
*   [Javadoc](https://web.archive.org/web/20221008004625/https://cxf.apache.org/javadoc/latest-3.5.x/)

## 9。 **JiBX**

JiBX 是一个将 XML 数据绑定到 Java 对象的工具。与 JAXB 等其他常用工具相比，它提供了可靠的性能。

### 9.1。有用的资源

*   [官网](https://web.archive.org/web/20221008004625/https://jibx.sourceforge.io/)
*   [JiBX 简介](/web/20221008004625/https://www.baeldung.com/jibx)

## 10。 XMLUnit 2

XMLUnit 2.x 是一个强大的库，可以帮助我们测试和验证 XML 内容，当我们确切知道 XML 应该包含什么内容时，它就变得非常方便。

### 10.1。有用的资源

*   [官网](https://web.archive.org/web/20221008004625/http://www.xmlunit.org/)
*   [XML unit 2 . x 简介](/web/20221008004625/https://www.baeldung.com/xmlunit2)

## 11。 **结论**

这是对 Java 中 XML 生态系统的快速介绍。

以此为指南，学习更多关于做 XML 工作的知识，并获得 Java XML 前景的高级视图。

如果你想在一个地方看到我们所有 XML 内容的链接，我们也有关于这个主题的文章集。