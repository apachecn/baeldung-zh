# 在 Java 中修改 XML 属性

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-modify-xml-attribute>

## 1。简介

当我们处理 XML 时，一个常见的活动就是处理它的属性。在本教程中，我们将探索如何使用 Java 修改 XML 属性。

## 2。依赖性

为了运行我们的测试，我们需要将 [JUnit](https://web.archive.org/web/20221127013950/https://search.maven.org/search?q=g:org.junit.jupiter%20AND%20a:junit-jupiter) 和`[xmlunit-assertj](https://web.archive.org/web/20221127013950/https://search.maven.org/search?q=g:org.xmlunit%20AND%20a:xmlunit-assertj)` 依赖项添加到我们的 [Maven](/web/20221127013950/https://www.baeldung.com/maven) 项目中:

```java
<dependency>
    <groupId>org.junit.jupiter</groupId>
    <artifactId>junit-jupiter</artifactId>
    <version>5.8.1</version>
    <scope>test</scope>
</dependency>
```

```java
<dependency>
    <groupId>org.xmlunit</groupId>
    <artifactId>xmlunit-assertj</artifactId>
    <version>2.6.3</version>
    <scope>test</scope>
</dependency>
```

## 3。使用 JAXP

让我们从一个 XML 文档开始:

```java
<?xml version="1.0" encoding="UTF-8"?>
<notification id="5">
    <to customer="true">[[email protected]](/web/20221127013950/https://www.baeldung.com/cdn-cgi/l/email-protection)</to>
    <from>[[email protected]](/web/20221127013950/https://www.baeldung.com/cdn-cgi/l/email-protection)</from>
</notification>
```

为了处理它，我们将**使用用于 XML 处理的 Java API(JAXP)**，它从版本 1.4 开始就与 Java 捆绑在一起。

让我们修改`customer`属性，将其值改为`false`。

首先，我们需要从 XML 文件构建一个`Document`对象，为此，我们将使用一个`DocumentBuilderFactory`:

```java
DocumentBuilderFactory factory = DocumentBuilderFactory.newInstance();
factory.setFeature(XMLConstants.FEATURE_SECURE_PROCESSING, true);
factory.setFeature("http://apache.org/xml/features/disallow-doctype-decl", true);
Document input = factory
  .newDocumentBuilder()
  .parse(resourcePath);
```

注意，为了**禁用`DocumentBuilderFactory`类的[外部实体处理(XXE)](https://web.archive.org/web/20221127013950/https://owasp.org/www-community/vulnerabilities/XML_External_Entity_(XXE)_Processing)** ，**我们配置了`XMLConstants.FEATURE_SECURE_PROCESSING`和`http://apache.org/xml/features/disallow-doctype-decl`特性**。当我们解析不可信的 XML 文件时，配置它是一个很好的实践。

在初始化我们的`input`对象之后，我们需要定位我们想要改变属性的节点。让我们使用一个 [XPath 表达式](/web/20221127013950/https://www.baeldung.com/java-xpath)来选择它:

```java
XPath xpath = XPathFactory
  .newInstance()
  .newXPath();
String expr = String.format("//*[contains(@%s, '%s')]", attribute, oldValue);
NodeList nodes = (NodeList) xpath.evaluate(expr, input, XPathConstants.NODESET);
```

在这种情况下，XPath `evaluate`方法返回一个包含匹配节点的节点列表。

让我们遍历列表来更改值:

```java
for (int i = 0; i < nodes.getLength(); i++) {
    Element value = (Element) nodes.item(i);
    value.setAttribute(attribute, newValue);
}
```

或者，我们可以使用一个`IntStream`来代替`for`循环:

```java
IntStream
    .range(0, nodes.getLength())
    .mapToObj(i -> (Element) nodes.item(i))
    .forEach(value -> value.setAttribute(attribute, newValue));
```

现在，让我们使用一个`Transformer`对象来应用更改:

```java
TransformerFactory factory = TransformerFactory.newInstance();
factory.setFeature(XMLConstants.FEATURE_SECURE_PROCESSING, true);
Transformer xformer = factory.newTransformer();
xformer.setOutputProperty(OutputKeys.INDENT, "yes");
Writer output = new StringWriter();
xformer.transform(new DOMSource(input), new StreamResult(output));
```

如果我们打印`output`对象内容，我们将得到修改了`customer`属性的结果 XML:

```java
<?xml version="1.0" encoding="UTF-8"?>
<notification id="5">
    <to customer="false">[[email protected]](/web/20221127013950/https://www.baeldung.com/cdn-cgi/l/email-protection)</to>
    <from>[[email protected]](/web/20221127013950/https://www.baeldung.com/cdn-cgi/l/email-protection)</from>
</notification>
```

同样，如果我们需要在单元测试中验证它，我们可以使用 [XMLUnit](/web/20221127013950/https://www.baeldung.com/xmlunit2) 的`assertThat`方法:

```java
assertThat(output.toString()).hasXPath("//*[contains(@customer, 'false')]");
```

## 4。使用 dom4j

[dom4j](https://web.archive.org/web/20221127013950/https://dom4j.github.io/) 是一个处理 XML 的开源框架，集成了 XPath，完全支持 dom、SAX、JAXP 和 Java 集合。

### 4.1。Maven 依赖关系

我们需要将 [dom4j](https://web.archive.org/web/20221127013950/https://search.maven.org/search?q=g:org.dom4j%20AND%20a:dom4j) 和 [jaxen](https://web.archive.org/web/20221127013950/https://search.maven.org/search?q=g:jaxen%20AND%20a:jaxen) 依赖项添加到我们的`pom.xml`中，以便在我们的项目中使用 dom4j:

```java
<dependency>
    <groupId>org.dom4j</groupId>
    <artifactId>dom4j</artifactId>
    <version>2.1.1</version>
</dependency>
<dependency>
    <groupId>jaxen</groupId>
    <artifactId>jaxen</artifactId>
    <version>1.2.0</version>
</dependency>
```

我们可以在 XML 库支持文章中了解更多关于 dom4j 的信息。

### 4.2。使用`org.dom4j.Element.addAttribute`

dom4j 提供了`Element`接口作为 XML 元素的抽象。我们将使用`addAttribute`方法来更新我们的`customer`属性。

让我们看看这是如何工作的。

首先，我们需要从 XML 文件构建一个`Document`对象——这一次，我们将使用一个`SAXReader`:

```java
SAXReader xmlReader = new SAXReader();
Document input = xmlReader.read(resourcePath);
xmlReader.setFeature("http://apache.org/xml/features/disallow-doctype-decl", true);
xmlReader.setFeature("http://xml.org/sax/features/external-general-entities", false);
xmlReader.setFeature("http://xml.org/sax/features/external-parameter-entities", false);
```

我们设置附加功能是为了防止 XXE。

像 JAXP 一样，我们可以使用 XPath 表达式来选择节点:

```java
String expr = String.format("//*[contains(@%s, '%s')]", attribute, oldValue);
XPath xpath = DocumentHelper.createXPath(expr);
List<Node> nodes = xpath.selectNodes(input);
```

现在，我们可以迭代并更新属性:

```java
for (int i = 0; i < nodes.size(); i++) {
    Element element = (Element) nodes.get(i);
    element.addAttribute(attribute, newValue);
}
```

注意，使用这种方法，如果给定名称的属性已经存在，它将被替换。否则，它将被添加。

为了打印结果，我们可以重用前面 JAXP 部分的代码。

## 5。使用 jOOX

[jOOX](https://web.archive.org/web/20221127013950/https://github.com/jOOQ/jOOX) (jOOX 面向对象的 XML)是`org.w3c.dom`包的包装器，允许流畅的 XML 文档创建和操作，其中需要 DOM 但过于冗长。jOOX 只包装底层文档，可以用来增强 DOM，不能作为替代方案。

### 5.1。Maven 依赖关系

我们需要将依赖项添加到我们的`pom.xml`中，以便在我们的项目中使用 jOOX。

对于 Java 9+的使用，我们可以使用:

```java
<dependency>
    <groupId>org.jooq</groupId>
    <artifactId>joox</artifactId>
    <version>1.6.2</version>
</dependency>
```

或者使用 Java 6+，我们有:

```java
<dependency>
    <groupId>org.jooq</groupId>
    <artifactId>joox-java-6</artifactId>
    <version>1.6.2</version>
</dependency>
```

我们可以在 Maven 中央存储库中找到`[joox](https://web.archive.org/web/20221127013950/https://search.maven.org/search?q=g:org.jooq%20AND%20a:joox)` 和`[joox-java-6](https://web.archive.org/web/20221127013950/https://search.maven.org/search?q=g:org.jooq%20AND%20a:joox-java-6)`的最新版本。

### 5.2。使用`org.w3c.dom.Element.setAttribute`

jOOX API 本身受到了 [jQuery](https://web.archive.org/web/20221127013950/https://jquery.com/) 的启发，我们可以在下面的例子中看到。让我们看看如何使用它。

首先，我们需要加载`Document`:

```java
DocumentBuilder builder = JOOX.builder();
Document input = builder.parse(resourcePath);
```

现在，我们需要选择它:

```java
Match $ = $(input);
```

为了选择`customer Element,`，我们可以使用`find`方法或 XPath 表达式。在这两种情况下，我们都会得到与之匹配的元素列表。

让我们看看`find`方法的实际应用:

```java
$.find("to")
    .get()
    .stream()
    .forEach(e -> e.setAttribute(attribute, newValue));
```

要获得作为`String`的结果，我们只需调用`toString()`方法:

```java
$.toString();
```

## 6。基准测试

为了比较这些库的性能，我们使用了一个 [JMH](/web/20221127013950/https://www.baeldung.com/java-microbenchmark-harness) 基准。

让我们看看结果:

```java
| Benchmark                          Mode  Cnt  Score   Error  Units |
|--------------------------------------------------------------------|
| AttributeBenchMark.dom4jBenchmark  avgt    5  0.150 ± 0.003  ms/op |
| AttributeBenchMark.jaxpBenchmark   avgt    5  0.166 ± 0.003  ms/op |
| AttributeBenchMark.jooxBenchmark   avgt    5  0.230 ± 0.033  ms/op |
```

正如我们所看到的，对于这个用例以及我们的实现，dom4j 和 JAXP 比 jOOX 有更好的成绩。

## 7。结论

在这篇快速教程中，我们介绍了如何使用 JAXP、dom4j 和 jOOX 修改 XML 属性。此外，我们用 JMH 基准测试了这些库的性能。

像往常一样，这里显示的所有代码样本都可以在 [GitHub](https://web.archive.org/web/20221127013950/https://github.com/eugenp/tutorials/tree/master/xml) 上获得。