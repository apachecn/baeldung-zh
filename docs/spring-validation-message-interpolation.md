# 弹簧验证消息插值

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-validation-message-interpolation>

## 1.介绍

消息插值是为 [Java bean 验证](/web/20220627181353/https://www.baeldung.com/javax-validation)约束创建错误消息的过程。例如，我们可以通过为用`javax.validation.constraints.NotNull`注释标注的字段提供一个`null`值来查看消息。

在本教程中，我们将学习如何使用默认的 Spring 消息插值以及如何创建我们自己的插值机制。

要查看除了`javax.validation`之外提供约束的其他库的例子，请看一下 [Hibernate Validator 特定约束](/web/20220627181353/https://www.baeldung.com/hibernate-validator-constraints)。我们还可以创建一个[定制的 Spring 验证注释](/web/20220627181353/https://www.baeldung.com/spring-mvc-custom-validator)。

## 2.默认消息插值

在进入代码片段之前，让我们考虑一个带有默认`@NotNull`约束违反消息的 HTTP 400 响应的例子:

```java
{
    ....
    "status": 400,
    "error": "Bad Request",
    "errors": [
        {
            ....
            "defaultMessage": "must not be null",
            ....
        }
    ],
    "message": "Validation failed for object='notNullRequest'. Error count: 1",
    ....
}
```

**Spring 从消息描述符中检索违反约束的消息细节。**每个约束使用`message`属性定义其默认消息描述符。但是，当然，我们可以用自定义值覆盖它。

作为一个例子，我们将使用 POST 方法创建一个简单的 REST 控制器:

```java
@RestController
public class RestExample {

    @PostMapping("/test-not-null")
    public void testNotNull(@Valid @RequestBody NotNullRequest request) {
        // ...
    }
}
```

请求体将被映射到`NotNullRequest`对象，该对象只有一个用`@NotNull`注释的`String`文件:

```java
public class NotNullRequest {

    @NotNull(message = "stringValue has to be present")
    private String stringValue;

    // getters, setters
}
```

现在，当我们发送一个没有通过验证检查的 POST 请求时，我们将看到我们的自定义错误消息:

```java
{
    ...
    "errors": [
        {
            ...
            "defaultMessage": "stringValue has to be present",
            ...
        }
    ],
    ...
}
```

唯一改变的值是`defaultMessage`。但是我们还是会得到很多关于错误码、对象名、字段名等信息。为了限制显示值的数量，我们可以为 REST API 实现[自定义错误消息处理。](/web/20220627181353/https://www.baeldung.com/global-error-handler-in-a-spring-rest-api)

## 3.消息表达式插值

在 Spring 中，我们可以使用统一表达式语言来定义我们的消息描述符。这允许基于条件逻辑定义**错误消息，也允许高级格式化选项**。

为了更清楚地理解它，我们来看几个例子。

在每个约束注释中，我们可以访问被验证的字段的实际值:

```java
@Size(
  min = 5,
  max = 14,
  message = "The author email '${validatedValue}' must be between {min} and {max} characters long"
)
private String authorEmail;
```

我们的错误消息将包含属性的实际值和`@Size`注释的`min`和`max`参数:

```java
"defaultMessage": "The author email '[[email protected]](/web/20220627181353/https://www.baeldung.com/cdn-cgi/l/email-protection)' must be between 5 and 14 characters long"
```

注意，对于访问外部变量，我们使用`${}`语法，但是对于从验证注释中访问其他属性，我们使用`{}`。

使用三元运算符也是可能的:

```java
@Min(
  value = 1,
  message = "There must be at least {value} test{value > 1 ? 's' : ''} in the test case"
)
private int testCount;
```

Spring 会将三元运算符转换为错误消息中的单个值:

```java
"defaultMessage": "There must be at least 2 tests in the test case"
```

我们也可以对外部变量调用方法:

```java
@DecimalMin(
  value = "50",
  message = "The code coverage ${formatter.format('%1$.2f', validatedValue)} must be higher than {value}%"
)
private double codeCoverage;
```

无效输入将产生一条带有格式化值的错误消息:

```java
"defaultMessage": "The code coverage 44.44 must be higher than 50%"
```

从这些例子中我们可以看到，在消息表达式中使用了一些字符如`{, }, $,`和`/`，所以我们需要在字面上使用它们之前用一个反斜杠字符对它们进行转义:`\{, \}, \$,`和`\\`。

## 4.自定义消息插值

在某些情况下，我们希望**实现一个定制的消息插值引擎**。为此，我们必须首先实现`javax.validation.MessageInterpolation`接口:

```java
public class MyMessageInterpolator implements MessageInterpolator {
    private final MessageInterpolator defaultInterpolator;

    public MyMessageInterpolator(MessageInterpolator interpolator) {
        this.defaultInterpolator = interpolator;
    }

    @Override
    public String interpolate(String messageTemplate, Context context) {
        messageTemplate = messageTemplate.toUpperCase();
        return defaultInterpolator.interpolate(messageTemplate, context);
    }

    @Override
    public String interpolate(String messageTemplate, Context context, Locale locale) {
        messageTemplate = messageTemplate.toUpperCase();
        return defaultInterpolator.interpolate(messageTemplate, context, locale);
    }
}
```

在这个简单的实现中，我们只是将错误消息改为大写。通过这样做，我们的错误消息将类似于:

```java
"defaultMessage": "THE CODE COVERAGE 44.44 MUST BE HIGHER THAN 50%"
```

我们还需要**在`javax.validation.Validation`工厂中注册我们的插补器**:

```java
Validation.byDefaultProvider().configure().messageInterpolator(
  new MyMessageInterpolator(
    Validation.byDefaultProvider().configure().getDefaultMessageInterpolator())
);
```

## 5.结论

在本文中，我们已经了解了默认的 Spring 消息内插是如何工作的，以及如何创建一个定制的消息内插引擎。

和往常一样，GitHub 上的所有源代码[都是可用的。](https://web.archive.org/web/20220627181353/https://github.com/eugenp/tutorials/tree/master/spring-web-modules/spring-mvc-basics-3)