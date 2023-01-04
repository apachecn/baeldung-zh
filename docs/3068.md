# 使用 JavaScript 点击 Selenium 中的元素

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-selenium-javascript>

## 1.介绍

在这个简短的教程中，我们将通过一个简单的例子来了解如何使用 JavaScript 点击并元素化 [Selenium](https://web.archive.org/web/20220814111612/https://www.selenium.dev/) WebDriver。

在我们的演示中，我们将使用 [JUnit 和 Selenium](/web/20220814111612/https://www.baeldung.com/java-selenium-with-junit-and-testng) 来打开`https://baeldung.com` 并搜索“Selenium”文章。

## 2.属国

首先，我们将`[selenium-java](https://web.archive.org/web/20220814111612/https://mvnrepository.com/artifact/org.seleniumhq.selenium/selenium-java)`和`[junit](https://web.archive.org/web/20220814111612/https://mvnrepository.com/artifact/junit/junit)`依赖项添加到我们项目的`pom.xml`中:

```
<dependency>
    <groupId>org.seleniumhq.selenium</groupId>
    <artifactId>selenium-java</artifactId>
    <version>3.141.59</version>
</dependency>
<dependency>
    <groupId>junit</groupId>
    <artifactId>junit</artifactId>
    <version>4.13</version>
    <scope>test</scope>
</dependency>
```

## 3.配置

接下来，我们需要配置 WebDriver。在这个例子中，在下载了它的最新版本之后，我们将使用它的 Chrome 实现:

```
@Before
public void setUp() {
    System.setProperty("webdriver.chrome.driver", new File("src/main/resources/chromedriver.mac").getAbsolutePath());
    driver = new ChromeDriver();
}
```

我们使用一个用`@Before`标注的方法在每次测试前进行初始设置。在里面我们设置了定义 [chrome 驱动](https://web.archive.org/web/20220814111612/https://chromedriver.chromium.org/downloads)位置的`webdriver.chrome.driver`属性。之后，我们实例化了`WebDriver`对象。

测试完成后，我们应该关闭浏览器窗口。我们可以通过将`driver.close()`语句放在用`@After`注释的方法中来实现。这确保了即使测试失败它也会被执行:

```
@After
public void cleanUp() {
    driver.close();
}
```

## 4.打开浏览器

现在，我们可以创建一个测试用例来完成我们的第一步——打开网站:

```
@Test
public void whenSearchForSeleniumArticles_thenReturnNotEmptyResults() {
    driver.get("https://baeldung.com");
    String title = driver.getTitle();
    assertEquals("Baeldung | Java, Spring and Web Development tutorials", title);
}
```

这里，我们使用`driver.get()`方法来加载网页。接下来，我们验证它的标题，以确保我们在正确的地方。

## 5.使用 JavaScript 单击元素

**Selenium 附带了一个方便的`WebElement#click`方法**，该方法调用给定元素上的点击事件。**但是在某些情况下点击动作是不可能的。**

一个例子是，如果我们想点击一个禁用的元素。在这种情况下，`WebElement#click`抛出一个`IllegalStateException`。相反，我们可以使用 Selenium 的 JavaScript 支持。

为此，我们首先需要的是`JavascriptExecutor`。由于我们使用的是`ChromeDriver` 实现，我们可以简单地将它转换成我们需要的:

```
JavascriptExecutor executor = (JavascriptExecutor) driver;
```

得到了`JavascriptExecutor`之后，我们就可以使用它的`executeScript`方法了。参数是脚本本身和一组脚本参数。在我们的例子中，我们对第一个参数调用 click 方法:

```
executor.executeScript("arguments[0].click();", element);
```

现在，让我们把它放在一个方法中，我们称之为`clickElement`:

```
private void clickElement(WebElement element) {
    JavascriptExecutor executor = (JavascriptExecutor) driver;
    executor.executeScript("arguments[0].click();", element);
}
```

最后，我们可以将它添加到我们的测试中:

```
@Test
public void whenSearchForSeleniumArticles_thenReturnNotEmptyResults() {
    // ... load https://baeldung.com
    WebElement searchButton = driver.findElement(By.className("nav--menu_item_anchor"));
    clickElement(searchButton);

    WebElement searchInput = driver.findElement(By.id("search"));
    searchInput.sendKeys("Selenium");

    WebElement seeSearchResultsButton = driver.findElement(By.cssSelector(".btn-search"));
    clickElement(seeSearchResultsButton);
}
```

## 6.不可点击的元素

使用 JavaScript 单击元素时最常见的问题之一是在元素可单击之前执行 click 脚本。在这种情况下，点击动作不会发生，但代码会继续执行。

为了解决这个问题，我们必须在点击可用之前暂停执行。我们可以使用`WebDriverWait#until`等待按钮被渲染。

第一，`W` `ebDriverWait`对象需要两个参数；驱动程序和超时:

```
WebDriverWait wait = new WebDriverWait(driver, 5000); 
```

然后，我们调用`until`，给出期望的`elementToBeClickable`条件:

```
wait.until(ExpectedConditions.elementToBeClickable(By.className("nav--menu_item_anchor"))); 
```

一旦成功返回，我们知道我们可以继续:

```
WebElement searchButton = driver.findElement(By.className("nav--menu_item_anchor"));
clickElement(searchButton);
```

有关更多可用的条件方法，请参考官方文件。

## 7.结论

在本教程中，我们学习了如何使用 JavaScript 点击 Selenium 中的元素。和往常一样，这篇文章的来源可以在 GitHub 上找到[。](https://web.archive.org/web/20220814111612/https://github.com/eugenp/tutorials/tree/master/testing-modules/selenium-junit-testng)