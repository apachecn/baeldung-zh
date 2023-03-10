# Serenity BDD 和剧本

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/serenity-screenplay>

## 1。概述

在本文中，我们将快速浏览 Serenity BDD 中的剧本模式。我们建议你在阅读本书之前先阅读 Serenity BDD 的基础知识。另外，关于 [Serenity BDD 与 Spring](/web/20221128111557/https://www.baeldung.com/serenity-spring-jbehave) 集成的文章可能也很有趣。

Serenity BDD 中引入的 screen script 旨在通过让团队编写更健壮和可靠的测试来鼓励良好的测试习惯和设计良好的测试套件。它基于 Selenium WebDriver 和页面对象模型。如果你读过我们的[对 Selenium](/web/20221128111557/https://www.baeldung.com/java-selenium-with-junit-and-testng) 的介绍，你会发现这些概念相当熟悉。

## 2。Maven 依赖关系

首先，让我们将以下依赖项添加到 *pom.xml* 文件中:

```java
<dependency>
    <groupId>net.serenity-bdd</groupId>
    <artifactId>serenity-junit</artifactId>
    <version>1.4.0</version>
</dependency>
<dependency>
    <groupId>net.serenity-bdd</groupId>
    <artifactId>serenity-screenplay</artifactId>
    <version>1.4.0</version>
</dependency>
<dependency>
    <groupId>net.serenity-bdd</groupId>
    <artifactId>serenity-screenplay-webdriver</artifactId>
    <version>1.4.0</version>
</dependency>
```

最新版本的[宁静-剧本](https://web.archive.org/web/20221128111557/https://search.maven.org/classic/#search%7Cga%7C1%7Ca%3A%22serenity-screenplay)和[宁静-剧本-网络驱动](https://web.archive.org/web/20221128111557/https://search.maven.org/classic/#search%7Cga%7C1%7Ca%3A%22serenity-screenplay-webdriver)可以从 Maven 中央存储库获取。

我们还需要网络驱动来执行剧本——chrome driver 或者 T2 Mozilla-GeckoDriver T3 都可以。在本文中，我们将使用 ChromeDriver。

启用 WebDriver 需要以下插件配置，其中`webdriver.chrome.driver`的值应该是我们 maven 项目中 ChromeDriver 二进制文件的相对路径:

```java
<plugin>
    <artifactId>maven-failsafe-plugin</artifactId>
    <version>2.20</version>
    <configuration>
        <systemProperties>
            <webdriver.chrome.driver>chromedriver</webdriver.chrome.driver>
        </systemProperties>
    </configuration>
</plugin>
```

## 3。网络驱动支持

我们可以通过在 WebDriver 变量上标记`@Managed`注释来让 Serenity 管理 WebDriver 实例。Serenity 将在每次测试开始时打开一个适当的驱动程序，并在测试结束时关闭它。

在下面的例子中，我们启动一个 ChromeDriver，打开 Google 搜索‘bael dung’。我们希望 Eugen 的名字出现在搜索结果中:

```java
@RunWith(SerenityRunner.class)
public class GoogleSearchLiveTest {

    @Managed(driver = "chrome") 
    private WebDriver browser;

    @Test
    public void whenGoogleBaeldungThenShouldSeeEugen() {
        browser.get("https://www.google.com/ncr");

        browser
          .findElement(By.name("q"))
          .sendKeys("baeldung", Keys.ENTER);

        new WebDriverWait(browser, 5)https://www.baeldung.com/serenity-screenplay
          .until(visibilityOfElementLocated(By.cssSelector("._ksh")));

        assertThat(browser
          .findElement(By.cssSelector("._ksh"))
          .getText(), containsString("Eugen (Baeldung)"));
    }
}
```

如果我们不为`@Managed`指定任何参数，Serenity BDD 在这种情况下将使用 Firefox。由`@Managed`注释:`firefox, chrome, iexplorer, htmlunit, phantomjs`支持的驱动程序的完整列表。

如果需要在 IExplorer 或者 Edge 中测试，可以分别从[这里(针对 IE)](https://web.archive.org/web/20221128111557/https://selenium-release.storage.googleapis.com/index.html) 和[这里(针对 Edge)](https://web.archive.org/web/20221128111557/https://developer.microsoft.com/en-us/microsoft-edge/tools/webdriver) 下载 web 驱动。Safari WebDriver 只在 MacOS 上`/usr/bin/safaridriver`下可用。

## 4。页面对象

Serenity 页面对象代表一个 WebDriver 页面对象。 [`PageObject`](https://web.archive.org/web/20221128111557/http://thucydides.info/docs/apidocs/net/thucydides/core/pages/PageObject.html) 隐藏 WebDriver 细节以供重用。

### 4.1。使用`PageObject` 重构示例

让我们首先通过提取元素定位、搜索和结果验证动作，使用 [`PageObject`](https://web.archive.org/web/20221128111557/http://thucydides.info/docs/apidocs/net/thucydides/core/pages/PageObject.html) 来改进我们之前的测试:

```java
@DefaultUrl("https://www.google.com/ncr")
public class GoogleSearchPageObject extends PageObject {

    @FindBy(name = "q") 
    private WebElement search;

    @FindBy(css = "._ksh") 
    private WebElement result;

    public void searchFor(String keyword) {
        search.sendKeys(keyword, Keys.ENTER);
    }

    public void resultMatches(String expected) {
        assertThat(result.getText(), containsString(expected));
    }
}
```

[`WebElement`](https://web.archive.org/web/20221128111557/https://seleniumhq.github.io/selenium/docs/api/java/org/openqa/selenium/WebElement.html) 代表一个 HTML 元素。我们可以通过界面的 API 与网页进行交互。在上面的例子中，我们使用了两种方法在页面中定位 web 元素:通过元素名称和元素的 CSS 类。

在查找 web 元素时，有更多的方法可以应用，例如通过标签名称查找、通过链接文本查找等。更多详情请参考我们的[硒指南](/web/20221128111557/https://www.baeldung.com/java-selenium-with-junit-and-testng)。

我们也可以用 [`WebElementFacade`](https://web.archive.org/web/20221128111557/http://thucydides.info/docs/apidocs/net/thucydides/core/pages/WebElementFacade.html) 代替`WebElement`，提供更流畅的 API 来处理 web 元素。

由于 **Serenity 将自动实例化 JUnit 测试**中的任何`PageObject`字段，因此之前的测试可以重写为一个更加简洁的测试:

```java
@RunWith(SerenityRunner.class)
public class GoogleSearchPageObjectLiveTest {

    @Managed(driver = "chrome") 
    private WebDriver browser;

    GoogleSearchPageObject googleSearch;

    @Test
    public void whenGoogleBaeldungThenShouldSeeEugen() {
        googleSearch.open();

        googleSearch.searchFor("baeldung");

        googleSearch.resultMatches("Eugen (Baeldung)");
    }
}
```

现在，我们可以使用其他关键字进行搜索，并匹配相关的搜索结果，而无需对`GoogleSearchPageObject`进行任何更改。

### 4.2。异步支持

如今，许多网页是动态提供或呈现的。为了处理这种情况，`PageObject`还支持许多丰富的特性，使我们能够检查元素的状态。**我们可以检查元素是否可见，或者等到它们可见后再继续。**

让我们通过确保我们想要看到的元素可见来增强`resultMatches`方法:

```java
public void resultMatches(String expected) {
    waitFor(result).waitUntilVisible();
    assertThat(result.getText(), containsString(expected));
}
```

如果我们不希望等待太久，我们可以显式指定等待操作的超时时间:

```java
public void resultMatches(String expected) {
    withTimeoutOf(5, SECONDS)
      .waitFor(result)
      .waitUntilVisible();
    assertThat(result.getText(), containsString(expected));
}
```

## 5。剧本模式

剧本模式将可靠的设计原则应用到自动化验收测试中。对剧本模式的一般理解可以在`given_when_then`上下文中解释为:

*   `given`–能够执行一些`Task`的`Actor`
*   `when`–由`Actor`执行`Task`
*   `then –` `Actor`应该看到效果并验证结果

现在让我们将之前的测试场景融入到剧本模式中:给定一个可以使用 Google 的用户 Kitty，当她在 Google 上搜索“baeldung”时，Kitty 应该会在结果中看到 Eugen 的名字。

首先，定义 Kitty 可以执行的任务。

1.  Kitty 会用谷歌:

    ```java
    public class StartWith implements Task {

        public static StartWith googleSearchPage() {
            return instrumented(StartWith.class);
        }

        GoogleSearchPage googleSearchPage;

        @Step("{0} starts a google search")
        public <T extends Actor> void performAs(T t) {
            t.attemptsTo(Open
              .browserOn()
              .the(googleSearchPage));
        }
    }
    ```

2.  Kitty 可以在谷歌上搜索:

    ```java
    public class SearchForKeyword implements Task {

        @Step("{0} searches for '#keyword'")
        public <T extends Actor> void performAs(T actor) {
            actor.attemptsTo(Enter
              .theValue(keyword)
              .into(GoogleSearchPage.SEARCH_INPUT_BOX)
              .thenHit(Keys.RETURN));
        }

        private String keyword;

        public SearchForKeyword(String keyword) {
            this.keyword = keyword;
        }

        public static Task of(String keyword) {
            return Instrumented
              .instanceOf(SearchForKeyword.class)
              .withProperties(keyword);
        }
    }
    ```

3.  Kitty 可以看到谷歌搜索结果:

    ```java
    public class GoogleSearchResults implements Question<List<String>> {

        public static Question<List<String>> displayed() {
            return new GoogleSearchResults();
        }

        public List<String> answeredBy(Actor actor) {
            return Text
              .of(GoogleSearchPage.SEARCH_RESULT_TITLES)
              .viewedBy(actor)
              .asList();
        }
    }
    ```

此外，我们已经定义了谷歌搜索`PageObject`:

```java
@DefaultUrl("https://www.google.com/ncr")
public class GoogleSearchPage extends PageObject {

    public static final Target SEARCH_RESULT_TITLES = Target
      .the("search results")
      .locatedBy("._ksh");

    public static final Target SEARCH_INPUT_BOX = Target
      .the("search input box")
      .locatedBy("#lst-ib");
}
```

现在我们的主要测试类看起来像这样:

```java
@RunWith(SerenityRunner.class)
public class GoogleSearchScreenplayLiveTest {

    @Managed(driver = "chrome") 
    WebDriver browser;

    Actor kitty = Actor.named("kitty");

    @Before
    public void setup() {
        kitty.can(BrowseTheWeb.with(browser));
    }

    @Test
    public void whenGoogleBaeldungThenShouldSeeEugen() {
        givenThat(kitty).wasAbleTo(StartWith.googleSearchPage());

        when(kitty).attemptsTo(SearchForKeyword.of("baeldung"));

        then(kitty).should(seeThat(GoogleSearchResults.displayed(), 
          hasItem(containsString("Eugen (Baeldung)"))));
    }
}
```

运行该测试后，我们将看到 Kitty 在测试报告中执行的每个步骤的截图:

[![kitty search baeldung](img/961963c33be8d5d625a48b8f9777d309.png)](/web/20221128111557/https://www.baeldung.com/wp-content/uploads/2017/06/kitty-search-baeldung.png)

## 6。总结

在本文中，我们介绍了如何在 Serenity BDD 中使用剧本模式。同样，在`PageObject`的帮助下，我们不必直接与 WebDrivers 交互，这使得我们的测试更容易阅读、维护和扩展。

关于 Serenity BDD 中`PageObject`和剧本模式的更多细节，请查看 Serenity 文档的相关部分。

和往常一样，完整的示例代码可以在 Github 上的[中找到。](https://web.archive.org/web/20221128111557/https://github.com/eugenp/tutorials/tree/master/libraries-testing)