# 编写自定义 Gradle 插件

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/gradle-create-plugin>

[This article is part of a series:](javascript:void(0);)[• Introduction to Gradle](/web/20220630135208/https://www.baeldung.com/gradle)
[• Ant vs Maven vs Gradle](/web/20220630135208/https://www.baeldung.com/ant-maven-gradle)
• Writing Custom Gradle Plugins (current article)[• Creating a Fat Jar in Gradle](/web/20220630135208/https://www.baeldung.com/gradle-fat-jar)

## 1.介绍

Gradle 是一个非常流行的构建工具，它通常因高度可定制的构建过程而受到赞赏。

今天我们将展示如何创建定制的 Gradle 插件，这将允许我们修改构建过程，超越我们用标准配置所能实现的。

## 2。插件源位置

我们可以将代码放在几个不同的位置。它们都有一些优点和缺点。

### 2.1。构建脚本

我们可以简单地将插件的源代码放在构建脚本中。这将给我们自动编译和包含插件。

非常简单，然而，我们的插件在构建脚本之外是不可见的。因此，我们不能在其他构建脚本中重用它。

### 2.2。`BuildSrc`文件夹

我们可以使用的另一种可能性是将插件的源代码放在`buildSrc/src/main/java` 文件夹中。

当你运行 Gradle 时，它会检查`buildSrc`文件夹是否存在。如果存在，Gradle 会自动构建并包含我们的插件。

这将使我们有可能在不同的构建脚本之间共享我们的插件，但是我们仍然不能在其他项目中使用它。

### 2.3。独立项目

最后，我们可以创建我们的插件作为一个单独的项目，这使得插件在各种项目中完全可重用。

然而，要在外部项目中使用它，我们需要将它捆绑到一个 jar 文件中，并添加到一个项目中。

## 3。我们的第一个插件

让我们从基础开始——**每个 Gradle 插件都必须实现`com.gradle.api.Plugin` 接口*。***

该接口是通用的，因此我们可以用各种参数类型对其进行参数化。通常，参数类型为`org.gradle.api.Project.`

然而，我们可以使用不同的类型参数，以便插件在不同的生命周期阶段得到应用:

*   使用`org.gradle.api.Settings`会将插件应用到设置脚本中
*   使用`org.gradle.api.Gradle`将导致插件应用于初始化脚本

我们可以创建的最简单的插件是一个`hello world`应用程序:

```java
public class GreetingPlugin implements Plugin<Project> {
    @Override
    public void apply(Project project) {
        project.task("hello")
          .doLast(task -> System.out.println("Hello Gradle!"));
    }
}
```

我们现在可以通过在构建脚本中添加一行来应用它:

```java
apply plugin: GreetingPlugin
```

现在，在调用`gradle hello,`之后，我们将在日志中看到`“Hello Gradle”` 消息。

## 4。插件配置

大多数插件需要从构建脚本访问外部配置。

我们可以通过使用`extension objects`来做到这一点:

```java
public class GreetingPluginExtension {
    private String greeter = "Baeldung";
    private String message = "Message from the plugin!"
    // standard getters and setters
}
```

现在让我们将新的`extension object`添加到我们的插件类中:

```java
@Override
public void apply(Project project) {
    GreetingPluginExtension extension = project.getExtensions()
      .create("greeting", GreetingPluginExtension.class);

    project.task("hello")
      .doLast(task -> {
          System.out.println(
            "Hello, " + extension.getGreeter());
          System.out.println(
            "I have a message for You: " + extension.getMessage());
      });
}
```

现在，当我们调用`gradle hello,`时，我们将看到在`GreetingPluginExtension.` 中定义的默认消息

但是既然我们已经创建了扩展，我们可以在构建脚本中使用闭包来完成这项工作:

```java
greeting {
    greeter = "Stranger"
    message = "Message from the build script" 
}
```

## 5。独立插件项目

为了创建一个独立的 Gradle 插件，我们需要做更多的工作。

### 5.1。设置

首先，我们需要导入 Gradle API 依赖——这非常简单:

```java
dependencies {
    compile gradleApi()
}
```

注意，在 Maven 中做同样的事情需要`gradle-tooling-api` 依赖——来自 Gradle 存储库:

```java
<dependencies>
    <dependency>
        <groupId>org.gradle</groupId>
        <artifactId>gradle-tooling-api</artifactId>
        <version>3.0</version>
    </dependency>
    <dependency>
        <groupId>org.gradle</groupId>
        <artifactId>gradle-core</artifactId>
        <version>3.0</version>
        <scope>provided</scope>
    </dependency>
</dependencies>
<repositories>
    <repository>
        <id>repo.gradle.org</id>
        <url>https://repo.gradle.org/gradle/libs-releases-local/</url>
    </repository>
</repositories>
```

### 5.2。连接插件

为了让 Gradle 找到我们独立插件的实现，我们需要在`src/main/resources/META-INF/` gradle `-plugins`中创建属性文件。

资源文件需要有一个匹配插件 id 的名称。**所以如果我们的插件有一个 id 为`org.baeldung.greeting,` 的文件，那么文件的确切路径应该是`META-INF/gradle-plugins/org.baeldung.greeting.properties.`**

接下来，我们可以定义插件的实现类:

```java
 implementation-class=org.gradle.GreetingPlugin
```

`implementation-class`应该等于我们插件类的完整包名。

### 5.3。创建插件 ID

在 Gradle 中，插件 ID 必须遵循一些规则和约定。它们中的大多数类似于 Java 中的包名规则:

*   它们只能包含字母数字字符，“”和“-”
*   id 必须至少有一个"."将域名与插件名称分开
*   名称空间`org.gradle` 和`com.gradleware` 受到限制
*   id 不能以“.”开头或结尾
*   没有两个或两个以上的连续“.”允许字符

最后，有一个约定，插件 Id 应该是小写名称，遵循反向域名约定。

Java 包名和 Gradle 插件名的主要区别在于包名通常比插件 ID 更详细。

### 5.4。发布插件

当我们想要发布我们的插件以便能够在外部项目中重用它时，我们有两种方法可以实现。

首先，**我们可以将我们的插件 JAR 发布到一个外部库，比如`Maven` 或`Ivy.`**

或者，我们可以使用 Gradle 插件门户。这将允许我们的插件被广大的 Gradle 社区访问。关于将项目发布到 Gradle 资源库的更多信息可以在 [Gradle 插件门户文档中找到。](https://web.archive.org/web/20220630135208/https://plugins.gradle.org/docs/submit)

### 5.5。Java 梯度开发插件

当我们用 Java 编写插件时，我们可以从`Java Gradle Development Plugin.` 中受益

这将自动编译并添加`gradleApi()`依赖项。它还将执行插件元数据验证，作为`gradle jar`任务的一部分。

我们可以通过在构建脚本中添加以下代码块来添加插件:

```java
plugins {
    id 'java-gradle-plugin'
}
```

## 6。测试插件

为了测试我们的插件工作正常，并且正确地应用到了`Project,` 中，我们可以使用`org.gradle.testfixtures.ProjectBuilder` 来创建一个`Project`的实例。

然后，我们可以检查插件是否被应用，适当的任务是否出现在我们的`Project` 实例中。我们可以使用标准的`JUnit`测试来实现:

```java
@Test
public void greetingTest(){
    Project project = ProjectBuilder.builder().build();
    project.getPluginManager().apply("com.baeldung.greeting");

    assertTrue(project.getPluginManager()
      .hasPlugin("com.baeldung.greeting"));

    assertNotNull(project.getTasks().getByName("hello"));
}
```

## 7 .**。总结**

在本文中，我们展示了在 Gradle 中编写定制插件的基础。要更深入地了解插件创建，请看一下 [Gradle 文档](https://web.archive.org/web/20220630135208/https://docs.gradle.org/current/userguide/custom_plugins.html)。

和往常一样，所有的代码样本都可以在 Github 上找到[。](https://web.archive.org/web/20220630135208/https://github.com/eugenp/tutorials/tree/master/gradle)

Next **»**[Creating a Fat Jar in Gradle](/web/20220630135208/https://www.baeldung.com/gradle-fat-jar)**«** Previous[Ant vs Maven vs Gradle](/web/20220630135208/https://www.baeldung.com/ant-maven-gradle)