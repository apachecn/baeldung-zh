# 在 Java 中使用图像

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-images>

## 1。概述

在本教程中，我们将看看一些可用的图像处理库，并执行简单的图像处理操作——加载图像并在其上绘制形状。

我们将尝试 AWT(和一点 Swing)库、ImageJ、OpenIMAJ 和 TwelveMonkeys。

## 2。AWT

AWT 是一个内置的 Java 库，它允许用户执行与显示相关的简单操作，比如创建窗口、定义按钮和监听器等等。它还包括允许用户编辑图像的方法。它不需要安装，因为它随 Java 一起提供。

### 2.1。加载图像

第一件事是从我们硬盘上保存的图片创建一个`BufferedImage`对象:

```java
String imagePath = "path/to/your/image.jpg";
BufferedImage myPicture = ImageIO.read(new File(imagePath)); 
```

### 2.2。编辑图像

要在图像上绘制形状，我们必须使用与加载图像相关的`Graphics`对象。对象封装了执行基本渲染操作所需的属性。`Graphics2D`是`Graphics`的延伸类。它提供了对二维形状的更多控制。

在这种特殊情况下，我们需要`Graphic2D`来扩展形状宽度，使其清晰可见。我们通过增加它的 s `troke`属性来实现它。然后，我们设置一种颜色，并绘制一个矩形，形状将从图像边界 10 像素:

```java
Graphics2D g = (Graphics2D) myPicture.getGraphics();
g.setStroke(new BasicStroke(3));
g.setColor(Color.BLUE);
g.drawRect(10, 10, myPicture.getWidth() - 20, myPicture.getHeight() - 20); 
```

### 2.3。显示图像

现在我们已经在我们的图像上绘制了一些东西，我们想要显示它。我们可以使用 Swing 库对象来实现。首先，我们创建代表文本或/和图像显示区域的`JLabel`对象:

```java
JLabel picLabel = new JLabel(new ImageIcon(myPicture));
```

然后将我们的`JLabel`添加到`JPanel`，我们可以将它视为基于 Java 的 GUI 的`<div></div>`:

```java
JPanel jPanel = new JPanel();
jPanel.add(picLabel);
```

最后，我们将所有内容添加到屏幕上显示的窗口`JFrame`。我们必须设置大小，这样我们就不必在每次运行程序时都扩展这个窗口:

```java
JFrame f = new JFrame();
f.setSize(new Dimension(myPicture.getWidth(), myPicture.getHeight()));
f.add(jPanel);
f.setVisible(true);
```

## 3。ImageJ

ImageJ 是一个基于 Java 的软件，用于处理图像。它有相当多的插件，这里有。我们将只使用 API，因为我们想自己执行处理。

这是一个非常强大的库，比 Swing 和 AWT 要好，因为它的创建目的是图像处理，而不是 GUI 操作。插件包含许多免费使用的算法，当我们想学习图像处理并快速看到结果时，这是一件好事，而不是解决铺设在 IP 算法下的数学和优化问题。

### 3.1。Maven 依赖关系

要开始使用 ImageJ，只需在项目的`pom.xml`文件中添加一个依赖项:

```java
<dependency>
    <groupId>net.imagej</groupId>
    <artifactId>ij</artifactId>
    <version>1.51h</version>
</dependency>
```

你会在 [Maven 资源库](https://web.archive.org/web/20220628145241/https://search.maven.org/classic/#search%7Cgav%7C1%7Cg%3A%22net.imagej%22%20AND%20a%3A%22ij%22)中找到最新版本。

### 3.2。加载图像

要加载图像，您需要使用来自`IJ`类的`openImage()` 静态方法:

```java
ImagePlus imp = IJ.openImage("path/to/your/image.jpg");
```

### 3.3。编辑图像

要编辑一个图像，我们将不得不使用来自附加到我们的`ImagePlus` 对象的`ImageProcessor`对象的方法。把它想象成 AWT 中关于`Graphics`的对象:

```java
ImageProcessor ip = imp.getProcessor();
ip.setColor(Color.BLUE);
ip.setLineWidth(4);
ip.drawRect(10, 10, imp.getWidth() - 20, imp.getHeight() - 20);
```

### 3.4。显示图像

你只需要调用`ImagePlus` 对象的`show()`方法:

```java
imp.show();
```

## 4。OpenIMAJ

OpenIMAJ 是一组 Java 库，不仅专注于计算机视觉和视频处理，还专注于机器学习、音频处理、Hadoop 工作等等。OpenIMAJ 项目的所有部分都可以在这里的“模块”下找到。我们只需要图像处理部分。

### 4.1。Maven 依赖关系

要开始使用 OpenIMAJ，只需在项目的 *pom.xml* 文件中添加一个依赖项:

```java
<dependency>
    <groupId>org.openimaj</groupId>
    <artifactId>core-image</artifactId>
    <version>1.3.5</version>
</dependency>
```

你可以在这里找到最新的版本。

### 4.1。加载图像

要加载图像，使用`ImageUtilities.readMBF()`方法:

```java
MBFImage image = ImageUtilities.readMBF(new File("path/to/your/image.jpg")); 
```

MBF 代表多波段浮点图像(本例中为 RGB，但这不是表示颜色的唯一方式)。

### 4.2。编辑图像

要绘制矩形，我们需要定义它的形状，它是由 4 个点组成的多边形(左上、左下、右下、右上):

```java
Point2d tl = new Point2dImpl(10, 10);
Point2d bl = new Point2dImpl(10, image.getHeight() - 10);
Point2d br = new Point2dImpl(image.getWidth() - 10, image.getHeight() - 10);
Point2d tr = new Point2dImpl(image.getWidth() - 10, 10);
Polygon polygon = new Polygon(Arrays.asList(tl, bl, br, tr));
```

您可能已经注意到，在图像处理中，Y 轴是反的。定义形状后，我们需要绘制它:

```java
image.drawPolygon(polygon, 4, new Float[] { 0f, 0f, 255.0f });
```

绘制方法取 3 个参数:形状、线条粗细和用`Float`数组表示的 RGB 通道值。

### 4.3。显示图像

我们需要使用`DisplayUtilities`:

```java
DisplayUtilities.display(image);
```

## 5。`TwelveMonkeys` `ImageIO`

`TwelveMonkeys` `ImageIO`库旨在作为 Java `ImageIO` API 的扩展，支持更多的格式。

大多数情况下，代码看起来与内置的 Java 代码一样，但是在添加了必要的依赖项之后，它可以使用其他图像格式。

默认情况下，Java 只支持这五种图像格式:`JPEG`、 `PNG`、 `BMP`、 `WEBMP`、 `GIF`。

如果我们试图处理不同格式的图像文件，我们的应用程序将无法读取它，并在访问`BufferedImage` 变量时抛出一个`NullPointerException`。

`TwelveMonkeys`增加对以下格式的支持:`PNM`、`PSD`、`TIFF`、`HDR`、`IFF`、`PCX`、`PICT`、`SGI`、`TGA`、`ICNS`、`ICO`、`CUR`、`Thumbs.db`、`SVG`、`WMF`。

**为了处理特定格式的图像，我们需要添加相应的依赖关系**，比如 [imageio-jpeg](https://web.archive.org/web/20220628145241/https://search.maven.org/classic/#search%7Cga%7C1%7Ca%3A%22imageio-jpeg%22%20AND%20g%3A%22com.twelvemonkeys.imageio%22) 或者 [imageio-tiff](https://web.archive.org/web/20220628145241/https://search.maven.org/classic/#search%7Cga%7C1%7Ca%3A%22imageio-tiff%22%20AND%20g%3A%22com.twelvemonkeys.imageio%22) 。

您可以在 [TwelveMonkeys](https://web.archive.org/web/20220628145241/https://github.com/haraldk/TwelveMonkeys) 文档中找到依赖项的完整列表。

让我们创建一个读取`.ico`图像的例子。代码看起来与`AWT`部分相同，除了我们将打开一个不同的图像:

```java
String imagePath = "path/to/your/image.ico";
BufferedImage myPicture = ImageIO.read(new File(imagePath));
```

为了让这个例子工作，我们需要添加包含对`.ico`图像支持的`TwelveMonkeys`依赖项，即 [imageio-bmp](https://web.archive.org/web/20220628145241/https://search.maven.org/classic/#search%7Cga%7C1%7Ca%3A%22imageio-bmp%22%20AND%20g%3A%22com.twelvemonkeys.imageio%22) 依赖项，以及 [imageio-core](https://web.archive.org/web/20220628145241/https://search.maven.org/classic/#search%7Cga%7C1%7Ca%3A%22imageio-core%22%20AND%20g%3A%22com.twelvemonkeys.imageio%22) 依赖项:

```java
<dependency>
    <groupId>com.twelvemonkeys.imageio</groupId>
    <artifactId>imageio-bmp</artifactId>
    <version>3.3.2</version>
</dependency>
<dependency>
    <groupId>com.twelvemonkeys.imageio</groupId>
    <artifactId>imageio-core</artifactId>
    <version>3.3.2</version>
</dependency>
```

而这就是全部！**内置的`ImageIO` Java API 在运行时自动加载插件。**现在我们的项目也将使用 `.ico`图像。

## 6。总结

已经向您介绍了 4 个可以帮助您处理图像的库。更进一步，你可能想寻找一些图像处理算法，如提取边缘，增强对比度，使用过滤器或人脸检测。

出于这些目的，最好开始学习 ImageJ 或 OpenIMAJ。两者都很容易包含在项目中，并且在图像处理方面比 AWT 强大得多。

这些图像处理的例子可以在[GitHub 项目](https://web.archive.org/web/20220628145241/https://github.com/eugenp/tutorials/tree/master/image-processing)中找到。