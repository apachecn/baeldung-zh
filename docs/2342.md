# Maven 原型指南

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/maven-archetype>

## 1。概述

Maven 原型是一种项目的抽象，可以实例化为具体的定制 Maven 项目。简而言之，**它是一个模板项目模板，从这个模板项目模板中可以创建其他项目**。

使用原型的主要好处是标准化项目开发，并使开发人员能够在更快地引导他们的项目的同时轻松地遵循最佳实践。

在本教程中，我们将看看如何创建一个定制原型，然后如何使用它通过`maven-archetype-plugin.`生成一个 Maven 项目

## 2。Maven 原型描述符

**Maven 原型描述符是原型项目**的核心。它是一个名为`archetype-metadata.xml`的 XML 文件，位于 jar 的`META-INF/maven`目录中。

它被用来描述原型的元数据:

```
<archetype-descriptor
  ...
  name="custom-archetype">

    <requiredProperties>
        <requiredProperty key="foo">
            <defaultValue>bar</defaultValue>
        </requiredProperty>
    </requiredProperties>

    <fileSets>
        <fileSet filtered="true" packaged="true">
            <directory>src/main/java</directory>
            <includes>
                <include>**/*.java</include>
            </includes>
        </fileSet>
    </fileSets>

    <modules>
        <module name="sub-module"></module>
    </modules>

</archetype-descriptor>
```

`requiredProperties`标签用于在项目生成期间提供属性。因此，我们将被提示为它们提供值，并选择接受默认值。

`fileSets,`另一方面，用于配置哪些资源将被复制到具体生成的项目中。过滤后的文件意味着占位符将在生成过程中被提供的值替换。

并且，通过在`fileSet`中使用`packaged=”true”`，我们说选择的文件将被添加到由`package `属性指定的文件夹层次结构中。

如果我们想要生成一个多模块项目，那么标签`modules`可以帮助配置生成项目的所有模块。

注意，这个文件是关于原型 2 及以上的。在 1.0.x 版本中，这个文件被称为`archetype.xml`，它有一个不同的结构。

要了解更多信息，一定要看看官方的 Apache 文档。

## 3。如何创建原型

**原型是一个普通的 Maven 项目，具有以下额外内容**:

*   `src/main/resources/archetype-resources `是将资源复制到新创建项目的模板
*   用于描述原型元数据的描述符

要手动创建原型，我们可以从新创建的 Maven 项目开始，然后我们可以添加上面提到的资源。

或者，我们可以使用`archetype-maven-plugin, `生成它，然后定制`archetype-resources`目录和`archetype-metadata.xml`文件的内容。

为了生成原型，我们可以使用:

```
mvn archetype:generate -B -DarchetypeArtifactId=maven-archetype-archetype \
  -DarchetypeGroupId=maven-archetype \
  -DgroupId=com.baeldung \
  -DartifactId=test-archetype
```

我们还可以从现有的 Maven 项目中创建原型:

```
mvn archetype:create-from-project
```

它在`target/generated-sources/archetype,`中生成，随时可以使用。

无论我们如何创建原型，我们都将以下面的结构结束:

```
archetype-root/
├── pom.xml
└── src
    └── main
        ├── java
        └── resources
            ├── archetype-resources
            │   ├── pom.xml
            │   └── src
            └── META-INF
                └── maven
                    └── archetype-metadata.xml
```

我们现在可以通过将资源放入`archetype-resources`目录并在`archetype-metadata.xml`文件中配置它们来开始构建我们的原型。

## 4。构建原型

现在我们准备定制我们的原型。作为这个过程的亮点，我们将展示一个简单的 Maven 原型的创建，用于生成基于 JAX RS 2.1 的 RESTful 应用程序。

姑且称之为`maven-archetype`。

### 4.1。原型包装

让我们从修改位于`the maven-archetype directory`下的原型项目的`pom.xml `开始:

```
<packaging>maven-archetype</packaging>
```

得益于`archetype-packaging`扩展，这种类型的包装是可用的:

```
<build>
    <extensions>
        <extension>
            <groupId>org.apache.maven.archetype</groupId>
            <artifactId>archetype-packaging</artifactId>
            <version>3.0.1</version>
        </extension>
    </extensions>
    <!--....-->
</build>
```

### 4.2。添加`pom.xml`

现在让我们在`archetype-resources`目录下创建一个`pom.xml`文件:

```
<project ...>

    <groupId>${groupId}</groupId>
    <artifactId>${artifactId}</artifactId>
    <version>${version}</version>
    <packaging>war</packaging>

    <dependencies>
        <dependency>
            <groupId>javax.ws.rs</groupId>
            <artifactId>javax.ws.rs-api</artifactId>
            <version>2.1</version>
            <scope>provided</scope>
        </dependency>        
    </dependencies>

</project>
```

我们可以看到，`groupId, artifactId`和`version`是参数化的。在根据这个原型创建新项目的过程中，它们将被替换。

我们可以将生成的项目所需的一切，如依赖项和插件，放在`pom.xml`中。这里，我们添加了 JAX RS 依赖，因为原型将用于生成基于 RESTful 的应用程序。

### 4.3。添加所需资源

接下来，我们可以在`archetype-resources/src/main/java`中为我们的应用程序添加一些 Java 代码。

用于配置 JAX 遥感应用程序的类:

```
package ${package};
// import
@ApplicationPath("${app-path}")
public class AppConfig extends Application {
}
```

和 ping 资源的类:

```
@Path("ping")
public class PingResource{
    //...
}
```

最后，将服务器配置文件`server.xml`放在`archetype-resources/src/main/config/liberty`中。

### 4.4。配置元数据

在添加了所有需要的资源之后，我们现在可以通过`archetype-metadata.xml`文件来配置在生成过程中将复制哪些资源。

我们可以告诉我们的原型，我们希望复制所有的 Java 源文件:

```
<archetype-descriptor
  name="maven-archetype">

    <!--...-->
    <fileSets>
        <fileSet filtered="true" packaged="true">
            <directory>src/main/java</directory>
            <includes>
                <include>**/*.java</include>
            </includes>
        </fileSet>
        <fileSet>
            <directory>src/main/config/liberty</directory>
            <includes>
                <include>server.xml</include>
            </includes>
        </fileSet>
    </fileSets>

</archetype-descriptor>
```

这里，我们希望复制来自`src/main/java`目录的所有 Java 文件，以及来自`src/main/config/liberty,`的`server.xml`文件。

### 4.5。安装原型

既然我们已经完成了所有这些，我们可以通过调用以下命令来安装原型:

```
mvn install
```

此时，原型被注册到位于 Maven 本地存储库中的文件`archetype-catalog.xml,`中，因此可以使用了。

## 5.使用安装的原型

**`maven-archetype-plugin`允许用户通过`generate`目标和现有原型** `.`创建一个 Maven 项目，关于这个插件的更多信息，可以访问[主页](https://web.archive.org/web/20220628124317/https://maven.apache.org/archetype/maven-archetype-plugin/index.html)。

这个命令使用这个插件从我们的原型生成一个 Maven 项目:

```
mvn archetype:generate -DarchetypeGroupId=com.baeldung.archetypes
                       -DarchetypeArtifactId=maven-archetype
                       -DarchetypeVersion=1.0-SNAPSHOT
                       -DgroupId=com.baeldung.restful
                       -DartifactId=cool-jaxrs-sample
                       -Dversion=1.0-SNAPSHOT
```

然后，我们应该将原型的 GAV 作为参数传递给`maven-archetype-plugin:generate`目标。我们也可以通过我们想要生成的具体项目的 GAV，否则，我们可以以交互方式提供它们。

因此，具体的`cool-jaxrs-sample`生成的项目已经准备好运行，无需任何更改。因此，我们可以通过调用以下命令来运行它:

```
mvn package liberty:run
```

然后我们可以访问这个 URL:

```
http://localhost:9080/cool-jaxrs-sample/<app-path>/ping
```

## 6。结论

在本文中，我们展示了如何构建和使用 Maven 原型。

我们已经演示了如何创建原型，然后如何通过`archetype-metadata.xml`文件配置模板资源。

像往常一样，代码可以在 Github 上找到[。](https://web.archive.org/web/20220628124317/https://github.com/eugenp/tutorials/tree/master/maven-modules/maven-archetype)