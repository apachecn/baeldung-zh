# 如何在 Java 中打印屏幕

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/print-screen-in-java>

## 1。概述

当您需要在桌面上执行打印屏幕操作时，键盘上有一个内置的“PrntScr”按钮来帮助您。有时候这就够了。

但是，当您需要以编程方式执行该操作时，问题就出现了。简单来说，你可能需要用 Java 将当前的截图保存为图片文件。

让我们来看看如何做到这一点。

## 2。`Robot`班

Java `**java.awt.Robot**` class 是我们将要使用的主要 API。该调用包含一个名为'`createScreenCapture`'的方法，当一个特定的形状通过时，该方法会截取一个屏幕截图:

```java
robot.createScreenCapture(rectangle); 
```

由于上面的方法返回了一个`java.awt.image.BufferedImage`实例，您所要做的就是使用 **`javax.imageio.ImageIO`** 实用程序类将检索到的图像写到一个文件中。

## 3。捕获并保存图像文件

用于图像捕获和保存的 Java 代码如下:

```java
public void getScreenshot(int timeToWait) throws Exception {
    Rectangle rec = new Rectangle(
      Toolkit.getDefaultToolkit().getScreenSize());
    Robot robot = new Robot();
    BufferedImage img = robot.createScreenCapture(rectangle);

    ImageIO.write(img, "jpg", setupFileNamePath());
}
```

在这里，可以通过将所需的大小设置为`java.awt.Rectangle` 实例来捕获屏幕的一部分。然而，在上面的例子中，通过设置当前屏幕尺寸，它已经被设置为捕获全屏。

## 4。结论

在本教程中，我们快速浏览了 Java 中打印屏幕的用法。上面例子的源代码可以在[GitHub 项目](https://web.archive.org/web/20220121025227/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-os)中找到。