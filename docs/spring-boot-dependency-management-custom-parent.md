# 具有自定义父项的 Spring Boot 相关性管理

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-boot-dependency-management-custom-parent>

## 1。概述

`Spring Boot`提供父 POM，以便更轻松地创建 Spring Boot 应用程序。

然而，如果我们已经有了一个可以继承的父 POM，使用父 POM 可能并不总是理想的。

在这个快速教程中，我们将看看如何在没有父启动器的情况下使用 Boot。

## 2。`Spring Boot`无父 POM

**父`pom.xml`负责依赖性和插件管理。**由于这个原因，继承它在应用程序中提供了有价值的支持，所以在创建`Boot`应用程序时，这通常是首选的做法。你可以在我们之前的文章的[中找到更多关于如何基于父启动器构建应用程序的细节。](/web/20220627083827/https://www.baeldung.com/spring-boot-start)

然而在实践中，我们可能会受到设计规则或其他偏好的约束，使用不同的父代。

幸运的是，`Spring Boot`提供了一种替代从父启动器继承的方法，这种方法仍然可以为我们提供一些它的优点。

**如果我们不使用父 POM，我们仍然可以通过添加带有`scope=import`的`spring-boot-dependencies` 工件从依赖管理**中获益:

```
<dependencyManagement>
     <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-dependencies</artifactId>
            <version>2.4.0</version>
            <type>pom</type>
            <scope>import</scope>
        </dependency>
    </dependencies>
</dependencyManagement>
```

接下来，我们可以简单地开始添加 Spring 依赖项并利用`Spring Boot`特性:

```
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
</dependency>
```

**另一方面，没有了父 POM，我们不再受益于插件管理。**这意味着我们需要显式地添加`spring-boot-maven-plugin`:

```
<build>
    <plugins>
        <plugin>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-maven-plugin</artifactId>
        </plugin>
    </plugins>
</build>
```

## 3。覆盖依赖版本

如果我们想为某个依赖项使用不同于 Boot 管理的版本，我们需要在声明`spring-boot-dependencies`之前，在`dependencyManagement`部分声明它:

```
<dependencyManagement>
    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-data-jpa</artifactId>
            <version>2.4.0</version>
        </dependency>
    </dependencies>
    // ...
</dependencyManagement>
```

相比之下，仅仅在`dependencyManagement`标签之外声明依赖项的版本将不再有效。

## 4。结论

在这个快速教程中，我们已经看到了如何在没有父`pom.xml.`的情况下使用`Spring Boot`

示例的源代码可以在 GitHub 的[中找到。](https://web.archive.org/web/20220627083827/https://github.com/eugenp/tutorials/tree/master/spring-boot-modules/spring-boot-artifacts)