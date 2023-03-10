# 在 Java 中使用带有 Selenium WebDriver 的 Cookies

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-selenium-webdriver-cookies>

## 1.概观

在本文中，我们将快速浏览一下[如何在 Java 中使用 cookie](/web/20221205161812/https://www.baeldung.com/cookies-java)和 [Selenium WebDriver](/web/20221205161812/https://www.baeldung.com/java-selenium-with-junit-and-testng) 。

我们将讨论一些用例，然后直接进入代码。

## 2.使用 Cookies

操纵 cookies 的一个日常用例是在测试之间保持我们的会话。

一个更简单的场景是当我们想要测试我们的后端是否正确设置了 cookies。

在接下来的几节中，我们将简要讨论如何处理 cookies，同时提供简单的代码示例。

### 2.1.设置

我们需要将 [selenium-java](https://web.archive.org/web/20221205161812/https://search.maven.org/search?q=g:org.seleniumhq.selenium%20a:selenium-java) 依赖项添加到我们的项目中:

```java
<dependency>
    <groupId>org.seleniumhq.selenium</groupId>
    <artifactId>selenium-java</artifactId>
    <version>3.14.0</version>
</dependency>
```

接下来要下载最新版本的[壁虎驱动](https://web.archive.org/web/20221205161812/https://github.com/mozilla/geckodriver/releases)。

现在让我们来设置我们的测试类:

```java
public class SeleniumCookiesJUnitLiveTest {

    private WebDriver driver;
    private String navUrl;

    @Before
    public void setUp() {
        Capabilities capabilities = DesiredCapabilities.firefox();
        driver = new FirefoxDriver(capabilities);
        navUrl = "https://baeldung.com";
    }
}
```

### 2.2.阅读饼干

接下来，我们将执行一个简单的测试，在导航到一个网页后，验证 cookies 是否存在于我们的驱动程序中:

```java
@Test
public void whenNavigate_thenCookiesExist() {
    driver.navigate().to(navUrl);
    Set<Cookie> cookies = driver.manage().getCookies();

    assertThat(cookies, is(not(empty())));
}
```

**通常，我们可能想要搜索特定的 cookie** :

```java
@Test
public void whenNavigate_thenLpCookieIsHasCorrectValue() {
    driver.navigate().to(navUrl);
    Cookie lpCookie = driver.manage().getCookieNamed("lp_120073");

    assertThat(lpCookie.getValue(), containsString("www.baeldung.com"));
}
```

### 2.3.Cookie 属性

一个 cookie 可以与一个域相关联，有一个截止日期，等等。

让我们来看看一些常见的 cookie 属性:

```java
@Test
public void whenNavigate_thenLpCookieHasCorrectProps() {
    driver.navigate().to(navUrl);
    Cookie lpCookie = driver.manage().getCookieNamed("lp_120073");

    assertThat(lpCookie.getDomain(), equalTo(".baeldung.com"));
    assertThat(lpCookie.getPath(), equalTo("/"));
    assertThat(lpCookie.getExpiry(), is(not(nullValue())));
    assertThat(lpCookie.isSecure(), equalTo(false));
    assertThat(lpCookie.isHttpOnly(), equalTo(false));
}
```

### 2.4.添加 Cookies

添加 cookie 是一个简单的过程。

我们创建 cookie 并使用`addCookie`方法将其添加到驱动程序中:

```java
@Test
public void whenAddingCookie_thenItIsPresent() {
    driver.navigate().to(navUrl);
    Cookie cookie = new Cookie("foo", "bar");
    driver.manage().addCookie(cookie);
    Cookie driverCookie = driver.manage().getCookieNamed("foo");

    assertThat(driverCookie.getValue(), equalTo("bar"));
}
```

### 2.5.删除 Cookies

**正如我们所料，我们也可以使用`deleteCookie`方法删除 cookie:**

```java
@Test
public void whenDeletingCookie_thenItIsAbsent() {
    driver.navigate().to(navUrl);
    Cookie lpCookie = driver.manage().getCookieNamed("lp_120073");

    assertThat(lpCookie, is(not(nullValue())));

    driver.manage().deleteCookie(lpCookie);
    Cookie deletedCookie = driver.manage().getCookieNamed("lp_120073");

    assertThat(deletedCookie, is(nullValue()));
}
```

### 2.6.覆盖 Cookies

虽然没有显式的方法来覆盖 cookie，但是有一个简单的方法。

我们可以删除 cookie 并添加一个名称相同但值不同的新 cookie:

```java
@Test
public void whenOverridingCookie_thenItIsUpdated() {
    driver.navigate().to(navUrl);
    Cookie lpCookie = driver.manage().getCookieNamed("lp_120073");
    driver.manage().deleteCookie(lpCookie);

    Cookie newLpCookie = new Cookie("lp_120073", "foo");
    driver.manage().addCookie(newLpCookie);

    Cookie overriddenCookie = driver.manage().getCookieNamed("lp_120073");

    assertThat(overriddenCookie.getValue(), equalTo("foo"));
}
```

## 3.结论

在这个快速教程中，我们通过快速实用的例子学习了如何在 Java 中使用 Selenium WebDriver 处理 cookies。

和往常一样，代码可以在 GitHub 的[上获得。](https://web.archive.org/web/20221205161812/https://github.com/eugenp/tutorials/tree/master/testing-modules/selenium-junit-testng)