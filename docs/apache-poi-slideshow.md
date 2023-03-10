# 用 Java 创建 MS PowerPoint 演示文稿

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/apache-poi-slideshow>

## 1。简介

在本文中，我们将了解如何使用 [Apache POI](https://web.archive.org/web/20221208143856/https://poi.apache.org/) 创建演示文稿。

这个库让我们有可能创建 PowerPoint 演示文稿，阅读现有的演示文稿，并修改它们的内容。

## 2。Maven 依赖关系

首先，我们需要将以下依赖项添加到我们的`pom.xml`中:

```java
<dependency>
    <groupId>org.apache.poi</groupId>
    <artifactId>poi</artifactId>
    <version>3.17</version>
</dependency>
<dependency>
    <groupId>org.apache.poi</groupId>
    <artifactId>poi-ooxml</artifactId>
    <version>3.17</version>
</dependency>
```

两个[库](https://web.archive.org/web/20221208143856/https://search.maven.org/classic/#search%7Cga%7C1%7Cg%3A%22org.apache.poi%22%20AND%20(a%3A%22poi%22%20OR%20a%3A%22poi-ooxml%22))的最新版本都可以从 Maven Central 下载。

## 3。阿帕奇兴趣点

**[Apache POI](https://web.archive.org/web/20221208143856/https://poi.apache.org/) 库支持`.ppt`和`.pptx`文件**，它为 Powerpoint’97(-2007)文件格式提供 HSLF 实现，为 PowerPoint 2007 OOXML 文件格式提供 XSLF 实现。

由于两种实现都不存在一个公共接口，**我们必须记住在处理较新的`.pptx` 文件格式**时使用`XMLSlideShow`、`XSLFSlide`和`XSLFTextShape`类。

当需要使用旧的`.ppt` 格式时，使用`HSLFSlideShow`、`HSLFSlide` 和`HSLFTextParagraph` 类。

在我们的示例中，我们将使用新的`.pptx`文件格式，我们要做的第一件事是创建一个新的演示文稿，添加一张幻灯片(可能使用预定义的布局)并保存它。

一旦这些操作清楚了，我们就可以开始处理图像、文本和表格了。

### 3.1。创建新的演示文稿

让我们首先创建新的演示文稿:

```java
XMLSlideShow ppt = new XMLSlideShow();
ppt.createSlide();
```

### 3.2。添加新幻灯片

当向演示文稿添加新幻灯片时，我们也可以选择从预定义的布局创建它。为了实现这一点，我们首先必须检索保存布局的`XSLFSlideMaster`(第一个是默认的母版):

```java
XSLFSlideMaster defaultMaster = ppt.getSlideMasters().get(0);
```

现在，我们可以检索`XSLFSlideLayout`并在创建新幻灯片时使用它:

```java
XSLFSlideLayout layout 
  = defaultMaster.getLayout(SlideLayout.TITLE_AND_CONTENT);
XSLFSlide slide = ppt.createSlide(layout);
```

让我们看看如何在模板中填充占位符:

```java
XSLFTextShape titleShape = slide.getPlaceholder(0);
XSLFTextShape contentShape = slide.getPlaceholder(1);
```

记住，每个模板都有自己的占位符，即`XSLFAutoShape`子类的实例，每个模板的占位符数量可能不同。

让我们看看如何快速检索幻灯片中的所有占位符:

```java
for (XSLFShape shape : slide.getShapes()) {
    if (shape instanceof XSLFAutoShape) {
        // this is a template placeholder
    }
}
```

### 3.3。保存演示文稿

一旦我们创建了幻灯片，下一步就是保存它:

```java
FileOutputStream out = new FileOutputStream("powerpoint.pptx");
ppt.write(out);
out.close();
```

## 4。使用对象

既然我们已经看到了如何创建一个新的演示文稿，添加一个幻灯片(使用或不使用预定义的模板)并保存它，我们可以开始添加文本、图像、链接和表格。

先说正文。

### 4.1。正文

当在演示文稿中处理文本时，如在 MS PowerPoint 中，我们必须在幻灯片中创建文本框，添加一个段落，然后将文本添加到该段落中:

```java
XSLFTextBox shape = slide.createTextBox();
XSLFTextParagraph p = shape.addNewTextParagraph();
XSLFTextRun r = p.addNewTextRun();
r.setText("Baeldung");
r.setFontColor(Color.green);
r.setFontSize(24.);
```

当配置`XSLFTextRun`时，可以通过选择字体系列以及文本是否应该是粗体、斜体或下划线来定制其样式。

### 4.2。超链接

向演示文稿添加文本时，有时添加超链接会很有用。

一旦我们创建了`XSLFTextRun`对象，我们现在可以添加一个链接:

```java
XSLFHyperlink link = r.createHyperlink();
link.setAddress("http://www.baeldung.com");
```

### 4.3。图像

我们还可以添加图像:

```java
byte[] pictureData = IOUtils.toByteArray(
  new FileInputStream("logo-leaf.png"));

XSLFPictureData pd
  = ppt.addPicture(pictureData, PictureData.PictureType.PNG);
XSLFPictureShape picture = slide.createPicture(pd);
```

但是，**如果没有适当的配置，图像将被放置在幻灯片**的左上角。要正确放置它，我们必须配置它的锚点:

```java
picture.setAnchor(new Rectangle(320, 230, 100, 92));
```

`XSLFPictureShape`接受一个`Rectangle`作为锚点，这允许我们用前两个参数配置 x/y 坐标，用后两个参数配置图像的宽度/高度。

### 4.4。列表

演示文稿中的文本通常以列表的形式表示，不管有没有编号。

现在，让我们定义一系列要点:

```java
XSLFTextShape content = slide.getPlaceholder(1);
XSLFTextParagraph p1 = content.addNewTextParagraph();
p1.setIndentLevel(0);
p1.setBullet(true);
r1 = p1.addNewTextRun();
r1.setText("Bullet");
```

类似地，我们可以定义一个编号列表:

```java
XSLFTextParagraph p2 = content.addNewTextParagraph();
p2.setBulletAutoNumber(AutoNumberingScheme.alphaLcParenRight, 1);
p2.setIndentLevel(1);
XSLFTextRun r2 = p2.addNewTextRun();
r2.setText("Numbered List Item - 1");
```

如果我们使用多个列表，定义`indentLevel`来实现条目的适当缩进总是很重要的。

### 4.5。表格

表格是演示文稿中的另一个关键对象，在我们想要显示数据时非常有用。

让我们首先创建一个表:

```java
XSLFTable tbl = slide.createTable();
tbl.setAnchor(new Rectangle(50, 50, 450, 300));
```

现在，我们可以添加一个标题:

```java
int numColumns = 3;
XSLFTableRow headerRow = tbl.addRow();
headerRow.setHeight(50);

for (int i = 0; i < numColumns; i++) {
    XSLFTableCell th = headerRow.addCell();
    XSLFTextParagraph p = th.addNewTextParagraph();
    p.setTextAlign(TextParagraph.TextAlign.CENTER);
    XSLFTextRun r = p.addNewTextRun();
    r.setText("Header " + (i + 1));
    tbl.setColumnWidth(i, 150);
}
```

完成标题后，我们可以向表格中添加行和单元格来显示数据:

```java
for (int rownum = 1; rownum < numRows; rownum++) {
    XSLFTableRow tr = tbl.addRow();
    tr.setHeight(50);

    for (int i = 0; i < numColumns; i++) {
        XSLFTableCell cell = tr.addCell();
        XSLFTextParagraph p = cell.addNewTextParagraph();
        XSLFTextRun r = p.addNewTextRun();
        r.setText("Cell " + (i*rownum + 1));
    }
}
```

使用表格时，需要提醒的是，可以自定义每个单元格的边框和背景。

## 5。更改演示文稿

并不总是在做幻灯片的时候，我们必须创建一个新的，但是我们必须改变一个已经存在的。

让我们看一下我们在上一节中创建的那个，然后我们可以开始修改它:

[![presentation 1](img/4537c7555f0779714914c7462de4ea8d.png)](/web/20221208143856/https://www.baeldung.com/wp-content/uploads/2017/12/presentation-1.jpg)

### 5.1。阅读演示文稿

阅读演示非常简单，可以使用接受`FileInputStream`的`XMLSlideShow`重载构造函数来完成:

```java
XMLSlideShow ppt = new XMLSlideShow(
  new FileInputStream("slideshow.pptx"));
```

### 5.2。改变幻灯片顺序

在我们的演示文稿中添加幻灯片时，最好将它们按正确的顺序排列，以保证幻灯片的正确流动。

如果没有发生这种情况，可以重新安排幻灯片的顺序。让我们看看如何将第四张幻灯片移动到第二张幻灯片:

```java
List<XSLFSlide> slides = ppt.getSlides();

XSLFSlide slide = slides.get(3);
ppt.setSlideOrder(slide, 1);
```

### 5.3。删除幻灯片

也可以从演示文稿中删除幻灯片。

让我们看看如何删除第四张幻灯片:

```java
ppt.removeSlide(3);
```

## 6。结论

这个快速教程从 Java 的角度说明了如何使用`Apache POI` API 来读写 PowerPoint 文件。

一如既往，本文的完整源代码可以在 GitHub 的[中找到。](https://web.archive.org/web/20221208143856/https://github.com/eugenp/tutorials/tree/master/apache-poi-2)