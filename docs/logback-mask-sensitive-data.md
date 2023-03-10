# 使用日志回溯屏蔽日志中的敏感数据

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/logback-mask-sensitive-data>

## 1.概观

随着大量数据被记录，在记录时屏蔽用户的敏感细节是很重要的。在新的 GDPR 时代，在众多问题中，我们必须特别注意记录个人的敏感数据。

在本教程中，我们将了解如何使用 Logback 来屏蔽日志中的敏感数据。总的来说，这种方法不是解决问题的真正方法——它是我们日志文件的最后一道防线。

## 2.回溯

[Logback](/web/20221208143830/https://www.baeldung.com/logback) 是 Java 社区中使用最广泛的日志框架之一。它是其前身 Log4j 的替代产品。它提供了比 Log4j 更快的实现，并且在归档旧日志文件时提供了更多的配置选项和更大的灵活性。

敏感数据是任何旨在防止未经授权的访问的信息。这可能包括从个人身份信息(PII)到银行信息、登录凭证、地址、电子邮件等任何信息。

在我们的应用程序中登录时，我们将屏蔽属于用户的敏感数据。

## 3.屏蔽数据

假设我们在 web 请求的上下文中记录用户详细信息。我们需要屏蔽与用户相关的敏感数据。假设我们的应用程序收到了我们记录的以下请求或响应:

```java
{
    "user_id":"87656",
    "ssn":"786445563",
    "address":"22 Street",
    "city":"Chicago",
    "Country":"U.S.",
    "ip_address":"192.168.1.1",
    "email_id":"[[email protected]](/web/20221208143830/https://www.baeldung.com/cdn-cgi/l/email-protection)"
 }
```

在这里，我们可以看到我们有敏感数据，如`ssn`、`address`、`ip_address`和`email_id`。因此，我们必须在记录时屏蔽这些数据。

我们将通过为 Logback 生成的所有日志条目配置屏蔽规则来集中屏蔽日志。**为了做到这一点，我们必须实现一个自定义`ch.qos.logback.classic.PatternLayout`** 。

### 3.1.`PatternLayout`

这个配置背后的想法是**用一个定制的布局**扩展我们需要的每一个 Logback appender。在我们的例子中，我们将编写一个`MaskingPatternLayout`类作为`PatternLayout.`的实现，每个掩码模式代表匹配一种类型的敏感数据的正则表达式。

让我们构建`MaskingPatternLayout` 类:

```java
public class MaskingPatternLayout extends PatternLayout {

    private Pattern multilinePattern;
    private List<String> maskPatterns = new ArrayList<>();

    public void addMaskPattern(String maskPattern) {
        maskPatterns.add(maskPattern);
        multilinePattern = Pattern.compile(maskPatterns.stream().collect(Collectors.joining("|")), Pattern.MULTILINE);
    }

    @Override
    public String doLayout(ILoggingEvent event) {
        return maskMessage(super.doLayout(event));
    }

    private String maskMessage(String message) {
        if (multilinePattern == null) {
            return message;
        }
        StringBuilder sb = new StringBuilder(message);
        Matcher matcher = multilinePattern.matcher(sb);
        while (matcher.find()) {
            IntStream.rangeClosed(1, matcher.groupCount()).forEach(group -> {
                if (matcher.group(group) != null) {
                    IntStream.range(matcher.start(group), matcher.end(group)).forEach(i -> sb.setCharAt(i, '*'));
                }
            });
        }
        return sb.toString();
    }
}
```

`PatternLayout`的实现。`doLayout()`负责屏蔽我们的应用程序的每个日志消息中匹配的数据，如果它匹配一个配置的模式。

来自 `logback.xml` 的`maskPatterns` 列表构建了一个多行模式。遗憾的是，日志回溯引擎不支持构造函数注入。如果它是一个属性列表，那么每个配置条目都会调用`addMaskPattern`。因此，我们必须在每次向列表中添加新的正则表达式时编译模式。

### 3.2.配置

一般来说，我们可以**使用正则表达式模式来屏蔽敏感的用户信息**。

例如，对于 SSN，我们可以使用这样的正则表达式:

```java
\"SSN\"\s*:\s*\"(.*)\"
```

对于地址，我们可以使用:

```java
\"address\"\s*:\s*\"(.*?)\" 
```

此外，对于 IP 地址数据模式(192.169.0.1)，我们可以使用正则表达式:

```java
(\d+\.\d+\.\d+\.\d+)
```

最后，对于电子邮件，我们可以这样写:

```java
(\[[email protected]](/web/20221208143830/https://www.baeldung.com/cdn-cgi/l/email-protection)\w+\.\w+)
```

现在，我们将把这些正则表达式模式添加到我们的`logback.xml`文件中的`maskPattern`标签中:

```java
<configuration>
    <appender name="mask" class="ch.qos.logback.core.ConsoleAppender">
        <encoder class="ch.qos.logback.core.encoder.LayoutWrappingEncoder">
           <layout class="com.baeldung.logback.MaskingPatternLayout">
	       <maskPattern>\"SSN\"\s*:\s*\"(.*?)\"</maskPattern> <!-- SSN JSON pattern -->
	       <maskPattern>\"address\"\s*:\s*\"(.*?)\"</maskPattern> <!-- Address JSON pattern -->
	       <maskPattern>(\d+\.\d+\.\d+\.\d+)</maskPattern> <!-- Ip address IPv4 pattern -->
	       <maskPattern>(\[[email protected]](/web/20221208143830/https://www.baeldung.com/cdn-cgi/l/email-protection)\w+\.\w+)</maskPattern> <!-- Email pattern -->
	       <pattern>%-5p [%d{ISO8601,UTC}] [%thread] %c: %m%n%rootException</pattern>
            </layout>
        </encoder>
    </appender>
</ configuration>
```

### 3.3.执行

现在，我们将为上面的例子创建 JSON，并使用`logger.info()`来记录细节:

```java
Map<String, String> user = new HashMap<String, String>();
user.put("user_id", "87656");
user.put("SSN", "786445563");
user.put("address", "22 Street");
user.put("city", "Chicago");
user.put("Country", "U.S.");
user.put("ip_address", "192.168.1.1");
user.put("email_id", "[[email protected]](/web/20221208143830/https://www.baeldung.com/cdn-cgi/l/email-protection)");
JSONObject userDetails = new JSONObject(user);

logger.info("User JSON: {}", userDetails);
```

执行此操作后，我们可以看到输出:

```java
INFO  [2021-06-01 16:04:12,059] [main] com.baeldung.logback.MaskingPatternLayoutExample: User JSON: 
{"email_id":"*******************","address":"*********","user_id":"87656","city":"Chicago","Country":"U.S.", "ip_address":"***********","SSN":"*********"} 
```

在这里，我们可以看到记录器中的用户 JSON 被屏蔽了:

```java
{
    "user_id":"87656",
    "ssn":"*********",
    "address":"*********",
    "city":"Chicago",
    "Country":"U.S.",
    "ip_address":"*********",
    "email_id":"*****************"
 }
```

使用这种方法，我们只能屏蔽日志文件中那些我们已经在`logback.xml`的`maskPattern`中定义了正则表达式的数据。

## 4.结论

在本教程中，我们介绍了如何使用`PatternLayout `特性通过 Logback 屏蔽应用程序日志中的敏感数据。此外，我们看到了如何在`logback.xml`中添加正则表达式模式来屏蔽特定的数据。

像往常一样，代码片段可以在 GitHub 上找到[。](https://web.archive.org/web/20221208143830/https://github.com/eugenp/tutorials/tree/master/logging-modules/logback)