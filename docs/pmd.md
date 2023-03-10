# PMD 简介

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/pmd>

## 1。概述

简而言之， [PMD](https://web.archive.org/web/20221206063442/https://pmd.github.io/) 是一个源代码分析器，用来发现常见的编程缺陷，比如未使用的变量、空的 catch 块、不必要的对象创建等等。

它支持 Java，JavaScript，Salesforce.com Apex，PLSQL，Apache Velocity，XML，XSL。

在本文中，我们将关注如何使用 PMD 在 Java 项目中执行静态分析。

## 2。先决条件

让我们从在 Maven 项目中设置 PMD 开始——使用和配置`maven-pmd-plugin`:

```java
<project>
    ...
    <reporting>
        <plugins>
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-pmd-plugin</artifactId>
                <version>3.7</version>
                <configuration>
                    <rulesets>
                        <ruleset>/rulesets/java/braces.xml</ruleset>
                        <ruleset>/rulesets/java/naming.xml</ruleset>
                    </rulesets>
                </configuration>
            </plugin>
        </plugins>
    </reporting>
</project> 
```

你可以在这里找到最新版本的`maven-pmd-plugin` [。](https://web.archive.org/web/20221206063442/https://search.maven.org/classic/#search%7Cgav%7C1%7Cg%3A%22org.apache.maven.plugins%22%20AND%20a%3A%22maven-pmd-plugin%22)

请注意我们是如何在这里的配置中添加规则集的——这些是已经从 PMD 核心库定义了规则的相对路径。

最后，在运行所有程序之前，让我们创建一个简单的 Java 类，它有一些突出的问题——PMD 可以开始报告这些问题:

```java
public class Ct {

    public int d(int a, int b) {
        if (b == 0)
            return Integer.MAX_VALUE;
        else
            return a / b;
    }
} 
```

## 3。跑 PMD

使用简单的 PMD 配置和示例代码，让我们在构建目标文件夹中生成一个报告:

```java
mvn site
```

生成的报告名为`pmd.html`，位于`target/site`文件夹中:

```java
Files

com/baeldung/pmd/Cnt.java

Violation                                                                             Line

Avoid short class names like Cnt                                   1–10 
Avoid using short method names                                  3 
Avoid variables with short names like b                        3 
Avoid variables with short names like a                        3 
Avoid using if...else statements without curly braces 5 
Avoid using if...else statements without curly braces 7 
```

如你所见，我们没有得到结果。根据 PMD 的说法，报告显示了 Java 代码中的违规和行号。

## 4。规则集

PMD 插件使用五个默认规则集:

*   `basic.xml`
*   `empty.xml`
*   `imports.xml`
*   `unnecessary.xml`
*   `unusedcode.xml`

您可以使用其他规则集或创建自己的规则集，并在插件中配置这些规则集:

```java
<project>
    ...
    <reporting>
        <plugins>
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-pmd-plugin</artifactId>
                <version>3.7</version>
                <configuration>
                    <rulesets>
                        <ruleset>/rulesets/java/braces.xml</ruleset>
                        <ruleset>/rulesets/java/naming.xml</ruleset>
                        <ruleset>/usr/pmd/rulesets/strings.xml</ruleset>
                        <ruleset>http://localhost/design.xml</ruleset>
                    </rulesets>
                </configuration>
            </plugin>
        </plugins>
    </reporting>
</project> 
```

请注意，我们使用相对地址、绝对地址甚至 URL 作为配置中“ruleset”值的值。

定制项目使用的规则的一个简单策略是**编写一个定制的规则集文件**。在这个文件中，我们可以定义要使用的规则，添加自定义规则，以及自定义要在正式规则集中包含/排除的规则。

## 5。自定义规则集

现在，让我们从 PMD 的现有规则集中选择我们想要使用的特定规则，并对它们进行自定义。

首先，我们将创建一个新的`ruleset.xml`文件。当然，我们可以使用一个现有的规则集文件作为示例，将其复制并粘贴到我们的新文件中，删除其中的所有旧规则，并更改名称和描述:

```java
<?xml version="1.0"?>
<ruleset name="Custom ruleset"

  xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://pmd.sourceforge.net/ruleset/2.0.0
  http://pmd.sourceforge.net/ruleset_2_0_0.xsd">
    <description>
        This ruleset checks my code for bad stuff
    </description>
</ruleset>
```

其次，让我们添加一些规则引用:

```java
<!-- We'll use the entire 'strings' ruleset -->
<rule ref="rulesets/java/strings.xml"/> 
```

或者添加一些具体的规则:

```java
<rule ref="rulesets/java/unusedcode.xml/UnusedLocalVariable"/>
<rule ref="rulesets/java/unusedcode.xml/UnusedPrivateField"/>
<rule ref="rulesets/java/imports.xml/DuplicateImports"/>
<rule ref="rulesets/java/basic.xml/UnnecessaryConversionTemporary"/> 
```

我们可以自定义规则的消息和优先级:

```java
<rule ref="rulesets/java/basic.xml/EmptyCatchBlock"
  message="Must handle exceptions">
    <priority>2</priority>
</rule> 
```

您还可以自定义规则的属性值，如下所示:

```java
<rule ref="rulesets/java/codesize.xml/CyclomaticComplexity">
    <properties>
        <property name="reportLevel" value="5"/>
    </properties>
</rule>
```

请注意，您可以自定义单个引用的规则。在您的自定义规则集中，除了规则的类之外的所有内容都可以被覆盖。

接下来，您还可以从规则集中排除规则:

```java
<rule ref="rulesets/java/braces.xml">
    <exclude name="WhileLoopsMustUseBraces"/>
    <exclude name="IfElseStmtsMustUseBraces"/>
</rule> 
```

**接下来–您还可以使用排除模式从规则集**中排除文件，并使用可选的覆盖包含模式。

当有匹配的排除模式，但没有匹配的包含模式时，文件将被排除在处理之外。

源文件路径中的路径分隔符被规范化为“/”字符，因此同一个规则集可以透明地在多个平台上使用。

此外，无论如何使用 PMD(例如，命令行、IDE、Ant)，这种排除/包含技术都可以工作，从而更容易在整个环境中保持 PMD 规则应用的一致性。

这里有一个简单的例子:

```java
<?xml version="1.0"?>
<ruleset ...>
    <description>My ruleset</description>
    <exclude-pattern>.*/some/package/.*</exclude-pattern>
    <exclude-pattern>
       .*/some/other/package/FunkyClassNamePrefix.*
    </exclude-pattern>
    <include-pattern>.*/some/package/ButNotThisClass.*</include-pattern>
    <rule>...
</ruleset>
```

## 6。结论

在这篇简短的文章中，我们介绍了 PMD——一个灵活且高度可配置的工具，专注于 Java 代码的静态分析

和往常一样，本教程中的完整代码可以从 Github 上的[处获得。](https://web.archive.org/web/20221206063442/https://github.com/eugenp/tutorials/tree/master/static-analysis)