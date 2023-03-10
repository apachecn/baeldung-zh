# 在 Maven 中生成一个 WAR 文件

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/maven-generate-war-file>

## 1.概观

Web 应用程序资源或 web 应用程序档案通常称为 WAR 文件。WAR 文件用于在应用服务器中部署 Java EE web 应用程序。在一个 WAR 文件中，所有的 web 组件都被打包到一个单元中。这些文件包括 JAR 文件、JavaServer Pages、Java servlets、Java 类文件、XML 文件、HTML 文件和其他 web 应用程序需要的资源文件。

Maven 是一个流行的构建管理工具，广泛用于 Java EE 项目中处理编译、打包和工件管理等构建任务。我们可以**使用 Maven WAR 插件将项目构建为[战争](/web/20221208143856/https://www.baeldung.com/java-jar-war-packaging#war)文件**。

在本教程中，我们将考虑在 Java EE 应用程序中使用 Maven WAR 插件。为此，我们将创建一个简单的 Maven Spring Boot web 应用程序，并从中生成一个 WAR 文件。

## 2.设置 Spring Boot Web 应用程序

让我们创建一个简单的 Maven、Spring Boot 和 [Thymeleaf](/web/20221208143856/https://www.baeldung.com/thymeleaf-in-spring-mvc) web 应用程序来演示 WAR 文件生成过程。

首先，我们将向构建我们的 Spring Boot web 应用程序所需的`pom.xml`文件添加依赖项:

```java
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-thymeleaf</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-tomcat</artifactId>
    <scope>provided</scope>
</dependency>
```

接下来，让我们创建我们的`MainController`类。在这个类中，我们将创建一个单独的`GET`控制器方法来查看我们的 HTML 文件:

```java
@Controller
public class MainController {

    @GetMapping("/")
    public String viewIndexPage(Model model) {
        model.addAttribute("header", "Maven Generate War");
        return "index";
    }
}
```

最后，是时候创建我们的`index.html`文件了。项目中还包含了引导 CSS 文件，我们的`index.html`文件中使用了一些 CSS 类:

```java
<!DOCTYPE html>
<html lang="en" xmlns:th="http://www.thymeleaf.org">
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Index</title>
    <!-- Bootstrap core CSS -->
    <link th:href="@{/css/bootstrap.min.css}" rel="stylesheet">
</head>
<body>
    <nav class="navbar navbar-light bg-light">
        <div class="container-fluid">
            <a class="navbar-brand" href="#">
                Baeldung Tutorial
            </a>
        </div>
    </nav>
    <div class="container">
        <h1>[[${header}]]</h1>
    </div>
</body>
</html>
```

## 3.Maven 战争插件

Maven WAR 插件负责将 web 应用的所有依赖、类和资源收集和编译到 web 应用档案中。

Maven WAR 插件中有一些明确的目标:

*   `war`:这是在项目打包阶段调用的默认目标。如果`packaging`类型是`war`，它会建立一个 WAR 文件。
*   这个目标通常用于项目开发阶段，以加速测试。它在指定的目录中生成一个展开的 web 应用程序。
*   `inplace`:这是`exploded`目标的变体。它在 web 应用程序文件夹中生成一个展开的 web 应用程序。

让我们将 Maven WAR 插件添加到我们的`pom.xml`文件中:

```java
<plugin>
    <artifactId>maven-war-plugin</artifactId>
    <version>3.3.1</version>
</plugin>
```

现在，一旦我们执行了`mvn install`命令，WAR 文件将在目标文件夹中生成。

使用`mvn:war:exploded`命令，我们可以将展开的 WAR 生成为目标目录中的一个目录。这是一个普通目录，WAR 文件中的所有文件都包含在展开的 WAR 目录中。

## 4.包括或排除 WAR 文件内容

使用 Maven WAR 插件，我们可以过滤一个 WAR 文件的内容。让我们配置 Maven WAR 插件，在 WAR 文件中包含一个`additional_resources`文件夹:

```java
<plugin>
    <artifactId>maven-war-plugin</artifactId>
    <version>3.3.1</version>
    <configuration>
        <webResources>
            <resource>
                <directory>additional_resources</directory>
            </resource>
        </webResources>
    </configuration>
</plugin>
```

一旦我们执行了`mvn install`命令，`additional_resources`文件夹下的所有内容都可以在 WAR 文件中找到。当我们需要向 WAR 文件中添加一些额外的资源(例如报告)时，这很有用。

## 5.编辑清单文件

**Maven WAR 插件允许定制`manifest`文件。**例如，我们可以将类路径添加到`manifest`文件。当 WAR 文件在一个更复杂的结构下，当我们需要在几个模块之间共享项目依赖关系时，这是非常有用的。

让我们配置 Maven WAR 插件，将类路径添加到`manifest`文件中:

```java
<plugin>
    <artifactId>maven-war-plugin</artifactId>
    <version>3.3.1</version>
    <configuration>
        <archive>
            <manifest>
                <addClasspath>true</addClasspath>
            </manifest>
        </archive>
    </configuration>
</plugin>
```

## 6.结论

在这个简短的教程中，我们讨论了如何使用 Maven 构建工具生成 WAR 文件。我们创建了一个 Maven Spring Boot web 应用程序来演示这项工作。为了生成 WAR 文件，我们使用了一个叫做 Maven WAR 插件的特殊插件。

GitHub 上的[提供了完整的源代码示例。](https://web.archive.org/web/20221208143856/https://github.com/eugenp/tutorials/tree/master/maven-modules/maven-generate-war)