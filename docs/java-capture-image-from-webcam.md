# 用 Java 从网络摄像头捕捉图像

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-capture-image-from-webcam>

## 1.概观

通常，Java 不提供对计算机硬件的简单访问。这就是为什么我们可能会发现使用 Java 很难访问网络摄像头。

在本教程中，我们将探索一些 Java 库，这些库允许我们通过访问网络摄像头来捕捉图像。

## 2.JavaCV

首先，我们将考察 [`javacv`](https://web.archive.org/web/20220524115457/https://github.com/bytedeco/javacv) 库。这是 **[Bytedeco](https://web.archive.org/web/20220524115457/http://bytedeco.org/) 的[对 OpenCV](/web/20220524115457/https://www.baeldung.com/java-opencv) 计算机视觉库**的 Java 实现。

让我们将最新的 [`javacv-platform`](https://web.archive.org/web/20220524115457/https://search.maven.org/search?q=g:org.bytedeco%20a:javacv-platform) Maven 依赖添加到我们的`pom.xml`:

```java
<dependency>
    <groupId>org.bytedeco</groupId>
    <artifactId>javacv-platform</artifactId>
    <version>1.5.5</version>
</dependency>
```

类似地，当使用 Gradle 时，我们可以在`build.gradle`文件中添加`javacv-platform`依赖项:

```java
compile group: 'org.bytedeco', name: 'javacv-platform', version: '1.5.5'
```

现在我们已经准备好了设置，让我们使用**[`OpenCVFrameGrabber`](https://web.archive.org/web/20220524115457/http://bytedeco.org/javacv/apidocs/org/bytedeco/javacv/OpenCVFrameGrabber.html)类来访问网络摄像头并捕捉一帧**:

```java
FrameGrabber grabber = new OpenCVFrameGrabber(0);
grabber.start();
Frame frame = grabber.grab();
```

这里，**我们将设备号作为`0`传递，指向系统的默认摄像头**。但是，如果我们有多个可用的摄像机，那么第二个摄像机在 1 处可用，第三个在 2 处可用，依此类推。

然后，我们可以使用`OpenCVFrameConverter`将捕获的帧转换成图像。同样，我们将使用`opencv_imgcodecs`类的`cvSaveImage`方法**保存图像:**

```java
OpenCVFrameConverter.ToIplImage converter = new OpenCVFrameConverter.ToIplImage();
IplImage img = converter.convert(frame);
opencv_imgcodecs.cvSaveImage("selfie.jpg", img); 
```

最后，我们可以使用 [`CanvasFrame`](https://web.archive.org/web/20220524115457/http://bytedeco.org/javacv/apidocs/org/bytedeco/javacv/CanvasFrame.html) 类来显示捕获的帧:

```java
CanvasFrame canvas = new CanvasFrame("Web Cam");
canvas.showImage(frame); 
```

让我们来看一个完整的解决方案，它访问网络摄像头，捕捉图像，在一个框架中显示图像，并在两秒钟后自动关闭框架:

```java
CanvasFrame canvas = new CanvasFrame("Web Cam");
canvas.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);

FrameGrabber grabber = new OpenCVFrameGrabber(0);
OpenCVFrameConverter.ToIplImage converter = new OpenCVFrameConverter.ToIplImage();

grabber.start();
Frame frame = grabber.grab();

IplImage img = converter.convert(frame);
cvSaveImage("selfie.jpg", img);

canvas.showImage(frame);

Thread.sleep(2000);

canvas.dispatchEvent(new WindowEvent(canvas, WindowEvent.WINDOW_CLOSING));
```

## 3.`webcam-capture`

接下来，我们将研究 `webcam-capture`库，它通过支持多种捕捉框架来允许使用网络摄像头。

首先，让我们将最新的 [`webcam-capture`](https://web.archive.org/web/20220524115457/https://search.maven.org/search?q=g:com.github.sarxos%20a:webcam-capture) Maven 依赖添加到我们的`pom.xml`:

```java
<dependency>
    <groupId>com.github.sarxos</groupId>
    <artifactId>webcam-capture</artifactId>
    <version>0.3.12</version>
</dependency>
```

或者，我们可以在 Gradle 项目的`build.gradle`中添加`webcam-capture`:

```java
compile group: 'com.github.sarxos', name: 'webcam-capture', version: '0.3.12'
```

然后，让我们编写一个简单的例子来使用 [`Webcam`](https://web.archive.org/web/20220524115457/https://javadoc.io/static/com.github.sarxos/webcam-capture/0.3.12/com/github/sarxos/webcam/Webcam.html) 类捕捉图像:

```java
Webcam webcam = Webcam.getDefault();
webcam.open();

BufferedImage image = webcam.getImage();

ImageIO.write(image, ImageUtils.FORMAT_JPG, new File("selfie.jpg"));
```

在这里，我们访问默认的网络摄像头来捕捉图像，然后将图像保存到一个文件中。

或者，我们可以使用 [`WebcamUtils`](https://web.archive.org/web/20220524115457/https://javadoc.io/static/com.github.sarxos/webcam-capture/0.3.12/com/github/sarxos/webcam/WebcamUtils.html) 类来捕捉一个图像:

```java
WebcamUtils.capture(webcam, "selfie.jpg");
```

同样，我们可以使用[`WebcamPanel`](https://web.archive.org/web/20220524115457/https://javadoc.io/static/com.github.sarxos/webcam-capture/0.3.12/com/github/sarxos/webcam/WebcamPanel.html)[类在](/web/20220524115457/https://www.baeldung.com/java-images#3-displaying-an-image) 帧中显示捕获的图像:

```java
Webcam webcam = Webcam.getDefault();
webcam.setViewSize(WebcamResolution.VGA.getSize());

WebcamPanel panel = new WebcamPanel(webcam);
panel.setImageSizeDisplayed(true);

JFrame window = new JFrame("Webcam");
window.add(panel);
window.setResizable(true);
window.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);
window.pack();
window.setVisible(true);
```

在这里，我们将 **`VGA`设置为网络摄像头的视图大小，创建了 [`JFrame`](https://web.archive.org/web/20220524115457/https://docs.oracle.com/en/java/javase/11/docs/api/java.desktop/javax/swing/JFrame.html) 对象，并将`WebcamPanel`组件添加到帧**中。

## 4.马文框架

最后，我们将探索 Marvin 框架来访问网络摄像头和捕捉图像。

像往常一样，我们将最新的 [`marvin`](https://web.archive.org/web/20220524115457/https://search.maven.org/search?q=g:com.github.downgoon%20a:marvin) 依赖项添加到我们的`pom.xml`:

```java
<dependency>
    <groupId>com.github.downgoon</groupId>
    <artifactId>marvin</artifactId>
    <version>1.5.5</version>
</dependency>
```

或者，对于 Gradle 项目，我们将在`build.gradle`文件中添加`marvin`依赖项:

```java
compile group: 'com.github.downgoon', name: 'marvin', version: '1.5.5'
```

现在设置已经准备好了，让我们**使用 [`MarvinJavaCVAdapter`](https://web.archive.org/web/20220524115457/http://marvinproject.sourceforge.net/javadoc/marvin/video/MarvinJavaCVAdapter.html) 类通过为设备号**提供 0 来连接到默认的网络摄像头:

```java
MarvinVideoInterface videoAdapter = new MarvinJavaCVAdapter();
videoAdapter.connect(0);
```

接下来，我们可以使用`getFrame`方法捕获帧，然后我们将使用 [`MarvinImageIO`](https://web.archive.org/web/20220524115457/http://marvinproject.sourceforge.net/javadoc/marvin/io/MarvinImageIO.html) 类的`saveImage`方法**保存图像:**

```java
MarvinImage image = videoAdapter.getFrame();
MarvinImageIO.saveImage(image, "selfie.jpg");
```

同样，我们可以使用 [`MarvinImagePanel`](https://web.archive.org/web/20220524115457/http://marvinproject.sourceforge.net/javadoc/marvin/gui/MarvinImagePanel.html) 类来显示一帧图像:

```java
MarvinImagePanel imagePanel = new MarvinImagePanel();
imagePanel.setImage(image);

imagePanel.setSize(800, 600);
imagePanel.setVisible(true);
```

## 5.结论

在这篇短文中，我们研究了几个 Java 库，它们提供了对网络摄像头的简单访问。

首先，我们探索了提供 OpenCV 项目的 Java 实现的`javacv-platform`库。然后，我们看到了使用网络摄像头捕捉图像的`webcam-capture`库的示例实现。最后，我们看一下使用 Marvin 框架捕捉图像的简单示例。

像往常一样，所有的代码实现都可以在 GitHub 上获得[。](https://web.archive.org/web/20220524115457/https://github.com/eugenp/tutorials/tree/master/image-processing)