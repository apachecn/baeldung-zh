# Maven 属性的默认值

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/maven-properties-defaults>

## 1.概观

Apache Maven 是一个强大的构建自动化工具，主要用于 Java 项目。Maven 使用项目对象模型(POM)来构建项目，它包含了关于项目和配置细节的信息。在 POM 内部，我们能够定义可以在 POM 本身或者一个[多模块](/web/20220926182749/https://www.baeldung.com/maven-multi-module)配置项目中的任何子 POM 中使用的属性。

Maven 属性允许我们在一个地方定义值，并在项目定义中的几个不同位置使用它们。

在这篇短文中，我们将介绍如何配置默认值，以及如何使用它们。

## 2.POM 中的默认值

**最常见的是，我们在 POM** 中定义 Maven 属性的默认值——为了演示这一点，我们将创建一个保存库依赖项默认值的属性。让我们从在 POM 中定义属性及其默认值开始:

```java
<properties>
    <junit.version>4.13</junit.version>
</properties> 
```

在这个例子中，我们已经创建了一个名为`junit.version`的属性，并赋予了默认值`4.13`。

## 3.`settings.xml`中的默认值

**我们也可以在用户的`settings.xml`** 中定义 Maven 属性。如果用户需要为属性设置自己的默认值，这将非常有用。我们在`settings.xml` 中定义属性和它们的值，就像我们在 POM 中定义它们一样。

我们在用户主目录的`.m2`目录中找到`settings.xml`。

## 4.命令行上的默认值

**当执行 Maven 命令**时，我们可以在[命令行](/web/20220926182749/https://www.baeldung.com/maven-arguments)上定义属性的默认值。在本例中，我们将默认值`4.13`更改为`4.12`:

```java
mvn install -Djunit.version=4.12
```

## 5.在 POM 中使用属性

我们可以在 POM 的其他地方引用我们的默认属性值，所以让我们继续定义`junit` 依赖项，并使用我们的属性来检索版本号:

```java
<dependencies>
    <dependency>
        <groupId>junit</groupId>
        <artifactId>junit</artifactId>
        <version>${junit.version}</version>
    </dependency>
</dependencies> 
```

我们通过使用`${junit.version}` 语法来引用值`junit.version`。

## 6.结论

在这篇短文中，我们看到了如何以三种不同的方式定义 Maven 属性的默认值，正如我们所看到的，它们有助于我们在不同的地方重用相同的值，而只需要在一个地方管理它。

和往常一样，示例代码可以在 GitHub 上找到[。](https://web.archive.org/web/20220926182749/https://github.com/eugenp/tutorials/tree/master/maven-modules/maven-properties)