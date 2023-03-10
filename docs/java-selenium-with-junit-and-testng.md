# 使用 JUnit / TestNG 的 Selenium 指南

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-selenium-with-junit-and-testng>

## 1。简介

这篇文章快速、实用地介绍了如何使用 [Selenium](https://web.archive.org/web/20220815045426/http://www.seleniumhq.org/) 以及如何使用 [JUnit](https://web.archive.org/web/20220815045426/http://junit.org/) 和 [TestNG](https://web.archive.org/web/20220815045426/http://testng.org/) 编写测试。

## 2。硒整合

在本节中，我们将从一个简单的场景开始——打开一个浏览器窗口，导航到一个给定的 URL，并在页面上查找一些想要的内容。

### 2.1。Maven 依赖关系

在`pom.xml`文件中，添加以下依赖关系:

```java
<dependency>
    <groupId>org.seleniumhq.selenium</groupId>
    <artifactId>selenium-java</artifactId>
    <version>3.4.0</version>
</dependency>
```

最新版本可以在 [Maven 中央存储库](https://web.archive.org/web/20220815045426/https://search.maven.org/classic/#search%7Cga%7C1%7Cselenium-java)中找到。

### 2.2。硒配置

首先，创建一个名为`SeleniumConfig`的新 Java 类文件:

```java
public class SeleniumConfig {

    private WebDriver driver;

    //...

}
```

假设我们使用的是 Selenium 3.x 版本，我们必须使用名为`webdriver.gecko.driver`的系统属性指定可执行文件`GeckoDriver`的路径(基于您的操作系统)。GeckoDriver 的最新版本可以从[Github GeckoDriver Releases](https://web.archive.org/web/20220815045426/https://github.com/mozilla/geckodriver/releases)下载。

现在让我们初始化构造函数中的`WebDriver`，我们还将设置 5 秒钟作为`WebDriver`等待页面上元素出现的超时时间:

```java
public SeleniumConfig() {
    Capabilities capabilities = DesiredCapabilities.firefox();
    driver = new FirefoxDriver(capabilities);
    driver.manage().timeouts().implicitlyWait(5, TimeUnit.SECONDS);
}

static {
    System.setProperty("webdriver.gecko.driver", findFile("geckodriver.mac"));
}

static private String findFile(String filename) {
   String paths[] = {"", "bin/", "target/classes"};
   for (String path : paths) {
      if (new File(path + filename).exists())
          return path + filename;
   }
   return "";
}
```

这个配置类包含了一些方法，我们现在将忽略它们，但是我们将在本系列的第二部分中看到更多关于它们的内容。

接下来，我们需要实现一个`SeleniumExample`类:

```java
public class SeleniumExample {

    private SeleniumConfig config;
    private String url = "http://www.baeldung.com/";

    public SeleniumExample() {
        config = new SeleniumConfig();
        config.getDriver().get(url);
    }

    // ...
}
```

在这里，我们将初始化`SeleniumConfig`并设置想要导航到的 URL。类似地，我们将实现一个简单的 API 来关闭浏览器并获取页面标题:

```java
public void closeWindow() {
    this.config.getDriver().close();
}

public String getTitle() {
    return this.config.getDriver().getTitle();
}
```

为了导航到 baeldung.com 的 About 部分，我们需要创建一个`closeOverlay()`方法来检查和关闭主页加载时的覆盖。此后，我们使用`getAboutBaeldungPage()`方法导航到 About Baeldung 页面:

```java
public void getAboutBaeldungPage() {
    closeOverlay();
    clickAboutLink();
    clickAboutUsLink();
}

private void closeOverlay() {
    List<WebElement> webElementList = this.config.getDriver()
      .findElements(By.tagName("a"));
    if (webElementList != null) {
       webElementList.stream()
         .filter(webElement -> "Close".equalsIgnoreCase(webElement.getAttribute("title")))
         .filter(WebElement::isDisplayed)
         .findAny()
         .ifPresent(WebElement::click);
    }
}

private void clickAboutLink() {
    Actions actions = new Actions(config.getDriver());
    WebElement aboutElement = this.config.getDriver()
        .findElement(By.id("menu-item-6138"));

    actions.moveToElement(aboutElement).perform();
}

private void clickAboutUsLink() {
    WebElement element = this.config.getDriver()
        .findElement(By.partialLinkText("About Baeldung."));
    element.click();
}
```

我们可以检查显示的页面上是否有所需的信息:

```java
public boolean isAuthorInformationAvailable() {
    return this.config.getDriver()
        .getPageSource()
        .contains("Hey ! I'm Eugen");
}
```

接下来，我们将使用 JUnit 和 TestNG 测试这个类。

## 3。使用 JUnit

让我们创建一个新的测试类`SeleniumWithJUnitLiveTest:`

```java
public class SeleniumWithJUnitLiveTest {

    private static SeleniumExample seleniumExample;
    private String expectedTitle = "About Baeldung | Baeldung";

    // more code goes here...

}
```

我们将使用来自`org.junit.BeforeClass`的`**@**BeforeClass`注释来进行初始设置。在这个`setUp()`方法中，我们将初始化`SeleniumExample`对象:

```java
@BeforeClass
public static void setUp() {
    seleniumExample = new SeleniumExample();
}
```

同样，当我们的测试用例完成时，我们应该关闭新打开的浏览器。我们将用`@AfterClass`注释来做这件事——当测试用例执行完成时，清理设置:

```java
@AfterClass
public static void tearDown() {
    seleniumExample.closeWindow();
}
```

请注意我们的`SeleniumExample`成员变量上的`static`修饰符——因为我们需要在`setUp()`和`tearDown()` 静态方法中使用这个变量——`@BeforeClass`和`@AfterClass`只能在静态方法上调用。

最后，我们可以创建完整的测试:

```java
@Test
public void whenAboutBaeldungIsLoaded_thenAboutEugenIsMentionedOnPage() {
    seleniumExample.getAboutBaeldungPage();
    String actualTitle = seleniumExample.getTitle();

    assertNotNull(actualTitle);
    assertEquals(expectedTitle, actualTitle);
    assertTrue(seleniumExample.isAuthorInformationAvailable());
}
```

这个测试方法断言网页的标题不是`null`，而是按照预期设置的。除此之外，我们检查页面是否包含预期的信息。

当测试运行时，它只是在 Firefox 中打开 URL，然后在网页标题和内容得到验证后关闭它。

## 4。带测试

现在让我们使用 TestNG 来运行我们的测试用例/套件。

注意，如果您使用的是 Eclipse，TestNG 插件可以从 [Eclipse Marketplace](https://web.archive.org/web/20220815045426/https://marketplace.eclipse.org/) 下载并安装。

首先，让我们创建一个新的测试类:

```java
public class SeleniumWithTestNGLiveTest {

    private SeleniumExample seleniumExample;
    private String expectedTitle = "About Baeldung | Baeldung";

    // more code goes here...

}
```

我们将使用来自`org.testng.annotations.BeforeSuite`的`@BeforeSuite`注释来实例化我们的`SeleniumExample class`。`setUp()`方法将在测试套件被激活之前启动:

```java
@BeforeSuite
public void setUp() {
    seleniumExample = new SeleniumExample();
}
```

类似地，一旦测试套件完成，我们将使用来自`org.testng.annotations.AfterSuite`的`@AfterSuite`注释来关闭我们打开的浏览器:

```java
@AfterSuite
public void tearDown() {
    seleniumExample.closeWindow();
}
```

最后，让我们实现我们的测试:

```java
@Test
public void whenAboutBaeldungIsLoaded_thenAboutEugenIsMentionedOnPage() {
    seleniumExample.getAboutBaeldungPage();
    String actualTitle = seleniumExample.getTitle();

    assertNotNull(actualTitle);
    assertEquals(expectedTitle, actualTitle);
    assertTrue(seleniumExample.isAuthorInformationAvailable());
}
```

在测试套件成功完成之后，我们在项目的`test-output`文件夹中找到 HTML 和 XML 报告。这些报告总结了测试结果。

## 5。结论

在这篇简短的文章中，我们重点介绍了如何用 JUnit 和 TestNG 编写 Selenium 3 测试。

和往常一样，这篇文章的来源可以在 GitHub 上找到。