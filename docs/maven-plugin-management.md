# Maven 中的插件管理

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/maven-plugin-management>

## 1.概观

Apache Maven 是一个强大的工具，它使用[插件](/web/20220628065049/https://www.baeldung.com/maven#introduction-10)来自动化和执行 Java 项目中的所有构建和报告任务。

然而，在构建中可能会使用几个不同版本和配置的插件，尤其是在一个多模块项目中。这可能会导致复杂的 POM 文件出现冗余或重复的插件工件，以及分散在各个子项目中的配置。

在本文中，我们将看到如何使用 Maven 的插件管理机制来处理这样的问题，并在整个项目中有效地维护插件。

## 2.插件配置

Maven 有两种类型的插件:

*   构建–在构建过程中执行。示例包括清理、安装和确保插件。这些应在 POM 的`build`部分进行配置。
*   报告–在网站生成期间执行，以生成各种项目报告。例子包括 Javadoc 和 Checkstyle 插件。这些在项目 POM 的`reporting`部分进行配置。

Maven 插件提供了执行和管理项目构建所需的所有有用的功能。

例如，我们可以在 POM 中声明 [Jar](https://web.archive.org/web/20220628065049/https://maven.apache.org/plugins/maven-jar-plugin/) 插件:

```java
<build>
    ....
    <plugins>
        <plugin>
            <groupId>org.apache.maven.plugins</groupId>
            <artifactId>maven-jar-plugin</artifactId>
            <version>3.2.0</version>
            ....
        </plugin>
    ....
    </plugins>
</build>
```

这里，我们在`build`部分包含了插件，以增加将我们的项目编译成`jar`的能力。

## 3.插件管理

除了`plugins`，我们还可以在 POM 的`pluginManagement`部分声明插件。这包含了`plugin`元素，就像我们之前看到的一样。然而，通过在`pluginManagement`部分添加插件，**对这个 POM 和所有继承的子 POM**变得可用。

这意味着任何子 POM 将简单地通过引用它们的`plugin`部分中的插件来继承插件执行。我们需要做的就是添加相关的`groupId`和`artifactId`，而不必复制配置或管理版本。

类似于[依赖管理](https://web.archive.org/web/20220628065049/https://maven.apache.org/guides/introduction/introduction-to-dependency-mechanism.html#dependency-management)机制，这在多模块项目中特别有用，因为它提供了一个管理插件版本和任何相关配置的中心位置。

## 4.例子

让我们从创建一个简单的包含两个子模块的多模块项目开始。我们将在父 POM 中包含[构建助手插件](https://web.archive.org/web/20220628065049/https://www.mojohaus.org/build-helper-maven-plugin/index.html)，它包含几个小目标来帮助构建生命周期。在我们的示例中，我们将使用它将一些额外的资源复制到子项目的项目输出中。

### 4.1.父 POM 配置

首先，我们将插件添加到父 POM 的`pluginManagement`部分:

```java
<pluginManagement>
    <plugins>
        <plugin>
            <groupId>org.codehaus.mojo</groupId>
            <artifactId>build-helper-maven-plugin</artifactId>
            <version>3.3.0</version>
            <executions>
                <execution>
                    <id>add-resource</id>
                    <phase>generate-resources</phase>
                    <goals>
                        <goal>add-resource</goal>
                    </goals>
                    <configuration>
                        <resources>
                            <resource>
                                <directory>src/resources</directory>
                                <targetPath>json</targetPath>
                            </resource>
                        </resources>
                    </configuration>
                </execution>
            </executions>
        </plugin>
   </plugins>
</pluginManagement>
```

这将插件的`add-resource`目标绑定到[默认 POM 生命周期](https://web.archive.org/web/20220628065049/https://maven.apache.org/guides/introduction/introduction-to-the-lifecycle.html#Built-in_Lifecycle_Bindings)中的`generate-resources`阶段。我们还指定了包含附加资源的`src/resources`目录。插件执行会根据需要将这些资源复制到项目输出中的目标位置。

接下来，让我们运行 maven 命令来确保配置有效并且构建成功:

```java
$ mvn clean test
```

运行此命令后，目标位置还不包含预期的资源。

### 4.2.子 POM 配置

现在，让我们参考子 POM 中的这个插件:

```java
<build>
    <plugins>
        <plugin>
            <groupId>org.codehaus.mojo</groupId>
            <artifactId>build-helper-maven-plugin</artifactId>
        </plugin>
    </plugins>
</build>
```

类似于依赖管理，我们不声明版本或任何插件配置。相反，子项目从父 POM 中的声明继承这些值。

最后，让我们再次运行构建并查看输出:

```java
....
[INFO] --- build-helper-maven-plugin:3.3.0:add-resource (add-resource) @ submodule-1 ---
[INFO]
[INFO] --- maven-resources-plugin:2.6:resources (default-resources) @ submodule-1 ---
[INFO] Using 'UTF-8' encoding to copy filtered resources.

[INFO] Copying 1 resource to json
....
```

在这里，插件在构建期间执行，但是只在带有相应声明的子项目中执行。因此，项目输出现在包含了来自指定项目位置的额外资源，如预期的那样。

我们应该注意到**只有** **父 POM 包含插件声明和配置**，而子项目只是根据需要引用这些。

如果需要，子项目可以自由地[修改继承的配置](/web/20220628065049/https://www.baeldung.com/maven-plugin-override-parent)。

## 5.核心插件

默认情况下，有一些 Maven [核心插件](/web/20220628065049/https://www.baeldung.com/core-maven-plugins)被用作构建生命周期的一部分。例如，`clean`和`compiler`插件不需要显式声明。

然而，我们可以在 POM 的`pluginManagement`元素中显式声明和配置这些。主要的区别是**核心插件配置自动生效，不需要子项目**中的任何引用。

让我们通过将`compiler`插件添加到熟悉的`pluginManagement`部分来尝试一下:

```java
<pluginManagement>
    ....
    <plugin>
        <groupId>org.apache.maven.plugins</groupId>
        <artifactId>maven-compiler-plugin</artifactId>
        <version>3.10.0</version>
        <configuration>
            <source>1.8</source>
            <target>1.8</target>
        </configuration>
    </plugin>
    ....
</pluginManagement>
```

这里，我们锁定了插件版本，并将其配置为使用 Java 8 来构建项目。然而，在任何子项目中都不需要额外的`plugin`声明。默认情况下，构建框架会激活此配置。因此，这种配置意味着构建必须使用 Java 8 来跨所有模块编译这个项目。

总的来说，**明确声明配置并锁定多模块项目**中所需的任何插件的版本是一个好的做法。因此，不同的子项目只能从父 POM 继承所需的插件配置，并根据需要应用它们。

这消除了重复的声明，简化了 POM 文件，并提高了构建的可重复性。

## 6.结论

在本文中，我们看到了如何集中和管理构建项目所需的 Maven 插件。

首先，我们看了插件和它们在 POM 项目中的用法。然后，我们更详细地研究了 Maven 插件管理机制，以及它如何帮助减少重复和提高构建配置的可维护性。

与往常一样，GitHub 上的示例代码[可用。](https://web.archive.org/web/20220628065049/https://github.com/eugenp/tutorials/tree/master/maven-modules/maven-simple/plugin-management)