# Java 8 中的国际化和本地化

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-8-localization>

## 1.概观

国际化是准备应用程序以支持各种语言、地区、文化或政治特定数据的过程。这是任何现代多语言应用的一个重要方面。

为了进一步阅读**、**我们应该知道国际化有一个非常流行的缩写(可能比实际的名字更流行)——`i18n`由于‘I’和‘n’之间有 18 个字母。

对于当今的企业计划来说，为来自世界不同地区或多种文化领域的人服务是至关重要的。不同的文化或语言区域不仅决定了特定语言的描述，还决定了货币、数字表示，甚至不同的日期和时间组成。

例如，让我们关注特定国家的数字。它们有各种小数点和千位分隔符:

*   102，300.45(美国)
*   102 300.45(波兰)
*   102.300，45(德国)

还有不同的日期格式:

*   欧洲中部时间 2018 年 1 月 1 日星期一下午 3:20:34(美国)
*   2018 年 1 月 1 日星期一下午 3:20 cet(法国)。
*   Monday, January 1st, 2018 at 03: 20: 34 PM CET (China)

此外，不同的国家有独特的货币符号:

*   1 200.60 英镑(联合王国)
*   1.200.60 欧元(意大利)
*   1 200.60 欧元(法国)
*   1 200.60 美元(美国)

需要知道的一个重要事实是，即使国家拥有相同的货币和货币符号——比如法国和意大利——它们的货币符号的位置也可能不同。

## 2.本地化

在 Java 中，我们有一个奇妙的特性，叫做`Locale`类。

它允许我们快速区分不同的文化区域，并适当地设置内容的格式。这在国际化过程中至关重要。和 i18n 一样，本地化也有它的缩写——`**l10n**.`

使用`Locale`的主要原因是无需重新编译就可以访问所有必需的特定于地区的格式。一个应用程序可以同时处理多个地区，所以支持新的语言很简单。

区域设置通常由下划线分隔的语言、国家和变体缩写表示:

*   德(德语)
*   it_CH(意大利语，瑞士)
*   en_US_UNIX(美国，UNIX 平台)

### 2.1。字段

我们已经知道 **`Locale`由语言代码、国家代码和变体组成。还有两个可能的字段需要设置:脚本和扩展**。

让我们看一下字段列表，看看规则是什么:

*   **语言**可以是`ISO 639 alpha-2 or alpha-3`代码或注册语言子标签。
*   **地区**(国家)为`ISO 3166 alpha-2`国家代码或`UN numeric-3`区号。
*   **变量**是一个区分大小写的值或一组值，指定了`Locale`的变量。
*   **脚本**必须是有效的`ISO 15924 alpha-4`代码。
*   **扩展**是由单字符键和`String`值组成的映射。

[IANA 语言子标记注册表](https://web.archive.org/web/20220908233401/https://www.iana.org/assignments/language-subtag-registry/language-subtag-registry)包含`language`、`region`、`variant`和`script`的可能值。

没有可能的`extension`值的列表，但是这些值必须是格式良好的 [`BCP-47`](https://web.archive.org/web/20220908233401/https://docs.oracle.com/javase/tutorial/i18n/locale/extensions.html) 子标记。键和值总是被转换成小写。

### 2.2。`Locale.Builder`

创建`Locale`对象有几种方法。一种可能的方式是利用`Locale.Builder`。`Locale.Builder`有五个 setter 方法，我们可以用它们来构建对象，同时验证这些值:

```java
Locale locale = new Locale.Builder()
  .setLanguage("fr")
  .setRegion("CA")
  .setVariant("POSIX")
  .setScript("Latn")
  .build();
```

上述`Locale`的`String`表示为`fr_CA_POSIX_#Latn`。

很高兴知道**设置‘variant’可能有点棘手，因为对 variant 值没有官方限制，尽管 setter 方法要求它与`BCP-47`兼容**。

否则会抛出`IllformedLocaleException`。

在我们需要使用没有通过验证的值的情况下，我们可以使用`Locale`构造函数，因为它们不验证值。

### 2.3。构造函数

`Locale`有三个构造函数:

*   `new Locale(String language)`
*   `new Locale(String language, String country)`
*   `new Locale(String language, String country, String variant)`

三参数构造函数:

```java
Locale locale = new Locale("pl", "PL", "UNIX");
```

有效的`variant`必须是由 5 到 8 个字母数字组成的`String`,或者是单个数字后跟 3 个字母数字。我们只能通过构造函数将“UNIX”应用于`variant`字段，因为它不符合这些要求。

然而，使用构造函数创建`Locale`对象有一个缺点——我们不能设置扩展和脚本字段。

### 2.4。常数

这可能是最简单也是最受限制的获取`Locales`的方式。`Locale`类有几个静态常量，代表最流行的国家或语言:

```java
Locale japan = Locale.JAPAN;
Locale japanese = Locale.JAPANESE;
```

### 2.5。语言标签

创建`Locale`的另一种方式是调用静态工厂方法`forLanguageTag(String languageTag)`。这种方法需要一个符合`IETF BCP 47`标准的`String`。

这就是我们创建英国的方式:

```java
Locale uk = Locale.forLanguageTag("en-UK");
```

### 2.6。可用区域设置

**即使我们可以创建`Locale`对象的多种组合，我们也可能无法使用它们。**

需要注意的一点是，平台上的`Locales`依赖于 Java 运行时中已经安装的那些。

当我们使用`Locales`进行格式化时，不同的格式化程序可能会有一个更小的安装在运行时的`Locales`集合。

让我们看看如何检索可用区域设置的数组:

```java
Locale[] numberFormatLocales = NumberFormat.getAvailableLocales();
Locale[] dateFormatLocales = DateFormat.getAvailableLocales();
Locale[] locales = Locale.getAvailableLocales();
```

之后，我们可以检查我们的`Locale`是否在可用的`Locales.`中

我们应该记住，对于 Java 平台 **的各种实现和各种功能领域**，可用的语言环境是不同的。

Oracle Java SE Development Kit 网页上提供了受支持语言环境的完整列表。

### 2.7。默认区域设置

在处理本地化时，我们可能需要知道我们的`JVM`实例上的默认`Locale`是什么。幸运的是，有一个简单的方法可以做到:

```java
Locale defaultLocale = Locale.getDefault();
```

此外，我们可以通过调用类似的 setter 方法来指定默认的`Locale`:

```java
Locale.setDefault(Locale.CANADA_FRENCH);
```

当我们想要创建不依赖于`JVM`实例的`JUnit`测试时，这尤其重要。

## 3.数字和货币

本节涉及应该符合不同地区特定约定的数字和货币格式化程序。

为了格式化原始数字类型(`int`、`double`)以及它们的等价对象(`Integer`、`Double`)，我们应该使用`NumberFormat`类及其静态工厂方法。

我们对两种方法感兴趣:

*   `NumberFormat.getInstance(Locale locale)`
*   `NumberFormat.getCurrencyInstance(Locale locale)`

让我们检查一个示例代码:

```java
Locale usLocale = Locale.US;
double number = 102300.456d;
NumberFormat usNumberFormat = NumberFormat.getInstance(usLocale);

assertEquals(usNumberFormat.format(number), "102,300.456");
```

正如我们所见，创建`Locale`并使用它来检索`NumberFormat`实例和格式化一个样本号一样简单。我们可以注意到**输出包括特定于地区的小数点和千位分隔符**。

这是另一个例子:

```java
Locale usLocale = Locale.US;
BigDecimal number = new BigDecimal(102_300.456d);

NumberFormat usNumberFormat = NumberFormat.getCurrencyInstance(usLocale); 
assertEquals(usNumberFormat.format(number), "$102,300.46");
```

格式化货币的步骤与格式化数字的步骤相同。唯一的区别是格式化程序将货币符号和四舍五入的小数部分附加到两位数字上。

## 4.日期和时间

现在，我们将学习日期和时间的格式化，这可能比格式化数字更复杂。

首先，我们应该知道日期和时间格式在 Java 8 中有了显著的改变，因为它包含了全新的`Date/Time` API。因此，我们将查看不同的格式化程序类。

### 4.1。`DateTimeFormatter`

**自从 Java 8 推出以来，本地化日期和时间的主要类是`DateTimeFormatter`类**。它对实现`TemporalAccessor`接口的类进行操作，例如`LocalDateTime`、`LocalDate, LocalTime`或`ZonedDateTime. `要创建一个`DateTimeFormatter`我们必须至少提供一个模式，然后`Locale. `让我们看一个示例代码:

```java
Locale.setDefault(Locale.US);
LocalDateTime localDateTime = LocalDateTime.of(2018, 1, 1, 10, 15, 50, 500);
String pattern = "dd-MMMM-yyyy HH:mm:ss.SSS";

DateTimeFormatter defaultTimeFormatter = DateTimeFormatter.ofPattern(pattern);
DateTimeFormatter deTimeFormatter = DateTimeFormatter.ofPattern(pattern, Locale.GERMANY);

assertEquals(
  "01-January-2018 10:15:50.000", 
  defaultTimeFormatter.format(localDateTime));
assertEquals(
  "01-Januar-2018 10:15:50.000", 
  deTimeFormatter.format(localDateTime));
```

我们可以看到，在检索了`DateTimeFormatter`之后，我们所要做的就是调用`format()`方法。

为了更好地理解，我们应该熟悉可能的模式字母。

让我们以字母为例:

```java
Symbol  Meaning                     Presentation      Examples
  ------  -------                     ------------      -------
   y       year-of-era                 year              2004; 04
   M/L     month-of-year               number/text       7; 07; Jul; July; J
   d       day-of-month                number            10

   H       hour-of-day (0-23)          number            0
   m       minute-of-hour              number            30
   s       second-of-minute            number            55
   S       fraction-of-second          fraction          978
```

所有可能的带解释的模式字母都可以在[`DateTimeFormatter`](https://web.archive.org/web/20220908233401/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/time/format/DateTimeFormatter.html)`.`**的 Java 文档中找到，值得一提的是最终值取决于符号**的个数。在打印完整月份名称的示例中有“MMMM ”,而单个“M”字母将给出不带前导 0 的月份编号。

为了结束对`DateTimeFormatter`的讨论，让我们看看如何格式化`LocalizedDateTime`:

```java
LocalDateTime localDateTime = LocalDateTime.of(2018, 1, 1, 10, 15, 50, 500);
ZoneId losAngelesTimeZone = TimeZone.getTimeZone("America/Los_Angeles").toZoneId();

DateTimeFormatter localizedTimeFormatter = DateTimeFormatter
  .ofLocalizedDateTime(FormatStyle.FULL);
String formattedLocalizedTime = localizedTimeFormatter.format(
  ZonedDateTime.of(localDateTime, losAngelesTimeZone));

assertEquals("Monday, January 1, 2018 10:15:50 AM PST", formattedLocalizedTime);
```

为了格式化`LocalizedDateTime`，我们可以使用`ofLocalizedDateTime(FormatStyle dateTimeStyle)`方法并提供一个预定义的`FormatStyle.`

为了更深入地了解 Java 8 `Date/Time` API，我们已经有一篇文章[在这里](/web/20220908233401/https://www.baeldung.com/java-8-date-time-intro)。

### 4.2。`DateFormat`和`SimpleDateFormatter`

由于在项目中使用`Dates`和`Calendars`仍然很常见，我们将简要介绍用`DateFormat` 和`SimpleDateFormat`类格式化日期和时间的能力。

让我们分析一下第一个人的能力:

```java
GregorianCalendar gregorianCalendar = new GregorianCalendar(2018, 1, 1, 10, 15, 20);
Date date = gregorianCalendar.getTime();

DateFormat ffInstance = DateFormat.getDateTimeInstance(
  DateFormat.FULL, DateFormat.FULL, Locale.ITALY);
DateFormat smInstance = DateFormat.getDateTimeInstance(
  DateFormat.SHORT, DateFormat.MEDIUM, Locale.ITALY);

assertEquals("giovedì 1 febbraio 2018 10.15.20 CET", ffInstance.format(date));
assertEquals("01/02/18 10.15.20", smInstance.format(date));
```

`DateFormat`与`Dates`一起工作，有三个有用的方法:

*   `getDateTimeInstance`
*   `getDateInstance`
*   `getTimeInstance`

它们都将预定义的值`DateFormat` 作为参数。每个方法都是重载的，所以传递`Locale`也是可能的。如果我们想使用自定义模式，就像在`DateTimeFormatter`中所做的那样，我们可以使用`SimpleDateFormat`。让我们来看一小段代码:

```java
GregorianCalendar gregorianCalendar = new GregorianCalendar(
  2018, 1, 1, 10, 15, 20);
Date date = gregorianCalendar.getTime();
Locale.setDefault(new Locale("pl", "PL"));

SimpleDateFormat fullMonthDateFormat = new SimpleDateFormat(
  "dd-MMMM-yyyy HH:mm:ss:SSS");
SimpleDateFormat shortMonthsimpleDateFormat = new SimpleDateFormat(
  "dd-MM-yyyy HH:mm:ss:SSS");

assertEquals(
  "01-lutego-2018 10:15:20:000", fullMonthDateFormat.format(date));
assertEquals(
  "01-02-2018 10:15:20:000" , shortMonthsimpleDateFormat.format(date));
```

## 5.用户化

由于一些好的设计决策，我们不局限于特定于地区的格式模式，我们可以配置几乎每个细节以完全满足输出。

**要自定义数字格式，我们可以使用`DecimalFormat`和`DecimalFormatSymbols`。**

让我们考虑一个简短的例子:

```java
Locale.setDefault(Locale.FRANCE);
BigDecimal number = new BigDecimal(102_300.456d);

DecimalFormat zeroDecimalFormat = new DecimalFormat("000000000.0000");
DecimalFormat dollarDecimalFormat = new DecimalFormat("$###,###.##");

assertEquals(zeroDecimalFormat.format(number), "000102300,4560");
assertEquals(dollarDecimalFormat.format(number), "$102 300,46"); 
```

[`DecimalFormat`文档](https://web.archive.org/web/20220908233401/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/text/DecimalFormat.html)显示了所有可能的模式字符。我们现在只需要知道“000000000.000”决定前导零或尾随零，“，”是千位分隔符，而“.”是十进制一。

也可以添加货币符号。我们可以在下面看到，使用`DateFormatSymbol`类可以获得相同的结果:

```java
Locale.setDefault(Locale.FRANCE);
BigDecimal number = new BigDecimal(102_300.456d);

DecimalFormatSymbols decimalFormatSymbols = DecimalFormatSymbols.getInstance();
decimalFormatSymbols.setGroupingSeparator('^');
decimalFormatSymbols.setDecimalSeparator('@');
DecimalFormat separatorsDecimalFormat = new DecimalFormat("$###,###.##");
separatorsDecimalFormat.setGroupingSize(4);
separatorsDecimalFormat.setCurrency(Currency.getInstance(Locale.JAPAN));
separatorsDecimalFormat.setDecimalFormatSymbols(decimalFormatSymbols);

assertEquals(separatorsDecimalFormat.format(number), "$10^[[email protected]](/web/20220908233401/https://www.baeldung.com/cdn-cgi/l/email-protection)");
```

正如我们所见，`DecimalFormatSymbols`类使我们能够指定我们能想象的任何数字格式。

**要自定义`SimpleDataFormat,` 我们可以用`DateFormatSymbols`** 。

让我们看看更改日期名称有多简单:

```java
Date date = new GregorianCalendar(2018, 1, 1, 10, 15, 20).getTime();
Locale.setDefault(new Locale("pl", "PL"));

DateFormatSymbols dateFormatSymbols = new DateFormatSymbols();
dateFormatSymbols.setWeekdays(new String[]{"A", "B", "C", "D", "E", "F", "G", "H"});
SimpleDateFormat newDaysDateFormat = new SimpleDateFormat(
  "EEEE-MMMM-yyyy HH:mm:ss:SSS", dateFormatSymbols);

assertEquals("F-lutego-2018 10:15:20:000", newDaysDateFormat.format(date));
```

## 6.资源包

最后，`JVM`中国际化的关键部分是`Resource Bundle` 机制。

`ResourceBundle`的目的是为应用程序提供本地化的消息/描述，这些消息/描述可以具体化为单独的文件。我们在之前的一篇文章中讨论了资源包的使用和配置——[资源包指南](/web/20220908233401/https://www.baeldung.com/java-resourcebundle)。

## 7.结论

利用它们的格式化程序是帮助我们创建国际化应用程序的工具。这些工具允许我们创建一个应用程序，它可以动态地适应用户的语言或文化设置，而不需要多次构建，甚至不需要担心 Java 是否支持`Locale`。

在这个世界上，用户可以在任何地方使用任何语言，应用这些变化的能力意味着我们的应用程序可以更直观，更容易被全球更多的用户理解。

在使用 Spring Boot 应用程序时，我们还有一篇关于 [Spring Boot 国际化](/web/20220908233401/https://www.baeldung.com/spring-boot-internationalization)的便利文章。

这篇教程的源代码，以及完整的例子，可以在 GitHub 上找到[。](https://web.archive.org/web/20220908233401/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-8)