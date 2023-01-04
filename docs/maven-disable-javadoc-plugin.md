# 禁用 Maven Javadoc 插件

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/maven-disable-javadoc-plugin>

## 1.概观

Apache Maven Javadoc 插件允许我们在 Maven 构建期间为指定的项目生成 Javadoc。此外，该插件非常方便，因为它使用[标准`javadoc`工具](https://web.archive.org/web/20221208143917/https://docs.oracle.com/en/java/javase/11/tools/javadoc.html)自动生成 Javadocs。

在这个快速教程中，我们将看看如何在 Maven 构建中临时禁用 Javadoc 生成。

## 2.问题简介

我们可以在我们的`pom.xml`中配置 Maven Javadoc 插件来生成 Javadoc 并将它们附加到构建的`jar`文件中，例如:

```
...
<plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-javadoc-plugin</artifactId>
    <executions>
        <execution>
            <id>attach-javadocs</id>
            <goals>
                <goal>jar</goal>
            </goals>
        </execution>
    </executions>
</plugin>
...
```

这很方便。然而，有时候，当我们发布时，我们不想将 Javadocs 附加到`jar`文件中。但是，我们也不想删除 Maven Javadoc 插件。

因此，我们需要一种在构建中跳过 Javadoc 生成的方法。接下来，我们来看看如何实现。

## 3.`maven.javadoc.skip`选项

Maven Javadoc 插件提供了一个 [`maven.javadoc.skip`](https://web.archive.org/web/20221208143917/https://maven.apache.org/plugins/maven-javadoc-plugin/javadoc-mojo.html#skip) 选项来跳过 Javadoc 的生成。

如果我们在构建项目时使用值`true`传递这个选项，我们的 Maven 构建将不会生成 Javadocs:

```
mvn clean install -Dmaven.javadoc.skip=true
```

## 4.使用 Maven 发布插件跳过 Javadoc 生成

Maven 发布插件广泛用于自动发布管理。

假设我们已经在项目中配置了 Maven Release 插件和 Javadoc 插件。

现在，我们想让 Maven Javadoc 插件为常规构建生成 Javadoc，但只为发布构建跳过 Javadoc 生成。

我们可以使用两种方法来实现这个目标。

第一种方法是，当我们开始一个发布构建时，向`mvn`命令行传递一个参数:

```
mvn release:perform -Darguments="-Dmaven.javadoc.skip=true"
```

或者，我们可以在 Maven 发布插件配置中添加`maven.javadoc.skip=true`参数:

```
<plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-release-plugin</artifactId>
    <configuration>
        <arguments>-Dmaven.javadoc.skip=true</arguments>
    </configuration>
</plugin>
```

这样，所有带有 Maven Release 插件的构建都将跳过 Javadoc 的生成。

## 5.结论

在这篇快速的文章中，我们已经解决了当 Maven Javadoc 插件在我们的`pom.xml`中配置时如何跳过 Javadoc 生成。