# 用 Selenium 处理浏览器标签

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-handle-browser-tabs-selenium>

## 1.概观

在本教程中，我们将看看如何使用 Selenium 处理浏览器标签。有些情况下，点击链接或按钮会在新标签页打开页面。在这种情况下，我们必须正确处理标签以继续我们的测试。本教程涵盖了在新标签页中打开页面、在标签页之间切换以及关闭标签页。对于我们的例子，我们将使用[https://testpages.herokuapp.com](https://web.archive.org/web/20230103152530/https://testpages.herokuapp.com/)。

## 2.设置

基于 [`WebDriver`设置](/web/20230103152530/https://www.baeldung.com/java-selenium-webdriver-path-error)，我们将创建一个`SeleniumTestBase`类来处理`WebDriver`的设置和拆卸。我们的测试类将扩展这个类。我们还将为 Selenium 的选项卡处理定义一个助手类。这个助手类将包含打开、切换和关闭标签的方法。这些方法将在下面的章节中展示和解释。我们将在`SeleniumTestBase`中初始化该助手类的一个实例。所有示例都使用 JUnit5。

初始化是在`init() `方法中完成的，需要用`@BeforeAll`进行注释:

```
@BeforeAll
public static void init() {
    setupChromeDriver();
}
```

在所有测试之后，teardown 或 cleanup 方法将关闭整个浏览器:

```
@AfterAll
public static void cleanup() {
    if (driver != null) {
        driver.quit();
    }
}
```

## 3.基础

### 3.1.链接目标属性

我们需要处理选项卡的一个典型情况是，当点击一个链接或按钮时，一个新的选项卡被打开。目标属性设置为`_blank`的网站上的链接将在新标签中打开，例如:

```
<a href="https://www.baeldung.com/" target="_blank">Baeldung.com</a>
```

这样的链接在新的标签页中打开目标，例如[https://www.baeldung.com](/web/20230103152530/https://www.baeldung.com/)。

### 3.2.窗户把手

在 Selenium 中，每个选项卡都有一个窗口句柄，它是唯一的字符串。对于 Chrome 标签，字符串以`CDwindow,`开头，后跟 32 个十六进制字符，例如`CDwindow-CDE9BEF919431FDAA0FC9CB7EBBD4E1A`。**当切换到特定标签页时，我们需要窗口句柄。**因此，我们需要在测试期间存储窗口句柄。

## 4.标签处理

### 4.1.打开标签

**有了 Selenium > = 4.0，不用 Javascript 或者浏览器热键就可以打开新标签页。**`WebDriver`提供了以下方法来打开一个新的标签页:

```
driver.switchTo().newWindow(WindowType.TAB);
```

如前一节所述，存在与目标属性集的链接。在这些情况下，我们不需要自己打开标签页，但是我们需要注意切换和关闭它们。尽管这样的链接会在新标签页中打开页面，但它不会切换到新标签页。我们需要自己解决这个问题。一种方法是将单击链接之前的窗口句柄与单击链接之后的窗口句柄进行比较。应该有一个新的窗口句柄代表新打开的选项卡。

我们可以用下面的`WebDriver`方法来检索窗口句柄集合和活动标签页的窗口句柄:

```
driver.getWindowHandles();
driver.getWindowHandle()
```

在这些方法的帮助下，我们可以在`TabHelper`类中实现一个助手方法。它会在新标签页中打开一个链接，并切换到该标签页。只有在打开了新标签页的情况下，才会执行标签页切换。

```
String openLinkAndSwitchToNewTab(By link) {
    String windowHandle = driver.getWindowHandle();
    Set<String> windowHandlesBefore = driver.getWindowHandles();

    driver.findElement(link).click();
    Set<String> windowHandlesAfter = driver.getWindowHandles();
    windowHandlesAfter.removeAll(windowHandlesBefore);

    Optional<String> newWindowHandle = windowHandlesAfter.stream().findFirst();
    newWindowHandle.ifPresent(s -> driver.switchTo().window(s));

    return windowHandle;
}
```

查找窗口句柄是通过在点击链接前后检索窗口句柄来完成的。单击之前的窗口句柄集合将在单击之后从窗口句柄集合中移除。结果将是新选项卡的窗口句柄或一个空集。该方法还返回切换前选项卡的窗口句柄。

### 4.2.切换标签

每当我们打开多个选项卡时，我们需要手动在它们之间切换。我们可以使用下面的语句切换到一个特定的选项卡，其中`destinationWindowHandle`表示我们要切换到的选项卡的窗口句柄:

```
driver.switchTo().window(destinationWindowHandle)
```

可以实现一个助手方法，该方法将目标窗口句柄作为参数，并切换到相应的选项卡。此外，该方法在切换前返回活动选项卡的窗口句柄:

```
public String switchToTab(String destinationWindowHandle) {
    String currentWindowHandle = driver.getWindowHandle();
    driver.switchTo().window(destinationWindowHandle);
    return currentWindowHandle;
}
```

### 4.3.关闭标签

在我们的测试之后，或者当我们不再需要某个选项卡时，我们需要关闭它。Selenium 提供了一条语句来关闭当前选项卡，如果是最后一个打开的选项卡，则关闭整个浏览器:

```
driver.close();
```

我们可以使用该语句关闭所有不再需要的选项卡。使用以下方法，我们可以关闭除特定选项卡之外的所有选项卡:

```
void closeAllTabsExcept(String windowHandle) {
    for (String handle : driver.getWindowHandles()) {
        if (!handle.equals(windowHandle)) {
            driver.switchTo().window(handle);
            driver.close();
        }
    }
    driver.switchTo().window(windowHandle);
}
```

在这个方法中，我们遍历所有的窗口句柄。如果窗口句柄不同于提供的窗口句柄，我们将切换到它并关闭它。最后，我们将确保再次切换到我们想要的选项卡。

我们可以使用此方法关闭除当前打开的选项卡之外的所有选项卡:

```
void closeAllTabsExceptCurrent() {
    String currentWindow = driver.getWindowHandle();
    closeAllTabsExcept(currentWindow);
}
```

我们现在可以在每次测试后在我们的`SeleniumTestBase`类中使用这个方法来关闭所有剩余的选项卡。这在测试未能在开始下一个测试之前清理浏览器的情况下特别有用，以免影响结果。我们将调用带有`@AfterEach`注释的方法中的方法:

```
@AfterEach
public void closeTabs() {
    tabHelper.closeAllTabsExceptCurrent();
}
```

### 4.4.用热键处理标签

使用 Selenium，可以用浏览器热键处理标签。**不幸的是，由于`ChromeDriver`的改变，这似乎不再有效。**

## 5.试验

我们可以通过几个简单的测试来验证我们的`TabHelper`类的标签处理是否如预期的那样工作。正如本文介绍中提到的，我们的测试类需要扩展`SeleniumTestBase:`

```
class SeleniumTabsLiveTest extends SeleniumTestBase {
    //...
}
```

对于这些测试，我们在测试类中为 URL 和定位器声明了一些常量，如下所示:

```
By LINK_TO_ATTRIBUTES_PAGE_XPATH = By.xpath("//a[.='Attributes in new page']");
By LINK_TO_ALERT_PAGE_XPATH = By.xpath("//a[.='Alerts In A New Window From JavaScript']");

String MAIN_PAGE_URL = "https://testpages.herokuapp.com/styled/windows-test.html";
String ATTRIBUTES_PAGE_URL = "https://testpages.herokuapp.com/styled/attributes-test.html";
String ALERT_PAGE_URL = "https://testpages.herokuapp.com/styled/alerts/alert-test.html";
```

在我们的第一个测试案例中，我们在一个新标签页中打开一个链接。我们正在验证是否打开了两个选项卡，并且可以在它们之间切换。注意，我们总是存储各自的窗口句柄。下面的代码代表了这个测试用例:

```
void givenOneTab_whenOpenTab_thenTwoTabsOpen() {
    driver.get(MAIN_PAGE_URL);

    String mainWindow = tabHelper.openLinkAndSwitchToNewTab(LINK_TO_ATTRIBUTES_PAGE_XPATH);
    assertEquals(ATTRIBUTES_PAGE_URL, driver.getCurrentUrl());

    tabHelper.switchToTab(mainWindow);
    assertEquals(MAIN_PAGE_URL, driver.getCurrentUrl());
    assertEquals(2, driver.getWindowHandles().size());
}
```

在我们的第二个测试中，我们希望验证除了第一个打开的选项卡之外，我们可以关闭所有的选项卡。我们需要提供该选项卡的窗口句柄。以下代码显示了这个测试用例:

```
void givenTwoTabs_whenCloseAllExceptMainTab_thenOneTabOpen() {
    driver.get(MAIN_PAGE_URL);
    String mainWindow = tabHelper.openLinkAndSwitchToNewTab(LINK_TO_ATTRIBUTES_PAGE_XPATH);
    assertEquals(ATTRIBUTES_PAGE_URL, driver.getCurrentUrl());
    assertEquals(2, driver.getWindowHandles().size());

    tabHelper.closeAllTabsExcept(mainWindow);

    assertEquals(1, driver.getWindowHandles().size());
    assertEquals(MAIN_PAGE_URL, driver.getCurrentUrl());
}
```

## 6.结论

在本文中，我们学习了如何使用 Selenium 处理浏览器选项卡。我们讲述了如何用不同的窗口句柄来区分不同的选项卡。本文介绍了打开标签页、切换标签页以及完成后关闭标签页。

和往常一样，所有这些例子的实现都可以在 GitHub 上找到[。](https://web.archive.org/web/20230103152530/https://github.com/eugenp/tutorials/tree/master/testing-modules/selenium-junit-testng/)