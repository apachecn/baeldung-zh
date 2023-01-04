# 向 Spring Boot 应用程序添加构建属性

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-boot-build-properties>

## 1。简介

通常，我们项目的构建配置包含相当多的关于我们应用程序的信息。应用程序本身可能需要其中的一些信息。因此，我们可以从现有的构建配置中使用它，而不是硬编码这些信息。

在本文中，我们将看到如何在 Spring Boot 应用程序中使用来自项目构建配置的信息。

## 2。构建信息

假设我们想在我们网站的主页上显示应用程序描述和版本。

通常，该信息出现在`pom.xml`中:

```
<project 
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <artifactId>spring-boot</artifactId>
    <name>spring-boot</name>
    <packaging>war</packaging>
    <description>This is simple boot application for Spring boot actuator test</description>
    <version>0.0.1-SNAPSHOT</version>
...
</project>
```

## 3。引用应用程序属性文件中的信息

现在，要在我们的应用程序中使用上述信息，我们必须首先在我们的一个应用程序属性文件中引用它:

```
[[email protected]](/web/20220626073128/https://www.baeldung.com/cdn-cgi/l/email-protection)@
[[email protected]](/web/20220626073128/https://www.baeldung.com/cdn-cgi/l/email-protection)@
```

这里，我们使用构建属性`project.description`的值来设置应用程序属性`application-description`。同样，使用`project.version`设置`application-version`。

**这里最重要的一点是在属性名周围使用了`@`字符。这告诉 Spring 从 Maven 项目中展开命名属性。**

现在，当我们构建我们的项目时，这些属性将被替换为它们来自`pom.xml`的值。

这种扩展也称为资源过滤。**值得注意的是，这种过滤只适用于生产配置**。因此，我们不能使用`src/test/resources`下的文件中的构建属性。

另一件要注意的事情是，如果我们使用`addResources`标志，`spring-boot:run`目标会将`src/main/resources`直接添加到类路径中。虽然这对于热重装很有用，但是它绕过了资源过滤，因此也绕过了这个特性。

现在，**只有当我们使用`spring-boot-starter-parent`时，上面的属性扩展才能开箱即用。**

### 3.1。`spring-boot-starter-parent`扩展属性无

让我们看看如何在不使用`spring-boot-starter-parent`依赖的情况下启用这个特性。

首先，我们必须在我们的`pom.xml`中的`<build/>`元素内启用资源过滤:

```
<resources>
    <resource>
        <directory>src/main/resources</directory>
        <filtering>true</filtering>
    </resource>
</resources>
```

这里，我们只在`src/main/resources`下启用了资源过滤。

然后，我们可以为`maven-resources-plugin`添加分隔符配置:

```
<plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-resources-plugin</artifactId>
    <configuration>
        <delimiters>
            <delimiter>@</delimiter>
        </delimiters>
        <useDefaultDelimiters>false</useDefaultDelimiters>
    </configuration>
</plugin>
```

注意，我们已经将`useDefaultDelimiters`属性指定为`false`。这确保了＄{ placeholder }等标准 Spring 占位符不会被构建扩展。

## 4。使用 YAML 文件中的构建信息

如果我们使用 YAML 来存储应用程序属性，**我们可能无法使用`@`来指定构建属性**。这是因为`@`在 YAML 是一个保留字符。

但是，我们可以通过**在`maven-resources-plugin`** 中配置不同的分隔符来解决这个问题:

```
<plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-resources-plugin</artifactId>
    <configuration>
        <delimiters>
            <delimiter>^</delimiter>
        </delimiters>
        <useDefaultDelimiters>false</useDefaultDelimiters>
    </configuration>
</plugin>
```

或者，简单地通过**覆盖我们`pom.xml`的属性块**中的`resource.delimiter`属性:

```
<properties>
    <resource.delimiter>^</resource.delimiter>
</properties>
```

然后，我们可以在 YAML 文件中使用`^`:

```
application-description: ^project.description^
application-version: ^project.version^
```

## 5。结论

在本文中，我们看到了如何在应用程序中使用 Maven 项目信息。这可以帮助我们避免在应用程序属性文件中对项目构建配置中已经存在的信息进行硬编码。

当然，伴随本教程的代码可以在 GitHub 上找到[。](https://web.archive.org/web/20220626073128/https://github.com/eugenp/tutorials/tree/master/spring-boot-modules/spring-boot-properties)