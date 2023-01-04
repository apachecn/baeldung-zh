# 使用 Apache POI 的 Java 中的 Microsoft 字处理

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-microsoft-word-with-apache-poi>

## 1。概述

Apache POI 是一个 Java 库，用于处理基于 Office Open XML 标准(OOXML)和微软 OLE2 复合文档格式(OLE 2)的各种文件格式。

本教程重点介绍 [Apache POI 对最常用的 Office 文件格式 Microsoft Word](https://web.archive.org/web/20220122151303/https://poi.apache.org/) 的支持。它介绍了格式化和生成 MS Word 文件所需的步骤，以及如何解析该文件。

## 2。Maven 依赖关系

Apache POI 处理 MS Word 文件所需的唯一依赖项是:

```java
<dependency>
    <groupId>org.apache.poi</groupId>
    <artifactId>poi-ooxml</artifactId>
    <version>3.15</version>
</dependency>
```

请点击[此处](https://web.archive.org/web/20220122151303/https://search.maven.org/classic/#search%7Cga%7C1%7Cg%3A%22org.apache.poi%22%20AND%20a%3A%22poi-ooxml%22)获取该神器的最新版本。

## 3。准备工作

现在，让我们来看一下用来帮助生成 MS Word 文件的一些元素。

### 3.1。资源文件

我们将收集三个文本文件的内容，并将它们写入一个名为`rest-with-spring.docx`的 MS Word 文件。

此外，`logo-leaf.png`文件用于将图像插入到新文件中。所有这些文件都存在于类路径中，由几个静态变量表示:

```java
public static String logo = "logo-leaf.png";
public static String paragraph1 = "poi-word-para1.txt";
public static String paragraph2 = "poi-word-para2.txt";
public static String paragraph3 = "poi-word-para3.txt";
public static String output = "rest-with-spring.docx";
```

对于那些好奇的人来说，资源库中这些资源文件的内容(其链接在本教程的最后一节中给出)是从网站上的[本课程页面](/web/20220122151303/https://www.baeldung.com/rest-with-spring-course?utm_source=blog&utm_medium=web&utm_content=menu&utm_campaign=rws)中摘录的。

### 3.2。助手方法

由用于生成 MS Word 文件的逻辑组成的主方法(将在下一节中描述)利用了帮助器方法:

```java
public String convertTextFileToString(String fileName) {
    try (Stream<String> stream 
      = Files.lines(Paths.get(ClassLoader.getSystemResource(fileName).toURI()))) {

        return stream.collect(Collectors.joining(" "));
    } catch (IOException | URISyntaxException e) {
        return null;
    }
}
```

该方法提取位于类路径上的文本文件中包含的内容，该文件的名称是传入的`String`参数。然后，它连接这个文件中的行并返回连接的`String`。

## 4。MS Word 文件生成

本节给出了如何格式化和生成 Microsoft Word 文件的说明。在处理文件的任何部分之前，我们需要有一个`XWPFDocument`实例:

```java
XWPFDocument document = new XWPFDocument();
```

### 4.1。格式化标题和副标题

为了创建标题，我们需要首先实例化`XWPFParagraph`类，并在新对象上设置对齐:

```java
XWPFParagraph title = document.createParagraph();
title.setAlignment(ParagraphAlignment.CENTER);
```

段落的内容需要包装在一个`XWPFRun`对象中。我们可以配置这个对象来设置一个文本值及其相关的样式:

```java
XWPFRun titleRun = title.createRun();
titleRun.setText("Build Your REST API with Spring");
titleRun.setColor("009933");
titleRun.setBold(true);
titleRun.setFontFamily("Courier");
titleRun.setFontSize(20);
```

人们应该能够从它们的名字中推断出 set 方法的用途。

以类似的方式，我们创建一个包含字幕的`XWPFParagraph`实例:

```java
XWPFParagraph subTitle = document.createParagraph();
subTitle.setAlignment(ParagraphAlignment.CENTER);
```

让我们也格式化副标题:

```java
XWPFRun subTitleRun = subTitle.createRun();
subTitleRun.setText("from HTTP fundamentals to API Mastery");
subTitleRun.setColor("00CC44");
subTitleRun.setFontFamily("Courier");
subTitleRun.setFontSize(16);
subTitleRun.setTextPosition(20);
subTitleRun.setUnderline(UnderlinePatterns.DOT_DOT_DASH);
```

`setTextPosition`方法设置字幕和后续图像之间的距离，而`setUnderline`决定下划线模式。

请注意，我们对标题和副标题的内容进行了硬编码，因为这些语句太短，不适合使用辅助方法。

### 4.2。插入图像

图像也需要包装在一个`XWPFParagraph`实例中。我们希望图像水平居中，并放在副标题下，因此下面的代码片段必须放在上面给出的代码下面:

```java
XWPFParagraph image = document.createParagraph();
image.setAlignment(ParagraphAlignment.CENTER);
```

以下是如何设置此图像与其下方文本之间的距离:

```java
XWPFRun imageRun = image.createRun();
imageRun.setTextPosition(20);
```

从类路径上的文件中取出一个图像，然后插入到具有指定尺寸的 MS Word 文件中:

```java
Path imagePath = Paths.get(ClassLoader.getSystemResource(logo).toURI());
imageRun.addPicture(Files.newInputStream(imagePath),
  XWPFDocument.PICTURE_TYPE_PNG, imagePath.getFileName().toString(),
  Units.toEMU(50), Units.toEMU(50));
```

### 4.3。格式化段落

下面是我们如何用从`poi-word-para1.txt`文件中提取的内容创建第一段:

```java
XWPFParagraph para1 = document.createParagraph();
para1.setAlignment(ParagraphAlignment.BOTH);
String string1 = convertTextFileToString(paragraph1);
XWPFRun para1Run = para1.createRun();
para1Run.setText(string1);
```

显然，段落的创建类似于标题或副标题的创建。这里唯一的区别是使用了 helper 方法，而不是硬编码的字符串。

以类似的方式，我们可以使用文件`poi-word-para2.txt`和`poi-word-para3.txt`中的内容创建另外两个段落:

```java
XWPFParagraph para2 = document.createParagraph();
para2.setAlignment(ParagraphAlignment.RIGHT);
String string2 = convertTextFileToString(paragraph2);
XWPFRun para2Run = para2.createRun();
para2Run.setText(string2);
para2Run.setItalic(true);

XWPFParagraph para3 = document.createParagraph();
para3.setAlignment(ParagraphAlignment.LEFT);
String string3 = convertTextFileToString(paragraph3);
XWPFRun para3Run = para3.createRun();
para3Run.setText(string3);
```

这三个段落的创建几乎相同，除了一些样式如对齐或斜体。

### 4.4。正在生成 MS Word 文件

现在我们准备从`document`变量向内存中写出一个 Microsoft Word 文件:

```java
FileOutputStream out = new FileOutputStream(output);
document.write(out);
out.close();
document.close();
```

本节中的所有代码片段都包装在一个名为`handleSimpleDoc`的方法中。

## 5。解析和测试

本节概述了 MS Word 文件的解析和结果验证。

### 5.1。准备工作

我们在测试类中声明了一个静态字段:

```java
static WordDocument wordDocument;
```

该字段用于引用类的一个实例，该实例包含第 3 节和第 4 节中显示的所有代码片段。

在解析和测试之前，我们需要初始化上面声明的静态变量，并通过调用`handleSimpleDoc`方法在当前工作目录中生成`rest-with-spring.docx`文件:

```java
@BeforeClass
public static void generateMSWordFile() throws Exception {
    WordTest.wordDocument = new WordDocument();
    wordDocument.handleSimpleDoc();
}
```

让我们进入最后一步:解析 MS Word 文件并验证结果。

### 5.2。解析 MS Word 文件并验证

首先，我们从项目目录中给定的 MS Word 文件中提取内容，并将内容存储在`XWPFParagraph`的`List`中:

```java
Path msWordPath = Paths.get(WordDocument.output);
XWPFDocument document = new XWPFDocument(Files.newInputStream(msWordPath));
List<XWPFParagraph> paragraphs = document.getParagraphs();
document.close();
```

接下来，让我们确保标题的内容和样式与我们之前设置的相同:

```java
XWPFParagraph title = paragraphs.get(0);
XWPFRun titleRun = title.getRuns().get(0);

assertEquals("Build Your REST API with Spring", title.getText());
assertEquals("009933", titleRun.getColor());
assertTrue(titleRun.isBold());
assertEquals("Courier", titleRun.getFontFamily());
assertEquals(20, titleRun.getFontSize());
```

为了简单起见，我们只验证文件其他部分的内容，忽略样式。对其风格的验证类似于我们对标题所做的工作:

```java
assertEquals("from HTTP fundamentals to API Mastery",
  paragraphs.get(1).getText());
assertEquals("What makes a good API?", paragraphs.get(3).getText());
assertEquals(wordDocument.convertTextFileToString
  (WordDocument.paragraph1), paragraphs.get(4).getText());
assertEquals(wordDocument.convertTextFileToString
  (WordDocument.paragraph2), paragraphs.get(5).getText());
assertEquals(wordDocument.convertTextFileToString
  (WordDocument.paragraph3), paragraphs.get(6).getText());
```

现在我们可以确信,`rest-with-spring.docx`文件的创建已经成功。

## 6。结论

本教程介绍了 Apache POI 对 Microsoft Word 格式的支持。它经历了生成 MS Word 文件并验证其内容所需的步骤。

所有这些例子和代码片段的实现都可以在 GitHub 项目中找到。