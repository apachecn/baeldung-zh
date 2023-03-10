# JavaFaker 指南

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-faker>

## 1.概观

JavaFaker 是一个库，可以用来生成从地址到流行文化参考的大量真实数据。

在本教程中，我们将看看如何使用 JavaFaker 的类来生成假数据。我们将从介绍`Faker`类和`FakeValueService`开始，然后继续介绍地区，使数据更具体地针对一个地方。

最后，我们将讨论数据的独特性。为了测试 JavaFaker 的类，我们将使用正则表达式，你可以在这里阅读更多关于它们的内容。

## 2.属国

下面是我们开始使用 JavaFaker 时需要的单个[依赖项](https://web.archive.org/web/20220523234448/https://search.maven.org/classic/#search%7Cgav%7C1%7Cg%3A%22com.github.javafaker%22%20AND%20a%3A%22javafaker%22)。

首先，我们需要基于 Maven 的项目的依赖性:

```java
<dependency>
    <groupId>com.github.javafaker</groupId>
    <artifactId>javafaker</artifactId>
    <version>0.15</version>
</dependency>
```

对于 Gradle 用户，您可以将以下内容添加到您的`build.gradle `文件中:

```java
compile group: 'com.github.javafaker', name: 'javafaker', version: '0.15'
```

## 3.`FakeValueService`

`FakeValueService`类提供了 **[方法，用于生成随机序列](https://web.archive.org/web/20220523234448/https://dius.github.io/java-faker/apidocs/index.html)** 以及解析与[地区](#locales)相关的`.yml`文件。

在这一节中，我们将介绍一些`FakerValueService`必须提供的有用方法。

### 3.1.`Letterify`、`Numerify`和`Bothify`

三种有用的方法是`Letterify`、`Numberify`和`Bothify`。`Letterify`帮助生成**字母字符的随机序列**。

接下来，`Numerify`简单地生成数字序列。

最后，`Bothify`是两者的组合，可以**创建随机的字母数字序列**——用于模仿 ID 字符串之类的东西。

`FakeValueService`需要一个有效的`Locale,`以及一个`RandomService:`

```java
@Test
public void whenBothifyCalled_checkPatternMatches() throws Exception {

    FakeValuesService fakeValuesService = new FakeValuesService(
      new Locale("en-GB"), new RandomService());

    String email = fakeValuesService.bothify("????##@gmail.com");
    Matcher emailMatcher = Pattern.compile("\\w{4}\\d{2}@gmail.com").matcher(email);

    assertTrue(emailMatcher.find());
}
```

在这个单元测试中，我们**创建了一个新的`FakeValueService`** ，其区域设置为`en-GB `，而**使用`bothify `方法生成一个唯一的假 Gmail 地址**。

它的工作原理是用随机字母**代替“`?'`，用随机数**代替`‘#' `。然后，我们可以通过简单的`Matcher`检查来检查输出是否正确。

### 3.2.重新流放

类似地， **`regexify`根据选择的正则表达式模式**生成一个随机序列。

在这个代码片段中，我们将使用`FakeValueService`来创建一个遵循指定正则表达式的随机序列:

```java
@Test
public void givenValidService_whenRegexifyCalled_checkPattern() throws Exception {

    FakeValuesService fakeValuesService = new FakeValuesService(
      new Locale("en-GB"), new RandomService());

    String alphaNumericString = fakeValuesService.regexify("[a-z1-9]{10}");
    Matcher alphaNumericMatcher = Pattern.compile("[a-z1-9]{10}").matcher(alphaNumericString);

    assertTrue(alphaNumericMatcher.find());
}
```

我们的**代码创建了一个长度为 10** 的小写字母数字字符串。我们的模式根据正则表达式检查生成的字符串。

## 4.JavaFaker 的`Faker`类

`Faker`类**允许我们使用 JavaFaker 的假数据类**。

在这一节中，我们将看到如何实例化一个`Faker`对象并使用它来调用一些假数据:

```java
Faker faker = new Faker();

String streetName = faker.address().streetName();
String number = faker.address().buildingNumber();
String city = faker.address().city();
String country = faker.address().country();

System.out.println(String.format("%s\n%s\n%s\n%s",
  number,
  streetName,
  city,
  country));
```

上面，我们用 **`Faker` `Address`对象生成一个随机地址**。

当我们运行这段代码时，我们将得到一个输出示例:

```java
3188
Dayna Mountains
New Granvilleborough
Tonga
```

我们可以看到，**数据没有单一的地理位置**,因为我们没有指定地区。为了改变这一点，我们将在下一节中学习使数据与我们的位置更加相关。

我们还可以以类似的方式使用这个`faker`对象来创建与更多对象相关的数据，例如:

*   商业
*   啤酒
*   食物
*   电话号码

你可以在这里找到完整的列表[。](https://web.archive.org/web/20220523234448/https://github.com/DiUS/java-faker)

## 5.语言环境介绍

在这里，我们将介绍如何**使用 locales 来使生成的数据更特定于单个位置**。我们将介绍一个带有美国地区和英国地区的`Faker`:

```java
@Test
public void givenJavaFakersWithDifferentLocals_thenHeckZipCodesMatchRegex() {

    Faker ukFaker = new Faker(new Locale("en-GB"));
    Faker usFaker = new Faker(new Locale("en-US"));

    System.out.println(String.format("American zipcode: %s", usFaker.address().zipCode()));
    System.out.println(String.format("British postcode: %s", ukFaker.address().zipCode()));

    Pattern ukPattern = Pattern.compile(
      "([Gg][Ii][Rr] 0[Aa]{2})|((([A-Za-z][0-9]{1,2})|"
      + "(([A-Za-z][A-Ha-hJ-Yj-y][0-9]{1,2})|(([A-Za-z][0-9][A-Za-z])|([A-Za-z][A-Ha-hJ-Yj-y]" 
      + "[0-9]?[A-Za-z]))))\\s?[0-9][A-Za-z]{2})");
    Matcher ukMatcher = ukPattern.matcher(ukFaker.address().zipCode());

    assertTrue(ukMatcher.find());

    Matcher usMatcher = Pattern.compile("^\\d{5}(?:[-\\s]\\d{4})?$")
      .matcher(usFaker.address().zipCode());

    assertTrue(usMatcher.find());
}
```

上图中，我们看到两个带有地区的`Fakers`与它们的国家邮政编码的正则表达式相匹配。

**如果传递给`Faker`的区域设置不存在，`Faker`抛出一个`LocaleDoesNotExistException`** 。

我们将用下面的单元测试对此进行测试:

```java
@Test(expected = LocaleDoesNotExistException.class)
public void givenWrongLocale_whenFakerInitialised_testExceptionThrown() {
    Faker wrongLocaleFaker = new Faker(new Locale("en-seaWorld"));
}
```

## 6.独特性

虽然 JavaFaker **看似随机生成数据，但无法保证唯一性**。

**JavaFaker 支持以`RandomService`的形式播种其伪随机数生成器(PRNG)** ，以提供重复方法调用的确定性输出。

简而言之，伪随机性是一个看似随机，实则不然的过程。

我们可以通过用相同的种子创建两个`Fakers`来看看这是如何工作的:

```java
@Test
public void givenJavaFakersWithSameSeed_whenNameCalled_CheckSameName() {

    Faker faker1 = new Faker(new Random(24));
    Faker faker2 = new Faker(new Random(24));

    assertEquals(faker1.name().firstName(), faker2.name().firstName());
} 
```

上面的代码从两个`different fakers.`返回相同的名字

## 7.结论

在本教程中，我们探索了 **JavaFaker 库来生成看起来真实的假数据**。我们还介绍了两个有用的类`Faker`类和 `FakeValueService`类。

我们探索了如何使用区域设置来生成特定于位置的数据。

最后，我们讨论了生成的**数据看起来仅仅是随机的**，而数据的唯一性并没有得到保证。

像往常一样，代码片段可以在 GitHub 上找到。