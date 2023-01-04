# 修复 Selenium WebDriver 可执行路径错误

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-selenium-webdriver-path-error>

## 1.概观

在本教程中，我们将看看常见的 Selenium 错误:“`The path to the driver executable must be set by the webdriver.chrome.driver system property`”。这个错误阻止 Selenium 启动浏览器。这是由不完整的配置造成的。我们将学习如何通过手动或自动的正确设置来解决这个问题。

## 2.错误的原因

Selenium 在使用之前需要一些设置步骤，比如设置 WebDriver 的路径。**如果不配置 WebDriver 的路径，就无法运行它来控制浏览器，就会得到一个`java.lang.IllegalStateException`。**

让我们来看看导致此错误的不完整设置:

```
WebDriver driver = new ChromeDriver();
```

有了这个语句，我们想要创建一个新的`ChromeDriver`实例，但是由于我们没有提供到`WebDriver`的路径，Selenium 无法运行它，并且它失败并显示错误“`java.lang.IllegalStateException: The path to the driver executable must be set by the webdriver.chrome.driver system property`”。

要解决此问题，我们需要执行正确的设置。我们可以使用专用库手动或自动完成这项工作。

## 3.手动设置

首先，我们需要为我们的浏览器下载正确的 web 驱动程序。根据我们的浏览器下载正确的版本是很重要的，因为，否则，在运行它时可能会出现不可预见的问题。

可以从以下网站下载正确的 web 驱动程序:

*   chrome:[https://chromedriver.chromium.org/downloads](https://web.archive.org/web/20221110072459/https://chromedriver.chromium.org/downloads)
*   edge:[https://developer . Microsoft . com/en-us/Microsoft-edge/tools/web driver/](https://web.archive.org/web/20221110072459/https://developer.microsoft.com/en-us/microsoft-edge/tools/webdriver/)
*   火狐:[https://github.com/mozilla/geckodriver/releases](https://web.archive.org/web/20221110072459/https://github.com/mozilla/geckodriver/releases)

然后 Selenium 需要下载的驱动程序的路径，这样它就可以运行它来控制浏览器。**我们可以用一个[系统属性](/web/20221110072459/https://www.baeldung.com/java-system-get-property-vs-system-getenv)** 来设置驱动的路径。每个浏览器的属性的键都不同:

*   Chrome: `webdriver.chrome.driver`
*   火狐:`webdriver.gecko.driver`
*   边缘:`webdriver.edge.driver`

让我们来看看 Chrome 的手动设置。我们设置先前下载的 WebDriver 的路径，然后创建一个`ChromeDriver`实例:

```
WebDriver driver;

void setupChromeDriver() {
    System.setProperty("webdriver.chrome.driver", "src/test/resources/chromedriver.exe");
    driver = new ChromeDriver();
    options();
}

void options() {
    driver.manage().window().maximize();
}
```

路径可以是相对的，也可以是绝对的。此外，我们可以设置各种设置，比如在上面的例子中最大化浏览器窗口。

其他浏览器的设置非常相似。正如我们在下面看到的，我们只需要替换驱动程序设置方法，并为各个驱动程序设置路径:

```
void setupGeckoDriver() {
    System.setProperty("webdriver.gecko.driver", "src/test/resources/geckodriver.exe");
    driver = new FirefoxDriver();
    options();
}

void setupEdgeDriver() {
    System.setProperty("webdriver.edge.driver", "src/test/resources/msedgedriver.exe");
    driver = new EdgeDriver();
    options();
}
```

为了验证设置，我们可以对进行一个小检查:

```
String TITLE_XPATH = "//a[@href='/']";
String URL = "https://www.baeldung.com";

@Test
void givenChromeDriver_whenNavigateToBaeldung_thenFindTitleIsSuccessful() {
    setupChromeDriver();
    driver.get(URL);
    final WebElement title = driver.findElement(By.xpath(TITLE_XPATH));

    assertEquals("Baeldung", title.getAttribute("title"));
}
```

如果设置仍然无效，我们需要确保网络驱动的路径是正确的。

## 4.自动设置

手动设置可能很麻烦，因为我们需要手动下载特定的 web 驱动程序。我们还需要确保使用正确的版本。如果安装的浏览器启用了自动更新，这可能需要我们定期用更新版本替换 WebDriver。

为了克服这一点，我们可以利用 [WebDriverManager](https://web.archive.org/web/20221110072459/https://bonigarcia.dev/webdrivermanager/) 库，它在我们每次运行时为我们处理这些任务。

首先，我们需要将[依赖项](https://web.archive.org/web/20221110072459/https://search.maven.org/search?q=g:io.github.bonigarcia%20AND%20a:webdrivermanager)添加到我们的`pom.xml`中:

```
<dependency>
    <groupId>io.github.bonigarcia</groupId>
    <artifactId>webdrivermanager</artifactId>
    <version>5.3.0</version>
</dependency>
```

使用该库的设置非常简单，只需要一行代码:

```
WebDriver driver;

void setupChromeDriver() {
    WebDriverManager.chromedriver().setup();
    driver = new ChromeDriver();
    options();
}

void options() {
    driver.manage().window().maximize();
}
```

在设置过程中，`WebDriverManager`会检查安装的浏览器版本，并自动下载正确的网络驱动程序版本。它设置系统属性，然后运行浏览器。

为其他浏览器调整设置也很简单:

```
void setupGeckoDriver() {
    WebDriverManager.firefoxdriver().setup();
    driver = new FirefoxDriver();
    options();
}

void setupEdgeDriver() {
    WebDriverManager.edgedriver().setup();
    driver = new EdgeDriver();
    options();
}
```

同样，我们可以通过在上的一个小测试来验证这个设置:

```
String TITLE_XPATH = "//a[@href='/']";
String URL = "https://www.baeldung.com";

@Test
void givenChromeDriver_whenNavigateToBaeldung_thenFindTitleIsSuccessful() {
    setupChromeDriver();
    driver.get(URL);
    final WebElement title = driver.findElement(By.xpath(TITLE_XPATH));

    assertEquals("Baeldung", title.getAttribute("title"));
}
```

## 5.结论

在本文中，我们看到了导致 Selenium 错误"`The path to the driver executable must be set by the webdriver.chrome.driver system property`"的原因，以及如何修复它。

我们可以进行手动设置，但这会导致一些维护工作。使用 WebDriverManager 库的自动设置减少了使用 Selenium 时的维护。

和往常一样，所有这些例子的实现都可以在 GitHub 上找到[。](https://web.archive.org/web/20221110072459/https://github.com/eugenp/tutorials/tree/master/testing-modules/selenium-junit-testng/)