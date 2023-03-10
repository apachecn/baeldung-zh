# 用 Java 漂亮地打印 XML

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-pretty-print-xml>

## 1.概观

当我们需要手动读取一个 [XML](/web/20221109185607/https://www.baeldung.com/java-xml) 文件时，通常我们希望以一种漂亮的打印格式读取内容。许多文本编辑器或 ide 可以重新格式化 XML 文档。如果我们在 Linux 中工作，我们可以从命令行中[漂亮地打印 XML 文件。](/web/20221109185607/https://www.baeldung.com/linux/pretty-print-xml)

然而，有时，我们需要在 Java 程序中将原始的 XML 字符串转换成打印精美的格式。例如，我们可能希望在用户界面中显示一个印刷精美的 XML 文档，以便更好地理解。

在本教程中，我们将探索如何用 Java 漂亮地打印 XML。

## 2.问题简介

为简单起见，我们将采用一个未格式化的`emails.xml` 文件作为输入:

```java
<emails> <email> <from>Kai</from> <to>Amanda</to> <time>2018-03-05</time>
<subject>I am flying to you</subject></email> <email>
<from>Jerry</from> <to>Tom</to> <time>1992-08-08</time> <subject>Hey Tom, catch me if you can!</subject>
</email> </emails> 
```

正如我们所看到的，`emails.xml` 文件是格式良好的。然而，由于格式混乱，不容易阅读。

我们的目标是创建一种方法，将这个难看的原始 XML 字符串转换成格式美观的字符串。

此外，我们将讨论定制两个常见的输出属性:indent-size ( `integer`)和 suppress XML declaration(`boolean`)。

indent-size 属性非常简单:它是要缩进的空格数(每一级)。另一方面，抑制 XML 声明选项决定我们是否希望在生成的 XML 中包含 XML 声明标记。典型的 XML 声明如下所示:

```java
<?xml version="1.0" encoding="UTF-8"?>
```

在本教程中，我们将使用标准的 Java API 和另一种使用外部库的方法来解决这个问题。

接下来，让我们看看他们的行动。

## 3.用`Transformer`类漂亮地打印 XML

Java API 提供了`[Transformer](/web/20221109185607/https://www.baeldung.com/java-write-xml-document-file#transformer)`类来进行 XML 转换。

### 3.1.使用默认`Transformer`

首先，让我们看看使用`Transformer`类的漂亮打印解决方案:

```java
public static String prettyPrintByTransformer(String xmlString, int indent, boolean ignoreDeclaration) {

    try {
        InputSource src = new InputSource(new StringReader(xmlString));
        Document document = DocumentBuilderFactory.newInstance().newDocumentBuilder().parse(src);

        TransformerFactory transformerFactory = TransformerFactory.newInstance();
        transformerFactory.setAttribute("indent-number", indent);
        Transformer transformer = transformerFactory.newTransformer();
        transformer.setOutputProperty(OutputKeys.ENCODING, "UTF-8");
        transformer.setOutputProperty(OutputKeys.OMIT_XML_DECLARATION, ignoreDeclaration ? "yes" : "no");
        transformer.setOutputProperty(OutputKeys.INDENT, "yes");

        Writer out = new StringWriter();
        transformer.transform(new DOMSource(document), new StreamResult(out));
        return out.toString();
    } catch (Exception e) {
        throw new RuntimeException("Error occurs when pretty-printing xml:\n" + xmlString, e);
    }
} 
```

现在，让我们快速浏览一下这个方法，并弄清楚它是如何工作的:

*   首先，我们解析原始的 XML 字符串，得到一个 [`Document`](https://web.archive.org/web/20221109185607/https://docs.oracle.com/en/java/javase/11/docs/api/java.xml/org/w3c/dom/Document.html) 对象。
*   接下来，我们获取一个`[TransformerFactory](https://web.archive.org/web/20221109185607/https://docs.oracle.com/en/java/javase/11/docs/api/java.xml/javax/xml/transform/TransformerFactory.html)`实例并设置所需的 indent-size 属性。
*   然后，我们可以从配置好的`tranformerFactory`对象中获得一个默认的 transformer 实例。
*   `transformer`对象支持各种输出属性。**为了决定我们是否要跳过声明，我们设置了`OutputKeys.OMIT_XML_DECLARATION`属性。**
*   由于我们希望有一个格式良好的`String`对象，最后，我们将解析后的 XML `Document `转换为`StringWriter`，并返回转换后的`String`。

我们已经在上面的方法中设置了`TransformerFactory`对象的缩进大小。**或者，我们也可以在`transformer`实例**上定义`indent-amount`属性:

```java
transformer.setOutputProperty("{http://xml.apache.org/xslt}indent-amount", String.valueOf(indent));
```

接下来，让我们测试该方法是否如预期的那样工作。

### 3.2.测试方法

我们的 Java 项目是一个 Maven 项目，我们已经将`emails.xml `放在了`src/main/resources/xml/email.xml`下。我们已经创建了`readFromInputStream`方法来作为`String`读取输入文件。但是，我们不会深入这个方法的细节，因为它与我们这里的主题没有太大关系。假设我们希望设置 indent-size=2，并跳过结果中的 XML 声明:

```java
public static void main(String[] args) throws IOException {
    InputStream inputStream = XmlPrettyPrinter.class.getResourceAsStream("/xml/emails.xml");
    String xmlString = readFromInputStream(inputStream);
    System.out.println("Pretty printing by Transformer");
    System.out.println("=============================================");
    System.out.println(prettyPrintByTransformer(xmlString, 2, true));
}
```

如`main`方法所示，我们将输入文件作为一个`String`读取，然后调用我们的`prettyPrintByTransformer`方法来获得一个漂亮的 XML `String`。

接下来，**让我们用 Java 8** 运行`main `方法:

```java
Pretty printing by Transformer
=============================================
<emails>
  <email>
    <from>Kai</from>
    <to>Amanda</to>
    <time>2018-03-05</time>
    <subject>I am flying to you</subject>
  </email>
  <email>
    <from>Jerry</from>
    <to>Tom</to>
    <time>1992-08-08</time>
    <subject>Hey Tom, catch me if you can!</subject>
  </email>
</emails>
```

正如上面的输出所示，我们的方法按预期工作。

但是，如果我们用 Java 9 或更高版本再次测试它，我们可能会看到不同的输出。

接下来，**让我们看看如果用 Java 9** 运行它会产生什么:

```java
Pretty printing by Transformer
=============================================
<emails>

  <email>

    <from>Kai</from>

    <to>Amanda</to>

    <time>2018-03-05</time>

    <subject>I am flying to you</subject>
  </email>

  <email>

    <from>Jerry</from>

    <to>Tom</to>

    <time>1992-08-08</time>

    <subject>Hey Tom, catch me if you can!</subject>

  </email>

</emails>

=============================================
```

正如我们在上面的输出中看到的，输出中有意外的空行。

这是因为我们的原始输入在元素之间包含空白，例如:

```java
<emails> <email> <from>Kai</from> ...
```

**从 Java 9 开始，`Transformer`类的漂亮打印特性没有定义实际的格式。因此，也将输出只有空白的节点。**这个已经在本 [JDK 虫票](https://web.archive.org/web/20221109185607/https://bugs.openjdk.java.net/browse/JDK-8262285?attachmentViewMode=list)中讨论过了。另外， [Java 9 的发行说明](https://web.archive.org/web/20221109185607/https://www.oracle.com/java/technologies/javase/9-notes.html)已经在 xml/jaxp 部分解释了这一点。

如果我们希望我们的 pretty-print 方法在各种 Java 版本下总是生成相同的格式，我们需要提供一个样式表文件。

接下来，让我们创建一个简单的`xsl`文件来实现它。

### 3.3.提供 XSLT 文件

首先，让我们创建`prettyprint.xsl`文件来定义输出格式:

```java
<xsl:stylesheet version="1.0" xmlns:xsl="http://www.w3.org/1999/XSL/Transform">
    <xsl:strip-space elements="*"/>
    <xsl:output method="xml" encoding="UTF-8"/>

    <xsl:template match="@*|node()">
        <xsl:copy>
            <xsl:apply-templates select="@*|node()"/>
        </xsl:copy>
    </xsl:template>

</xsl:stylesheet>
```

正如我们所看到的，在`prettyprint.xsl`文件中，**我们使用了< `xsl:strip-space/>`元素来删除只有空白的节点，这样它们就不会出现在输出**中。

接下来，我们仍然需要对我们的方法做一个小的改变。我们将不再使用默认的变压器。相反，**我们将使用 XSLT 文档**创建一个`Transformer`对象:

```java
Transformer transformer = transformerFactory.newTransformer(new StreamSource(new StringReader(readPrettyPrintXslt())));
```

这里，`readPrettyPrintXslt()`方法读取`prettyprint.xsl`内容。

现在，如果我们在 Java 8 和 Java 9 中测试该方法，两者都会产生相同的输出:

```java
Pretty printing by Transformer
=============================================
<emails>
  <email>
    <from>Kai</from>
    <to>Amanda</to>
    <time>2018-03-05</time>
    <subject>I am flying to you</subject>
  </email>
...
</emails>
```

我们已经用标准的 Java API 解决了这个问题。接下来，让我们使用外部库漂亮地打印出`emails.xml`。

## 4.用 Dom4j 库漂亮地打印 XML

Dom4j 是一个流行的 XML 库。它允许我们轻松地打印 XML 文档。

首先，让我们将 Dom4j 依赖项添加到我们的`pom.xml`中:

```java
<dependency>
    <groupId>org.dom4j</groupId>
    <artifactId>dom4j</artifactId>
    <version>2.1.3</version>
</dependency>
```

我们以 2.1.3 版本为例。我们可以在 Maven 中央存储库中找到最新版本的。

接下来，让我们看看如何使用 Dom4j 库来美化 XML:

```java
public static String prettyPrintByDom4j(String xmlString, int indent, boolean skipDeclaration) {
    try {
        OutputFormat format = OutputFormat.createPrettyPrint();
        format.setIndentSize(indent);
        format.setSuppressDeclaration(skipDeclaration);
        format.setEncoding("UTF-8");

        org.dom4j.Document document = DocumentHelper.parseText(xmlString);
        StringWriter sw = new StringWriter();
        XMLWriter writer = new XMLWriter(sw, format);
        writer.write(document);
        return sw.toString();
    } catch (Exception e) {
        throw new RuntimeException("Error occurs when pretty-printing xml:\n" + xmlString, e);
    }
}
```

**D0m4j 的`OutputFormat`类提供了一个`createPrettyPrint`方法来创建一个预定义的漂亮打印`OutputFormat`对象。**如上面的方法所示，我们可以在默认的漂亮打印格式上添加一些定制。在这种情况下，我们设置缩进大小，并决定是否要在结果中包含声明。

接下来，我们解析原始 XML 字符串，并用准备好的`OutputFormat`实例创建一个`XMLWritter`对象。

最后，`XMLWriter`对象将按照要求的格式编写解析后的 XML 文档。

接下来，让我们测试它是否能漂亮地打印出`emails.xml`文件。这一次，假设我们想要包含声明，并且在结果中缩进大小为 8:

```java
System.out.println("Pretty printing by Dom4j");
System.out.println("=============================================");
System.out.println(prettyPrintByDom4j(xmlString, 8, false));
```

当我们运行该方法时，我们将看到输出:

```java
Pretty printing by Dom4j
=============================================
<?xml version="1.0" encoding="UTF-8"?>

<emails> 
        <email> 
                <from>Kai</from>  
                <to>Amanda</to>  
                <time>2018-03-05</time>  
                <subject>I am flying to you</subject>
        </email>  
        <email> 
                <from>Jerry</from>  
                <to>Tom</to>  
                <time>1992-08-08</time>  
                <subject>Hey Tom, catch me if you can!</subject> 
        </email> 
</emails>
```

正如上面的输出所示，该方法已经解决了这个问题。

## 5.结论

在本文中，我们介绍了两种用 Java 美化 XML 文件的方法。

我们可以使用标准的 Java API 漂亮地打印 XML。然而，我们需要记住，根据 Java 版本的不同，`Transformer`对象可能会产生不同的结果。解决方案是提供一个 XSLT 文件。

或者，Dom4j 库可以直接解决问题。

和往常一样，GitHub 上有完整版本的代码[。](https://web.archive.org/web/20221109185607/https://github.com/eugenp/tutorials/tree/master/xml-2)