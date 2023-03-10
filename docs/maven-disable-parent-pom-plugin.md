# 如何禁用父 POM 中定义的 Maven 插件

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/maven-disable-parent-pom-plugin>

## 1.概观

Maven 允许我们使用继承的概念来构建项目。当一个父 POM 定义了一个插件，所有的子模块都会继承它。

但是如果我们不想从父 POM 继承一个插件，并且我们不能修改父 POM，会发生什么呢？

在本教程中，我们将看看禁用 Maven 插件的几种不同方法，特别是在父 POM 中定义的 [Maven Enforcer 插件](https://web.archive.org/web/20220524061646/https://maven.apache.org/enforcer/maven-enforcer-plugin/)。

## 2.我们什么时候禁用父 POM 中定义的插件？

在我们继续之前，让我们思考一下为什么我们可能需要这样做。

**Maven 更喜欢常规而不是配置**。我们需要记住，虽然禁用插件可能是我们最快的解决方案，但它可能不是项目的最佳解决方案。

当 Maven 项目的原作者没有预见到我们的情况，并且我们没有办法自己修改父模块时，可能会出现禁用父 POM 中的插件的需求。

假设原作者假设某个特定的文件应该一直存在。但是，我们的模块拥有这个文件是没有意义的。例如，父 POM 可能在每个模块中强制存在一个许可文件，而我们没有。我们希望禁用规则实施，而不是添加一个可能会引起混淆的空文件。

让我们通过在 Maven 项目中添加一个实现`maven-enforcer-plugin`的父模块来设置这个场景:

```java
<plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-enforcer-plugin</artifactId>
    <version>3.0.0</version>
</plugin> 
```

接下来，让我们向插件添加一个执行来强制执行一个规则，即名为`file-that-must-exist.txt` 的文件必须存在于每个模块的`src`目录中:

```java
<executions>
    <execution>
        <id>enforce-file-exists</id>
	<goals>
	    <goal>enforce</goal>
	</goals>
	<configuration>
	    <rules>
	        <requireFilesExist>
		    <files>
		        <file>${project.basedir}/src/file-that-must-exist.txt</file>
		    </files>
		</requireFilesExist>
            </rules>
	</configuration>
    </execution>
</executions> 
```

如果`file-that-must-exist.txt`不存在，那么构建将会失败。

当子模块从它们的父模块继承插件时，所有的子模块都必须遵守这个规则。

让我们来看看在我们的子 POM 中禁用此规则的几种方法。

## 3.我们如何禁用父 POM 中定义的插件？

首先，让我们假设重构 Maven 项目或改变父 POM 是不可接受的解决方案。**如果我们可以修改父模块，那么我们可以通过在父 POM 中实现一个`[pluginManagement](https://web.archive.org/web/20220524061646/http://baeldung.com/maven-plugin-management)` 部分来解决这个问题。**

我们可能无法修改父模块，因为我们不拥有该项目，所以我们无权在我们的模块之外进行更改。这可能是由于时间限制——重组一个项目需要时间，所以在子模块中禁用插件更方便。

此外，**我们将假设该插件实际上需要被禁用。许多插件运行起来不会有任何问题，即使是在他们不打算使用的模块上。**

例如，假设我们有一个复制 Java 文件的插件。如果我们有一个没有 Java 文件的子项目，那么插件很可能不会复制任何文件。它会这样做而不会引起问题。在这种情况下，让插件运行更简单也更常规。

让我们假设在考虑了上述内容后，我们肯定需要用我们的模块禁用插件。

我们可以这样做的一个方法是配置`skip`参数。

### 3.1.配置`Skip`参数

**许多插件都有一个`skip`参数。我们可以使用`skip`参数来禁用插件。**

如果我们看一下`maven-enforcer-plugin`的文档[，我们可以看到它有一个我们可以实现的`skip`参数。](https://web.archive.org/web/20220524061646/https://maven.apache.org/enforcer/maven-enforcer-plugin/enforce-mojo.html#skip)

**对`skip`参数的支持应该是我们检查的第一件事，因为这是最简单的解决方案**，也是最常规的。

让我们添加一个空的子模块，只包含 POM。如果我们使用`mvn clean install` 命令构建模块，我们会看到构建失败。这是因为`file-that-must-exist.txt`在我们的模块中不存在，这是从我们的父模块继承规则所需要的。

让我们将下面几行添加到子 POM 中，以启用`skip`参数:

```java
<plugin>
    <artifactId>maven-enforcer-plugin</artifactId>
    <configuration>
        <skip>true</skip>
    </configuration>
</plugin>
```

现在，如果我们运行项目，我们会看到构建是成功的。

然而，并不是所有的插件都会有一个`skip`参数。所以，如果我们使用一个没有插件的插件，我们能做什么呢？

### 3.2.移除`Phase`参数

Maven 目标只有在绑定到构建阶段时才会运行。

在我们的父 POM 中，我们已经将 enforce 目标配置为使用 id `enforce-file-exists`运行。

因为我们还没有为`enforce-file-exists`指定一个`phase` 参数，所以它将使用默认值来执行目标。我们可以从[文档](https://web.archive.org/web/20220524061646/https://maven.apache.org/enforcer/maven-enforcer-plugin/enforce-mojo.html)中看到，默认是`validate`构建阶段。

我们可以通过为`phase`参数指定一个替代值，在替代构建阶段执行目标。

利用这一点，**我们可以将`phase`参数设置为一个不存在的值。这意味着构建阶段永远不会被执行。**因此目标不会被执行，有效地禁用了插件:

```java
<plugin>
    <artifactId>maven-enforcer-plugin</artifactId>
    <executions>
        <execution>
            <id>enforce-file-exists</id>
	    <phase>any-value-that-is-not-a-phase</phase>
	</execution>
    </executions>
</plugin>
```

为了让以后看我们代码的人清楚，我们希望将`phase`设置为一个清晰的名称，比如“none”或“null”。

然而，也许最清楚的方法是完全清除`phase`参数:

```java
<execution>
    <id>enforce-file-exists</id>
    <phase/>
</execution>
```

因为 execution empty 的`phase`现在是空的，所以目标不会被绑定到运行的构建阶段。这有效地禁用了插件。

我们可以看到，当我们运行构建时，`enforce-file-exists`根本没有为我们的子模块运行。

## 4.结论

在本文中，我们讨论了为什么我们可能选择禁用父 POM 中定义的插件。我们看到禁用插件可能并不总是最好的办法，因为 Maven 更喜欢约定胜于配置。

然后，我们看了一个简单的例子，其中我们禁用了父 POM 声明的`maven-enforcer-plugin`。

首先，**我们演示了如果插件有参数的话，我们可以配置插件的`skip`参数。**我们发现这是最传统的方法。

最后，**我们了解到清除插件的`phase`参数将有效地禁用它。**

与往常一样，GitHub 上的示例项目[可用。](https://web.archive.org/web/20220524061646/https://github.com/eugenp/tutorials/tree/master/maven-modules/maven-parent-pom-resolution)