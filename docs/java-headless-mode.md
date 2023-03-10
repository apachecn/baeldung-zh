# Java 无头模式

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-headless-mode>

## 1.概观

有时，我们需要**在 Java 中使用基于图形的应用程序，而不需要实际的显示器、键盘或鼠标**，比如说，在服务器或容器上。【T2

在这个简短的教程中，我们将学习 Java 的无头模式来处理这个场景。我们还将看看在无头模式下我们能做什么，不能做什么。

## 2.设置无头模式

有许多方法可以在 Java 中显式设置无头模式:

*   以编程方式将系统属性`java.awt.headless`设置为`true`
*   使用命令行参数:`java -Djava.awt.headless=true`
*   在服务器启动脚本中将`-Djava.awt.headless=true`添加到`JAVA_OPTS` 环境变量中

如果环境实际上是无头的，JVM 将会隐含地意识到它。但是，在某些场景中会有细微的差别。我们很快就会见到他们。

## 3.无头模式下的 UI 组件示例

在无头环境中运行的 UI 组件的一个典型用例是图像转换器应用程序。虽然图像处理需要图形数据，但显示器并不是真正必要的。该应用程序可以在服务器上运行，并通过网络将转换后的文件保存或发送到另一台机器上进行显示。

让我们来看看实际情况。

首先，我们将在一个`JUnit` 类中以编程方式打开无头模式:

```java
@Before
public void setUpHeadlessMode() {
    System.setProperty("java.awt.headless", "true");
} 
```

为了确保设置正确，我们可以使用`java.awt.GraphicsEnvironment` # `isHeadless`:

```java
@Test
public void whenSetUpSuccessful_thenHeadlessIsTrue() {
    assertThat(GraphicsEnvironment.isHeadless()).isTrue();
} 
```

我们应该记住，即使没有明确打开模式，上面的测试也会在无头环境中成功。

现在让我们看看简单的图像转换器:

```java
@Test
public void whenHeadlessMode_thenImagesWork() {
    boolean result = false;
    try (InputStream inStream = HeadlessModeUnitTest.class.getResourceAsStream(IN_FILE); 
      FileOutputStream outStream = new FileOutputStream(OUT_FILE)) {
        BufferedImage inputImage = ImageIO.read(inStream);
        result = ImageIO.write(inputImage, FORMAT, outStream);
    }

    assertThat(result).isTrue();
}
```

在下一个示例中，我们可以看到所有字体的信息，包括字体规格:

```java
@Test
public void whenHeadless_thenFontsWork() {
    GraphicsEnvironment ge = GraphicsEnvironment.getLocalGraphicsEnvironment();
    String fonts[] = ge.getAvailableFontFamilyNames();

    assertThat(fonts).isNotEmpty();

    Font font = new Font(fonts[0], Font.BOLD, 14);
    FontMetrics fm = (new Canvas()).getFontMetrics(font);

    assertThat(fm.getHeight()).isGreaterThan(0);
    assertThat(fm.getAscent()).isGreaterThan(0);
    assertThat(fm.getDescent()).isGreaterThan(0);
}
```

## 4.`HeadlessException`

有些组件需要外围设备，无法在无头模式下工作。在非交互环境中使用时，它们抛出一个`HeadlessException`:

```java
Exception in thread "main" java.awt.HeadlessException
	at java.awt.GraphicsEnvironment.checkHeadless(GraphicsEnvironment.java:204)
	at java.awt.Window.<init>(Window.java:536)
	at java.awt.Frame.<init>(Frame.java:420)
```

这个测试断言在无头模式下使用`Frame`确实会抛出一个`HeadlessException`:

```java
@Test
public void whenHeadlessmode_thenFrameThrowsHeadlessException() {
    assertThatExceptionOfType(HeadlessException.class).isThrownBy(() -> {
        Frame frame = new Frame();
        frame.setVisible(true);
        frame.setSize(120, 120);
    });
} 
```

作为一个经验法则，记住顶层组件如`Frame` 和`Button`总是需要一个交互环境，并且会抛出这个异常。但是，**如果没有显式设置**的无头模式，就会被抛出为不可恢复的`Error`。

## 5.在无头模式下绕过重量级组件

在这一点上，我们可能会问自己一个问题——但是如果我们有带 GUI 组件的代码可以在两种类型的环境下运行——有头的生产机器和无头的源代码分析服务器？

在上面的例子中，我们已经看到了重量级组件不能在服务器上工作，并将抛出一个异常。

所以，我们可以使用条件方法:

```java
public void FlexibleApp() {
    if (GraphicsEnvironment.isHeadless()) {
        System.out.println("Hello World");
    } else {
        JOptionPane.showMessageDialog(null, "Hello World");
    }
}
```

使用这种模式，我们可以创建一个灵活的应用程序，根据环境调整其行为。

## 6.结论

通过不同的代码示例，我们看到了 java 中无头模式的方式和原因。[这篇技术文章](https://web.archive.org/web/20220626082650/https://www.oracle.com/technical-resources/articles/javase/headless.html)提供了在无头模式下可以完成的所有工作的完整列表。

和往常一样，上面例子的源代码可以在 GitHub 的[上找到。](https://web.archive.org/web/20220626082650/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-lang-2)