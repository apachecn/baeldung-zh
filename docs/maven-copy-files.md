# 用 Maven 复制文件

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/maven-copy-files>

## 1.介绍

Maven 是一个构建自动化工具，允许 Java 开发人员从一个集中的位置 POM(项目对象模型)管理项目的构建、报告和文档。

当我们构建一个 Java 项目时，我们经常需要将任意的项目资源复制到输出构建中的特定位置——我们可以通过使用几个不同的插件使用 Maven 来实现这一点。

**在本教程中，我们将构建一个 Java 项目，并在构建输出**中将一个特定文件复制到一个目的地，使用:

*   `[maven-resources-plugin](https://web.archive.org/web/20221128040854/https://maven.apache.org/plugins/maven-resources-plugin/)`
*   `[maven-antrun-plugin](https://web.archive.org/web/20221128040854/https://maven.apache.org/plugins/maven-antrun-plugin/)`
*   `[copy-rename-maven-plugin](https://web.archive.org/web/20221128040854/https://coderplus.github.io/copy-rename-maven-plugin/)`

## 2.使用 Maven 资源插件

[`maven-resources-plugin`](/web/20221128040854/https://www.baeldung.com/maven-resources-plugin) 处理将项目资源复制到输出目录。

让我们从添加插件到我们的`pom.xml`开始:

```java
<project>
    ...
    <build>
        <pluginManagement>
            <plugins>
                <plugin>
                    <groupId>org.apache.maven.plugins</groupId>
                    <artifactId>maven-resources-plugin</artifactId>
                    <version>3.2.0</version>
                </plugin>
                ...
            </plugins>
        </pluginManagement>
        <!-- To use the plugin goals in your POM or parent POM -->
        <plugins>
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-resources-plugin</artifactId>
                <version>3.2.0</version>
            </plugin>
            ...
        </plugins>
    </build>
    ...
</project>
```

然后，让我们在项目根目录下创建一个名为`source-files.`的文件夹，其中包含一个我们想要复制的文本文件:`foo.txt`。然后，我们将向`maven-resources-plugin`添加一个配置元素，以将该文件复制到`target/destination-folder`:

```java
<plugin>
    <artifactId>maven-resources-plugin</artifactId>
    <version>3.0.2</version>
    <executions>
        <execution>
            <id>copy-resource-one</id>
            <phase>generate-sources</phase>
            <goals>
                <goal>copy-resources</goal>
            </goals>
            <configuration>
                <outputDirectory>${basedir}/target/destination-folder</outputDirectory>
                <resources>
                    <resource>
                        <directory>source-files</directory>
                        <includes>
                            <include>foo.txt</include>
                        </includes>
                    </resource>
                </resources>
            </configuration>
        </execution>
    </executions>
</plugin>
```

建立项目后，我们可以在`target/destination-folder`中找到`foo.txt`。

## 3.使用 Maven Antrun 插件

[`maven-antrun-plugin`](/web/20221128040854/https://www.baeldung.com/maven-ant-task) 提供了从 Maven 内部运行 Ant 任务的能力。我们将在这里使用它来指定一个将源文件复制到目的地的 Ant 任务。

`pom.xml`中的插件定义如下:

```java
<project>
    [...]
    <build>
        <plugins>
            <plugin>
                <artifactId>maven-antrun-plugin</artifactId>
                <version>3.0.0</version>
                <executions>
                    <execution>
                        <phase>
                            <phase>generate-sources</phase>
                        </phase>
                        <configuration>
                            <target>
                                <!-- Place any Ant task here. -->
                            </target>
                        </configuration>
                        <goals>
                            <goal>run</goal>
                        </goals>
                    </execution>
                </executions>
            </plugin>
        </plugins>
    </build>
</project>
```

我们将执行与上面相同的示例:将`source-files/foo.txt`复制到`target/destination-folder/foo.txt`——我们将通过定义一个 Ant 任务来执行复制来实现这一点:

```java
<configuration>
    <target>
        <mkdir dir="${basedir}/target/destination-folder" />
        <copy todir="${basedir}/target/destination-folder">
            <fileset dir="${basedir}/source-files" includes="foo.txt" />
        </copy>
    </target>
</configuration>
```

构建项目后，我们将在`target/destination-folder`中找到`foo.txt`。

## 4.使用复制重命名 Maven 插件

`copy-rename-maven-plugin` 有助于在 Maven 构建生命周期中复制文件或重命名文件/目录。

可以通过将以下条目添加到我们的`pom.xml`来安装该插件:

```java
<project>
    ...
    <build>
        <plugins>
            <plugin>
                <groupId>com.coderplus.maven.plugins</groupId>
                <artifactId>copy-rename-maven-plugin</artifactId>
                <version>1.0</version>
                <executions>
                    <execution>
                        <id>copy-file</id>
                        <phase>generate-sources</phase>
                        <goals>
                            <goal>copy</goal>
                        </goals>
                        <configuration>
                            <!-- Place config here -->
                        </configuration>
                    </execution>
                </executions>
            </plugin>
        </plugins>
    </build>
</project> 
```

我们现在将添加一些配置来执行复制:`source-files/foo.txt`到`target/destination-folder/foo.txt`:

```java
<configuration>
    <sourceFile>source-files/foo.txt</sourceFile>
    <destinationFile>target/destination-folder/foo.txt</destinationFile>
</configuration>
```

在构建项目时，我们将在`target/destination-folder`中看到`foo.txt`。

## 5.结论

我们已经使用三个不同的 Maven 插件成功地将一个源文件复制到一个目的地。每个插件的操作都略有不同，虽然我们在这里介绍了复制单个文件，但是插件能够复制多个文件，在某些情况下，可以复制整个目录。

我们的其他文章以及每个插件的官方文档将进一步详细介绍如何执行更复杂的操作。

这些例子的源代码可以在 GitHub 上找到[。](https://web.archive.org/web/20221128040854/https://github.com/eugenp/tutorials/tree/master/maven-modules/maven-copy-files)