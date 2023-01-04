# Maven 中的可选依赖项

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/maven-optional-dependency>

## 1.概观

这个简短的教程将描述 Maven 的`<optional>`标签，以及我们如何使用它来减少 Maven 项目工件的大小和范围，比如 WAR、EAR 或 JAR。

为了复习 Maven，[查看我们的综合指南](/web/20220628114425/https://www.baeldung.com/maven)。

## 2.什么是`<optional>`？

有时我们会创建一个 Maven 项目，作为其他 Maven 项目的依赖项。在处理这样的项目时，可能需要包含一个或多个依赖项，这些依赖项只对该项目的一部分功能有用。

如果最终用户不使用那个特性子集，项目仍然会暂时引入那些依赖项。这不必要地扩大了用户的项目规模，甚至可能引入与其他项目依赖相冲突的依赖版本。

理想情况下，我们应该将项目的特性子集拆分到它自己的模块中，因此不会污染项目的其余部分。然而，这并不总是可行的。

为了从主项目中排除这些特殊的依赖项，我们可以对它们应用 Maven 的`<optional>`标签。这迫使任何想要使用这些依赖项的用户显式地声明它们。然而，它不会强迫那些依赖项进入一个不需要它们的项目。

## 3.如何使用`<optional>`

正如我们将要看到的，我们可以包含值为`true`的`<optional>`元素，使任何 Maven 依赖项都是可选的。

假设我们有以下项目 pom:

```java
<project>
    ...
    <artifactId>project-with-optionals</artifactId>
    ...
    <dependencies>
        <dependency>
            <groupId>com.baeldung</groupId>
            <artifactId>optional-project</artifactId>
            <version>0.0.1-SNAPSHOT</version>
            <optional>true</optional>
        </dependency>
    </dependencies>
</project>
```

在这个例子中，虽然`optional-project`被标记为可选的，但是它仍然是`project-with-optionals`的一个可用的依赖项，就好像`<optional>`标签从来没有出现过一样。

为了看到`<optional>`标签的效果，我们需要创建一个依赖于`project-with-optionals`的新项目:

```java
<project>
    ...
    <artifactId>main-project</artifactId>
    ...
    <dependencies>
        <dependency>
            <groupId>com.baeldung</groupId>
            <artifactId>project-with-optionals</artifactId>
            <version>0.0.1-SNAPSHOT</version>
        </dependency>
    </dependencies>
</project>
```

现在，如果我们试图从`main-project`内部引用`optional-project`，我们会发现`optional-project`并不存在。这是因为`<optional>`标签阻止它被包含在传递中。

如果我们发现在我们的`main-project`中需要`optional-project`，我们只需要将它声明为一个依赖项。

## 4.结论

在本文中，我们看了 Maven 的`<optional>`标签。使用标签的主要好处是它可以减少项目的大小，并有助于防止版本冲突。我们还看到标签不会影响使用它的项目。

本文中的源代码可以在 Github 上找到。