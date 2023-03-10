# 用 JCommander 解析命令行参数

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/jcommander-parsing-command-line-parameters>

## 1.概观

在本教程中，**我们将学习如何使用 [JCommander](https://web.archive.org/web/20221208143830/https://github.com/cbeust/jcommander) 解析命令行参数。**在构建一个简单的命令行应用程序时，我们将探究它的几个特性。

## 2.为什么是 JCommander？

`“Because life is too short to parse command line parameters”`–塞德里克·贝乌斯特

JCommander 由 Cédric Beust 创建，是一个基于**注释的库，用于** **解析命令行参数**。它可以减少构建命令行应用程序的工作量，并帮助我们为他们提供良好的用户体验。

有了 JCommander，我们可以卸载一些棘手的任务，比如解析、验证和类型转换，让我们可以专注于应用程序逻辑。

## 3.设置 JCommander

### 3.1.Maven 配置

让我们首先在我们的`pom.xml`中添加 [`jcommander`](https://web.archive.org/web/20221208143830/https://search.maven.org/search?q=g:com.beust%20AND%20a:%20jcommander) 依赖项:

```java
<dependency>
    <groupId>com.beust</groupId>
    <artifactId>jcommander</artifactId>
    <version>1.78</version>
</dependency>
```

### 3.2.你好世界

让我们创建一个简单的`HelloWorldApp`，它接受一个名为`name`的输入，并打印一个问候语`“Hello <name>”`。

由于 **JCommander 将命令行参数绑定到 Java 类**中的字段，我们将首先定义一个带有字段`name`的`HelloWorldArgs`类，字段`name`用`@Parameter`注释:

```java
class HelloWorldArgs {

    @Parameter(
      names = "--name",
      description = "User name",
      required = true
    )
    private String name;
}
```

现在，让我们使用`JCommander`类来解析命令行参数，并在我们的`HelloWorldArgs`对象中分配字段:

```java
HelloWorldArgs jArgs = new HelloWorldArgs();
JCommander helloCmd = JCommander.newBuilder()
  .addObject(jArgs)
  .build();
helloCmd.parse(args);
System.out.println("Hello " + jArgs.getName());
```

最后，让我们从控制台用相同的参数调用主类:

```java
$ java HelloWorldApp --name JavaWorld
Hello JavaWorld
```

## 4.在 JCommander 中构建真正的应用程序

现在我们已经启动并运行了，让我们考虑一个更复杂的用例——一个命令行 API 客户端，它与一个计费应用程序如 [Stripe](/web/20221208143830/https://www.baeldung.com/java-stripe-api) 交互，特别是[计量](https://web.archive.org/web/20221208143830/https://stripe.com/docs/billing/subscriptions/metered-billing)(或基于使用的)计费场景。该第三方计费服务管理我们的订阅和发票。

假设我们在经营一家 SaaS 公司，我们的客户购买我们的服务，并按每月调用我们服务的 API 数量付费。我们将在客户机中执行两个操作:

*   `submit`:针对给定的订阅提交客户的使用数量和单价
*   `fetch`:根据客户当月部分或全部订阅的消费情况获取费用——我们可以将这些费用汇总到所有订阅中，或者按每个订阅逐项列出

我们将在浏览该库的特性时构建 API 客户端。

我们开始吧！

## 5.定义参数

让我们从定义应用程序可以使用的参数开始。

### 5.1.`@Parameter`注解

用 **[`@Parameter`](https://web.archive.org/web/20221208143830/https://jcommander.org/) 注释字段告诉 JCommander 将匹配的命令行参数绑定到它**。`@Parameter`有描述主参数的属性，如:

*   `names –`选项的一个或多个名称，例如“name”或“-n”
*   `description `–选项背后的含义，帮助最终用户
*   `required `–该选项是否为必填项，默认为`false`
*   `arity`–选项消耗的附加参数的数量

让我们在计量计费场景中配置一个参数`customerId`:

```java
@Parameter(
  names = { "--customer", "-C" },
  description = "Id of the Customer who's using the services",
  arity = 1,
  required = true
)
String customerId; 
```

现在，让我们使用新的“–customer”参数来执行命令:

```java
$ java App --customer cust0000001A
Read CustomerId: cust0000001A. 
```

同样，我们可以使用更短的"-C "参数来达到同样的效果:

```java
$ java App -C cust0000001A
Read CustomerId: cust0000001A. 
```

### 5.2.必需的参数

在参数是强制的情况下，如果用户没有指定参数，应用程序会抛出一个`ParameterException` :

```java
$ java App
Exception in thread "main" com.beust.jcommander.ParameterException:
  The following option is required: [--customer | -C]
```

我们要注意的是，一般情况下，**解析参数的任何错误都会导致 JCommander 中出现 [`ParameterException`](https://web.archive.org/web/20221208143830/https://jcommander.org/#_exception)** 。

## 6.内置类型

### 6.1.`IStringConverter`界面

JCommander 执行从命令行输入到参数类中的 Java 类型的类型转换。**[`IStringConverter`](https://web.archive.org/web/20221208143830/https://jcommander.org/#_custom_types_converters_and_splitters)接口处理参数从`String`到任意类型的类型转换。**所以，JCommander 的所有内置转换器都实现了这个接口。

JCommander 自带对常见数据类型的支持，如`String`、`Integer`、`Boolean`、`BigDecimal`和`Enum`。

### 6.2.单一 Arity 类型

Arity 与选项消耗的附加参数的数量有关。JCommander 的**内置参数类型默认有一个**的 arity，除了`Boolean`和`List.` ，所以常见的类型如`String`、`Integer`、`BigDecimal`、`Long,`和`Enum`都是单 arity 类型。

### 6.3.`Boolean`类型

**类型为`boolean`或`Boolean`的字段不需要任何额外的参数-**这些选项的`arity`为零。

让我们看一个例子。也许我们想获取客户的费用，按订阅逐项列出。我们可以添加一个`boolean`字段`itemized`，默认为`false`:

```java
@Parameter(
  names = { "--itemized" }
)
private boolean itemized; 
```

我们的应用程序将返回将`itemized`设置为`false`的总费用。当我们使用`itemized`参数调用命令行时，我们将该字段设置为`true`:

```java
$ java App --itemized
Read flag itemized: true. 
```

这很好地工作，除非我们有一个用例，我们总是想要逐项收费`,` ，除非另有说明。我们可以将参数改为`notItemized,`，但是提供`false`作为`itemized`的值可能会更清楚。

让我们通过使用字段的默认值`true`并将其`arity`设置为 1 来介绍这种行为:

```java
@Parameter(
  names = { "--itemized" },
  arity = 1
)
private boolean itemized = true; 
```

现在，当我们指定选项时，该值将被设置为`false`:

```java
$ java App --itemized false
Read flag itemized: false. 
```

## 7.`List`类型

JCommander 提供了几种将参数绑定到`List `字段的方法。

### 7.1.多次指定参数

假设我们只想获取客户订阅的一个子集的费用:

```java
@Parameter(
  names = { "--subscription", "-S" }
)
private List<String> subscriptionIds; 
```

该字段不是必需的，如果没有提供参数，应用程序将获取所有订阅的费用。然而，我们可以通过多次使用参数名来指定多个订阅**:**

```java
$ java App -S subscriptionA001 -S subscriptionA002 -S subscriptionA003
Read Subscriptions: [subscriptionA001, subscriptionA002, subscriptionA003]. 
```

### 7.2.使用拆分器绑定`Lists`

让我们尝试通过传递一个逗号分隔的`String`来绑定列表，而不是多次指定选项:

```java
$ java App -S subscriptionA001,subscriptionA002,subscriptionA003
Read Subscriptions: [subscriptionA001, subscriptionA002, subscriptionA003]. 
```

这使用单个参数值(arity = 1)来表示一个列表。JCommander 将使用类`CommaParameterSplitter`将逗号分隔的`String`绑定到我们的`List`。

### 7.3.使用自定义拆分器绑定`Lists`

我们可以通过实现`IParameterSplitter`接口来覆盖默认的拆分器:

```java
class ColonParameterSplitter implements IParameterSplitter {

    @Override
    public List split(String value) {
        return asList(value.split(":"));
    }
}
```

然后将实现映射到`@Parameter`中的`splitter`属性:

```java
@Parameter(
  names = { "--subscription", "-S" },
  splitter = ColonParameterSplitter.class
)
private List<String> subscriptionIds; 
```

让我们试一试:

```java
$ java App -S "subscriptionA001:subscriptionA002:subscriptionA003"
Read Subscriptions: [subscriptionA001, subscriptionA002, subscriptionA003]. 
```

### 7.4.变量 Arity `Lists`

**变量 arity 允许我们声明** **列表，这些列表可以接受不确定的参数，直到下一个选项**。我们可以将属性`variableArity`设置为`true`来指定这种行为。

让我们尝试这样来解析订阅:

```java
@Parameter(
  names = { "--subscription", "-S" },
  variableArity = true
)
private List<String> subscriptionIds; 
```

当我们运行命令时:

```java
$ java App -S subscriptionA001 subscriptionA002 subscriptionA003 --itemized
Read Subscriptions: [subscriptionA001, subscriptionA002, subscriptionA003]. 
```

JCommander 将选项“-S”后面的所有输入参数绑定到列表字段，直到下一个选项或命令结束。

### 7.5.固定 Arity `Lists`

到目前为止，我们已经看到了无界列表，我们可以传递任意多的列表项。有时，我们可能希望限制传递给`List`字段的项目数量。为此，我们可以**为`List`字段指定一个整数 arity 值，使其有界**:

```java
@Parameter(
  names = { "--subscription", "-S" },
  arity = 2
)
private List<String> subscriptionIds; 
```

固定 arity 强制检查传递给`List`选项的参数数量，并在违反时抛出`ParameterException`:

```java
$ java App -S subscriptionA001 subscriptionA002 subscriptionA003
Was passed main parameter 'subscriptionA003' but no main parameter was defined in your arg class 
```

错误消息表明，由于 JCommander 只需要两个参数，它试图将额外的输入参数“subscriptionA003”解析为下一个选项。

## 8.自定义类型

我们也可以通过编写自定义转换器来绑定参数。像内置转换器一样，定制转换器必须实现`IStringConverter`接口。

让我们编写一个转换器来解析一个 [ISO8601 时间戳](https://web.archive.org/web/20221208143830/https://en.wikipedia.org/wiki/ISO_8601#Combined_date_and_time_representations):

```java
class ISO8601TimestampConverter implements IStringConverter<Instant> {

    private static final DateTimeFormatter TS_FORMATTER = 
      DateTimeFormatter.ofPattern("uuuu-MM-dd'T'HH:mm:ss");

    @Override
    public Instant convert(String value) {
        try {
            return LocalDateTime
              .parse(value, TS_FORMATTER)
              .atOffset(ZoneOffset.UTC)
              .toInstant();
        } catch (DateTimeParseException e) {
            throw new ParameterException("Invalid timestamp");
        }
    }
} 
```

这段代码将解析输入`String`并返回一个`Instant`，如果有转换错误就抛出一个`ParameterException`。我们可以通过使用`@Parameter`中的`converter`属性将该转换器绑定到类型为 [`Instant`](https://web.archive.org/web/20221208143830/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/time/Instant.html) 的字段来使用它:

```java
@Parameter(
  names = { "--timestamp" },
  converter = ISO8601TimestampConverter.class
)
private Instant timestamp; 
```

让我们来看看它的实际应用:

```java
$ java App --timestamp 2019-10-03T10:58:00
Read timestamp: 2019-10-03T10:58:00Z.
```

## 9.验证参数

JCommander 提供了一些默认验证:

*   是否提供了所需的参数
*   如果指定的参数数量与字段的 arity 匹配
*   每个`String`参数是否可以转换成对应字段的类型

此外，**我们可能希望添加自定义验证**。例如，让我们假设客户 ID 必须是 [UUID](/web/20221208143830/https://www.baeldung.com/java-uuid) s

我们可以为客户字段编写一个验证器，实现接口 [`IParameterValidator`](https://web.archive.org/web/20221208143830/https://jcommander.org/#_parameter_validation) :

```java
class UUIDValidator implements IParameterValidator {

    private static final String UUID_REGEX = 
      "[0-9a-fA-F]{8}(-[0-9a-fA-F]{4}){3}-[0-9a-fA-F]{12}";

    @Override
    public void validate(String name, String value) throws ParameterException {
        if (!isValidUUID(value)) {
            throw new ParameterException(
              "String parameter " + value + " is not a valid UUID.");
        }
    }

    private boolean isValidUUID(String value) {
        return Pattern.compile(UUID_REGEX)
          .matcher(value)
          .matches();
    }
} 
```

然后，我们可以将它与参数的`validateWith`属性联系起来:

```java
@Parameter(
  names = { "--customer", "-C" },
  validateWith = UUIDValidator.class
)
private String customerId; 
```

如果我们使用非 UUID 客户 ID 调用该命令，应用程序将退出，并显示一条验证失败消息:

```java
$ java App --C customer001
String parameter customer001 is not a valid UUID. 
```

## 10.子命令

现在我们已经学习了参数绑定，让我们一起来构建我们的命令。

在 JCommander 中，我们可以支持多个命令，称为子命令，每个命令都有一组不同的选项。

### 10.1.`@Parameters`注释

我们可以用 [`@Parameters`](https://web.archive.org/web/20221208143830/https://jcommander.org/#_main_parameter) 来定义子命令。`@Parameters`包含属性`commandNames`来识别一个命令。

让我们将`submit`和`fetch`建模为子命令:

```java
@Parameters(
  commandNames = { "submit" },
  commandDescription = "Submit usage for a given customer and subscription, " +
    "accepts one usage item"
)
class SubmitUsageCommand {
    //...
}

@Parameters(
  commandNames = { "fetch" },
  commandDescription = "Fetch charges for a customer in the current month, " +
    "can be itemized or aggregated"
)
class FetchCurrentChargesCommand {
    //...
} 
```

JCommander 使用`@Parameters`中的属性来配置子命令，例如:

*   `commandNames`–子命令的名称；将命令行参数绑定到用`@Parameters`注释的类
*   `commandDescription`–记录子命令的目的

### 10.2.向`JCommander`添加子命令

我们用`addCommand`方法将子命令添加到`JCommander` :

```java
SubmitUsageCommand submitUsageCmd = new SubmitUsageCommand();
FetchCurrentChargesCommand fetchChargesCmd = new FetchCurrentChargesCommand();

JCommander jc = JCommander.newBuilder()
  .addCommand(submitUsageCmd)
  .addCommand(fetchChargesCmd)
  .build(); 
```

`addCommand`方法注册子命令及其各自的名称，如在`@Parameters`注释的`commandNames `属性中所指定的。

### 10.3.解析子命令

要访问用户选择的命令，我们必须首先解析参数:

```java
jc.parse(args); 
```

接下来，我们可以用`getParsedCommand`提取子命令:

```java
String parsedCmdStr = jc.getParsedCommand(); 
```

除了识别命令之外，JCommander 还将其余的命令行参数绑定到子命令中的字段。现在，我们只需调用我们想要使用的命令:

```java
switch (parsedCmdStr) {
    case "submit":
        submitUsageCmd.submit();
        break;

    case "fetch":
        fetchChargesCmd.fetch();
        break;

    default:
        System.err.println("Invalid command: " + parsedCmdStr);
} 
```

## 11.JCommander 用法帮助

我们可以调用`usage`来呈现使用指南。这是我们的应用程序使用的所有选项的汇总。在我们的应用程序中，我们可以调用主命令，或者分别调用两个命令“提交”和“获取”。

用法显示可以在几个方面帮助我们:显示帮助选项和在错误处理期间。

### 11.1.显示帮助选项

我们可以在命令中绑定一个帮助选项，使用一个`boolean`参数以及设置为`true`的属性`help`:

```java
@Parameter(names = "--help", help = true)
private boolean help; 
```

然后，我们可以检测参数中是否传递了“–help”，并调用`usage`:

```java
if (cmd.help) {
  jc.usage();
} 
```

让我们看看“提交”子命令的帮助输出:

```java
$ java App submit --help
Usage: submit [options]
  Options:
  * --customer, -C     Id of the Customer who's using the services
  * --subscription, -S Id of the Subscription that was purchased
  * --quantity         Used quantity; reported quantity is added over the 
                       billing period
  * --pricing-type, -P Pricing type of the usage reported (values: [PRE_RATED, 
                       UNRATED]) 
  * --timestamp        Timestamp of the usage event, must lie in the current 
                       billing period
    --price            If PRE_RATED, unit price to be applied per unit of 
                       usage quantity reported 
```

`usage`方法使用`@Parameter`属性，比如`description`来显示有用的摘要。标有星号(*)的参数是必需的。

### 11.2.错误处理

我们可以捕捉`ParameterException`并调用`usage` 来帮助用户理解为什么他们的输入不正确。`ParameterException`包含显示帮助的`JCommander`实例:

```java
try {
  jc.parse(args);

} catch (ParameterException e) {
  System.err.println(e.getLocalizedMessage());
  jc.usage();
} 
```

## 12.**结论**

在本教程中，我们使用 JCommander 构建了一个命令行应用程序。虽然我们涵盖了许多主要特性，但在官方[文档](https://web.archive.org/web/20221208143830/http://jcommander.org/)中还有更多。

像往常一样，所有例子的源代码都可以在 GitHub 的[上找到。](https://web.archive.org/web/20221208143830/https://github.com/eugenp/tutorials/tree/master/libraries-3)