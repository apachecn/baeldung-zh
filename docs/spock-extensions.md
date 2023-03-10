# Spock 扩展指南

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spock-extensions>

## 1.概观

在本教程中，我们将看看 [Spock](/web/20221226061918/https://www.baeldung.com/groovy-spock) 扩展。

有时，我们可能需要修改或增强我们规范的生命周期。例如，我们想要添加一些条件执行，在随机失败的集成测试上重试，等等。为此，我们可以使用斯波克的扩展机制。

**`Spock `有各种各样的扩展**，我们可以把它们挂在一个规范的生命周期上。

让我们来看看如何使用最常见的扩展。

## 2.Maven 依赖性

在我们开始之前，让我们设置一下我们的 [Maven 依赖关系](https://web.archive.org/web/20221226061918/https://search.maven.org/classic/#search%7Cga%7C1%7C%20(g%3A%22org.spockframework%22%20AND%20a%3A%22spock-core%22)%20OR%20(g%3A%22org.codehaus.groovy%22%20AND%20a%3A%22groovy-all%22)):

```java
<dependency>
    <groupId>org.spockframework</groupId>
    <artifactId>spock-core</artifactId>z
    <version>1.3-groovy-2.4</version>
    <scope>test</scope>
</dependency>
<dependency>
    <groupId>org.codehaus.groovy</groupId>
    <artifactId>groovy-all</artifactId>
    <version>2.4.7</version>
    <scope>test</scope>
</dependency>
```

## 3.基于注释的扩展

**`Spock`的内置扩展大部分都是基于注释的。**

我们可以在等级库类或特征上添加注释来触发特定的行为。

### 3.1.`@Ignore`

有时我们需要忽略一些特性方法或规范类。比如，我们可能需要尽快合并我们的变更，但是持续集成仍然失败。我们可以忽略一些规格，仍然可以成功合并。

**我们可以在方法级别使用`@Ignore` 来跳过一个单独的规格说明方法:**

```java
@Ignore
def "I won't be executed"() {
    expect:
    true
}
```

**Spock 不会执行这个测试方法。**并且大多数 ide 会将测试标记为`skipped`。

此外，我们可以在类级别使用`@Ignore `:

```java
@Ignore
class IgnoreTest extends Specification
```

我们可以简单地**提供为什么**我们的测试套件或方法被忽略的原因:

```java
@Ignore("probably no longer needed")
```

### 3.2.`@IgnoreRest`

同样，我们可以忽略所有规范，只有一个除外，我们可以用`@IgnoreRest` 注释来标记它:

```java
def "I won't run"() { }

@IgnoreRest
def 'I will run'() { }

def "I won't run too"() { }
```

### 3.3.`@IgnoreIf`

有时，我们希望有条件地忽略一两个测试。在这种情况下，**我们可以使用 `@IgnoreIf,` ，它接受一个谓词**作为参数:

```java
@IgnoreIf({System.getProperty("os.name").contains("windows")})
def "I won't run on windows"() { }
```

Spock 提供了一组属性和助手类，使我们的谓词更容易读写:

*   `os `–关于操作系统的信息(参见`spock.util.environment.OperatingSystem`)。
*   `jvm`–JVM 的信息(见`spock.util.environment.Jvm`)。
*   `sys `–地图中的系统属性。
*   `env`–地图中的环境变量。

我们可以在使用`os `属性的过程中重写前面的例子。实际上，它是带有一些有用方法的`spock.util.environment.OperatingSystem`类，例如`isWindows()`:

```java
@IgnoreIf({ os.isWindows() })
def "I'm using Spock helper classes to run only on windows"() {}
```

**注意，`Spock `使用`System.getProperty(…) `引擎盖下。主要目标是提供一个清晰的界面，而不是定义复杂的规则和条件。**

同样，和前面的例子一样，我们可以在类级别应用`@IgnoreIf`注释。

### 3.4.`@Requires`

有时，从`@IgnoreIf.` 反转我们的谓词逻辑更容易，在这种情况下，我们可以使用`@Requires`:

```java
@Requires({ System.getProperty("os.name").contains("windows") })
def "I will run only on Windows"()
```

因此， **`@Requires`只在操作系统是 Windows** 时才运行测试，而使用相同谓词的`@IgnoreIf, `只在操作系统是 Windows`not`时才运行测试。

一般来说， **最好是说在什么条件下测试将被执行，而不是当它被忽略的时候**。

### 3.5.`@PendingFeature`

在`TDD, `中，我们首先编写测试。然后，我们需要编写一个代码来通过这些测试。在某些情况下，我们需要在特性实现之前提交我们的测试。

这是`@PendingFeature:`的一个很好的用例

```java
@PendingFeature
def 'test for not implemented yet feature. Maybe in the future it will pass'()
```

`@Ignore`和`@PendingFeature`有一个主要区别。在`@PedingFeature, `中，测试被执行，但是任何失败都被忽略。

**如果标有`@PendingFeature `的测试无错误结束，则报告为失败，提醒移除注释。**

这样，我们最初可以忽略未实现特性的失败，但是在将来，这些规格将成为正常测试的一部分，而不是永远被忽略。

### 3.6.`@Stepwise`

我们可以用`@Stepwise`注释按照给定的顺序执行规范的方法:

```java
def 'I will run as first'() { }

def 'I will run as second'() { }
```

一般来说，测试应该是确定性的。一个人不应该依赖另一个人。这就是为什么我们应该避免使用`@Stepwise `注释。

但是如果我们必须这么做，我们需要意识到 **`@Stepwise`不会覆盖`@Ignore`、`@IgnoreRest`或** **`@IgnoreIf`** 的行为。我们应该小心地将这些注释与`@Stepwise`结合起来。

### 3.7.`@Timeout`

**我们可以限制一个 spec 的单个方法的执行时间，让它更早失败:**

```java
@Timeout(1)
def 'I have one second to finish'() { }
```

注意，这是单次迭代的超时，不包括在 fixture 方法中花费的时间。

默认情况下，`spock.lang.Timeout`使用秒作为基本时间单位。但是，**我们可以指定其他时间单位:**

```java
@Timeout(value = 200, unit = TimeUnit.SECONDS)
def 'I will fail after 200 millis'() { }
```

`@Timeout`在类级别上具有与将其分别应用于每个特性方法相同的效果:

```java
@Timeout(5)
class ExampleTest extends Specification {

    @Timeout(1)
    def 'I have one second to finish'() {

    }

    def 'I will have 5 seconds timeout'() {}
}
```

**在单个 spec 方法上使用`@Timeout`总是覆盖类级别。**

### 3.8.`@Retry`

有时候，我们会有一些非确定性的集成测试。由于异步处理或依赖于其他`HTTP`客户端的响应等原因，这些可能会在某些运行中失败。此外，带有 build 和 CI 的远程服务器将会失败，并迫使我们再次运行测试和构建。

为了避免这种情况，**我们可以在方法或类级别上使用`@Retry `注释，来重复失败的测试**:

```java
@Retry
def 'I will retry three times'() { }
```

默认情况下，它将重试三次。

确定我们应该重试测试的条件是非常有用的。我们可以指定例外列表:

```java
@Retry(exceptions = [RuntimeException])
def 'I will retry only on RuntimeException'() { }
```

或者出现特定异常消息时:

```java
@Retry(condition = { failure.message.contains('error') })
def 'I will retry with a specific message'() { }
```

延迟重试非常有用:

```java
@Retry(delay = 1000)
def 'I will retry after 1000 millis'() { }
```

最后，像通常一样，我们可以在类级别指定重试:

```java
@Retry
class RetryTest extends Specification
```

### 3.9.`@RestoreSystemProperties`

我们可以用`@RestoreSystemProperties`操纵环境变量。

应用该注释时，保存变量的当前状态，并在以后恢复它们。它还包括`setup`或`cleanup`方法:

```java
@RestoreSystemProperties
def 'all environment variables will be saved before execution and restored after tests'() {
    given:
    System.setProperty('os.name', 'Mac OS')
}
```

请注意，当我们操作系统属性时，我们不应该同时运行测试。我们的测试可能是不确定的。

### 3.10.人性化的标题

我们可以通过使用`@Title`注释来添加一个人性化的测试标题:

```java
@Title("This title is easy to read for humans")
class CustomTitleTest extends Specification
```

类似地，我们可以添加带有`@Narrative`注释和多行`Groovy S`字符串的规范描述:

```java
@Narrative("""
    as a user
    i want to save favourite items 
    and then get the list of them
""")
class NarrativeDescriptionTest extends Specification
```

### 3.11.`@See`

要链接一个或多个外部引用，我们可以使用` @See`注释:

```java
@See("https://example.org")
def 'Look at the reference'()
```

为了传递多个链接，我们可以使用 Groovy `[]`操作数来创建一个列表:

```java
@See(["https://example.org/first", "https://example.org/first"])
def 'Look at the references'()
```

### 3.12.`@Issue`

我们可以表示一个特征方法指的是一个或多个问题:

```java
@Issue("https://jira.org/issues/LO-531")
def 'single issue'() {

}

@Issue(["https://jira.org/issues/LO-531", "http://jira.org/issues/LO-123"])
def 'multiple issues'()
```

### 3.13.`@Subject`

最后，我们可以用`@Subject`指出哪个类是测试中的类:

```java
@Subject
ItemService itemService // initialization here...
```

目前，这只是为了提供信息。

## 4.配置扩展

我们可以在 Spock 配置文件中配置一些扩展。这包括描述每个扩展应该如何表现。

通常，我们在 Groovy `, `中创建一个配置文件，比如叫做`SpockConfig.groovy`。

当然，**斯波克需要找到我们的配置文件。**首先，它从`spock.configuration `系统属性中读取一个自定义位置，然后尝试在类路径中查找文件。如果找不到，它会转到文件系统中的某个位置。如果仍然没有找到，那么它将在测试执行类路径中寻找`SpockConfig.groovy `。

最终，Spock 转到 Spock 用户的主目录，这只是我们主目录中的一个目录`.spock `。我们可以通过设置名为`spock.user.home `的系统属性或者环境变量`SPOCK_USER_HOME.`来改变这个目录

对于我们的示例，我们将创建一个文件`SpockConfig` `.groovy`，并将其放在类路径中(`src/test/resources/SpockConfig.Groovy`)。

### 4.1.过滤堆栈跟踪

通过使用配置文件，我们可以过滤(或不过滤)堆栈跟踪:

```java
runner {
    filterStackTrace false
}
```

**默认值为`true.`**

为了了解它是如何工作和实践的，让我们创建一个简单的测试来抛出一个`RuntimeException:`

```java
def 'stacktrace'() {
    expect:
    throw new RuntimeException("blabla")
}
```

当`filterStackTrace `设置为 false 时，我们将在输出中看到:

```java
java.lang.RuntimeException: blabla

  at sun.reflect.NativeConstructorAccessorImpl.newInstance0(Native Method)
  at sun.reflect.NativeConstructorAccessorImpl.newInstance(NativeConstructorAccessorImpl.java:62)
  at sun.reflect.DelegatingConstructorAccessorImpl.newInstance(DelegatingConstructorAccessorImpl.java:45)
  at java.lang.reflect.Constructor.newInstance(Constructor.java:423)
  at org.codehaus.groovy.reflection.CachedConstructor.invoke(CachedConstructor.java:83)
  at org.codehaus.groovy.runtime.callsite.ConstructorSite$ConstructorSiteNoUnwrapNoCoerce.callConstructor(ConstructorSite.java:105)
  at org.codehaus.groovy.runtime.callsite.CallSiteArray.defaultCallConstructor(CallSiteArray.java:60)
  at org.codehaus.groovy.runtime.callsite.AbstractCallSite.callConstructor(AbstractCallSite.java:235)
  at org.codehaus.groovy.runtime.callsite.AbstractCallSite.callConstructor(AbstractCallSite.java:247)
  // 34 more lines in the stack trace... 
```

通过将该属性设置为`true,`,我们将得到:

```java
java.lang.RuntimeException: blabla

  at extensions.StackTraceTest.stacktrace(StackTraceTest.groovy:10)
```

尽管要记住，有时查看完整的堆栈跟踪是有用的。

### 4.2.Spock 配置文件中的条件特征

有时，我们可能需要有条件地过滤堆栈跟踪。例如，我们需要在持续集成工具中看到完整的堆栈跟踪，但这在我们的本地机器上是不必要的。

我们可以添加一个简单的条件，例如基于环境变量:

```java
if (System.getenv("FILTER_STACKTRACE") == null) {   
    filterStackTrace false
}
```

Spock 配置文件是一个 Groovy 文件，因此它可以包含 Groovy 代码片段。

### 4.3.`@Issue`中的前缀和网址

之前，我们讨论过`@Issue`注释。我们也可以使用配置文件**通过用`issueUrlPrefix. `** 定义一个公共的 URL 部分来进行配置

另一个属性是`issueNamePrefix.` ，那么每个`@Issue`值前面都有一个`issueNamePrefix`属性。

我们需要在`the report`中添加这两个属性:

```java
report {
    issueNamePrefix 'Bug '
    issueUrlPrefix 'https://jira.org/issues/'
}
```

### 4.4.优化运行顺序

**另一个非常有用的工具是`optimizeRunOrder`** 。Spock 可以记住哪些规范失败了，以及执行一个特征方法的频率和时间。

基于这一知识，`Spock`将首先运行上次运行失败的特性。首先，它将执行那些失败次数更多的规范。此外，最快的规格将首先运行。

该行为可在配置文件中启用。为了启用优化器，我们使用`optimizeRunOrder `属性:

```java
runner {
  optimizeRunOrder true
}
```

**默认情况下，运行命令的优化器被禁用。**

### 4.5.包括和不包括规格

斯波克可以排除或包括某些规格。我们可以依靠类、超类、接口或注释，它们应用于规范类。基于特征级的注释，该库能够排除或包括单个特征。

我们可以简单地通过使用`exclude`属性从类`TimeoutTest` 中排除一个测试套件:

```java
import extensions.TimeoutTest

runner {
    exclude TimeoutTest
}
```

**`TimeoutTest `及其所有子类都将被排除。**如果`TimeoutTest`是一个应用于规范类的注释，那么这个规范将被排除。

我们可以分别指定注释和基类:

```java
import extensions.TimeoutTest
import spock.lang.Issue
    exclude {
        baseClass TimeoutTest
        annotation Issue
}
```

上面的例子**将排除带有`@Issue `注释的测试类或方法，以及`TimeoutTest` 或它的任何子类。**

要包含任何规范，我们只需使用`include `属性。我们可以像定义`exclude`一样定义`include`的规则。

### 4.6.创建报告

**基于测试结果和先前已知的注释，我们可以生成一个带有`Spock.`** 的报告。此外，这个报告将包含像`@Title, @See, @Issue, and @Narrative`值这样的东西。

我们可以在配置文件中生成报告。默认情况下，它不会生成报告。

我们所要做的就是传递一些属性的值:

```java
report {
    enabled true
    logFileDir '.'
    logFileName 'report.json'
    logFileSuffix new Date().format('yyyy-MM-dd')
}
```

上述属性是:

*   `enabled `–是否应该生成报告
*   `logFileDir `–报告目录
*   `logFileName – `报告的名称
*   `logFileSuffix`–每个生成的报告基本名称的后缀，用破折号分隔

当我们将`enabled` 设置为`true,`时，必须设置`logFileDir`和`logFileName `属性。`logFileSuffix`是可选的。

我们也可以在系统属性中全部设置:`enabled`、`spock.logFileDir,` 、`spock.logFileName` 和`spock.logFileSuffix.`

## 5.结论

在本文中，我们描述了最常见的 Spock 扩展。

**我们知道大部分都是基于注解**。此外，我们还学习了如何创建一个`Spock `配置文件，以及有哪些可用的配置选项。简而言之，我们新获得的知识对于编写有效且易于阅读的测试非常有帮助。

我们所有示例的实现都可以在我们的 [Github 项目](https://web.archive.org/web/20221226061918/https://github.com/eugenp/tutorials/tree/master/testing-modules/groovy-spock)中找到。