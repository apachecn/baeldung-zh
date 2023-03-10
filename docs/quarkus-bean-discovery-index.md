# 使用 jandex 索引发现 quartus bean

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/quarkus-bean-discovery-index>

## 1.概观

在本文中，我们将了解 Quarkus 和经典 Jakarta EE 环境中 bean 发现的区别。我们将关注如何确保 Quarkus 能够发现外部模块中带注释的类。

## 2.为什么夸库斯需要索引

Quarkus 的一个主要优势是其极快的启动时间。为了实现这一点，Quarkus 将类路径注释扫描等步骤从运行时提前到构建时。**为此，我们需要在构建时宣布所有的依赖项。**

因此，在运行时环境中通过类路径扩展来增强应用程序是不可能的。在构建过程中收集元数据时，索引会加入游戏。索引意味着在索引文件中存储元数据。这允许应用程序在启动时或任何需要的时候快速读出它。

让我们用一个简单的草图来看看两者的区别:

[![](img/f0d522fa5c61937bb6f78a1729ce46ee.png)](/web/20220525135902/https://www.baeldung.com/wp-content/uploads/2021/11/q1.svg)

Quarkus 使用 [Jandex](https://web.archive.org/web/20220525135902/https://github.com/wildfly/jandex) 来创建和读取索引。

## 3.创建索引

对于我们 Quarkus 项目中的类，我们不需要做任何特殊的事情 Quarkus Maven 插件会自动生成索引。但是，我们需要注意依赖性——项目内部模块以及外部库。

### 3.1.Jandex Maven 插件

对于我们自己的模块来说，实现这一点最明显的方法是使用 [Jandex Maven 插件](https://web.archive.org/web/20220525135902/https://github.com/wildfly/jandex-maven-plugin):

```java
<build>
    <plugins>
        <plugin>
            <!-- https://github.com/wildfly/jandex-maven-plugin -->
            <groupId>org.jboss.jandex</groupId>
            <artifactId>jandex-maven-plugin</artifactId>
            <version>1.2.1</version>
            <executions>
                <execution>
                    <id>make-index</id>
                    <goals>
                        <!-- phase is 'process-classes by default' -->
                        <goal>jandex</goal>
                    </goals>
                </execution>
            </executions>
        </plugin>
    </plugins>
</build>
```

这个插件创建了一个打包到 JAR 中的`“META-INF/jandex.idx”`文件。当库在运行时提供时，Quarkus 将读出这个文件。因此，包含这样一个文件的每个库都隐式地增强了索引。

对于 Gradle 构建，我们可以使用`org.kordamp.gradle.jandex`插件:

```java
plugins {
    id 'org.kordamp.gradle.jandex' version '0.11.0'
}
```

### 3.2.应用程序属性

如果我们不能修改依赖项(例如，在外部库的情况下)，我们需要在 Quarkus 项目的`application.properties`文件中明确指定它们:

`quarkus.index-dependency.<name>.group-id=<groupId>
quarkus.index-dependency.<name>.artifact-id=<artifactId>
quarkus.index-dependency.<name>.classifier=(optional)`

### 3.3.框架特定的含义

除了使用 Jandex Maven 插件，一个模块还可以包含一个`META-INF/beans.xml`文件。这实际上是 CDI 技术的一部分，[通过一些调整](https://web.archive.org/web/20220525135902/https://quarkus.io/guides/cdi-reference)被 Quarkus 采用，但是**我们并不局限于只使用 CDI 管理的 beans】。例如，我们也可以声明 JAX 遥感资源，因为索引的范围是整个模块。**

## 4.结论

本文已经确定 Quarkus 需要一个 Jandex 索引来在运行时检测带注释的类。索引是在构建时生成的，因此标准技术不会检测到构建后添加到类路径中的带注释的类。

和往常一样，GitHub 上的所有代码[都是可用的。有一个多模块项目，包含一个 Quarkus 应用程序和一些提供 CDI 管理的 beans 的依赖项。](https://web.archive.org/web/20220525135902/https://github.com/eugenp/tutorials/tree/master/quarkus-jandex)