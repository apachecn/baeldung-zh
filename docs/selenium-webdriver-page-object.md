# 使用 Selenium/WebDriver 和页面对象模式进行测试

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/selenium-webdriver-page-object>

## 1。简介

在本文中，我们将在[之前的文章](/web/20221128115839/https://www.baeldung.com/java-selenium-with-junit-and-testng)的基础上，通过引入页面对象模式继续改进我们的 Selenium/WebDriver 测试。

## 2。添加硒

让我们在项目中添加一个新的依赖项，以编写更简单、更易读的断言:

```java
<dependency>
    <groupId>org.hamcrest</groupId>
    <artifactId>hamcrest-all</artifactId>
    <version>1.3</version>
</dependency>
```

最新版本可以在 [Maven 中央存储库](https://web.archive.org/web/20221128115839/https://search.maven.org/classic/#search%7Cgav%7C1%7Cg%3A%22org.hamcrest%22%20AND%20a%3A%22hamcrest-all%22)中找到。

## 2.1。附加方法

在本系列的第一部分中，我们使用了一些额外的实用方法，我们也将在这里使用。

我们将从`navigateTo(String url)` 方法开始——它将帮助我们浏览应用程序的不同页面:

```java
public void navigateTo(String url) {
    driver.navigate().to(url);
}
```

然后，`clickElement(WebElement element)`——顾名思义——将负责在指定的元素上执行点击操作:

```java
public void clickElement(WebElement element) {
    element.click();
}
```

## 3。页面对象模式

给了我们许多强大的、低级的 API，我们可以用它们来与 HTML 页面交互。

然而，随着测试复杂性的增加，与底层的、原始的 DOM 元素进行交互并不理想。我们的代码将更难更改，可能会在小的 UI 更改后中断，并且，简单地说，灵活性会更低。

相反，我们可以利用简单的封装，将所有这些底层细节移动到一个页面对象中。

在我们开始编写首页对象之前，最好对模式有一个清晰的理解——因为它应该允许我们模拟用户与应用程序的交互。

页面对象将表现为一种接口，它将封装我们的页面或元素的细节，并将公开一个高级 API 来与该元素或页面进行交互。

因此，一个重要的细节是为我们的方法提供描述性的名称(例如因为我们可以更容易地复制用户采取的动作，并且当我们将步骤链接在一起时，通常会产生更好的 API。

好了，现在让我们继续,**创建我们的页面对象**——在本例中，我们的主页:

```java
public class BaeldungHomePage {

    private SeleniumConfig config;

    @FindBy(css = ".nav--logo_mobile")
    private WebElement title;
    @FindBy(css = ".menu-start-here > a")
    private WebElement startHere;

    // ...

    public StartHerePage clickOnStartHere() {
        config.clickElement(startHere);

        StartHerePage startHerePage = new StartHerePage(config);
        PageFactory.initElements(config.getDriver(), startHerePage);

        return startHerePage;
    }
}
```

注意我们的实现是如何处理 DOM 的底层细节并公开一个漂亮的高级 API 的。

例如，`@FindBy`注释允许我们预先填充我们的`WebElements`，这也可以通过 API 使用[来表示:](https://web.archive.org/web/20221128115839/https://seleniumhq.github.io/selenium/docs/api/java/org/openqa/selenium/By.html)

```java
private WebElement title = By.cssSelector(".header--menu > a");
```

当然，这两种方法都是有效的，但是使用注释更干净一些。

另外，请注意链接——我们的`clickOnStartHere()` 方法返回一个`StartHerePage` 对象——在这里我们可以继续交互:

```java
public class StartHerePage {

    // Includes a SeleniumConfig attribute

    @FindBy(css = ".page-title")
    private WebElement title;

    // constructor

    public String getPageTitle() {
        return title.getText();
    }
}
```

让我们编写一个快速测试，我们只需导航到页面并检查其中一个元素:

```java
@Test
public void givenHomePage_whenNavigate_thenShouldBeInStartHere() {
    homePage.navigate();
    StartHerePage startHerePage = homePage.clickOnStartHere();

    assertThat(startHerePage.getPageTitle(), is("Start Here"));
}
```

重要的是要考虑到我们的主页有责任:

1.  根据给定的浏览器配置，导航到页面。
2.  在那里，验证页面的内容(在本例中是标题)。

我们的测试非常简单；我们导航到主页，单击“Start Here”元素，这将把我们带到同名的页面，最后，我们只需验证标题是否存在。

在我们的测试运行之后，`close()`方法将被执行，我们的浏览器将自动关闭。

## 3.1。分离关注点

我们可以考虑的另一种可能性可能是分离关注点(甚至更多)，通过拥有两个独立的类，一个将负责拥有我们页面的所有属性`(WebElement`或`By)`:

```java
public class BaeldungAboutPage {

    @FindBy(css = ".page-header > h1")
    public static WebElement title;
}
```

另一个负责我们想要测试的功能的所有实现:

```java
public class BaeldungAbout {

    private SeleniumConfig config;

    public BaeldungAbout(SeleniumConfig config) {
        this.config = config;
        PageFactory.initElements(config.getDriver(), BaeldungAboutPage.class);
    }

    // navigate and getTitle methods
}
```

如果我们使用属性作为`By`并且不使用注释特性，建议在我们的页面类中添加一个私有构造函数来防止它被实例化。

值得一提的是，我们需要传递包含注释的类，在本例中是`BaeldungAboutPage`类，这与我们在上一个示例中传递`this`关键字的做法不同。

```java
@Test
public void givenAboutPage_whenNavigate_thenTitleMatch() {
    about.navigateTo();

    assertThat(about.getPageTitle(), is("About Baeldung"));
}
```

请注意，我们现在如何在实现中保留与页面交互的所有内部细节，在这里，我们实际上可以在更高的可读级别上使用这个客户端。

## 4。结论

在这个快速教程中，我们着重于在页面对象模式的帮助下改进我们对 **Selenium/WebDriver 的使用。我们研究了不同的例子和实现，以了解利用该模式与我们的站点进行交互的实际方法。**

和往常一样，所有这些例子和片段的实现都可以在 GitHub 上找到[。这是一个基于 Maven 的项目，因此应该很容易导入和运行。](https://web.archive.org/web/20221128115839/https://github.com/eugenp/tutorials/tree/master/testing-modules/selenium-junit-testng)