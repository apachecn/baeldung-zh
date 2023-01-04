# 使用 DOM 解析在 Java 中处理 XML 文件

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-xerces-dom-parsing>

## 1。概述

在本教程中，我们将讨论如何使用 [Apache Xerces](https://web.archive.org/web/20220625222218/https://xerces.apache.org/) 解析 DOM，这是一个成熟的用于解析/操作 XML 的库。

解析 XML 文档有多种选择；在本文中，我们将重点讨论 DOM 解析。DOM 解析器加载一个文档，并在内存中创建一个完整的层次树。

关于 Java 中 XML 库支持的概述，请查看我们之前的[文章](/web/20220625222218/https://www.baeldung.com/java-xml-libraries)。

## 2。我们的文档

让我们从我们将在示例中使用的 XML 文档开始:

```
<?xml version="1.0"?>
<tutorials>
    <tutorial tutId="01" type="java">
        <title>Guava</title>
        <description>Introduction to Guava</description>
        <date>04/04/2016</date>
        <author>GuavaAuthor</author>
    </tutorial>
...
</tutorials>
```

请注意，我们的文档有一个名为“tutorials”的根节点，其中有 4 个“tutorials”子节点。其中每个都有两个属性:“tutId”和“type”。同样，每个“教程”有 4 个子节点:“标题”、“描述”、“日期”和“作者”。

现在我们可以继续解析这个文档。

## 3.正在加载 XML 文件

首先，我们应该注意到 Apache Xerces 库是和 JDK 打包在一起的，所以我们不需要任何额外的设置。

让我们直接开始加载 XML 文件:

```
DocumentBuilder builder = DocumentBuilderFactory.newInstance().newDocumentBuilder();
Document doc = builder.parse(new File("src/test/resources/example_jdom.xml"));
doc.getDocumentElement().normalize();
```

在上面的例子中，我们首先获得一个`DocumentBuilder`类的实例，然后在 XML 文档上使用`parse()`方法获得一个表示它的`Document`对象。

我们还需要使用`normalize()`方法来确保文档层次结构不会受到节点中任何额外空白或新行的影响。

## 4。解析 DOM

现在，让我们研究一下我们的 XML 文件。

让我们从检索标签为“tutorial”的所有元素开始。我们可以使用`getElementsByTagName()`方法来完成这项工作，该方法将返回一个`NodeList:`

```
@Test
public void whenGetElementByTag_thenSuccess() {
    NodeList nodeList = doc.getElementsByTagName("tutorial");
    Node first = nodeList.item(0);

    assertEquals(4, nodeList.getLength());
    assertEquals(Node.ELEMENT_NODE, first.getNodeType());
    assertEquals("tutorial", first.getNodeName());        
}
```

需要注意的是 **`Node`是 DOM 组件**的主要数据类型。所有的元素、属性、文本都被认为是节点。

接下来，让我们看看如何使用`getAttributes()`获得第一个元素的属性:

```
@Test
public void whenGetFirstElementAttributes_thenSuccess() {
    Node first = doc.getElementsByTagName("tutorial").item(0);
    NamedNodeMap attrList = first.getAttributes();

    assertEquals(2, attrList.getLength());

    assertEquals("tutId", attrList.item(0).getNodeName());
    assertEquals("01", attrList.item(0).getNodeValue());

    assertEquals("type", attrList.item(1).getNodeName());
    assertEquals("java", attrList.item(1).getNodeValue());
}
```

这里，我们获得了`NamedNodeMap`对象，然后使用`item(index)`方法来检索每个节点。

对于每个节点，我们可以使用`getNodeName()`和`getNodeValue()`来找到它们的属性。

## 5。遍历节点

接下来，让我们看看如何遍历 DOM 节点。

在下面的测试中，我们将遍历第一个元素的子节点并打印它们的内容:

```
@Test
public void whenTraverseChildNodes_thenSuccess() {
    Node first = doc.getElementsByTagName("tutorial").item(0);
    NodeList nodeList = first.getChildNodes();
    int n = nodeList.getLength();
    Node current;
    for (int i=0; i<n; i++) {
        current = nodeList.item(i);
        if(current.getNodeType() == Node.ELEMENT_NODE) {
            System.out.println(
              current.getNodeName() + ": " + current.getTextContent());
        }
    }
}
```

首先，我们使用`getChildNodes()`方法获得`NodeList`，然后遍历它，并打印节点名称和文本内容。

输出将显示我们文档中第一个“tutorial”元素的内容:

```
title: Guava
description: Introduction to Guava
date: 04/04/2016
author: GuavaAuthor
```

## 6。修改 DOM

我们还可以对 DOM 进行修改。

例如，让我们将`type`属性的值从“java”更改为“other”:

```
@Test
public void whenModifyDocument_thenModified() {
    NodeList nodeList = doc.getElementsByTagName("tutorial");
    Element first = (Element) nodeList.item(0);

    assertEquals("java", first.getAttribute("type")); 

    first.setAttribute("type", "other");
    assertEquals("other", first.getAttribute("type"));     
}
```

在这里，改变属性值是一个简单的调用`Element`的`setAttribute()`方法的事情。

## 7 .**。创建新文档**

除了修改 DOM，我们还可以从头开始创建新的 XML 文档。

让我们首先看看我们想要创建的文件:

```
<?xml version="1.0" encoding="UTF-8" standalone="no"?>
<users>
    <user id="1">
        <email>[[email protected]](/web/20220625222218/https://www.baeldung.com/cdn-cgi/l/email-protection)</email>
    </user>
</users>
```

我们的 XML 包含一个带有一个`user`元素的`users`根节点，该元素还有一个子节点`email.`

为了实现这一点，我们首先必须调用返回一个`Document`对象的`Builder`的`newDocument()`方法。

然后，我们将调用新对象的`createElement()`方法:

```
@Test
public void whenCreateNewDocument_thenCreated() throws Exception {
    Document newDoc = builder.newDocument();
    Element root = newDoc.createElement("users");
    newDoc.appendChild(root);

    Element first = newDoc.createElement("user");
    root.appendChild(first);
    first.setAttribute("id", "1");

    Element email = newDoc.createElement("email");
    email.appendChild(newDoc.createTextNode("[[email protected]](/web/20220625222218/https://www.baeldung.com/cdn-cgi/l/email-protection)"));
    first.appendChild(email);

    assertEquals(1, newDoc.getChildNodes().getLength());
    assertEquals("users", newDoc.getChildNodes().item(0).getNodeName());
}
```

为了将每个元素添加到 DOM 中，我们还调用了`appendChild()`方法。

## 8。保存文档

在修改我们的文档或从头开始创建一个文档后，我们需要将它保存在一个文件中。

我们将从创建一个`DOMSource`对象开始，然后使用一个简单的`Transformer`将文档保存到一个文件中:

```
private void saveDomToFile(Document document,String fileName) 
  throws Exception {

    DOMSource dom = new DOMSource(document);
    Transformer transformer = TransformerFactory.newInstance()
      .newTransformer();

    StreamResult result = new StreamResult(new File(fileName));
    transformer.transform(dom, result);
}
```

同样，我们可以在控制台中打印我们的文档:

```
private void printDom(Document document) throws Exception{
    DOMSource dom = new DOMSource(document);
    Transformer transformer = TransformerFactory.newInstance()
        .newTransformer();

    transformer.transform(dom, new StreamResult(System.out));
}
```

## 9。结论

在这篇简短的文章中，我们学习了如何使用 Xerces DOM 解析器创建、修改和保存 XML 文档。

与往常一样，GitHub 上的[提供了示例的完整源代码。](https://web.archive.org/web/20220625222218/https://github.com/eugenp/tutorials/tree/master/xml)