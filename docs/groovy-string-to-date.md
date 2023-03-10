# 在 Groovy 中将字符串转换为日期

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/groovy-string-to-date>

## 1。概述

在这个简短的教程中，我们将学习如何在 [Groovy](/web/20220628121310/https://www.baeldung.com/groovy-language) 中将代表日期的`String`转换成真正的`Date`对象。

但是，我们应该记住，这种语言是 Java 的增强。因此，除了新的 Groovy 方法之外，我们仍然可以使用每一个普通的旧 Java 方法。

## 2。使用`DateFormat`

首先，我们可以像往常一样，使用 Java `DateFormat`将字符串解析成日期:

```java
def pattern = "yyyy-MM-dd"
def input = "2019-02-28"

def date = new SimpleDateFormat(pattern).parse(input) 
```

然而，Groovy 允许我们更容易地执行这个操作。**它在便利静态方法** `[Date.parse(String format, String input)](https://web.archive.org/web/20220628121310/http://docs.groovy-lang.org/2.5.6/html/api/org/apache/groovy/dateutil/extensions/DateUtilStaticExtensions.html#parse(java.util.Date,java.lang.String,java.lang.String))`里面封装了相同的行为:

```java
def date = Date.parse(pattern, input) 
```

简而言之，该方法是`java.util.Date`对象的扩展，为了线程安全，它在每次调用时在内部实例化一个`java.text.DateFormat` **。**

### 2.1。兼容性问题

澄清一下，`Date.parse(String format, String input)`方法从 Groovy 的 1.5.7 版本开始就可用了。

版本 2.4.1 引入了一个变体，接受第三个参数来指示时区:`Date.parse(String format, String input, TimeZone zone)`。

然而，从 2.5.0 开始，[有了一个突破性的变化](https://web.archive.org/web/20220628121310/http://groovy-lang.org/releasenotes/groovy-2.5.html#Groovy2.5releasenotes-Breakingchanges)，那些增强功能不再随`[groovy-all](https://web.archive.org/web/20220628121310/https://search.maven.org/search?q=a:groovy-all%20AND%20g:org.codehaus.groovy).`一起发布

所以，今后，它们需要作为一个单独的模块被包含进来，命名为 [`groovy-dateutil`](https://web.archive.org/web/20220628121310/https://search.maven.org/search?q=a:groovy-dateutil) :

```java
<dependency>
    <groupId>org.codehaus.groovy</groupId>
    <artifactId>groovy-dateutil</artifactId>
    <version>2.5.6</version>
</dependency> 
```

还有 3.0.0 版本，但目前处于 Alpha 阶段。

## 3。使用 JSR-310 `LocalDate`

从版本 8 开始，Java 引入了一套全新的工具来处理日期:[日期/时间 API](/web/20220628121310/https://www.baeldung.com/java-8-date-time-intro) 。

这些 API 更好有几个原因，而且**应该比传统的**更好。

让我们看看如何利用 Groovy 的`java.time.LocalDate`解析功能:

```java
def date = LocalDate.parse(input, pattern) 
```

## 4。结论

我们已经看到了如何在 Groovy 语言中将一个`String`转换成一个`Date`，并注意到了特定版本之间的特性。

和往常一样，源代码和单元测试可以在 [GitHub](https://web.archive.org/web/20220628121310/https://github.com/eugenp/tutorials/tree/master/core-groovy-modules/core-groovy/) 上获得。