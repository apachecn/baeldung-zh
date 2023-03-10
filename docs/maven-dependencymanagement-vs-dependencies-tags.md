# Maven 依赖关系管理与依赖关系标签

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/maven-dependencymanagement-vs-dependencies-tags>

## 1.概观

在本教程中，我们将回顾两个重要的 [Maven](/web/20220829071246/https://www.baeldung.com/maven-guide) 标签— `dependencyManagement `和`dependencies`。

这些功能对于多模块项目尤其有用。

我们将回顾这两个标签的相似之处和不同之处，我们还将看看开发人员在使用它们时会犯的一些常见错误，这些错误可能会导致混淆。

## 2.使用

一般来说，当我们在`dependencies`标签中定义依赖关系时，我们使用`dependencyManagement` 标签来避免重复`version` 和`scope`标签。这样，所需的依赖关系就在一个中心 POM 文件中声明了。

### 2.1.`dependencyManagement`

这个标签由一个`dependencies`标签组成，标签本身可能包含多个`dependency`标签。每个`dependency`应该至少有三个主标签:`groupId`、`artifactId,`和`version`。让我们看一个例子:

```java
<dependencyManagement>
    <dependencies>
        <dependency>
            <groupId>org.apache.commons</groupId>
            <artifactId>commons-lang3</artifactId>
            <version>3.12.0</version>
        </dependency>
    </dependencies>
</dependencyManagement> 
```

上面的代码只是声明了新的工件`commons-lang3`，但是并没有真正将它添加到项目依赖资源列表中。

### 2.2.`dependencies`

这个标签包含一个`dependency`标签列表。每个`dependency `应该至少有两个主标签，分别是`groupId `和`artifactId.`

让我们看一个简单的例子:

```java
<dependencies>
    <dependency>
        <groupId>org.apache.commons</groupId>
        <artifactId>commons-lang3</artifactId>
        <version>3.12.0</version>
    </dependency>
</dependencies> 
```

**如果我们以前在 POM 文件**中使用过`dependencyManagement `标签，那么`version`和`scope`标签可以被隐式继承:

```java
<dependencies>
    <dependency>
        <groupId>org.apache.commons</groupId>
        <artifactId>commons-lang3</artifactId>
    </dependency>
</dependencies> 
```

## 3.类似

这两个标签都旨在声明某种第三方或子模块依赖性。它们相辅相成。

事实上，我们通常在`dependencies`标签之前定义一次`dependencyManagement` 标签。这用于声明 POM 文件中的依赖关系。**这个声明只是一个公告，并没有真正给项目添加依赖项。**

让我们看一个添加 JUnit 库依赖项的示例:

```java
<dependencyManagement>
    <dependencies>
        <dependency>
            <groupId>junit</groupId>
            <artifactId>junit</artifactId>
            <version>4.13.2</version>
            <scope>test</scope>
        </dependency>
    </dependencies>
</dependencyManagement> 
```

正如我们在上面的代码中看到的，有一个`dependencyManagement`标签，它本身包含另一个`dependencies` 标签。

现在，让我们看看代码的另一面，它向项目添加了实际的依赖关系:

```java
<dependencies>
    <dependency>
        <groupId>junit</groupId>
        <artifactId>junit</artifactId>
    </dependency>
</dependencies> 
```

所以，当前的标签与前一个非常相似。他们都将定义一个依赖列表。当然，还有一些小的差异，我们很快就会谈到。

相同的`groupId` 和`artifactId` 标记在两个代码片段中重复出现，它们之间存在有意义的关联:它们都引用同一个工件。

正如我们所看到的，在我们后面的`dependency`标签中没有任何`version`标签。令人惊讶的是，它是有效的语法，可以毫无问题地解析和编译。原因很容易猜到:它将使用`dependencyManagement`宣称的版本。

## 4.差异

### 4.1。结构差异

**正如我们前面提到的，这两个标签之间的主要结构差异是继承的逻辑。**我们在`dependencyManagement`标签中定义版本，然后我们可以使用提到的版本，而不用在下一个`dependencies` 标签中指定它。

### 4.2。行为差异

**`dependencyManagement` 只是一个声明，并没有真正添加依赖。**本节中声明的`dependencies`必须稍后由`dependencies`标签使用。正是`dependencies`标签导致了真正的依赖发生。在上面的例子中，`dependencyManagement`标签不会将`junit`库添加到任何范围内。它只是未来`dependencies`标签的一个声明。

## 5.真实世界的例子

几乎所有基于 Maven 的开源项目都使用这种机制。

让我们看一个来自[Maven 项目](https://web.archive.org/web/20220829071246/https://github.com/apache/maven/blob/master/pom.xml)本身的例子。我们看到了`hamcrest-core`依赖，它存在于 Maven 项目中。它首先在`dependencyManagement `标签中声明，然后由主`dependencies`标签导入:

```java
<dependencyManagement>
    <dependencies>
        <dependency>
            <groupId>org.hamcrest</groupId>
            <artifactId>hamcrest-core</artifactId>
            <version>2.2</version>
            <scope>test</scope>
        </dependency>
    </dependencies>
</dependencyManagement>

<dependencies>
    <dependency>
        <groupId>org.hamcrest</groupId>
        <artifactId>hamcrest-core</artifactId>
        <scope>test</scope>
    </dependency>
</dependencies> 
```

## 6.常见使用案例

此功能的一个非常常见的用例是多模块项目。

假设我们有一个由不同模块组成的大项目。每个模块都有自己的依赖项，每个开发人员可能会使用不同版本的依赖项。那么它可能导致不同工件版本的网状结构，这也可能导致困难和难以解决的冲突。

**这个问题的简单解决方案肯定是在根 POM 文件(通常称为“父”)中使用`dependencyManagement` 标记，然后在子 POM 文件(子模块)甚至父模块本身(如果适用)中使用`dependencies` 。**

如果我们只有一个模块，那么使用这个特性是否有意义呢？尽管这在多模块环境中非常有用，但即使在单模块项目中，也可以将它作为最佳实践来遵循。这有助于项目的可读性，也使它可以扩展到一个多模块的项目。

## 7.常见错误

一个常见的错误是只在`dependencyManagement` 部分定义了依赖关系，而没有将它包含在`dependencies` 标签中。在这种情况下，我们会遇到编译或运行时错误，这取决于提到的`scope`。

让我们看一个例子:

```java
<dependencyManagement>
    <dependencies>
        <dependency>
            <groupId>org.apache.commons</groupId>
            <artifactId>commons-lang3</artifactId>
            <version>3.12.0</version>
        </dependency>
        ...
    </dependencies>
</dependencyManagement> 
```

想象一下上面的 POM 代码片段。然后假设我们要在子模块源文件中使用这个库:

```java
import org.apache.commons.lang3.StringUtils;

public class Main {

    public static void main(String[] args) {
        StringUtils.isBlank(" ");
    }
} 
```

由于缺少库，此代码将无法编译。编译器报错:

```java
[ERROR] Failed to execute goal compile (default-compile) on project sample-module: Compilation failure
[ERROR] ~/sample-module/src/main/java/com/baeldung/Main.java:[3,32] package org.apache.commons.lang3 does not exist
```

为了避免这个错误，在子模块 POM 文件中添加下面的`dependencies`标记就足够了:

```java
<dependencies>
    <dependency>
        <groupId>org.apache.commons</groupId>
        <artifactId>commons-lang3</artifactId>
    </dependency>
</dependencies> 
```

## 8.结论

在本教程中，我们比较了 Maven 的`dependencyManagement`和`dependencies`标签。然后，我们回顾了它们的相同点和不同点，并了解了它们是如何协同工作的。

像往常一样，这些例子的代码可以在 GitHub 的[上找到。](https://web.archive.org/web/20220829071246/https://github.com/eugenp/tutorials/tree/master/maven-modules/maven-simple/maven-dependency)