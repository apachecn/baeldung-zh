# 如何在 Java 中找到异常的根本原因

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-exception-root-cause>

## 1。简介

在 Java 中使用嵌套异常很常见，因为它们可以帮助我们跟踪错误的来源。

当我们处理这类异常时，有时**我们可能想知道导致异常的原始问题，这样我们的应用程序就可以对每种情况做出不同的响应**。当我们使用将根异常封装到自己的框架中时，这尤其有用。

在这篇短文中，我们将展示如何使用普通 Java 以及外部库(如 [Apache Commons Lang](https://web.archive.org/web/20220525134741/https://commons.apache.org/proper/commons-lang/) 和 [Google Guava](https://web.archive.org/web/20220525134741/https://github.com/google/guava) )来获得根本原因异常。

## 2。一款年龄计算器 App

我们的应用程序将是一个年龄计算器，告诉我们一个人从一个给定的日期开始有多老，以 ISO 格式接收。我们将在解析日期时处理两种可能的错误情况:格式错误的日期和未来的日期。

让我们首先为我们的错误情况创建异常:

```java
static class InvalidFormatException extends DateParseException {

    InvalidFormatException(String input, Throwable thr) {
        super("Invalid date format: " + input, thr);
    }
}

static class DateOutOfRangeException extends DateParseException {

    DateOutOfRangeException(String date) {
        super("Date out of range: " + date);
    }

}
```

这两个异常都继承自一个公共的父异常，这将使我们的代码更加清晰:

```java
static class DateParseException extends RuntimeException {

    DateParseException(String input) {
        super(input);
    }

    DateParseException(String input, Throwable thr) {
        super(input, thr);
    }
}
```

之后，我们可以用解析日期的方法实现`AgeCalculator`类:

```java
static class AgeCalculator {

    private static LocalDate parseDate(String birthDateAsString) {
        LocalDate birthDate;
        try {
            birthDate = LocalDate.parse(birthDateAsString);
        } catch (DateTimeParseException ex) {
            throw new InvalidFormatException(birthDateAsString, ex);
        }

        if (birthDate.isAfter(LocalDate.now())) {
            throw new DateOutOfRangeException(birthDateAsString);
        }

        return birthDate;
    }
}
```

如我们所见，当格式错误时，我们将`DateTimeParseException`包装到我们的自定义`InvalidFormatException.`中

最后，让我们向我们的类添加一个公共方法，该方法接收日期，解析它，然后计算年龄:

```java
public static int calculateAge(String birthDate) {
    if (birthDate == null || birthDate.isEmpty()) {
        throw new IllegalArgumentException();
    }

    try {
        return Period
          .between(parseDate(birthDate), LocalDate.now())
          .getYears();
    } catch (DateParseException ex) {
        throw new CalculationException(ex);
    }
}
```

如图所示，我们再次包装了异常。在这种情况下，我们将它们包装到一个我们必须创建的`CalculationException`中:

```java
static class CalculationException extends RuntimeException {

    CalculationException(DateParseException ex) {
        super(ex);
    }
}
```

现在，我们已经准备好使用我们的计算器，以 ISO 格式传递任何日期:

```java
AgeCalculator.calculateAge("2019-10-01");
```

如果计算失败，知道问题出在哪里会很有用，不是吗？请继续阅读，了解我们如何做到这一点。

## 3。使用普通 Java 找到根本原因

我们将用来查找根本原因异常的第一种方法是通过**创建一个定制方法，该方法循环遍历所有原因，直到到达根本原因**:

```java
public static Throwable findCauseUsingPlainJava(Throwable throwable) {
    Objects.requireNonNull(throwable);
    Throwable rootCause = throwable;
    while (rootCause.getCause() != null && rootCause.getCause() != rootCause) {
        rootCause = rootCause.getCause();
    }
    return rootCause;
}
```

注意，我们在循环中添加了一个额外的条件，以避免在处理递归原因时出现无限循环。

如果我们传递一个无效的格式给我们的`AgeCalculator`，我们将得到`DateTimeParseException`作为根本原因:

```java
try {
    AgeCalculator.calculateAge("010102");
} catch (CalculationException ex) {
    assertTrue(findCauseUsingPlainJava(ex) instanceof DateTimeParseException);
}
```

然而，如果我们使用未来的日期，我们将得到一个`DateOutOfRangeException`:

```java
try {
    AgeCalculator.calculateAge("2020-04-04");
} catch (CalculationException ex) {
    assertTrue(findCauseUsingPlainJava(ex) instanceof DateOutOfRangeException);
}
```

此外，我们的方法也适用于非嵌套异常:

```java
try {
    AgeCalculator.calculateAge(null);
} catch (Exception ex) {
    assertTrue(findCauseUsingPlainJava(ex) instanceof IllegalArgumentException);
}
```

在这种情况下，我们得到一个`IllegalArgumentException`，因为我们传入了`null`。

## 4。使用 Apache Commons Lang 找到根本原因

我们现在将演示使用第三方库而不是编写我们的自定义实现来查找根本原因。

Apache Commons Lang 提供了一个`[ExceptionUtils](https://web.archive.org/web/20220525134741/https://github.com/apache/commons-lang/blob/master/src/main/java/org/apache/commons/lang3/exception/ExceptionUtils.java)`类，该类提供了一些处理异常的实用方法。

我们将在前面的例子中使用`getRootCause()` 方法:

```java
try {
    AgeCalculator.calculateAge("010102");
} catch (CalculationException ex) {
    assertTrue(ExceptionUtils.getRootCause(ex) instanceof DateTimeParseException);
}
```

我们得到了和以前一样的根本原因。同样的行为也适用于我们上面列出的其他例子。

## 5。使用番石榴找到根本原因

我们要尝试的最后一种方法是用番石榴。类似于 Apache Commons Lang，它提供了一个带有`getRootCause()` 实用方法的 [`Throwables`](https://web.archive.org/web/20220525134741/https://github.com/google/guava/blob/master/guava/src/com/google/common/base/Throwables.java) 类。

让我们用同一个例子来尝试一下:

```java
try {
    AgeCalculator.calculateAge("010102");
} catch (CalculationException ex) {
    assertTrue(Throwables.getRootCause(ex) instanceof DateTimeParseException);
}
```

行为与其他方法完全相同。

## 6。结论

在本文中，我们演示了如何在应用程序中使用嵌套异常，并实现了一个实用方法来查找异常的根本原因。

我们还展示了如何通过使用第三方库(如 Apache Commons Lang 和 Google Guava)来完成同样的工作。

与往常一样，GitHub 上的[提供了示例的完整源代码。](https://web.archive.org/web/20220525134741/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-exceptions-2)