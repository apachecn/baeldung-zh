# 如何用 Maven 创建可执行的 JAR

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/executable-jar-with-maven>

## 1。概述

在这个快速教程中，我们将重点关注**将一个 Maven 项目打包成一个可执行的 Jar 文件。**

当创建一个`jar` 文件时，我们通常想轻松地运行它，而不使用 IDE。为此，我们将讨论使用每种方法创建可执行文件的配置和优缺点。

## 延伸阅读:

## [Apache Maven 教程](/web/20220924195802/https://www.baeldung.com/maven)

A quick and practical guide to building and managing Java projects using Apache Maven.[Read more](/web/20220924195802/https://www.baeldung.com/maven) →

## [Maven 本地存储库在哪里？](/web/20220924195802/https://www.baeldung.com/maven-local-repository)

Quick tutorial showing you where Maven stores its local repo and how to change that.[Read more](/web/20220924195802/https://www.baeldung.com/maven-local-repository) →

## [带 Maven BOM 的春天](/web/20220924195802/https://www.baeldung.com/spring-maven-bom)

Learn how to use a BOM, Bill of Materials, in your Spring Maven project.[Read more](/web/20220924195802/https://www.baeldung.com/spring-maven-bom) →

## 2。配置

我们不需要任何额外的依赖来创建一个可执行文件`jar`。我们只需要创建一个 Maven Java 项目，并拥有至少一个带有`main(…)`方法的类。

在我们的例子中，我们创建了名为`ExecutableMavenJar`的 Java 类。

我们还需要确保我们的`pom.xml`包含这些元素:

```
<modelVersion>4.0.0</modelVersion>
<groupId>com.baeldung</groupId>
<artifactId>core-java</artifactId>
<version>0.1.0-SNAPSHOT</version>
<packaging>jar</packaging>
```

这里最重要的方面是类型——要创建可执行文件`jar`,请仔细检查配置是否使用了`jar`类型。

现在我们可以开始使用各种解决方案。

### 2.1。手动配置

让我们在`maven-dependency-plugin`的帮助下从手动方式开始。

我们首先将所有必需的依赖项复制到我们指定的文件夹中:

```
<plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-dependency-plugin</artifactId>
    <executions>
        <execution>
            <id>copy-dependencies</id>
            <phase>prepare-package</phase>
            <goals>
                <goal>copy-dependencies</goal>
            </goals>
            <configuration>
                <outputDirectory>
                    ${project.build.directory}/libs
                </outputDirectory>
            </configuration>
        </execution>
    </executions>
</plugin>
```

有两个重要方面需要注意。

首先，我们指定目标`copy-dependencies`，它告诉 Maven 将这些依赖项复制到指定的`outputDirectory`中。在我们的例子中，我们将在项目构建目录中创建一个名为`libs`的文件夹(通常是`target`文件夹)。

其次，我们将创建可执行文件和类路径感知的`jar`，其中包含第一步中复制的依赖项的链接:

```
<plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-jar-plugin</artifactId>
    <configuration>
        <archive>
            <manifest>
                <addClasspath>true</addClasspath>
                <classpathPrefix>libs/</classpathPrefix>
                <mainClass>
                    com.baeldung.executable.ExecutableMavenJar
                </mainClass>
            </manifest>
        </archive>
    </configuration>
</plugin>
```

其中最重要的部分是`manifest`配置。我们添加一个包含所有依赖项的类路径(文件夹`libs/`，并提供关于主类的信息。

请注意，我们需要提供一个完全限定的类名，这意味着它将包括包名。

这种方法的优点和缺点是:

*   **优点**–透明的流程，我们可以指定每一步
*   **缺点**–手动；依赖项不在最终的`jar`中，这意味着我们的可执行文件`jar`只有在`libs`文件夹对于`jar`是可访问和可见的情况下才会运行

### 2.2。Apache Maven 组装插件

Apache Maven Assembly 插件允许用户将项目输出及其依赖项、模块、站点文档和其他文件聚集到一个可运行的包中。

assembly 插件中的主要目标是`[single](https://web.archive.org/web/20220924195802/https://maven.apache.org/plugins/maven-assembly-plugin/single-mojo.html)`目标，用于创建所有的程序集(所有其他目标都被否决，并将在未来的版本中删除)。

我们来看看`pom.xml`中的配置:

```
<plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-assembly-plugin</artifactId>
    <executions>
        <execution>
            <phase>package</phase>
            <goals>
                <goal>single</goal>
            </goals>
            <configuration>
                <archive>
                <manifest>
                    <mainClass>
                        com.baeldung.executable.ExecutableMavenJar
                    </mainClass>
                </manifest>
                </archive>
                <descriptorRefs>
                    <descriptorRef>jar-with-dependencies</descriptorRef>
                </descriptorRefs>
            </configuration>
        </execution>
    </executions>
</plugin>
```

与手工方法类似，我们需要提供关于主类的信息。不同之处在于 Maven Assembly 插件会自动将所有需要的依赖项复制到一个`jar`文件中。

在配置代码的`descriptorRefs` 部分，我们提供了将被添加到项目名称中的名称。

我们示例中的输出将被命名为`core-java-jar-with-dependencies.jar`。

*   **优点**–依赖于`jar`文件，只有一个文件
*   **缺点**——包装我们的工件的基本控制，例如，没有类重定位支持

### 2.3。Apache Maven Shade 插件

Apache Maven Shade 插件提供了将工件打包到一个`uber-jar`中的能力，它包含了运行项目所需的所有依赖项。此外，它支持对一些依赖项的包进行着色，即重命名。

我们来看看配置:

```
<plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-shade-plugin</artifactId>
    <executions>
        <execution>
            <goals>
                <goal>shade</goal>
            </goals>
            <configuration>
                <shadedArtifactAttached>true</shadedArtifactAttached>
                <transformers>
                    <transformer implementation=
                      "org.apache.maven.plugins.shade.resource.ManifestResourceTransformer">
                        <mainClass>com.baeldung.executable.ExecutableMavenJar</mainClass>
                </transformer>
            </transformers>
        </configuration>
        </execution>
    </executions>
</plugin>
```

这种配置有三个主要部分。

首先，`<shadedArtifactAttached>`标记所有要打包到`jar`中的依赖项。

其次，我们需要指定[转换器实现](https://web.archive.org/web/20220924195802/https://maven.apache.org/plugins/maven-shade-plugin/usage.html)；在我们的例子中，我们使用了标准的。

最后，我们需要指定应用程序的主类。

输出文件将被命名为`core-java-0.1.0-SNAPSHOT-shaded.jar`，其中`core-java` 是我们的项目名称，后跟快照版本和插件名称。

*   **优点**—`jar`文件中的依赖关系，打包工件的高级控制，带有阴影和类重定位
*   **缺点**–复杂的配置(尤其是当我们想要使用高级功能时)

### 2.4。一个 Jar Maven 插件

创建可执行文件`jar`的另一个选项是 One Jar 项目。

这提供了一个定制的类加载器，它知道如何从归档中的 jar 加载类和资源，而不是从文件系统中的`jars`加载。

我们来看看配置:

```
<plugin>
    <groupId>com.jolira</groupId>
    <artifactId>onejar-maven-plugin</artifactId>
    <executions>
        <execution>
            <configuration>
                <mainClass>org.baeldung.executable.
                  ExecutableMavenJar</mainClass>
                <attachToBuild>true</attachToBuild>
                <filename>
                  ${project.build.finalName}.${project.packaging}
                </filename>
            </configuration>
            <goals>
                <goal>one-jar</goal>
            </goals>
        </execution>
    </executions>
</plugin>
```

如配置所示，我们需要通过使用`attachToBuild` `= true`来指定主类并附加所有要构建的依赖项。

此外，我们应该提供输出文件名。而且，Maven 的目标是`one-jar`。请注意，One Jar 是一个商业解决方案，它将使依赖关系`jars`在运行时不会扩展到文件系统中。

*   **优点**——干净的委托模型，允许类在一个 Jar 的顶层，支持外部`jars`并且可以支持本地库
*   **缺点**–自 2012 年起不再积极支持

### 2.5。Spring Boot Maven 插件

最后，我们要看的最后一个解决方案是 Spring Boot Maven 插件。

这允许打包可执行文件`jar`或`war`存档，并“就地”运行应用程序

要使用它，我们至少需要使用 Maven 版。详细描述见[此处](https://web.archive.org/web/20220924195802/https://docs.spring.io/spring-boot/docs/1.4.1.RELEASE/maven-plugin/)。

让我们看一下配置:

```
<plugin>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-maven-plugin</artifactId>
    <executions>
        <execution>
            <goals>
                <goal>repackage</goal>
            </goals>
            <configuration>
                <classifier>spring-boot</classifier>
                <mainClass>
                  com.baeldung.executable.ExecutableMavenJar
                </mainClass>
            </configuration>
        </execution>
    </executions>
</plugin>
```

Spring 插件和其他插件有两个不同之处:执行的目标被称为`repackage`，分类器被命名为`spring-boot`。

注意，我们不需要有 Spring Boot 应用程序来使用这个插件。

*   **优点**—`jar`文件中的依赖项，我们可以在每个可访问的位置运行它，高级控制我们工件的打包，从`jar`文件中排除依赖项等等。，打包的`war`文件也一样
*   增加潜在的不必要的 Spring 和 Spring Boot 相关的类

### 2.6。带有可执行 Tomcat 的 Web 应用程序

在最后一部分中，我们想介绍一个打包在 j `ar`文件中的独立 web 应用程序。

为了做到这一点，我们需要使用不同的插件，为创建可执行的 jar 文件而设计:

```
<plugin>
    <groupId>org.apache.tomcat.maven</groupId>
    <artifactId>tomcat7-maven-plugin</artifactId>
    <version>2.0</version>
    <executions>
        <execution>
            <id>tomcat-run</id>
            <goals>
                <goal>exec-war-only</goal>
            </goals>
            <phase>package</phase>
            <configuration>
                <path>/</path>
                <enableNaming>false</enableNaming>
                <finalName>webapp.jar</finalName>
                <charset>utf-8</charset>
            </configuration>
        </execution>
    </executions>
</plugin>
```

将`goal`设置为`exec-war-only`，我们服务器的`path`在`configuration` 标签中指定，带有附加属性，如`finalName`、 `charset` 等。

为了构建 j `ar`，我们运行`man package`，这将导致在我们的`target`目录中创建`webapp.jar`。

要运行应用程序，我们只需在控制台中编写 `java -jar target/webapp.jar` ，并尝试通过在浏览器中指定`localhost:8080`来测试它。

*   **优点**–只有一个文件，易于部署和运行
*   **缺点**–由于将 Tomcat 嵌入式发行版打包在一个 w `ar` 文件中，文件的大小要大得多

注意这是这个插件的最新版本，支持 Tomcat7 服务器。为了避免错误，我们可以检查我们对 Servlets 的依赖关系是否已经将`scope`设置为`provided`，否则，在可执行文件`jar`的`runtime`处将会发生冲突:

```
<dependency>
    <groupId>javax.servlet</groupId>
    <artifactId>javax.servlet-api</artifactId>
    <scope>provided</scope>
</dependency>
```

## 3。结论

在本文中，我们描述了用各种 Maven 插件创建可执行文件`jar`的许多方法。

本教程的完整实现可以在这些 GitHub 项目中找到:[可执行 jar](https://web.archive.org/web/20220924195802/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-jar "The Full Registration/Authentication Example Project on Github ") 和[可执行 war](https://web.archive.org/web/20220924195802/https://github.com/eugenp/tutorials/tree/master/spring-web-modules/spring-thymeleaf-5) 。

**如何测试？**为了将项目编译成可执行文件`jar`，请用`mvn clean package`命令运行 Maven。

这篇文章有望提供更多的见解，并帮助您根据自己的需要找到自己喜欢的方法。

最后一点:我们希望确保我们捆绑的 jar 的许可证不会禁止这种操作。一般情况下不会这样，但这是值得考虑的。