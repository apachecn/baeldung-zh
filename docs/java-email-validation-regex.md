# Java 中的电子邮件验证

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-email-validation-regex>

## 1.概观

在本教程中，我们将学习如何使用[正则表达式](/web/20220529013448/https://www.baeldung.com/regular-expressions-java)在 Java 中验证电子邮件地址。

## 2。Java 中的电子邮件验证

几乎每个有用户注册的应用程序都需要电子邮件验证。

电子邮件地址分为三个主要部分:本地部分、一个符号和一个域名。例如，如果“[【电子邮件保护】](/web/20220529013448/https://www.baeldung.com/cdn-cgi/l/email-protection)”是一封电子邮件，则:

*   本地部分=用户名
*   @ = @
*   域= domain.com

通过字符串操作技术来验证一个电子邮件地址可能需要很大的努力，因为我们通常需要计算和检查所有的字符类型和长度。但是在 Java 中，通过使用正则表达式，事情会简单得多。

正如我们所知道的，正则表达式是一个匹配模式的字符序列。在下面几节中，我们将看到如何通过使用几种不同的正则表达式方法来执行电子邮件验证。

## 3。简单的正则表达式验证

验证电子邮件地址最简单的正则表达式是 **`^(.+)@(\S+) $`** 。

它只检查电子邮件地址中是否存在`@`符号。如果存在，则验证结果返回`true,`，否则，结果为`false`。但是，这个正则表达式不检查电子邮件的本地部分和域。

比如按照这个正则表达式，`[[email protected]](/web/20220529013448/https://www.baeldung.com/cdn-cgi/l/email-protection)`会通过验证，但是`username#domain.com`会验证失败。

让我们定义一个简单的助手方法来匹配正则表达式模式:

```java
public static boolean patternMatches(String emailAddress, String regexPattern) {
    return Pattern.compile(regexPattern)
      .matcher(emailAddress)
      .matches();
}
```

我们还将编写代码，使用以下正则表达式来验证电子邮件地址:

```java
@Test
public void testUsingSimpleRegex() {
    emailAddress = "[[email protected]](/web/20220529013448/https://www.baeldung.com/cdn-cgi/l/email-protection)";
    regexPattern = "^(.+)@(\\S+)$";
    assertTrue(EmailValidation.patternMatches(emailAddress, regexPattern));
}
```

电子邮件地址中缺少`@`符号也会导致验证失败。

## 4。严格的正则表达式验证

现在让我们编写一个更严格的正则表达式，它将检查电子邮件的本地部分以及域部分:

`**^(?=.{1,64}@)[A-Za-z0-9_-]+(\\.[A-Za-z0-9_-]+)*@[^-][A-Za-z0-9-]+(\\.[A-Za-z0-9-]+)*(\\.[A-Za-z]{2,})$**`

通过使用此正则表达式，在电子邮件地址的本地部分施加了以下限制:

*   它允许从 0 到 9 的数值。
*   从 a 到 z 的大小写字母都允许。
*   允许使用下划线“_”、连字符“-”和点“.”
*   本地部分的开头和结尾不允许有点号。
*   不允许连续的点。
*   对于本地部分，最多允许 64 个字符。

此正则表达式中对域部分的限制包括:

*   它允许从 0 到 9 的数值。
*   我们允许从 a 到 z 的大写和小写字母。
*   连字符“-”和点“.”不允许出现在域部分的开头和结尾。
*   没有连续的点。

我们还将编写代码来测试这个正则表达式:

```java
@Test
public void testUsingStrictRegex() {
    emailAddress = "[[email protected]](/web/20220529013448/https://www.baeldung.com/cdn-cgi/l/email-protection)";
    regexPattern = "^(?=.{1,64}@)[A-Za-z0-9_-]+(\\.[A-Za-z0-9_-]+)*@" 
        + "[^-][A-Za-z0-9-]+(\\.[A-Za-z0-9-]+)*(\\.[A-Za-z]{2,})$";
    assertTrue(EmailValidation.patternMatches(emailAddress, regexPattern));
}
```

因此，通过这种电子邮件验证技术有效的一些电子邮件地址是:

*   [【电子邮件保护】](/web/20220529013448/https://www.baeldung.com/cdn-cgi/l/email-protection)
*   [【电子邮件保护】](/web/20220529013448/https://www.baeldung.com/cdn-cgi/l/email-protection)
*   [【电子邮件保护】](/web/20220529013448/https://www.baeldung.com/cdn-cgi/l/email-protection)
*   [【电子邮件保护】](/web/20220529013448/https://www.baeldung.com/cdn-cgi/l/email-protection)
*   [【电子邮件保护】](/web/20220529013448/https://www.baeldung.com/cdn-cgi/l/email-protection)

以下是通过此电子邮件验证将无效的一些电子邮件地址的名单:

*   [【电子邮件保护】](/web/20220529013448/https://www.baeldung.com/cdn-cgi/l/email-protection)
*   [【电子邮件保护】](/web/20220529013448/https://www.baeldung.com/cdn-cgi/l/email-protection)
*   [【电子邮件保护】](/web/20220529013448/https://www.baeldung.com/cdn-cgi/l/email-protection)
*   [【电子邮件保护】](/web/20220529013448/https://www.baeldung.com/cdn-cgi/l/email-protection)

## 5。验证非拉丁或 Unicode 字符的正则表达式电子邮件

我们在上一节中看到的正则表达式对于用英语书写的电子邮件地址很适用，但是对于非拉丁语的电子邮件地址就不适用了。

因此，我们将编写一个正则表达式，也可以用来验证 unicode 字符:

`**^(?=.{1,64}@)[\\p{L}0-9_-]+(\\.[\\p{L}0-9_-]+)*@[^-][\\p{L}0-9-]+(\\.[\\p{L}0-9-]+)*(\\.[\\p{L}]{2,})$**`

我们可以使用这个正则表达式来验证 Unicode 或非拉丁文电子邮件地址，以支持所有语言。

正如我们所看到的，这个正则表达式类似于我们在上一节中构建的严格正则表达式，只是我们用“`\\p{L}”`替换了“`A-Za-Z`”部分。这是为了支持 Unicode 字符。

让我们通过编写测试来检查这个正则表达式:

```java
@Test
public void testUsingUnicodeRegex() {
    emailAddress = "用户名@领域.电脑";
    regexPattern = "^(?=.{1,64}@)[\\p{L}0-9_-]+(\\.[\\p{L}0-9_-]+)*@" 
        + "[^-][\\p{L}0-9-]+(\\.[\\p{L}0-9-]+)*(\\.[\\p{L}]{2,})$";
    assertTrue(EmailValidation.patternMatches(emailAddress, regexPattern));
}
```

这个正则表达式不仅提供了一种更严格的验证电子邮件地址的方法，而且还支持非拉丁字符。

## 6。RFC 5322 用于电子邮件验证的正则表达式

我们可以使用 RFC 标准提供的正则表达式来验证电子邮件地址，而不是编写自定义的正则表达式。

[RFC 5322](https://web.archive.org/web/20220529013448/https://www.rfc-editor.org/info/rfc5322) ，是 [RFC 822](https://web.archive.org/web/20220529013448/https://www.rfc-editor.org/info/rfc822) 的更新版本，为电子邮件验证提供了一个正则表达式。

让我们来看看:

**T2`^[a-zA-Z0-9_!#$%&'*+/=?`{|}~^.-][[email protected]](/web/20220529013448/https://www.baeldung.com/cdn-cgi/l/email-protection)[a-zA-Z0-9.-]+$`**

正如我们所见，这是一个非常简单的正则表达式，允许电子邮件中的所有字符。

但是，它不允许管道字符(|)和单引号(')，因为这些字符在从客户端传递到服务器时会带来潜在的 [SQL 注入](/web/20220529013448/https://www.baeldung.com/sql-injection)风险。

让我们用这个正则表达式编写验证电子邮件的代码:

```java
@Test
public void testUsingRFC5322Regex() {
    emailAddress = "[[email protected]](/web/20220529013448/https://www.baeldung.com/cdn-cgi/l/email-protection)";
    regexPattern = "^[a-zA-Z0-9_!#$%&'*+/=?`{|}~^.-][[email protected]](/web/20220529013448/https://www.baeldung.com/cdn-cgi/l/email-protection)[a-zA-Z0-9.-]+$";
    assertTrue(EmailValidation.patternMatches(emailAddress, regexPattern));
}
```

## 7。检查顶级域名中字符的正则表达式

我们已经编写了 regex 来验证电子邮件地址的本地和域部分。现在我们还将编写一个正则表达式来检查电子邮件的顶级域。

以下正则表达式验证电子邮件地址的顶级域部分:

**T2`^[\\w!#$%&'*+/=?`{|}~^-]+(?:\\.[\\w!#$%&'*+/=?`{|}~^-]+)*@(?:[a-zA-Z0-9-]+\\.)+[a-zA-Z]{2,6}$`**

这个正则表达式主要检查电子邮件地址是否只有一个点，以及顶级域名中是否有最少两个最多六个字符。

我们还将编写一些代码，使用以下正则表达式来验证电子邮件地址:

```java
@Test
public void testTopLevelDomain() {
    emailAddress = "[[email protected]](/web/20220529013448/https://www.baeldung.com/cdn-cgi/l/email-protection)";
    regexPattern = "^[\\w!#$%&'*+/=?`{|}~^-]+(?:\\.[\\w!#$%&'*+/=?`{|}~^-]+)*" 
        + "@(?:[a-zA-Z0-9-]+\\.)+[a-zA-Z]{2,6}$";
    assertTrue(EmailValidation.patternMatches(emailAddress, regexPattern));
}
```

## 8 **。限制连续点、尾随点和前导点的正则表达式**

现在让我们编写一个正则表达式来限制电子邮件地址中点的使用:

**T2`^[a-zA-Z0-9_!#$%&'*+/=?`{|}~^-]+(?:\\.[a-zA-Z0-9_!#$%&'*+/=?`{|}~^-]+)*@[a-zA-Z0-9-]+(?:\\.[a-zA-Z0-9-]+)*$`**

上面的正则表达式用于限制连续点、前导点和尾随点。因此，电子邮件可以包含多个点，但在本地和域部分不是连续的。

让我们看一下代码:

```java
@Test
public void testRestrictDots() {
    emailAddress = "[[email protected]](/web/20220529013448/https://www.baeldung.com/cdn-cgi/l/email-protection)";
    regexPattern = "^[a-zA-Z0-9_!#$%&'*+/=?`{|}~^-]+(?:\\.[a-zA-Z0-9_!#$%&'*+/=?`{|}~^-]+)*@" 
        + "[a-zA-Z0-9-]+(?:\\.[a-zA-Z0-9-]+)*$";
    assertTrue(EmailValidation.patternMatches(emailAddress, regexPattern));
}
```

## 9。OWASP 验证正则表达式

这个正则表达式由 [OWASP 验证正则表达式库](https://web.archive.org/web/20220529013448/https://owasp.org/www-community/OWASP_Validation_Regex_Repository)提供，用于检查电子邮件验证:

`**^[a-zA-Z0-9_+&*-] + (?:\\.[a-zA-Z0-9_+&*-] + )*@(?:[a-zA-Z0-9-]+\\.) + [a-zA-Z]{2, 7}**`

这个正则表达式还支持标准电子邮件结构中的大多数验证。

让我们使用下面的代码来验证电子邮件地址:

```java
@Test
public void testOwaspValidation() {
    emailAddress = "[[email protected]](/web/20220529013448/https://www.baeldung.com/cdn-cgi/l/email-protection)";
    regexPattern = "^[a-zA-Z0-9_+&*-]+(?:\\.[a-zA-Z0-9_+&*-]+)*@(?:[a-zA-Z0-9-]+\\.)+[a-zA-Z]{2,7}$";
    assertTrue(EmailValidation.patternMatches(emailAddress, regexPattern));
}
```

## 10.Gmail 电子邮件的特殊情况

有一种特殊情况仅适用于 Gmail 域:允许在电子邮件的本地部分使用字符+字符。**对于 Gmail 域，两个电子邮件地址[【电子邮件保护】](/web/20220529013448/https://www.baeldung.com/cdn-cgi/l/email-protection)和[【电子邮件保护】](/web/20220529013448/https://www.baeldung.com/cdn-cgi/l/email-protection)是相同的。**

另外，[【邮件保护】](/web/20220529013448/https://www.baeldung.com/cdn-cgi/l/email-protection)与[【邮件保护】](/web/20220529013448/https://www.baeldung.com/cdn-cgi/l/email-protection)类似

我们必须实现一个略有不同的正则表达式，它也将通过这种特殊情况的电子邮件验证:

`**^(?=.{1,64}@)[A-Za-z0-9_-+]+(\\.[A-Za-z0-9_-+]+)*@[^-][A-Za-z0-9-+]+(\\.[A-Za-z0-9-+]+)*(\\.[A-Za-z]{2,})$**`

让我们写一个例子来测试这个用例:

```java
@Test
public void testGmailSpecialCase() {
    emailAddress = "[[email protected]](/web/20220529013448/https://www.baeldung.com/cdn-cgi/l/email-protection)";
    regexPattern = "^(?=.{1,64}@)[A-Za-z0-9\\+_-]+(\\.[A-Za-z0-9\\+_-]+)*@" 
        + "[^-][A-Za-z0-9\\+-]+(\\.[A-Za-z0-9\\+-]+)*(\\.[A-Za-z]{2,})$";
    assertTrue(EmailValidation.patternMatches(emailAddress, regexPattern));
}
```

## 11。电子邮件的 Apache Commons 验证器

Apache Commons Validator 是一个包含标准验证规则的验证包。因此，通过导入这个包，我们可以应用电子邮件验证。

我们可以使用`EmailValidator`类来验证电子邮件，它使用 RFC 822 标准。这个验证器混合了自定义代码和正则表达式来验证电子邮件。它不仅支持特殊字符，还支持我们讨论过的 Unicode 字符。

让我们在项目中添加 [commons-validator](https://web.archive.org/web/20220529013448/https://search.maven.org/artifact/commons-validator/commons-validator/1.7/jar) 依赖项:

```java
<dependency>
    <groupId>commons-validator</groupId>
    <artifactId>commons-validator</artifactId>
    <version>${validator.version}</version>
</dependency>
```

现在，我们可以使用下面的代码来验证电子邮件地址:

```java
@Test
public void testUsingEmailValidator() {
    emailAddress = "[[email protected]](/web/20220529013448/https://www.baeldung.com/cdn-cgi/l/email-protection)";
    assertTrue(EmailValidator.getInstance()
      .isValid(emailAddress));
}
```

## 12.我应该使用哪个正则表达式？

在本文中，我们研究了使用 regex 进行电子邮件地址验证的各种解决方案。显然，决定我们应该使用哪个解决方案取决于我们希望我们的验证有多严格，以及我们的确切需求。

例如，如果我们只需要一个简单的正则表达式来检查电子邮件中是否存在一个`@`符号，我们可以使用第 3 节中的简单正则表达式。然而，对于更详细的验证，我们可以选择第 6 节中基于 RFC5322 标准的更严格的正则表达式解决方案。

最后，如果我们要处理电子邮件中的 Unicode 字符，我们可以使用第 5 节中提供的 regex 解决方案。

## 13.结论

在本文中，我们学习了使用正则表达式在 Java 中验证电子邮件地址的各种方法。

这篇文章的完整代码可以在 GitHub 的[上找到。](https://web.archive.org/web/20220529013448/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-string-algorithms-3)