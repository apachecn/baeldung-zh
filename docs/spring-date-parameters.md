# 在 Spring 中使用日期参数

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-date-parameters>

## 1.介绍

在这个简短的教程中，我们将学习如何在 Spring REST 请求中接受`Date`、`LocalDate,`和`LocalDateTime`参数，包括请求级和应用级。

## 2.问题是

让我们考虑一个控制器，它有三种方法接受`Date`、`LocalDate`和`LocalDateTime`参数:

```java
@RestController
public class DateTimeController {

    @PostMapping("/date")
    public void date(@RequestParam("date") Date date) {
        // ...
    }

    @PostMapping("/localdate")
    public void localDate(@RequestParam("localDate") LocalDate localDate) {
        // ...
    }

    @PostMapping("/localdatetime")
    public void dateTime(@RequestParam("localDateTime") LocalDateTime localDateTime) {
        // ...
    }
}
```

当向这些方法中的任何一个发送 POST 请求时，如果其中的参数格式符合 ISO 8601，我们将会得到一个异常。

例如，当向`/date`端点发送“2018-10-22”时，我们会得到一个错误的请求错误，消息如下:

```java
Failed to convert value of type 'java.lang.String' to required type 'java.time.LocalDate'; 
  nested exception is org.springframework.core.convert.ConversionFailedException.
```

这是因为默认情况下，Spring 不能将字符串参数转换为任何日期或时间对象。

## 3.在请求级别转换日期参数

处理这个问题的方法之一是用`@DateTimeFormat `注释来注释参数，并提供一个格式化模式参数:

```java
@RestController
public class DateTimeController {

    @PostMapping("/date")
    public void date(@RequestParam("date") 
      @DateTimeFormat(iso = DateTimeFormat.ISO.DATE) Date date) {
        // ...
    }

    @PostMapping("/local-date")
    public void localDate(@RequestParam("localDate") 
      @DateTimeFormat(iso = DateTimeFormat.ISO.DATE) LocalDate localDate) {
        // ...
    }

    @PostMapping("/local-date-time")
    public void dateTime(@RequestParam("localDateTime") 
      @DateTimeFormat(iso = DateTimeFormat.ISO.DATE_TIME) LocalDateTime localDateTime) {
        // ...
    }
}
```

这样，如果字符串使用 ISO 8601 格式进行格式化，字符串将被正确地转换为日期对象。

我们也可以通过在`@DateTimeFormat`注释中提供一个模式参数来使用我们自己的转换模式:

```java
@PostMapping("/date")
public void date(@RequestParam("date") 
  @DateTimeFormat(pattern = "dd.MM.yyyy") Date date) {
    // ...
}
```

## 4.在应用程序级别转换日期参数

Spring 中处理日期和时间对象转换的另一种方式是提供一个全局配置。通过遵循[官方文档](https://web.archive.org/web/20221023123327/https://docs.spring.io/spring/docs/current/spring-framework-reference/core.html#format-configuring-formatting-globaldatetimeformat)，我们应该扩展`WebMvcConfigurationSupport`配置及其`mvcConversionService`方法:

```java
@Configuration
public class DateTimeConfig extends WebMvcConfigurationSupport {

    @Bean
    @Override
    public FormattingConversionService mvcConversionService() {
        DefaultFormattingConversionService conversionService = new DefaultFormattingConversionService(false);

        DateTimeFormatterRegistrar dateTimeRegistrar = new DateTimeFormatterRegistrar();
        dateTimeRegistrar.setDateFormatter(DateTimeFormatter.ofPattern("dd.MM.yyyy"));
        dateTimeRegistrar.setDateTimeFormatter(DateTimeFormatter.ofPattern("dd.MM.yyyy HH:mm:ss"));
        dateTimeRegistrar.registerFormatters(conversionService);

        DateFormatterRegistrar dateRegistrar = new DateFormatterRegistrar();
        dateRegistrar.setFormatter(new DateFormatter("dd.MM.yyyy"));
        dateRegistrar.registerFormatters(conversionService);

        return conversionService;
    }
}
```

首先，我们用 false 参数创建`DefaultFormattingConversionService`，这意味着 Spring 默认不会注册任何格式化程序。

然后，我们需要注册日期和日期时间参数的自定义格式。我们通过注册两个自定义格式注册器来实现这一点。第一个，`DateTimeFormatterRegistar,`将负责解析`LocalDate`和`LocaDateTime`对象。第二个，`DateFormattingRegistrar,`将处理`Date`对象。

## 5.在属性文件中配置日期时间

Spring 还为我们提供了通过应用程序属性文件设置全局日期时间格式的选项。日期、日期时间和时间格式有三个单独的参数:

```java
spring.mvc.format.date=yyyy-MM-dd
spring.mvc.format.date-time=yyyy-MM-dd HH:mm:ss
spring.mvc.format.time=HH:mm:ss
```

所有这些参数都可以用一个`iso`值代替。例如，将日期时间参数设置为:

```java
spring.mvc.format.date-time=iso
```

将等同于 ISO-8601 格式:

```java
spring.mvc.format.date-time=yyyy-MM-dd HH:mm:ss
```

## 6.结论

在本文中，我们学习了如何在 Spring MVC 请求中接受日期参数。我们讨论了如何在全局范围内根据每个请求实现这一点。

我们还学习了如何创建自己的日期格式模式。

和往常一样，GitHub 上的所有源代码[都是可用的。](https://web.archive.org/web/20221023123327/https://github.com/eugenp/tutorials/tree/master/spring-web-modules/spring-mvc-java-2)