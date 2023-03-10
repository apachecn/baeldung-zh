# 使用 Java 截图

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-taking-screenshots>

## 1.介绍

在本教程中，我们将看看 Java 中截图的几种不同方式。

## 2.使用`Robot`截图

在我们的第一个例子中，我们将截取一个主屏幕的截图。

为此，我们将使用来自`Robot`类的`createScreenCapture()`方法。它将一个`Rectangle`作为参数来设置屏幕截图的边界，并返回一个`BufferedImage` 对象。`BufferedImage`可以进一步用来创建一个图像文件:

```java
@Test
public void givenMainScreen_whenTakeScreenshot_thenSaveToFile() throws Exception {
    Rectangle screenRect = new Rectangle(Toolkit.getDefaultToolkit().getScreenSize());
    BufferedImage capture = new Robot().createScreenCapture(screenRect);

    File imageFile = new File("single-screen.bmp");
    ImageIO.write(capture, "bmp", imageFile );
    assertTrue(imageFile .exists());
}
```

通过使用`getScreenSize()`方法，可以通过`Toolkit`类访问屏幕的尺寸。在有多个屏幕的系统上，默认情况下使用主显示器。

在将屏幕捕获到`BufferedImage,`中后，我们可以用`ImageIO.write()`将其写入文件。为此，我们需要两个额外的参数。图像格式和图像文件本身。在我们的例子中，**我们使用了。`bmp`格式，但也有人喜欢。`png, .jpg` 或 `.gif`也可。**

## 3.拍摄多个屏幕的截图

**也可以一次对多个显示器进行截图**。就像前面的例子一样，我们可以使用来自`Robot`类的`createScreenCapture()`方法。但是这次截图的边界需要覆盖所有需要的屏幕。

为了获得所有的显示，我们将使用`GraphicsEnvironment`类及其`getScreenDevices()`方法。

接下来，我们将获取每个单独屏幕的边界，并创建一个适合所有屏幕的`Rectangle`:

```java
@Test
public void givenMultipleScreens_whenTakeScreenshot_thenSaveToFile() throws Exception {
    GraphicsEnvironment ge = GraphicsEnvironment.getLocalGraphicsEnvironment();
    GraphicsDevice[] screens = ge.getScreenDevices();

    Rectangle allScreenBounds = new Rectangle();
    for (GraphicsDevice screen : screens) {
        Rectangle screenBounds = screen.getDefaultConfiguration().getBounds();
        allScreenBounds.width += screenBounds.width;
        allScreenBounds.height = Math.max(allScreenBounds.height, screenBounds.height);
    }

    BufferedImage capture = new Robot().createScreenCapture(allScreenBounds);
    File imageFile = new File("all-screens.bmp");
    ImageIO.write(capture, "bmp", imageFile);
    assertTrue(imageFile.exists());
}
```

当迭代显示时，我们总是合计宽度并选择一个最大高度，因为屏幕将水平连接。

更进一步，我们需要保存截图。和前面的例子一样，我们可以使用`ImageIO.write()`方法。

## 4.获取给定 GUI 组件的屏幕截图

我们还可以截取给定 UI 组件的屏幕截图。

**通过`getBounds()`方法可以轻松获取尺寸，因为每个组件都知道其尺寸和位置。**

在这种情况下，我们不打算使用`Robot` API。相反，我们将使用来自`Component`类的`paint()`方法，该方法将内容直接提取到`BufferedImage`中:

```java
@Test
public void givenComponent_whenTakeScreenshot_thenSaveToFile(Component component) throws Exception {
    Rectangle componentRect = component.getBounds();
    BufferedImage bufferedImage = new BufferedImage(componentRect.width, componentRect.height, BufferedImage.TYPE_INT_ARGB);
    component.paint(bufferedImage.getGraphics());

    File imageFile = new File("component-screenshot.bmp");
    ImageIO.write(bufferedImage, "bmp", imageFile );
    assertTrue(imageFile.exists());
}
```

得到组件的边界后，我们需要为此创建`BufferedImage.`，我们需要宽度、高度和图像类型。在这种情况下，我们使用的是指 8 位彩色图像的`BufferedImage.TYPE_INT_ARGB`。

然后我们继续调用`paint()`方法来填充`BufferedImage`，和前面的例子一样，我们用`ImageIO.write()`方法将它保存到一个文件中。

## 5.结论

在本教程中，我们学习了几种使用 Java 截图的方法。

像往常一样，本教程中所有示例的源代码都可以在 GitHub 上的[处获得。](https://web.archive.org/web/20221128061531/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-os)