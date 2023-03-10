# Hibernate 验证器特定约束

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/hibernate-validator-constraints>

## 1。概述

在本教程中，我们将回顾 Hibernate Validator 约束，这些约束内置于 Hibernate Validator 中，但在 Bean 验证规范之外。

关于 Bean 验证的回顾，请参考我们关于 [Java Bean 验证基础知识](/web/20221127014958/https://www.baeldung.com/javax-validation)的文章。

## 2。Hibernate 验证程序设置

至少，**我们应该将 [Hibernate Validator](https://web.archive.org/web/20221127014958/https://search.maven.org/classic/#search%7Cgav%7C1%7Cg%3A%22org.hibernate%22%20AND%20a%3A%22hibernate-validator%22) 添加到我们的依赖项:**

```java
<dependency>
    <groupId>org.hibernate.validator</groupId>
    <artifactId>hibernate-validator</artifactId>
    <version>6.0.16.Final</version>
</dependency>
```

注意，Hibernate Validator 并不依赖于 [Hibernate，ORM，这一点我们已经在许多其他文章中讨论过了。](/web/20221127014958/https://www.baeldung.com/tag/hibernate/)

此外，我们将介绍的一些注释仅适用于我们的项目使用某些库的情况。因此，对于其中的每一个，我们将指出必要的依赖关系。

## 3。验证与金钱相关的价值

### 3.1.验证信用卡号码

有效的信用卡号码必须满足一个校验和，我们使用[卢恩的算法](https://web.archive.org/web/20221127014958/https://en.wikipedia.org/wiki/Luhn_algorithm)来计算。**当** **字符串满足校验和时，`@CreditCardNumber` 约束成功。**

**`@CreditCardNumber` 不对输入字符串执行任何其他检查。**特别是，它不检查输入的长度。因此，它只能检测由于一个小的错别字而无效的数字。

注意，默认情况下，如果字符串包含非数字字符，约束将失败，但是我们可以告诉它忽略它们:

```java
@CreditCardNumber(ignoreNonDigitCharacters = true)
private String lenientCreditCardNumber;
```

然后，我们可以包含空格或破折号等字符:

```java
validations.setLenientCreditCardNumber("7992-7398-713");
constraintViolations = validator.validateProperty(validations, "lenientCreditCardNumber");
assertTrue(constraintViolations.isEmpty());
```

### 3.2。验证货币价值

`@Currency` 验证器检查给定的货币金额是否是指定的货币:

```java
@Currency("EUR")
private MonetaryAmount balance;
```

类`MonetaryAmount` 是 Java Money 的一部分。因此， **`@Currency` 只在 Java Money 实现可用时才适用[。](/web/20221127014958/https://www.baeldung.com/java-money-and-currency)**

一旦我们正确设置了 Java Money，我们就可以检查约束条件:

```java
bean.setBalance(Money.of(new BigDecimal(100.0), Monetary.getCurrency("EUR")));
constraintViolations = validator.validateProperty(bean, "balance");
assertEquals(0, constraintViolations.size());
```

## 4。验证范围

### 4.1。数字和货币范围

bean validation 规范定义了几个约束，我们可以对数字字段实施这些约束。除此之外，Hibernate Validator 还提供了一个方便的注释，`@Range`，**作为`@Min` 和`@Max,`** 的组合来匹配一个范围:

```java
@Range(min = 0, max = 100)
private BigDecimal percent;
```

与`@Min`和`@Max`一样，`@Range` 适用于本原数字类型的字段及其包装器；`BigInteger` 、`BigDecimal`、`String` 表示上述内容，最后是`MonetaryValue` 字段。

### 4.2。持续时间

除了代表时间点的值的标准 JSR 380 注释，Hibernate Validator 还包括对`Duration`的约束。一定要先看看 Java Time 的`Period` 和`Duration` 类。

因此，**我们可以在属性上强制最小和最大持续时间:**

```java
@DurationMin(days = 1, hours = 2)
@DurationMax(days = 2, hours = 1)
private Duration duration;
```

即使我们没有在这里全部显示，注释也有从纳秒到天的所有时间单位的参数。

请注意，默认情况下，**最小值和最大值包含在内。**也就是说，与最小值或最大值完全相同的值将通过验证。

如果我们希望边界值无效，我们将`inclusive` 属性定义为 false:

```java
@DurationMax(minutes = 30, inclusive = false)
```

## 5。验证字符串

### 5.1。字符串长度

我们可以使用两个稍微不同的约束来强制字符串具有一定的长度。

通常，我们希望确保字符串的字符长度——我们用`length` 方法测量的长度——在最小值和最大值之间。在这种情况下，我们在字符串属性或字段上使用`@Length` :

```java
@Length(min = 1, max = 3)
private String someString;
```

**然而，由于 Unicode 的复杂性，有时字符长度和码位长度会有所不同。当我们想检查后者时，我们用`@CodePointLength:`**

```java
@CodePointLength(min = 1, max = 3)
private String someString;
```

例如，字符串“aa\uD835\uDD0A”有 4 个字符长，但它只包含 3 个码位，因此它将使第一个约束失败，并通过第二个约束。

同样，对于这两种注释，我们可以省略最小值或最大值。

### 5.2。检查数字串

我们已经看到了如何检查一个字符串是否是有效的信用卡号。然而，Hibernate Validator 包含了其他几个针对数字字符串的约束。

我们回顾的第一个是`@LuhnCheck.` 这是`@CreditCardNumber,`的一般化版本，因为**它执行相同的检查，但是允许额外的参数:**

```java
@LuhnCheck(startIndex = 0, endIndex = Integer.MAX_VALUE, checkDigitIndex = -1)
private String someString;
```

这里，我们已经展示了参数的默认值，所以上面的内容相当于一个简单的`@LuhnCheck` 注释。

但是，正如我们所看到的，我们可以对子串(`startIndex` 和`endIndex`)执行检查，并告诉约束哪个数字是校验和数字，用-1 表示检查的子串中的最后一个数字。

其他有趣的约束包括[模 10 检查](https://web.archive.org/web/20221127014958/https://www.activebarcode.com/codes/checkdigit/modulo10.html) ( `@Mod10Check`)和[模 11 检查](https://web.archive.org/web/20221127014958/https://www.activebarcode.com/codes/checkdigit/modulo11.html) ( `@Mod11Check`)，它们通常用于条形码和 ISBN 等其他代码。

然而，对于那些特定的情况，Hibernate Validator 恰好提供了一个约束来验证 ISBN 代码`@ISBN`，以及一个`@EAN`约束来验证 [EAN 条形码](https://web.archive.org/web/20221127014958/https://en.wikipedia.org/wiki/International_Article_Number)。

### 5.3。URL 和 HTML 验证

`@Url` 约束验证一个字符串是否是 URL 的有效表示。此外，我们可以检查 URL 的特定部分是否有特定值:

```java
@URL(protocol = "https")
private String url;
```

因此，我们可以检查协议、主机和端口。如果这还不够，我们可以使用一个`regexp` 属性来匹配 URL 和正则表达式。

我们还可以验证属性是否包含“安全”的 HTML 代码(例如，没有脚本标记):

```java
@SafeHtml
private String html;
```

`@SafeHtml` 使用[JSoup 库](/web/20221127014958/https://www.baeldung.com/java-with-jsoup)，它必须包含在我们的依赖项中。

我们可以使用内置的标签白名单(注释的`whitelist` 属性)并包含额外的标签和属性(`additionalTags`和`additionalTagsWithAttributes` 参数)，根据我们的需要定制 HTML 清理。

## 6。其他约束条件

让我们简单地提一下 Hibernate Validator 包含了一些特定于国家和地区的约束，特别是对于一些巴西和波兰的身份证号、纳税人代码等等。请参考[文档](https://web.archive.org/web/20221127014958/http://docs.jboss.org/hibernate/stable/validator/reference/en-US/html_single/#_country_specific_constraints)的相关章节，了解完整列表。

此外，我们可以用`@UniqueElements.`检查集合是否包含重复项

最后，对于现有注释没有涵盖的复杂情况，我们可以调用在 JSR-223 兼容脚本引擎中编写的脚本。当然，我们在关于 Nashorn 的文章中已经提到了 JSR-223，它是现代 JVM 中包含的 JavaScript 实现。

在这种情况下，注释是在类级别，脚本在整个实例上调用，作为变量`_this:`传递

```java
@ScriptAssert(lang = "nashorn", script = "_this.valid")
public class AdditionalValidations {
    private boolean valid = true;
    // standard getters and setters
}
```

然后，我们可以检查整个实例上的约束:

```java
bean.setValid(false);
constraintViolations = validator.validate(bean);
assertEquals(1, constraintViolations.size());
```

## 7 .**。结论**

在本文中，我们列出了 Hibernate Validator 中超出 Bean 验证规范中定义的最小集合的约束。

所有这些例子和代码片段的实现都可以在 GitHub 上找到[。](https://web.archive.org/web/20221127014958/https://github.com/eugenp/tutorials/tree/master/persistence-modules/hibernate-mapping/)