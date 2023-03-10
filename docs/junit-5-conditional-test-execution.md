# 带有注释的 JUnit 5 条件测试执行

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/junit-5-conditional-test-execution>

## 1.概观

在本教程中，我们将看看在[JUnit 5](/web/20221222081152/https://www.baeldung.com/junit-5)中带注释的**条件测试执行。**

这些注释来自于 [JUnit Jupiter 库的](/web/20221222081152/https://www.baeldung.com/junit-5-extensions) `condition`包，允许我们指定不同类型的条件，在这些条件下我们的测试应该或者不应该运行。

## 2.操作系统条件

有时，我们需要根据他们运行的操作系统(os)来改变我们的测试场景。在这些情况下，`@EnabledOnOs`注释就派上了用场。

`@EnabledOnOs`的用法很简单——我们只需要给它一个 OS 类型的值。此外，当我们想要定位多个操作系统时，它还接受数组参数。

例如，假设我们想让一个测试只在 Windows 和 macOS 上运行:

```java
@Test
@EnabledOnOs({OS.WINDOWS, OS.MAC})
public void shouldRunBothWindowsAndMac() {
    //...
}
```

现在，与`@EnabledOnOs`相反的是`@DisabledOnOs`。顾名思义，它根据 OS 类型参数禁用测试:

```java
@Test
@DisabledOnOs(OS.LINUX)
public void shouldNotRunAtLinux() {
    //...
} 
```

## 3.Java 运行时环境条件

我们还可以使用`@EnableOnJre`和`@DisableOnJre`注释，在特定的 [JRE](/web/20221222081152/https://www.baeldung.com/jvm-vs-jre-vs-jdk#jre) 版本上运行我们的测试。这些注释还接受一个数组来启用或禁用多个 Java 版本:

```java
@Test
@EnabledOnJre({JRE.JAVA_10, JRE.JAVA_11})
public void shouldOnlyRunOnJava10And11() {
    //...
}
```

从 JUnit 5.6 开始，我们可以使用`@EnabledForJreRange `对特定范围的 Java 版本进行测试:

```java
@Test
@EnabledForJreRange(min = JRE.JAVA_8, max = JRE.JAVA_13)
public void shouldOnlyRunOnJava8UntilJava13() {
    // this test will only run on Java 8, 9, 10, 11, 12, and 13.
}
```

默认情况下，最小值是`JAVA_8 `，最大值是最大可能的 JRE 版本。还有一个`@DisabledForJreRange `来禁用特定范围 Java 版本的测试:

```java
@Test
@DisabledForJreRange(min = JRE.JAVA_14, max = JRE.JAVA_15)
public void shouldNotBeRunOnJava14AndJava15() {
    // this won't run on Java 14 and 15.
}
```

此外，如果我们想要禁止我们的测试在除了 8、9、10 和 11 之外的 Java 版本上运行，我们可以使用`JRE.OTHER` `enum`属性:

```java
@Test
@DisabledOnJre(JRE.OTHER)
public void thisTestOnlyRunsWithUpToDateJREs() {
    // this test will only run on Java 8, 9, 10, and 11.
}
```

## 4.系统属性条件

现在，如果我们想要启用基于 JVM 系统属性的测试，我们可以使用@ `EnabledIfSystemProperty`注释。

要使用它，我们必须提供`named` 和`matches`参数。`named`参数用于指定一个确切的系统属性。`matches`用于用正则表达式定义属性值的模式。

例如，假设我们希望仅在虚拟机供应商名称以“Oracle”开头时运行测试:

```java
@Test
@EnabledIfSystemProperty(named = "java.vm.vendor", matches = "Oracle.*")
public void onlyIfVendorNameStartsWithOracle() {
    //...
}
```

同样，我们有`@DisabledIfSystemProperty` 来禁用基于 JVM 系统属性的测试。为了演示这种注释，让我们看一个例子:

```java
@Test
@DisabledIfSystemProperty(named = "file.separator", matches = "[/]")
public void disabledIfFileSeperatorIsSlash() {
    //...
}
```

## 5.环境可变条件

我们还可以用`@EnabledIfEnvironmentVariable`和`@DisabledIfEnvironmentVariable` 注释`.`为我们的测试指定环境变量条件

而且，就像对[系统属性条件](#system-property-condition)的注释一样，这些注释采用两个参数——`named`和`matches —`来指定环境变量名和正则表达式，以匹配环境变量值:

```java
@Test
@EnabledIfEnvironmentVariable(named = "GDMSESSION", matches = "ubuntu")
public void onlyRunOnUbuntuServer() {
    //...
}

@Test
@DisabledIfEnvironmentVariable(named = "LC_TIME", matches = ".*UTF-8.")
public void shouldNotRunWhenTimeIsNotUTF8() {
    //...
}
```

此外，我们可以参考其他教程[来了解更多关于系统属性和系统环境变量](/web/20221222081152/https://www.baeldung.com/java-system-get-property-vs-system-getenv)的信息。

## 6.基于脚本的条件

### 6.1.弃用通知

基于脚本的条件 API 及其实现在 JUnit 5.5 中被弃用，并从 JUnit 5.6 中删除。为了达到同样的效果，强烈建议使用内置条件的组合或者创建`ExecutionCondition.`的自定义实现

### 6.2.情况

在 JUnit 5.6 之前，我们可以通过在`@EnabledIf`和`@DisabledIf`注释中编写脚本来指定测试的运行条件。

这些注释接受三个参数:

*   `value`–包含要运行的实际脚本。
*   `engine`(可选)–指定要使用的脚本引擎；默认为[甲骨文 Nashorn](https://web.archive.org/web/20221222081152/https://www.oracle.com/technetwork/articles/java/jf14-nashorn-2126515.html) 。
*   `reason`(可选)–出于日志记录的目的，指定如果我们的测试失败，JUnit 应该打印的消息。

因此，让我们来看一个简单的例子，在这个例子中，我们只指定了一行脚本，注释中没有附加的参数:

```java
@Test
@EnabledIf("'FR' == systemProperty.get('user.country')")
public void onlyFrenchPeopleWillRunThisMethod() {
    //...
}
```

还有，`@DisabledIf`的用法也完全一样:

```java
@Test
@DisabledIf("java.lang.System.getProperty('os.name').toLowerCase().contains('mac')")
public void shouldNotRunOnMacOS() {
    //...
}
```

此外，**我们可以使用`value` 参数编写多行脚本。**

让我们编写一个简单的例子，在运行测试之前检查月份名称。

我们将用支持的占位符为`reason`定义一个句子:

*   `{annotation}`–表示注释实例的字符串。
*   `{script}`–在值参数内评估的脚本文本。
*   `{result}`–表示被评估脚本返回值的字符串。

对于这个实例，我们将在`value`参数中有一个多行脚本，并为`engine`和`reason`赋值:

```java
@Test
@EnabledIf(value = {
    "load('nashorn:mozilla_compat.js')",
    "importPackage(java.time)",
    "",
    "var thisMonth = LocalDate.now().getMonth().name()",
    "var february = Month.FEBRUARY.name()",
    "thisMonth.equals(february)"
},
    engine = "nashorn",
    reason = "On {annotation}, with script: {script}, result is: {result}")
public void onlyRunsInFebruary() {
    //...
}
```

**在写脚本时，我们可以使用几个脚本绑定**:

*   `systemEnvironment` –访问系统环境变量。
*   `systemProperty`–访问系统属性变量。
*   `junitConfigurationParameter`–访问配置参数。
*   `junitDisplayName`–测试或容器显示名称。
*   `junitTags`–访问测试或容器上的标签。
*   `anotherUniqueId`–获取测试或容器的唯一 id。

最后，让我们看另一个例子，看看如何使用带有绑定的脚本:

```java
@Test
@DisabledIf("systemEnvironment.get('XPC_SERVICE_NAME') != null" +
        "&& systemEnvironment.get('XPC_SERVICE_NAME').contains('intellij')")
public void notValidForIntelliJ() {
    //this method will not run on intelliJ
}
```

此外，请参考我们的其他教程之一来[了解更多关于` @EnabledIf`和 `@DisabledIf`注释](/web/20221222081152/https://www.baeldung.com/spring-5-enabledIf)。

## 7.创建自定义条件批注

JUnit 5 附带的一个非常强大的特性是创建定制注释的能力。我们可以通过使用现有条件注释的组合来定义自定义条件注释。

例如，假设我们想要定义所有的测试来运行特定的操作系统类型和特定的 JRE 版本。我们可以为此编写一个自定义注释:

```java
@Target(ElementType.METHOD)
@Retention(RetentionPolicy.RUNTIME)
@Test
@DisabledOnOs({OS.WINDOWS, OS.SOLARIS, OS.OTHER})
@EnabledOnJre({JRE.JAVA_9, JRE.JAVA_10, JRE.JAVA_11})
@interface ThisTestWillOnlyRunAtLinuxAndMacWithJava9Or10Or11 {
}

@ThisTestWillOnlyRunAtLinuxAndMacWithJava9Or10Or11
public void someSuperTestMethodHere() {
    // this method will run with Java9, 10, 11 and Linux or macOS.
}
```

此外，我们可以**使用基于脚本的注释来创建自定义注释**:

```java
@Target(ElementType.METHOD)
@Retention(RetentionPolicy.RUNTIME)
@DisabledIf("Math.random() >= 0.5")
@interface CoinToss {
}

@RepeatedTest(2)
@CoinToss
public void gamble() {
    // this method run run roughly 50% of the time
}
```

## 8.结论

在本文中，我们学习了 JUnit 5 中带注释的条件测试执行。此外，我们还浏览了一些它们的用法示例。

接下来，我们看到了如何创建定制的条件注释。

要了解关于这个主题的更多信息，我们可以参考 [JUnit 关于带注释](https://web.archive.org/web/20221222081152/https://junit.org/junit5/docs/current/user-guide/#writing-tests-conditional-execution)的条件测试执行的文档。

像往常一样，本文的所有示例代码都可以在 [GitHub](https://web.archive.org/web/20221222081152/https://github.com/eugenp/tutorials/tree/master/testing-modules/junit5-annotations) 上找到。