# 使用 Selenium WebDriver 截图

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-selenium-screenshots>

## 1。概述

当使用 [Selenium](https://web.archive.org/web/20221129014726/https://www.selenium.dev/) 进行自动化测试时，我们经常需要对网页或网页的一部分进行截图。这可能是有用的，特别是在调试测试失败或者验证我们的应用程序行为在不同浏览器之间是一致的时候。

在这个快速教程中，**我们将看看几种使用 Selenium WebDriver 从我们的 [JUnit](/web/20221129014726/https://www.baeldung.com/tag/junit/) 测试**中捕获屏幕截图的方法。要了解更多关于硒测试的信息，请查看我们伟大的[硒指南](/web/20221129014726/https://www.baeldung.com/java-selenium-with-junit-and-testng)。

## 2。依赖性和配置

让我们从添加 Selenium 依赖项到我们的`pom.xml`开始:

```java
<dependency>
    <groupId>org.seleniumhq.selenium</groupId>
    <artifactId>selenium-java</artifactId>
    <version>3.141.59</version>
</dependency>
```

一如既往，这个工件的最新版本可以在 [Maven Central](https://web.archive.org/web/20221129014726/https://search.maven.org/classic/#search%7Cga%7C1%7Cselenium-java) 中找到。此外，最新版本的 Chrome 驱动程序可以从其[网站](https://web.archive.org/web/20221129014726/https://chromedriver.chromium.org/downloads)下载。

现在让我们配置我们的驱动程序来使用单元测试中的 Chrome:

```java
private static ChromeDriver driver;

@BeforeClass
public static void setUp() {
    System.setProperty("webdriver.chrome.driver", resolveResourcePath("chromedriver.mac"));

    Capabilities capabilities = DesiredCapabilities.chrome();
    driver = new ChromeDriver(capabilities);
    driver.manage()
      .timeouts()
      .implicitlyWait(5, TimeUnit.SECONDS);

    driver.get("http://www.google.com/");
}
```

正如我们所看到的，这是一个非常标准的 Selenium 配置，可以让我们控制本地机器上运行的 Chrome 浏览器。我们还将驱动程序在搜索页面上的元素时应该等待的时间配置为 5 秒。

最后，在运行任何测试之前，我们在当前浏览器窗口中打开一个最喜欢的网页，`www.google.com`。

## 3。对可视区域进行截图

在第一个例子中，我们将看看 Selenium 提供的开箱即用的`TakesScreenShot`接口。顾名思义，我们可以使用这个界面对可视区域进行截图。

让我们使用这个界面创建一个简单的截图方法:

```java
public void takeScreenshot(String pathname) throws IOException {
    File src = ((TakesScreenshot) driver).getScreenshotAs(OutputType.FILE);
    FileUtils.copyFile(src, new File(pathname));
} 
```

在这个简洁的方法中，我们首先使用强制转换将我们的驱动程序转换成一个`TakesScreenshot`。**然后我们可以调用`getScreenshotAs`方法，用指定的`OutputType`创建一个图像文件**。

之后，我们可以使用 [Apache Commons IO](/web/20221129014726/https://www.baeldung.com/apache-commons-io) `copyFile`方法将文件复制到任何需要的位置。相当酷！**在短短两行代码中，我们就能捕捉到截图**。

现在让我们看看如何在单元测试中使用这个方法:

```java
@Test
public void whenGoogleIsLoaded_thenCaptureScreenshot() throws IOException {
    takeScreenshot(resolveTestResourcePath("google-home.png"));

    assertTrue(new File(resolveTestResourcePath("google-home.png")).exists());
}
```

在这个单元测试中，我们在断言之前使用文件名`google-home.png` 将结果图像文件保存到我们的`test/resources`文件夹中，以查看该文件是否存在。

## 4。捕捉页面上的元素

在下一节中，我们将看一下如何捕捉页面上单个元素的屏幕截图。为此，我们将使用一个名为 [aShot](https://web.archive.org/web/20221129014726/https://github.com/pazone/ashot) 的库，一个由 Selenium 3 及其后版本原生支持的截图实用程序库。

由于 aShot 可以从 [Maven Central](https://web.archive.org/web/20221129014726/https://search.maven.org/classic/#search%7Cga%7C1%7Cashot) 获得，我们可以将它包含在我们的`pom.xml`:

```java
<dependency>
    <groupId>ru.yandex.qatools.ashot</groupId>
    <artifactId>ashot</artifactId>
    <version>1.5.4</version>
</dependency>
```

aShot 库提供了一个 [Fluent API](https://web.archive.org/web/20221129014726/https://en.wikipedia.org/wiki/Fluent_interface) ,用于配置我们到底想要如何捕捉屏幕截图。

现在，让我们看看如何从我们的一个单元测试中从 Google 主页捕获徽标:

```java
@Test
public void whenGoogleIsLoaded_thenCaptureLogo() throws IOException {
    WebElement logo = driver.findElement(By.id("hplogo"));

    Screenshot screenshot = new AShot().shootingStrategy(ShootingStrategies.viewportPasting(1000))
      .coordsProvider(new WebDriverCoordsProvider())
      .takeScreenshot(driver, logo);

    ImageIO.write(screenshot.getImage(), "jpg", new File(resolveTestResourcePath("google-logo.png")));
    assertTrue(new File(resolveTestResourcePath("google-logo.png")).exists());
}
```

我们首先使用 id `hplogo.` 在页面上找到一个`WebElement`，然后创建一个新的`AShot`实例，并设置一个内置的拍摄策略—`ShootingStrategies.viewportPasting(1000)`。**这个策略会在我们截图的时候滚动视窗，最多一秒钟**。

现在我们有了如何配置截图的策略。

当我们想要捕获页面上的特定元素时，在内部，aShot 将找到元素的大小和位置，并裁剪原始图像。为此，我们调用`coordsProvider` 方法并传递一个`WebDriverCoordsProvider` 类，该类将使用 WebDriver API 来查找任何坐标。

**注意，默认情况下，aShot 使用 jQuery 进行坐标解析。但是有些驱动对 Javascript** 有问题。

现在我们可以调用`takeScreenshot`方法，传递我们的`driver`和`logo`元素，这将依次给我们一个包含屏幕截图结果的`Screenshot`对象。和以前一样，我们通过编写一个图像文件并验证它的存在来完成我们的测试。

## 5。结论

在这个快速教程中，我们看到了两种使用 Selenium WebDriver 捕获屏幕截图的方法。

在第一种方法中，我们看到了如何直接使用 Selenium 捕获整个屏幕。然后，我们学习了如何使用一个名为 aShot 的实用程序库来捕获页面上的特定元素。

使用 aShot 的一个主要好处是不同的 web 驱动在截图时表现不同。**使用 aShot 将我们从这种复杂性中抽离出来，并给出透明的结果，而不管我们使用的是什么驱动程序**。请务必查看完整的[文档](https://web.archive.org/web/20221129014726/https://github.com/pazone/ashot)，查看所有可用的支持功能。

和往常一样，GitHub 上的[提供了这篇文章的完整源代码。](https://web.archive.org/web/20221129014726/https://github.com/eugenp/tutorials/tree/master/testing-modules/selenium-junit-testng)