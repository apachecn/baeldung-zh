# 用 Maven 下载源代码和 Javadocs

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/maven-download-sources-javadoc>

## 1.概观

浏览不同库和框架的源代码和文档是了解它们的好方法。

在这个简短的教程中，我们将看到如何配置 Maven，或者让 Maven 为我们下载依赖源代码和它们的 Javadocs。

## 2.命令行

默认情况下，Maven 只下载每个依赖项的实际 JAR 文件，而不是源文件和文档文件。

**要下载源代码**，首先，我们应该**导航到包含`pom.xml `的目录，然后执行命令:**

```java
mvn dependency:sources
```

下载源代码可能需要一段时间。类似地，**为了只下载 Javadocs，我们可以发出命令**:

```java
mvn dependency:resolve -Dclassifier=javadoc
```

当然，我们也可以用一个命令下载这两个文件:

```java
mvn dependency:sources dependency:resolve -Dclassifier=javadoc
```

显然，如果我们在发出这些命令后添加一个新的依赖项，我们必须重新发出命令来为新的依赖项下载源代码和 Javadocs。

## 3.Maven 设置

**也可以下载所有 Maven 项目的源代码和文档**。为此，我们应该编辑或创建一个`~/m2/settings.xml `文件，并向其中添加以下配置:

```java
<settings>
    <!-- ... other settings omitted ... -->
    <profiles>
        <profile>
            <id>downloadSources</id>
            <properties>
                <downloadSources>true</downloadSources>
                <downloadJavadocs>true</downloadJavadocs>
            </properties>
        </profile>
    </profiles>

    <activeProfiles>
        <activeProfile>downloadSources</activeProfile>
    </activeProfiles>
</settings>
```

如上所示，我们正在创建一个配置文件，并在默认情况下激活它。在这个概要文件中，我们设置了两个属性，告诉 Maven 下载源代码和文档。此外，Maven 会将这些设置应用于所有项目。

## 4.`pom.xml`

甚至可以把这个配置放到`pom.xml`里。**通过这种方式，我们迫使所有项目贡献者下载源代码和文档，作为依赖性解决方案的一部分**:

```java
<build>
    <plugins>
        <plugin>
            <groupId>org.apache.maven.plugins</groupId>
            <artifactId>maven-dependency-plugin</artifactId>
            <version>3.1.2</version>
            <executions>
                <execution>
                    <goals>
                        <goal>sources</goal>
                        <goal>resolve</goal>
                    </goals>
                    <configuration>
                        <classifier>javadoc</classifier>
                    </configuration>
                </execution>
            </executions>
        </plugin>
    </plugins>
</build>
```

在这里，我们配置 [`maven-dependency-plugin`](https://web.archive.org/web/20220626210531/https://maven.apache.org/plugins/maven-dependency-plugin/) 来下载源代码和文档。

## 5\. IDE Setup

我们也可以设置我们最喜欢的 ide 来为我们做这件事。例如，在 IntelliJ IDEA 中，我们只需转到`Preference > Build, Execution, Deployment > Build Tools > Maven > importing `并检查源代码和文档复选框:

[![](img/a4cc672c4854a718e5c440c35fc86322.png)](/web/20220626210531/https://www.baeldung.com/wp-content/uploads/2020/07/idea-src-doc.png)

## 6.结论

在这个快速教程中，我们看到了如何以各种方式在 Maven 中下载依赖源代码和文档，从命令行解决方案到每个项目或系统范围的配置。