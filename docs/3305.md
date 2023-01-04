# 将 org.w3.dom.Document 写入文件

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-write-xml-document-file>

## 1。概述

XML 处理的一个重要部分是创建可供他人使用的 XML 文件。

当用 Java 处理 XML 时，我们经常会有一个需要导出的`[org.w3c.dom.Document](https://web.archive.org/web/20220526050643/https://docs.oracle.com/en/java/javase/11/docs/api/java.xml/org/w3c/dom/Document.html) `实例。

在这个快速教程中，**我们将会看到如何将一个`Document`写入一个文件，既可以是内嵌的，也可以是漂亮的打印格式**。

## 2。使用变压器

将`Document` s 写入文件时最重要的是`javax.xml.transform.Transformer.`

### 2.1。创建变压器

所以，我们先来弄个`TransformerFactory`。我们将使用这个工厂来创建变压器:

```
TransformerFactory transformerFactory = TransformerFactory.newInstance()
```

系统属性`javax.xml.transform.TransformerFactory`指定创建哪个工厂实现。因此，这个属性命名了`TransformerFactory `抽象类的一个具体子类。**但是，如果我们不定义这个属性，转换器将简单地使用平台默认值。**

**注意，从 Java 9 开始，我们可以使用`TransformerFactory.` newDefaultInstance( )来创建内置的系统默认实现。**

现在我们有了工厂，让我们创建`Transformer`:

```
Transformer transformer = transformerFactory.newTransformer();
```

### 2.2。指定来源和结果

**`Transformer`将源转换成结果。**在我们的例子中，源是 XML 文档，结果是输出文件。

首先，让我们指定转换的来源。这里，我们将使用我们的`Document`来构建一个 DOM 源:

```
DOMSource source = new DOMSource(document);
```

**注意，来源不一定是整个文档。**只要 XML 格式良好，我们就可以使用文档的子树。

接下来，我们将指定转换器应该在哪里写入转换结果:

```
FileWriter writer = new FileWriter(new File(fileName));
StreamResult result = new StreamResult(writer);
```

这里，我们告诉转换器结果是一个文件流。**但是，我们可以使用任何类型的`java.io.Writer` 或`java.io.OutputStream`来创建** `**StreamResult.** `例如，我们可以使用`StringWriter `来构造一个`String`，然后可以被记录。

### 2.3。创建 XML 文件

最后，我们将告诉转换器对源对象进行操作，并输出到结果对象:

```
transformer.transform(source, result);
```

**这将最终创建一个包含 XML 文档内容的文件:**

```
<?xml version="1.0" encoding="UTF-8" standalone="no"?><Company><Department name="Sales">
  <Employee name="John Smith"/><Employee name="Tim Dellor"/></Department></Company>
```

## 3。定制输出

我们可以通过指定各种输出属性来自定义写入文件的 XML。

### 3.1。美化输出

现在，我们的默认转换器只是将所有内容写在一行上，这读起来并不愉快。事实上，如果 XML 很大，阅读起来会更加困难。

**我们可以通过设置 transformer:** 的`OutputKeys.INDENT`属性来配置我们的 transformer 进行漂亮打印

```
transformer.setOutputProperty(OutputKeys.INDENT, "yes");
transformer.setOutputProperty("{http://xml.apache.org/xslt}indent-amount", "4");
```

注意，除了`OutputKeys.INDENT`，我们还在这里指定了`indent-amount`属性。这将正确地缩进输出，因为默认情况下缩进是零个空格。

通过设置上述属性，我们得到了更好的输出:

```
<?xml version="1.0" encoding="UTF-8" standalone="no"?>
<Company>
    <Department name="Sales">
        <Employee name="John Smith"/>
        <Employee name="Tim Dellor"/>
    </Department>
</Company>
```

### 3.2。省略 XML 声明

有时，我们可能想要排除 XML 声明。

我们可以通过设置`OutputKeys.OMIT_XML_DECLARATION `属性来配置我们的转换器:

```
transformer.setOutputProperty(OutputKeys.OMIT_XML_DECLARATION, "yes");
```

再次使用我们的变压器，我们得到:

```
<Company>
    <Department name="Sales">
        <Employee name="John Smith"/>
        <Employee name="Tim Dellor"/>
    </Department>
</Company>
```

### 3.3。其他输出属性

因此，除了美化打印和省略 XML 声明之外，我们还可以用其他方式定制输出:

*   我们可以使用`OutputKeys.VERSION, `指定 XML 版本，默认为“1.0”
*   我们可以使用`OutputKeys.ENCODING`指定我们的首选字符编码，默认为“utf-8”
*   此外，我们还可以指定其他典型的声明属性，如 [`SYSTEM`、`PUBLIC`和`STANDALONE`](https://web.archive.org/web/20220526050643/https://docs.oracle.com/en/java/javase/11/docs/api/java.xml/javax/xml/transform/OutputKeys.html) 。

## 4。结论

在本教程中，我们看到了如何将一个`org.w3c.Document `导出到一个文件，以及如何定制输出。

当然，附带的源代码可以在 GitHub 的[上找到。](https://web.archive.org/web/20220526050643/https://github.com/eugenp/tutorials/tree/master/xml)