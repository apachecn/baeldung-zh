# 查找未使用的 Maven 依赖项

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/maven-unused-dependencies>

## 1.概观

当使用 Maven 管理我们的项目依赖关系时，我们可能会忘记在我们的应用程序中使用了哪些依赖关系。

在这个简短的教程中，我们将介绍如何使用 [Maven 依赖插件](https://web.archive.org/web/20221205215412/https://mvnrepository.com/artifact/org.apache.maven.plugins/maven-dependency-plugin) ，这个插件可以帮助我们找到项目中未使用的依赖项。

## 2.项目设置

让我们从添加几个依赖项开始， `[slf4j-api](https://web.archive.org/web/20221205215412/https://mvnrepository.com/artifact/org.slf4j/slf4j-api)` (我们将使用的)和 `[common-collections](https://web.archive.org/web/20221205215412/https://mvnrepository.com/artifact/commons-collections/commons-collections)`(我们将不使用的):

```java
<dependencies>
    <dependency>
        <groupId>commons-collections</groupId>
        <artifactId>commons-collections</artifactId>
        <version>3.2.2</version>
    </dependency>
    <dependency>
        <groupId>org.slf4j</groupId>
        <artifactId>slf4j-api</artifactId>
        <version>1.7.25</version>
    </dependency>
</dependencies> 
```

我们可以访问[Maven 依赖插件](https://web.archive.org/web/20221205215412/https://search.maven.org/artifact/org.apache.maven.plugins/maven-dependency-plugin)，而无需在 pom 中指定它。在任何情况下，我们都可以使用`pom.xml`定义来指定版本，如果需要的话还可以指定一些属性:

```java
<build>
    <plugins>
        <plugin>
            <artifactId>maven-dependency-plugin</artifactId>
            <version>3.1.2</version>
        </plugin>
    </plugins>
</build>
```

## 3.代码示例

现在我们已经设置好了项目，**让我们写一行代码，其中使用了我们之前定义的依赖项之一:**

```java
public Logger getLogger() {
    return LoggerFactory.getLogger(UnusedDependenciesExample.class);
}
```

Slf4J 库中的`LoggerFactory`在方法**中返回，但是没有使用 common-collections 库，这使得它成为移除的候选对象。**

## 4.查找未使用的依赖项

使用 Maven 依赖插件，我们可以找到项目中没有使用的依赖项。为此，我们调用插件的分析目标:

```java
$ mvn dependency:analyze

[INFO] --- maven-dependency-plugin:3.1.1:analyze (default-cli) @ maven-unused-dependencies ---
[WARNING] Unused declared dependencies found:
[WARNING]    commons-collections:commons-collections:jar:3.2.2:compile
[INFO] ------------------------------------------------------------------------
[INFO] BUILD SUCCESS
[INFO] ------------------------------------------------------------------------
[INFO] Total time: 1.225 s
[INFO] Finished at: 2020-04-01T16:10:25-04:00
[INFO] ------------------------------------------------------------------------
```

对于我们项目中没有使用的每个依赖项，Maven 都会在分析报告中发出警告。

## 5.指定所使用的依赖关系

根据项目的性质，有时我们可能需要在运行时加载类，例如在面向插件的项目中。

由于在编译时不使用依赖项，`maven-dependency-plugin`会发出一个警告，声明一个依赖项没有被使用，而事实上它被使用了。为此，我们可以强制并指示插件正在使用一个库。

我们通过列出`usedDependencies`属性中的依赖项来做到这一点:

```java
<plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-dependency-plugin</artifactId>
    <configuration>
        <usedDependencies>
            <dependency>commons-collections:commons-collections</dependency>
        </usedDependencies>
    </configuration>
</plugin>
```

再次运行`analyze`目标，我们看到报告中不再考虑未使用的依赖项。

## 6.结论

在这个简短的教程中，我们学习了如何找到未使用的 maven 依赖项。定期检查未使用的依赖项是一个很好的实践，因为它提高了可维护性并减少了我们项目的库大小。

与往常一样，GitHub 上的[提供了包含所有示例的完整源代码。](https://web.archive.org/web/20221205215412/https://github.com/eugenp/tutorials/tree/master/maven-modules/maven-unused-dependencies)