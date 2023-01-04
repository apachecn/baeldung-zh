# 配置项目以排除某些声纳违规

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/sonar-exclude-violations>

## 1.概观

在我们的构建过程中，我们可以使用各种工具来报告源代码的质量。一个这样的工具是 [SonarQube](/web/20220627084841/https://www.baeldung.com/sonar-qube) ，它执行静态代码分析。

有时我们可能不同意返回的结果。因此，我们可能希望**排除一些被 SonarQube 错误标记的代码。**

在这个简短的教程中，我们将看看如何禁用声纳检查。虽然可以在 SonarQube 的服务器上更改规则集，但我们将只关注如何在源代码和项目配置中控制单独的检查。

## 2.违规示例

让我们看一个例子:

```java
public void printStringToConsoleWithDate(String str) {
    System.out.println(LocalDateTime.now().toString() + " " + str);
}
```

默认情况下，由于[违反`java:S106`规则](https://web.archive.org/web/20220627084841/https://rules.sonarsource.com/java/RSPEC-106)，SonarQube 将此代码报告为`Code Smell`:

[![](img/a6e04953476ceda252c1e43a8b1822f0.png)](/web/20220627084841/https://www.baeldung.com/wp-content/uploads/2020/12/sonarExclude_1.jpg)

然而，让我们想象一下，对于这个特殊的类，我们已经决定用`System.out` 记录的[是有效的](/web/20220627084841/https://www.baeldung.com/java-system-out-println-vs-loggers)。也许这是一个轻量级的实用程序，它将在一个容器中运行，不需要一个完整的日志库来记录到`stdout`。

我们应该注意，在 SonarQube 用户界面中，也可以将违规标记为误报。但是，如果在多个服务器上分析代码，或者在重构后代码行移动到另一个类，那么违规将再次出现。

有时，我们希望在源代码存储库中进行排除，以便它们能够持久存在。

因此，让我们看看如何通过配置项目从 SonarQube 报告中排除这些代码。

## 3.使用// `NOSONAR`

我们可以**禁用一行代码，方法是在**的末尾加上// `NOSONAR`:

```java
System.out.println(
  LocalDateTime.now()
    .toString() + " " + str); //NOSONAR lightweight logging
```

行尾的// `NOSONAR`标记隐藏了所有可能会在其上出现的问题。这种方法适用于 sonar cube 支持的大多数[语言。](https://web.archive.org/web/20220627084841/https://docs.sonarqube.org/latest/faq/)

我们还被允许在`NOSONAR`之后添加一些额外的注释，解释为什么我们禁用了检查。

让我们向前看一看一种特定于 Java 的禁用检查的方法。

## 4.使用`@SuppressWarnings`

### 4.1.注释代码

在 Java 中，我们可以使用内置的 [`@SuppressWarnings`](https://web.archive.org/web/20220627084841/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/lang/SuppressWarnings.html) 注释来**排除声纳检查。**

我们可以对该函数进行注释:

```java
@SuppressWarnings("java:S106")
public void printStringToConsoleWithDate(String str) {
    System.out.println(LocalDateTime.now().toString() + " " + str);
}
```

这与禁止编译器警告的工作方式完全相同。我们所要做的就是指定规则标识符，在本例中是`java:S106`。

### 4.2.如何获取标识符

我们可以使用 SonarQube 用户界面获得规则标识符。当我们查看违规时，我们可以点击`Why is this an issue?`:

[![](img/63e56bd5307767919baeb03035c74a01.png)](/web/20220627084841/https://www.baeldung.com/wp-content/uploads/2020/12/sonarExclude_3.jpg)

它向我们展示了定义。由此我们可以在右上角找到规则标识符:

[![](img/15ea4c19f811c80c5129b30358ec8198.png)](/web/20220627084841/https://www.baeldung.com/wp-content/uploads/2020/12/sonarExclude_2.jpg)

## 5.使用`sonar-project.properties`

我们也可以使用[分析属性](https://web.archive.org/web/20220627084841/https://docs.sonarqube.org/latest/analysis/analysis-parameters/) 在`sonar-project.properties`文件中**定义排除规则。**

让我们定义并添加`sonar-project.properties`文件到我们的资源目录:

```java
sonar.issue.ignore.multicriteria=e1

sonar.issue.ignore.multicriteria.e1.ruleKey=java:S106
sonar.issue.ignore.multicriteria.e1.resourceKey=**/SonarExclude.java
```

我们刚刚宣布了我们的第一个`[multicriteria](https://web.archive.org/web/20220627084841/https://docs.sonarqube.org/latest/analysis/scan/sonarscanner/)`，命名为`e1`。我们排除了`SonarExclude`类的`java:S106`规则。我们的定义可以使用规则标识符**和[文件匹配模式](https://web.archive.org/web/20220627084841/https://docs.sonarqube.org/latest/project-administration/narrowing-the-focus/#header-1)**`,` 分别在`ruleKey`和`resourceKey`属性前面加上`e1`名称标签来混合排除。

使用这种方法，我们可以构建一个复杂的配置，在多个文件中排除特定的规则:

```java
sonar.issue.ignore.multicriteria=e1,e2

# Console usage - ignore a single class
sonar.issue.ignore.multicriteria.e1.ruleKey=java:S106
sonar.issue.ignore.multicriteria.e1.resourceKey=**/SonarExclude.java
# Too many parameters - ignore the whole package
sonar.issue.ignore.multicriteria.e2.ruleKey=java:S107
sonar.issue.ignore.multicriteria.e2.resourceKey=com/baeldung/sonar/*.java
```

我们刚刚定义了`multicriteria`的一个子集。我们通过添加第二个定义来扩展我们的配置，并将其命名为`e2`。然后，我们将这两个规则合并到一个子集中，用逗号分隔名称。

## 6.使用 Maven 禁用

使用 [Maven 属性](https://web.archive.org/web/20220627084841/https://maven.apache.org/pom.html#Properties)也可以应用所有分析属性。类似的机制在 [Gradle](https://web.archive.org/web/20220627084841/https://docs.gradle.org/current/userguide/build_environment.html#sec:gradle_system_properties) 中也有。

### 六点一。`Multicriteria in `肚子

回到例子，让我们修改我们的`pom.xml`:

```java
<properties>
    <sonar.issue.ignore.multicriteria>e1</sonar.issue.ignore.multicriteria>
    <sonar.issue.ignore.multicriteria.e1.ruleKey>java:S106</sonar.issue.ignore.multicriteria.e1.ruleKey>
    <sonar.issue.ignore.multicriteria.e1.resourceKey>
      **/SonarExclude.java
    </sonar.issue.ignore.multicriteria.e1.resourceKey>
</properties>
```

该配置的工作方式与在`sonar-project.properties`文件中使用时完全相同。

### 6.2.缩小关注范围

有时，一个被分析的项目可能包含一些我们想要排除的生成代码，并且[缩小 SonarQube 检查的焦点](https://web.archive.org/web/20220627084841/https://docs.sonarqube.org/latest/project-administration/narrowing-the-focus/)。

让我们通过在我们的`pom.xml`中定义`sonar.exclusions`来排除我们的类:

```java
<properties>
    <sonar.exclusions>**/SonarExclude.java</sonar.exclusions>
</properties>
```

在这种情况下，我们根据文件名排除了一个文件。将对除该文件之外的所有文件进行检查。

我们也可以使用文件匹配模式。让我们通过定义来排除整个包:

```java
<properties>
    <sonar.exclusions>com/baeldung/sonar/*.java</sonar.exclusions>
</properties>
```

另一方面，通过使用`sonar.inclusions`属性，我们可以要求 SonarQube 只分析项目文件的特定子集:

```java
<properties>
    <sonar.inclusions>com/baeldung/sonar/*.java</sonar.inclusions>
</properties>
```

这个代码片段只为来自`com.baeldung.sonar`包的 java 文件定义了分析。

最后，我们还可以定义`sonar.skip`值:

```java
<properties>
    <sonar.skip>true</sonar.skip>
</properties>
```

这将整个 Maven 模块排除在 SonarQube 检查之外。

## 7.结论

在本文中，我们讨论了抑制代码中某些 SonarQube 分析的不同方法。

我们从排除个别行上的检查开始。然后，我们讨论了内置的`@SuppressWarnings`注释和特定规则的排除。这要求我们找到规则的标识符。

我们还看了配置分析属性。我们尝试了`multicriteria`和`sonar-project.properties`文件。

最后，我们将我们的属性移到了`pom.xml`中，并回顾了缩小关注范围的其他方法。