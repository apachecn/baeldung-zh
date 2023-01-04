# 在 Spring Boot 格式化 JSON 日期

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-boot-formatting-json-dates>

## 1。概述

在本教程中，我们将展示如何在 Spring Boot 应用程序中格式化 JSON 日期字段。

我们将探索使用 [**Jackson**](/web/20220523233330/https://www.baeldung.com/jackson) 格式化日期的各种方法，Spring Boot 将其用作默认的 JSON 处理器。

## 2。在`Date`字段上使用`@JsonFormat`

### 2.1。设置格式

**我们可以使用 [`@JsonFormat`](/web/20220523233330/https://www.baeldung.com/jackson-jsonformat) 注释来格式化特定字段**:

```
public class Contact {

    // other fields

    @JsonFormat(pattern="yyyy-MM-dd")
    private LocalDate birthday;

    @JsonFormat(pattern="yyyy-MM-dd HH:mm:ss")
    private LocalDateTime lastUpdate;

    // standard getters and setters

}
```

在`birthday`字段中，我们使用一种只呈现日期的模式，而在`lastUpdate` 字段中，我们还包含时间。

我们使用了 [**Java 8 日期类型**](/web/20220523233330/https://www.baeldung.com/java-8-date-time-intro) ，这对于处理时态类型非常方便。

当然，如果我们需要使用遗留类型如`java.util.Date`，我们可以用同样的方式使用注释:

```
public class ContactWithJavaUtilDate {

     // other fields

     @JsonFormat(pattern="yyyy-MM-dd")
     private Date birthday;

     @JsonFormat(pattern="yyyy-MM-dd HH:mm:ss")
     private Date lastUpdate;

     // standard getters and setters
}
```

最后，让我们来看看使用给定日期格式的`@JsonFormat `呈现的输出:

```
{
    "birthday": "2019-02-03",
    "lastUpdate": "2019-02-03 10:08:02"
}
```

**正如我们所见，使用`@JsonFormat `注释是格式化特定日期字段的一种很好的方式。**

然而，我们应该只在需要特定的字段格式时才使用它。如果我们希望应用程序中的所有日期都有一个通用的格式，我们稍后会看到有更好的方法来实现这一点。

### 2.2。设置时区

如果我们需要使用特定的时区，我们可以设置@ `JsonFormat`的`timezone`属性:

```
@JsonFormat(pattern="yyyy-MM-dd HH:mm:ss", timezone="Europe/Zagreb")
private LocalDateTime lastUpdate;
```

如果一个类型已经包含了时区，我们就不需要使用它，比如用`java.time.ZonedDatetime`。

## 3。配置默认格式

虽然 `@JsonFormat`本身就很强大，但是硬编码格式和时区会让我们吃尽苦头。

如果我们想为应用程序中的所有日期配置一个默认格式，**更灵活的方法是在`application.properties`** 中配置它:

```
spring.jackson.date-format=yyyy-MM-dd HH:mm:ss
```

如果我们想在 JSON 日期中使用特定的时区，也有一个属性:

```
spring.jackson.time-zone=Europe/Zagreb
```

虽然像这样设置默认格式非常方便和简单，**这种方法有一个缺点。不幸的是，它不适用于 Java 8 日期类型**，比如`LocalDate `和`LocalDateTime`。我们只能用它来格式化类型为`java.util.Date`或`java.util.Calendar`的字段。尽管如此，还是有希望的，我们很快就会看到。

## 4。定制杰克森的`ObjectMapper`

所以，如果我们想用 Java 8 的日期类型`and `设置一个默认的日期格式，我们需要看看**创建的 [`Jackson2ObjectMapperBuilderCustomizer`](https://web.archive.org/web/20220523233330/https://docs.spring.io/spring-boot/docs/current/api/org/springframework/boot/autoconfigure/jackson/Jackson2ObjectMapperBuilderCustomizer.html) bean** :

```
@Configuration
public class ContactAppConfig {

    private static final String dateFormat = "yyyy-MM-dd";
    private static final String dateTimeFormat = "yyyy-MM-dd HH:mm:ss";

    @Bean
    public Jackson2ObjectMapperBuilderCustomizer jsonCustomizer() {
        return builder -> {
            builder.simpleDateFormat(dateTimeFormat);
            builder.serializers(new LocalDateSerializer(DateTimeFormatter.ofPattern(dateFormat)));
            builder.serializers(new LocalDateTimeSerializer(DateTimeFormatter.ofPattern(dateTimeFormat)));
        };
    }

}
```

上面的例子显示了如何在我们的应用程序中配置默认格式。我们必须定义一个 bean 并覆盖它的`customize `方法来设置所需的格式。

尽管这种方法看起来有点麻烦，但好处是它对 Java 8 和遗留日期类型都有效。

## 5。结论

在本文中，我们探索了在 Spring Boot 应用程序中格式化 JSON 日期的多种方法。

和往常一样，例子的源代码可以在 GitHub 上找到[。](https://web.archive.org/web/20220523233330/https://github.com/eugenp/tutorials/tree/master/spring-boot-modules/spring-boot-data)