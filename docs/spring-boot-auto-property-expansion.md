# 使用 Spring Boot 自动扩展属性

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-boot-auto-property-expansion>

## 1。概述

在本文中，我们将通过 Maven 和 Gradle 构建方法探索 Spring 提供的属性扩展机制。

## 2。肚子

### 2.1。默认配置

对于使用*spring-boot-starter-parent*的 Maven 项目，不需要额外的配置来利用属性扩展:

```java
<parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>2.3.3.RELEASE</version>
</parent> 
```

现在我们可以使用 `@…@`占位符来扩展我们项目的属性。下面是一个例子，说明我们如何将取自 Maven 的项目版本保存到我们的属性中:

```java
[[email protected]](/web/20220926184421/https://www.baeldung.com/cdn-cgi/l/email-protection)@
[[email protected]](/web/20220926184421/https://www.baeldung.com/cdn-cgi/l/email-protection)@ 
```

我们只能在匹配这些模式的配置文件中使用这些扩展:

*   `**/application*.yml`
*   `**/application*.yaml`
*   `**/application*.properties`

### 2.2。手动配置

在没有*spring-boot-starter-parent*parent 的情况下，我们需要手动配置这个过滤和扩展。我们需要将*资源*元素包含到我们`pom.xml`文件的< *构建>* 部分:

```java
<resources>
    <resource>
        <directory>${basedir}/src/main/resources</directory>
        <filtering>true</filtering>
        <includes>
            <include>**/application*.yml</include>
            <include>**/application*.yaml</include>
            <include>**/application*.properties</include>
         </includes>
    </resource>
</resources> 
```

而在<*插件>中*:

```java
<plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-resources-plugin</artifactId>
    <version>2.7</version>
    <configuration>
        <delimiters>
            <delimiter>@</delimiter>
        </delimiters>
        <useDefaultDelimiters>false</useDefaultDelimiters>
    </configuration>
</plugin> 
```

如果需要使用类型为 *${variable.name}* 的标准占位符，我们需要将*usedefaultdelimiters*设置为 *true* ，您的 *application.properties* 将如下所示:

```java
expanded.project.version=${project.version}
expanded.project.property=${custom.property} 
```

## 3。度

### 3.1。标准梯度解决方案

来自 Spring Boot 文档的 Gradle [解决方案](https://web.archive.org/web/20220926184421/https://docs.spring.io/spring-boot/docs/current/reference/html/howto-properties-and-configuration.html)与 Maven 属性过滤和扩展不是 100%兼容。

为了允许我们使用属性扩展机制，我们需要将以下代码包含到 *build.gradle* 中:

```java
processResources {
    expand(project.properties)
} 
```

这是一个有限的解决方案，与 Maven 默认配置有以下区别:

1.  不支持带点的属性(如*用户名)。* Gradle 将点理解为对象属性分隔符
2.  过滤所有的资源文件，而不仅仅是一组特定的配置文件
3.  使用默认的美元符号占位符 *${…}* ，因此与标准的 Spring 占位符冲突

### 3.2。Maven 兼容解决方案

为了复制标准的 Maven 解决方案并利用`@…@`样式的占位符，我们需要将以下代码添加到我们的 *build.gradle* :

```java
import org.apache.tools.ant.filters.ReplaceTokens
processResources {
    with copySpec {
        from 'src/main/resources'
        include '**/application*.yml'
        include '**/application*.yaml'
        include '**/application*.properties'
        project.properties.findAll().each {
          prop ->
            if (prop.value != null) {
                filter(ReplaceTokens, tokens: [ (prop.key): prop.value])
                filter(ReplaceTokens, tokens: [ ('project.' + prop.key): prop.value])
            }
        }
    }
} 
```

这将解析项目的所有属性。我们仍然不能在*的 build.gradle* 中定义带点的属性(例如 user.name ),但是现在我们可以使用`gradle.properties`文件来定义标准 Java 属性格式的属性，它也支持带点的属性(例如`database.url).`)

这个构建只过滤项目配置文件，而不是所有的资源，它与 Maven 解决方案 100%兼容。

## 4。结论

在这个快速教程中，我们看到了如何使用 Maven 和 Gradle 构建方法自动扩展 Spring Boot 属性，以及如何轻松地从一个方法迁移到另一个方法。

完整的源代码示例可以在 GitHub 的[中找到。](https://web.archive.org/web/20220926184421/https://github.com/eugenp/tutorials/tree/master/spring-boot-modules/spring-boot-property-exp)