# 检查字符串在 Java 中是否是有效的日期

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-string-valid-date>

## 1.概观

在本教程中，我们将讨论在 Java 中检查`String`是否包含有效日期的各种方法。

我们将查看 Java 8 之前、之后的解决方案，并使用 [Apache Commons Validator](https://web.archive.org/web/20221206093158/https://commons.apache.org/proper/commons-validator/) 。

## 2.日期验证概述

无论何时我们在任何应用程序中接收数据，我们都需要在做进一步处理之前验证它的有效性。

对于日期输入，我们可能需要验证以下内容:

*   输入包含有效格式的日期，如 MM/DD/YYYY。
*   输入的各个部分都在有效范围内。
*   输入解析为日历中的有效日期。

我们可以用[正则表达式](/web/20221206093158/https://www.baeldung.com/java-date-regular-expressions)来做上面的事情。然而，处理各种输入格式和地区的正则表达式既复杂又容易出错。它们还会降低性能。

我们将讨论以灵活、健壮和高效的方式实现日期验证的不同方法。

首先，让我们为日期验证编写一个接口:

```java
public interface DateValidator {
   boolean isValid(String dateStr);
}
```

在接下来的部分中，我们将使用各种方法实现这个接口。

## 3.使用`DateFormat`进行验证

Java 从一开始就提供了格式化和解析日期的工具。该功能在 [`DateFormat `](https://web.archive.org/web/20221206093158/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/text/DateFormat.html) 抽象类及其实现——[`SimpleDateFormat`](/web/20221206093158/https://www.baeldung.com/java-simple-date-format)。

让我们使用`DateFormat`类的`parse`方法来实现日期验证:

```java
public class DateValidatorUsingDateFormat implements DateValidator {
    private String dateFormat;

    public DateValidatorUsingDateFormat(String dateFormat) {
        this.dateFormat = dateFormat;
    }

    @Override
    public boolean isValid(String dateStr) {
        DateFormat sdf = new SimpleDateFormat(this.dateFormat);
        sdf.setLenient(false);
        try {
            sdf.parse(dateStr);
        } catch (ParseException e) {
            return false;
        }
        return true;
    }
}
```

由于**`DateFormat`和相关的类不是线程安全的**，我们为每个方法调用创建一个新的实例。

接下来，让我们为这个类编写单元测试:

```java
DateValidator validator = new DateValidatorUsingDateFormat("MM/dd/yyyy");

assertTrue(validator.isValid("02/28/2019"));        
assertFalse(validator.isValid("02/30/2019"));
```

这是 Java 8 之前最常见的解决方案。

## 4.使用`LocalDate`进行验证

Java 8 引入了一个改进的日期和时间 API 。它增加了 **[`LocalDate`](https://web.archive.org/web/20221206093158/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/time/LocalDate.html) 类，表示没有时间的日期。这个类是不可变的和线程安全的。**

`LocalDate`提供了两个静态方法来解析日期，并且都使用一个 [`DateTimeFormatter`](/web/20221206093158/https://www.baeldung.com/java-datetimeformatter) 来做实际的解析:

```java
public static LocalDate parse​(CharSequence text)
// parses dates using using DateTimeFormatter.ISO_LOCAL_DATE

public static LocalDate parse​(CharSequence text, DateTimeFormatter formatter)
// parses dates using the provided formatter
```

让我们使用`parse`方法来实现日期验证:

```java
public class DateValidatorUsingLocalDate implements DateValidator {
    private DateTimeFormatter dateFormatter;

    public DateValidatorUsingLocalDate(DateTimeFormatter dateFormatter) {
        this.dateFormatter = dateFormatter;
    }

    @Override
    public boolean isValid(String dateStr) {
        try {
            LocalDate.parse(dateStr, this.dateFormatter);
        } catch (DateTimeParseException e) {
            return false;
        }
        return true;
    }
}
```

该实现使用一个`DateTimeFormatter`对象进行格式化。因为这个类是线程安全的，所以我们在不同的方法调用中使用同一个实例。

让我们也为这个实现添加一个单元测试:

```java
DateTimeFormatter dateFormatter = DateTimeFormatter.BASIC_ISO_DATE;
DateValidator validator = new DateValidatorUsingLocalDate(dateFormatter);

assertTrue(validator.isValid("20190228"));
assertFalse(validator.isValid("20190230"));
```

## 5.使用`DateTimeFormatter`进行验证

在上一节中，我们看到`LocalDate`使用一个`DateTimeFormatter`对象进行解析。我们也可以直接使用`DateTimeFormatter`类进行格式化和解析。

**`DateTimeFormatter `分两个阶段解析文本。**在第 1 阶段，它根据配置将文本解析成不同的日期和时间字段。在第 2 阶段，它将解析后的字段解析为日期和/或时间对象。

*ResolverStyle* 属性控制阶段 2。它是一个有三个可能值的`enum`:

*   宽松–宽松地解决日期和时间
*   智能–以智能的方式解析日期和时间
*   严格–严格解析日期和时间

现在让我们直接使用`DateTimeFormatter`编写日期验证:

```java
public class DateValidatorUsingDateTimeFormatter implements DateValidator {
    private DateTimeFormatter dateFormatter;

    public DateValidatorUsingDateTimeFormatter(DateTimeFormatter dateFormatter) {
        this.dateFormatter = dateFormatter;
    }

    @Override
    public boolean isValid(String dateStr) {
        try {
            this.dateFormatter.parse(dateStr);
        } catch (DateTimeParseException e) {
            return false;
        }
        return true;
    }
}
```

接下来，让我们为这个类添加单元测试:

```java
DateTimeFormatter dateFormatter = DateTimeFormatter.ofPattern("uuuu-MM-dd", Locale.US)
    .withResolverStyle(ResolverStyle.STRICT);
DateValidator validator = new DateValidatorUsingDateTimeFormatter(dateFormatter);

assertTrue(validator.isValid("2019-02-28"));
assertFalse(validator.isValid("2019-02-30"));
```

在上面的测试中，我们基于模式和地区创建了一个`DateTimeFormatter` 。我们对日期使用严格的解析。

## 6.使用 Apache Commons 验证器进行验证

Apache Commons 项目提供了一个验证框架。**这包含验证例程，如日期、时间、数字、货币、IP 地址、电子邮件和 URL。**

对于本文，让我们看看 [`GenericValidator`](https://web.archive.org/web/20221206093158/https://commons.apache.org/proper/commons-validator/apidocs/org/apache/commons/validator/GenericValidator.html) 类，它提供了两种方法来检查`String`是否包含有效日期:

```java
public static boolean isDate(String value, Locale locale)

public static boolean isDate(String value,String datePattern, boolean strict)
```

要使用这个库，让我们将 [`commons-validator`](https://web.archive.org/web/20221206093158/https://search.maven.org/search?q=g:commons-validator%20AND%20a:commons-validator) Maven 依赖项添加到我们的项目中:

```java
<dependency>
    <groupId>commons-validator</groupId>
    <artifactId>commons-validator</artifactId>
    <version>1.6</version>
</dependency>
```

接下来，让我们使用`GenericValidator`类来验证日期:

```java
assertTrue(GenericValidator.isDate("2019-02-28", "yyyy-MM-dd", true));
assertFalse(GenericValidator.isDate("2019-02-29", "yyyy-MM-dd", true));
```

## 7.结论

在本文中，我们研究了检查`String`是否包含有效日期的各种方法。

像往常一样，完整的源代码可以在 GitHub 上找到[。](https://web.archive.org/web/20221206093158/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-datetime-string)