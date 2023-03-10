# 在 LocalDate 和 SQL Date 之间转换

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-convert-localdate-sql-date>

## 1。概述

在这个快速教程中，我们将学习**如何在`java.time.LocalDate`和`java.sql.Date`T3 之间转换。**

## 2。直接转换

**到[从`LocalDate`](/web/20220630131815/https://www.baeldung.com/java-date-to-localdate-and-localdatetime) 转换到`java.sql.Date`，我们可以简单地使用** `**java.sql.Date**.`中可用的`valueOf()`方法。同样，要转换当前日期，我们可以使用:

```java
Date date = Date.valueOf(LocalDate.now());
```

或者，任何其他特定日期:

```java
Date date = Date.valueOf(LocalDate.of(2019, 01, 10));
```

此外，`valueOf()`在发生`null`争论时抛出`NullPointerException`。

现在，让我们从`java.sql.Date `转换到`LocalDate`。为此，我们可以使用`toLocalDate()`方法:

```java
LocalDate localDate = Date.valueOf("2019-01-10").toLocalDate();
```

## 3。使用`AttributeConverter`

首先，我们来了解一下问题。

Java 8 有很多有用的特性，包括[日期/时间 API](/web/20220630131815/https://www.baeldung.com/java-8-date-time-intro) 。

然而，在一些数据库或持久性框架中使用它需要比预期更多的工作。例如，JPA 会将`LocalDate`属性映射到一个 blob，而不是`java.sql.Date`对象。因此，数据库不会将`LocalDate`属性识别为`Date`类型。

一般来说，我们不想在`LocalDate`和`Date`之间执行显式转换。

例如，假设我们有一个带有`LocalDate`字段的实体对象。当持久化这个实体时，**我们需要告诉持久化上下文如何将`LocalDate`映射到** `**java.sql.Date**.`

让我们通过创建一个 [`AttributeConverter`](/web/20220630131815/https://www.baeldung.com/jpa-attribute-converters) 类来应用一个简单的解决方案:

```java
@Converter(autoApply = true)
public class LocalDateConverter implements AttributeConverter<LocalDate, Date> {

    @Override
    public Date convertToDatabaseColumn(LocalDate localDate) {
        return Optional.ofNullable(localDate)
          .map(Date::valueOf)
          .orElse(null);
    }

    @Override
    public LocalDate convertToEntityAttribute(Date date) {
        return Optional.ofNullable(date)
          .map(Date::toLocalDate)
          .orElse(null);
    }
}
```

正如我们所见，`AttributeConverter`接口接受两种类型:在我们的例子中是`LocalDate`和`Date`。

简而言之，`convertToDatabaseColumn()`和`convertToEntityAttribute()` 方法将负责转换过程。在实现内部，我们使用 [`Optional`](/web/20220630131815/https://www.baeldung.com/java-optional) 来轻松处理可能的`null`引用。

此外，我们还使用了`@Converter`注释。**使用`autoApply=true`属性，转换器将被应用于实体类型的所有映射属性。**

## 4。结论

在这个快速教程中，我们展示了两种在`java.time.LocalDate`和`java.sql.Date.`之间转换的方法，此外，我们展示了使用直接转换和使用自定义`AttributeConverter`类的例子。

和往常一样，这篇文章的完整代码可以在 GitHub 的[上找到。](https://web.archive.org/web/20220630131815/https://github.com/eugenp/tutorials/tree/master/persistence-modules/java-jpa)