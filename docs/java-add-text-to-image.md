# 在 Java 中向图像添加文本

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-add-text-to-image>

## 1。概述

有时我们需要给一张图片或一组图片添加一些文本。使用图像编辑工具可以很容易地手动完成这项工作。但是，当我们想要以同样的方式向大量图片添加相同的文本时，以编程方式实现这一点将非常有用。

在这个快速教程中，我们将学习如何使用 Java 给图片添加一些文本。

## 2。给图像添加文本

为了读取图像并添加一些文本，我们可以使用不同的类。在接下来的部分中，我们将看到几个选项。

### 2.1。`ImagePlus`和`ImageProcessor`

首先，让我们看看**如何使用 [ImageJ 库](https://web.archive.org/web/20221105185655/https://imagej.nih.gov/ij/developer/api/index.html)中可用的 [`ImagePlus`](https://web.archive.org/web/20221105185655/https://imagej.nih.gov/ij/source/ij/ImagePlus.java) 和 [`ImageProcessor`](https://web.archive.org/web/20221105185655/https://imagej.nih.gov/ij/developer/api/ij/ij/process/ImageProcessor.html)** 类。为了使用这个库，我们需要在我们的项目中包含这个依赖项:

```java
<dependency>
    <groupId>net.imagej</groupId>
    <artifactId>ij</artifactId>
    <version>1.51h</version>
</dependency>
```

为了读取图像，我们将使用`openImage`静态方法。该方法的结果将使用一个`ImagePlus`对象存储在内存中:

```java
ImagePlus image = IJ.openImage(path);
```

一旦我们将图像加载到内存中，让我们使用类`ImageProcessor`向它添加一些文本:

```java
Font font = new Font("Arial", Font.BOLD, 18);

ImageProcessor ip = image.getProcessor();
ip.setColor(Color.GREEN);
ip.setFont(font);
ip.drawString(text, 0, 20);
```

使用这段代码，我们所做的是在图像的左上角添加指定的绿色文本。注意，我们使用`drawString`方法的第二个和第三个参数来设置位置，这两个参数分别表示从左边和上边开始的像素数。

### 2.2。`BufferedImage`和`Graphics`

接下来，我们将看到**如何使用 [`BufferedImage`](https://web.archive.org/web/20221105185655/https://docs.oracle.com/en/java/javase/11/docs/api/java.desktop/java/awt/image/BufferedImage.html) 和 [`Graphics`](https://web.archive.org/web/20221105185655/https://docs.oracle.com/en/java/javase/11/docs/api/java.desktop/java/awt/Graphics.html#drawString(java.text.AttributedCharacterIterator,int,int))** 类来实现相同的结果。Java 的标准版本包括这些类，所以不需要额外的库。

和我们使用`ImageJ`的`openImage`一样，我们将使用`ImageIO`中可用的`read`方法:

```java
BufferedImage image = ImageIO.read(new File(path));
```

一旦我们将图像加载到内存中，让我们使用类`Graphics`向它添加一些文本:

```java
Font font = new Font("Arial", Font.BOLD, 18);

Graphics g = image.getGraphics();
g.setFont(font);
g.setColor(Color.GREEN);
g.drawString(text, 0, 20);
```

正如我们所看到的，这两种选择在使用方式上非常相似。在这种情况下，方法`drawString`的第二个和第三个参数的指定方式与我们对`ImageProcessor`方法的指定方式相同。

### 2.3。根据`AttributedCharacterIterator`绘制

`Graphics`中可用的方法`drawString`允许我们**使用 [`AttributedCharacterIterator`](https://web.archive.org/web/20221105185655/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/text/AttributedCharacterIterator.html)** 打印文本。这意味着我们可以使用带有相关属性的文本，而不是使用普通的`String`。让我们看一个例子:

```java
Font font = new Font("Arial", Font.BOLD, 18);

AttributedString attributedText = new AttributedString(text);
attributedText.addAttribute(TextAttribute.FONT, font);
attributedText.addAttribute(TextAttribute.FOREGROUND, Color.GREEN);

Graphics g = image.getGraphics();
g.drawString(attributedText.getIterator(), 0, 20);
```

这种打印文本的方式让我们有机会直接将格式与`String`相关联，这比我们想改变格式时改变`Graphics`对象属性更简洁。

## 3。文本对齐

既然我们已经学习了如何在图像的左上角添加一个简单的文本，现在让我们看看如何在特定位置添加这个**文本。**

### 3.1。居中文本

我们要处理的第一种对齐方式是**将文本**居中。为了动态地设置我们想要书写文本的正确位置，我们需要找出一些信息:

*   图像尺寸
*   字体大小

这个信息很容易获得。在图像大小的情况下，可以通过`BufferedImage`对象的方法`getWidth`和`getHeight`来访问该数据。另一方面，为了获得与字体大小相关的数据，我们需要使用对象`FontMetrics`。

让我们来看一个例子，我们计算文本的正确位置并绘制它:

```java
Graphics g = image.getGraphics();

FontMetrics metrics = g.getFontMetrics(font);
int positionX = (image.getWidth() - metrics.stringWidth(text)) / 2;
int positionY = (image.getHeight() - metrics.getHeight()) / 2 + metrics.getAscent();

g.drawString(attributedText.getIterator(), positionX, positionY);
```

### 3.2。文本右下方对齐

**我们将要看到的下一种对齐方式是右下角的**。在这种情况下，我们需要动态地获得正确的位置:

```java
int positionX = (image.getWidth() - metrics.stringWidth(text));
int positionY = (image.getHeight() - metrics.getHeight()) + metrics.getAscent();
```

### 3.3。位于左上角的文本

最后，让我们看看**如何在左上角**打印我们的文本:

```java
int positionX = 0;
int positionY = metrics.getAscent();
```

其余的排列可以从我们看到的三条中推断出来。

## 4。根据图像调整文本大小

当我们在图像中绘制文本时，我们可能会发现文本超出了图像的大小。为了解决这个问题，我们必须**根据图像大小调整我们使用的字体**的大小。

首先，我们需要使用基本字体获得文本的预期宽度和高度。为了实现这一点，我们将利用类 [`FontMetrics`](https://web.archive.org/web/20221105185655/https://docs.oracle.com/en/java/javase/11/docs/api/java.desktop/java/awt/FontMetrics.html) 、`[GlyphVector](https://web.archive.org/web/20221105185655/https://docs.oracle.com/en/java/javase/11/docs/api/java.desktop/java/awt/font/GlyphVector.html),`和 [`Shape`](https://web.archive.org/web/20221105185655/https://docs.oracle.com/en/java/javase/11/docs/api/java.desktop/java/awt/Shape.html) 。

```java
FontMetrics ruler = graphics.getFontMetrics(baseFont);
GlyphVector vector = baseFont.createGlyphVector(ruler.getFontRenderContext(), text);

Shape outline = vector.getOutline(0, 0);

double expectedWidth = outline.getBounds().getWidth();
double expectedHeight = outline.getBounds().getHeight(); 
```

下一步是检查是否有必要调整字体的大小。为此，让我们比较一下文本的预期大小和图像的大小:

```java
boolean textFits = image.getWidth() >= expectedWidth && image.getHeight() >= expectedHeight;
```

最后，如果我们的文本不适合图像，我们必须减小字体大小。为此，我们将使用方法`deriveFont`:

```java
double widthBasedFontSize = (baseFont.getSize2D()*image.getWidth())/expectedWidth;
double heightBasedFontSize = (baseFont.getSize2D()*image.getHeight())/expectedHeight;

double newFontSize = widthBasedFontSize < heightBasedFontSize ? widthBasedFontSize : heightBasedFontSize;
newFont = baseFont.deriveFont(baseFont.getStyle(), (float)newFontSize);
```

请注意，我们需要基于宽度和高度获得新的字体大小，并应用它们中最小的一个。

## 5。总结

在本文中，我们看到了如何使用不同的方法在图像中书写文本。

我们还学习了如何根据图像大小和字体属性动态地获取我们想要打印文本的位置。

最后，我们看到了如何调整文本的字体大小，以防它超过我们正在绘制的图像的大小。

和往常一样，GitHub 上的[提供了这篇文章的完整源代码。](https://web.archive.org/web/20221105185655/https://github.com/eugenp/tutorials/tree/master/image-processing)