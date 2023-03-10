# Docx4J 简介

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/docx4j>

## 1。概述

在本文中，我们将关注使用 [docx4j](https://web.archive.org/web/20220627085506/https://www.docx4java.org/) 库创建一个.`docx`文档。

Docx4j 是一个 Java 库，用于创建和操作 Office `OpenXML`文件——这意味着它只能处理`.docx`文件类型，而旧版本的 Microsoft Word 使用`.doc`扩展名(二进制文件)。

**请注意，从 2007 版本开始，Microsoft Office 支持`OpenXML` 格式。**

## 2。Maven 设置

要开始使用 docx4j，我们需要将所需的依赖项添加到我们的`pom.xml`中:

```java
<dependency>
    <groupId>org.docx4j</groupId>
    <artifactId>docx4j</artifactId>
    <version>3.3.5</version>
</dependency>
<dependency> 
    <groupId>javax.xml.bind</groupId>
    <artifactId>jaxb-api</artifactId>
    <version>2.1</version>
</dependency>
```

注意，我们总是可以在 [Maven 中央存储库](https://web.archive.org/web/20220627085506/https://search.maven.org/classic/#search%7Cgav%7C1%7Cg%3A%22org.docx4j%22%20AND%20a%3A%22docx4j%22)中查找最新的依赖版本。

需要`JAXB`依赖项，因为 docx4j 使用这个库来编组/解组一个`docx`文件中的 XML 部分。

## 3。创建一个 Docx 文件文档

### 3.1。文本元素和样式

让我们首先看看如何创建一个简单的`docx`文件——带有一个文本段落:

```java
WordprocessingMLPackage wordPackage = WordprocessingMLPackage.createPackage();
MainDocumentPart mainDocumentPart = wordPackage.getMainDocumentPart();
mainDocumentPart.addStyledParagraphOfText("Title", "Hello World!");
mainDocumentPart.addParagraphOfText("Welcome To Baeldung");
File exportFile = new File("welcome.docx");
wordPackage.save(exportFile); 
```

下面是生成的`welcome.docx`文件:

[![im1](img/a7f44f40d87f05b5009a6dbe5211ca62.png)](/web/20220627085506/https://www.baeldung.com/wp-content/uploads/2017/10/im1.png)

为了创建一个新的文档，我们必须使用`WordprocessingMLPackage`，它代表一个`OpenXML`格式的`docx`文件，而`MainDocumentPart`类持有一个主要`document.xml`部分的表示。

为了弄清楚，让我们解压缩`welcome.docx`文件，并打开`word/document.xml`文件，看看 XML 表示是什么样子的:

```java
<w:body>
    <w:p>
        <w:pPr>
            <w:pStyle w:val="Title"/>
        </w:pPr>
        <w:r>
            <w:t>Hello World!</w:t>
        </w:r>
    </w:p>
    <w:p>
        <w:r>
            <w:t>Welcome To Baeldung!</w:t>
        </w:r>
    </w:p>
</w:body>
```

正如我们所看到的，**每个句子由一个段落(`p` )** 内的一串(`r`)文本(`t`)表示，这就是`addParagraphOfText()`方法的用途。

`addStyledParagraphOfText()`做的比这多一点；它创建一个段落属性(`pPr`)，保存应用于该段落的样式。

简单地说，段落声明单独的运行，每个运行包含一些文本元素:

[![p-r-t](img/6ecacc7105abd9af66b842c6adca7534.png)](/web/20220627085506/https://www.baeldung.com/wp-content/uploads/2017/10/p-r-t.png)

为了创建一个好看的文档，我们需要完全控制这些元素`(paragraph, run,`和`text).`

因此，让我们来看看如何使用`runProperties` ( `RPr`)对象来样式化我们的内容:

```java
ObjectFactory factory = Context.getWmlObjectFactory();
P p = factory.createP();
R r = factory.createR();
Text t = factory.createText();
t.setValue("Welcome To Baeldung");
r.getContent().add(t);
p.getContent().add(r);
RPr rpr = factory.createRPr();       
BooleanDefaultTrue b = new BooleanDefaultTrue();
rpr.setB(b);
rpr.setI(b);
rpr.setCaps(b);
Color green = factory.createColor();
green.setVal("green");
rpr.setColor(green);
r.setRPr(rpr);
mainDocumentPart.getContent().add(p);
File exportFile = new File("welcome.docx");
wordPackage.save(exportFile);
```

结果是这样的:

[![im2a](img/fbcc14d98d2eea6c7f276db8ea7af442.png)](/web/20220627085506/https://www.baeldung.com/wp-content/uploads/2017/10/im2a.png)

在我们分别使用`createP()`、`createR()`和`createText()`创建了一个段落、一段文字和一个文本元素之后，我们声明了一个新的`runProperties`对象(`RPr`)来给文本元素添加一些样式。

`rpr`对象用于设置格式属性，粗体(`B`)、斜体(`I`)和大写(`Caps`)，这些属性使用`setRPr()`方法应用于文本运行。

### 3.2。处理图像

Docx4j 提供了一种向 Word 文档添加图像的简单方法:

```java
File image = new File("image.jpg" );
byte[] fileContent = Files.readAllBytes(image.toPath());
BinaryPartAbstractImage imagePart = BinaryPartAbstractImage
  .createImagePart(wordPackage, fileContent);
Inline inline = imagePart.createImageInline(
  "Baeldung Image (filename hint)", "Alt Text", 1, 2, false);
P Imageparagraph = addImageToParagraph(inline);
mainDocumentPart.getContent().add(Imageparagraph);
```

下面是`addImageToParagraph()`方法的实现:

```java
private static P addImageToParagraph(Inline inline) {
    ObjectFactory factory = new ObjectFactory();
    P p = factory.createP();
    R r = factory.createR();
    p.getContent().add(r);
    Drawing drawing = factory.createDrawing();
    r.getContent().add(drawing);
    drawing.getAnchorOrInline().add(inline);
    return p;
}
```

首先，我们创建了一个文件，其中包含了我们想要添加到主文档部件中的图像，然后，我们将表示图像的字节数组与`wordMLPackage`对象链接起来。

一旦创建了图像部分，我们需要使用`createImageInline(`方法创建一个`Inline`对象。

`addImageToParagraph()`方法将`Inline`对象嵌入到`Drawing`中，这样就可以将它添加到`run.`中

最后，像文本段落一样，包含图像的段落被添加到`mainDocumentPart`中。

这是生成的文档:

[![im3a](img/9124a9082928748931c7547e0450fee2.png)](/web/20220627085506/https://www.baeldung.com/wp-content/uploads/2017/10/im3a.png)

### 3.3。创建表格

Docx4j 还使得操纵表(Tbl)、行(Tr)和列(Tc)变得非常容易。

让我们看看如何创建一个 3×3 的表格并向其中添加一些内容:

```java
int writableWidthTwips = wordPackage.getDocumentModel()
  .getSections().get(0).getPageDimensions().getWritableWidthTwips();
int columnNumber = 3;
Tbl tbl = TblFactory.createTable(3, 3, writableWidthTwips/columnNumber);     
List<Object> rows = tbl.getContent();
for (Object row : rows) {
    Tr tr = (Tr) row;
    List<Object> cells = tr.getContent();
    for(Object cell : cells) {
        Tc td = (Tc) cell;
        td.getContent().add(p);
    }
}
```

给定一些行和列，`createTable()`方法创建一个新的`Tbl`对象，第三个参数指的是以缇为单位的列宽(这是一种距离度量——1/1440 英寸)。

一旦创建完毕，我们就可以迭代`tbl`对象的内容，并将`Paragraph`对象添加到每个单元格中。

让我们看看最后的结果是什么样的:

[![im4a](img/259dc725d6dd9a4eed4b1cfc6625c279.png)](/web/20220627085506/https://www.baeldung.com/wp-content/uploads/2017/10/im4a.png)

## 4。读取 Docx 文件文档

现在我们已经了解了如何使用 docx4j 创建文档，让我们看看如何读取现有的 docx 文件，并打印其内容:

```java
File doc = new File("helloWorld.docx");
WordprocessingMLPackage wordMLPackage = WordprocessingMLPackage
  .load(doc);
MainDocumentPart mainDocumentPart = wordMLPackage
  .getMainDocumentPart();
String textNodesXPath = "//w:t";
List<Object> textNodes= mainDocumentPart
  .getJAXBNodesViaXPath(textNodesXPath, true);
for (Object obj : textNodes) {
    Text text = (Text) ((JAXBElement) obj).getValue();
    String textValue = text.getValue();
    System.out.println(textValue);
}
```

在这个例子中，我们使用`load()`方法，基于现有的`helloWorld.docx`文件创建了一个`WordprocessingMLPackage`对象。

之后，我们使用一个`XPath`表达式(`//w:t`)从主文档部分获取所有文本节点。

`getJAXBNodesViaXPath()`方法返回一个`JAXBElement`对象的列表。

因此，`mainDocumentPart`对象中的所有文本元素都会在控制台中打印出来。

请注意，我们总是可以解压缩 docx 文件，以便更好地理解 XML 结构，这有助于分析问题，并更好地了解如何解决问题。

## 5。结论

在本文中，我们发现了 docx4j 如何使在 MSWord 文档上执行复杂操作变得更加容易，例如创建段落、表格、文档部分和添加图像。

代码片段一如既往地可以在 GitHub 的[上找到。](https://web.archive.org/web/20220627085506/https://github.com/eugenp/tutorials/tree/master/libraries-data-io)