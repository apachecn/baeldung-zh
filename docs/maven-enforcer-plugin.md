# Maven Enforcer 插件

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/maven-enforcer-plugin>

## 1。概述

在本教程中，我们将学习 [Maven Enforcer 插件](https://web.archive.org/web/20221207004925/https://maven.apache.org/enforcer/maven-enforcer-plugin/)以及我们如何使用它来保证我们项目的合规性水平。

当我们有分散在世界各地的分布式团队时，这个插件特别方便。

## 2。依赖性

为了在我们的项目中使用插件，我们需要向我们的`pom.xml`添加以下依赖项:

```java
<plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-enforcer-plugin</artifactId>
    <version>3.0.0-M2</version>
</plugin>
```

该插件的最新版本可在 [Maven Central](https://web.archive.org/web/20221207004925/https://search.maven.org/search?q=a:maven-enforcer-plugin%20AND%20g:org.apache.maven.plugins) 上获得。

## 3。插件配置和目标

Maven Enforcer 有两个目标:`enforcer:enforce `和`enforcer:display-info.`

`enforce `目标在项目构建期间运行，以执行配置中指定的规则，而`display-info`目标显示关于项目的`pom.xml`中存在的内置规则的当前信息。

让我们在`executions`标签中定义`enforce`目标。此外，我们将添加保存项目的`rules`定义的`configuration`标签:

```java
...
<executions>
    <execution>
        <id>enforce</id>
        <goals>
            <goal>enforce</goal>
        </goals>
        <configuration>
            <rules>
                <banDuplicatePomDependencyVersions/>
            </rules>
        </configuration>
    </execution>
</executions>
...
```

## 4。Maven Enforcer 规则

关键字`enforce`微妙地暗示了需要遵守的规则的存在。这就是 Maven Enforcer 插件的工作方式。我们用一些将在项目的构建阶段强制执行的规则来配置它。

在这一节中，我们将看看可以应用于我们的项目以提高其质量的可用规则。

### 4.1。禁止重复依赖

在多模块项目中，在`POMs`之间存在父子关系，确保项目的有效最终`POM`中没有重复的依赖关系可能是一项棘手的任务。但是，有了`banDuplicatePomDependencyVersions`规则，我们可以很容易地确保我们的项目没有这样的小故障。

我们需要做的就是将`banDuplicatePomDependencyVersions`标签添加到插件配置的`rules`部分:

```java
...
<rules>
    <banDuplicatePomDependencyVersions/>
</rules>
...
```

为了检查规则的行为，我们可以在`pom.xml`中复制一个依赖项并运行`mvn clean compile. `,这将在控制台上产生以下错误行:

```java
...
[WARNING] Rule 0: org.apache.maven.plugins.enforcer.BanDuplicatePomDependencyVersions failed with message:
Found 1 duplicate dependency declaration in this project:
 - dependencies.dependency[io.vavr:vavr:jar] ( 2 times )

[INFO] ------------------------------------------------------------------------
[INFO] BUILD FAILURE
[INFO] ------------------------------------------------------------------------
[INFO] Total time:  1.370 s
[INFO] Finished at: 2019-02-19T10:17:57+01:00
...
```

### 4.2。需要 Maven 和 Java 版本

`requireMavenVersion`和`requireJavaVersion`规则分别支持在项目范围内锁定所需的 Maven 和 Java 版本。这将有助于消除在开发环境中使用不同版本的 Maven 和 JDK 可能产生的差异。

让我们更新插件配置的`rules`部分:

```java
<requireMavenVersion>
    <version>3.0</version>
</requireMavenVersion>
<requireJavaVersion>
    <version>1.8</version>
</requireJavaVersion>
```

这些允许我们以灵活的方式指定版本号，只要它们符合插件的[版本范围规范模式。](https://web.archive.org/web/20221207004925/https://maven.apache.org/enforcer/enforcer-rules/versionRanges.html)

此外，这两个规则还接受一个用于指定定制消息的`message`参数:

```java
...
<requireMavenVersion>
    <version>3.0</version>
    <message>Invalid Maven version. It should, at least, be 3.0</message>
</requireMavenVersion>
...
```

### 4.3。需要环境变量

使用`requireEnvironmentVariable`规则，我们可以确保在执行环境中设置某个环境变量。

它可以重复，以适应一个以上的所需变量:

```java
<requireEnvironmentVariable>
    <variableName>ui</variableName>
</requireEnvironmentVariable>
<requireEnvironmentVariable>
    <variableName>cook</variableName>
</requireEnvironmentVariable>
```

### 4.4。需要活动配置文件

Maven 中的概要文件帮助我们配置当我们的应用程序被部署到不同的环境中时将被激活的属性。

因此，当我们需要确保一个或多个指定的概要文件是活动的时，我们可以使用`requireActiveProfile`规则，从而保证我们的应用程序的成功执行:

```java
<requireActiveProfile>
    <profiles>local,base</profiles>
    <message>Missing active profiles</message>
</requireActiveProfile>
```

在上面的代码片段中，我们使用了`message`属性来提供一条定制消息，以显示规则检查是否失败。

### 4.5。其他规则

Maven Enforcer 插件有许多其他规则[来促进项目质量和一致性，而不管开发环境如何。](https://web.archive.org/web/20221207004925/https://maven.apache.org/enforcer/enforcer-rules/index.html)

此外，该插件有一个命令来显示一些当前配置的规则的信息:

```java
mvn enforcer:display-info
```

## 5。自定义规则

到目前为止，我们一直在探索插件的内置规则。现在，是时候看看如何创建我们自己的自定义规则了。

首先，我们需要创建一个包含自定义规则的新 Java 项目。自定义规则是一个类`Object`，它由**实现`EnforceRule `接口并覆盖`execute() `方法**:

```java
public void execute(EnforcerRuleHelper enforcerRuleHelper) throws EnforcerRuleException {
    try {
        String groupId = (String) enforcerRuleHelper.evaluate("${project.groupId}");
        if (groupId == null || !groupId.startsWith("com.baeldung")) {
            throw new EnforcerRuleException("Project group id does not start with com.baeldung");
        }
    }
    catch (ExpressionEvaluationException ex) {
        throw new EnforcerRuleException( "Unable to lookup an expression " 
          + ex.getLocalizedMessage(), ex );
    }
}
```

我们的定制规则只是检查目标项目的`groupId`是否以`com.baeldung`开头。

注意，我们不需要返回`boolean`或任何类似的东西来表明规则不被满足。我们只是抛出一个描述错误的`EnforcerRuleException `。

**我们可以通过将自定义规则作为依赖项添加到 Maven Enforcer 插件**来使用它:

```java
...
<rules>
    <myCustomRule implementation="com.baeldung.enforcer.MyCustomRule"/>
</rules>
...
```

请注意，如果定制规则项目不是 Maven Central 上发布的工件，我们可以通过运行`mvn clean install.`将它安装到本地 Maven repo 中

这将使它在编译具有 Maven Enforcer 插件的目标项目时可用。请查看自定义规则的插件[文档以了解更多信息。](https://web.archive.org/web/20221207004925/https://maven.apache.org/enforcer/enforcer-api/writing-a-custom-rule.html)

**要查看它的运行，我们可以将带有 Enforcer 插件的项目的`groupId`属性设置为除“com.baeldung”之外的任何属性，然后运行`mvn clean compile`。**

## 6。结论

在这个快速教程中，我们看到了 Maven Enforcer 插件如何成为我们现有插件的有益补充。编写自定义规则能力增强了它的应用范围。

请注意，我们需要在 GitHub 上的完整示例源代码[中取消对自定义规则示例的依赖关系和规则的注释。](https://web.archive.org/web/20221207004925/https://github.com/eugenp/tutorials/tree/master/maven-modules/maven-plugins)