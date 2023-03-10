# 用 Maven 运行 Ant 任务

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/maven-ant-task>

## 1。简介

Maven 和 Ant 都是众所周知的 Java 构建自动化工具。虽然大多数时间我们只使用其中的一个，但在某些情况下，将两者结合使用是有意义的。

**一个常见的用例是在处理一个使用 ant 的遗留项目时，我们希望逐渐引入 Maven】，同时仍然保留一些现有的 Ant 任务。**

在本教程中，我们将介绍如何使用 Maven AntRun 插件做到这一点。

## 2。Maven `AntRun`插件

Maven 插件允许我们在 Maven 中运行 Ant 任务。

### 2.1。添加插件

要使用这个插件，我们需要将它添加到我们的 Maven 项目的构建插件中:

```java
<plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-antrun-plugin</artifactId>
    <version>1.8</version>
    <executions>
        ...
    </executions>
</plugin>
```

最新的插件版本可以在 [Maven Central](https://web.archive.org/web/20221129013036/https://search.maven.org/classic/#search%7Cgav%7C1%7Cg%3A%22org.apache.maven.plugins%22%20AND%20a%3A%22maven-antrun-plugin%22) 上找到(虽然很久没更新了)。

### 2.2。插件执行

与任何其他 Maven 插件一样，要使用 AntRun 插件，我们需要定义执行。

在下面的例子中，我们定义了一个绑定到 Maven 的*包*阶段的执行，它将从项目的目标目录中压缩最终的 JAR 文件:

```java
<plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-ant-run-plugin</artifactId>
    <version>1.8</version>
    <executions>
        <execution>
            <id>zip-artifacts</id>
            <phase>package</phase>
            <goals>
                <goal>run</goal>
            </goals>
            <configuration>
                <target>
                    <zip destfile="${project.basedir}/package.zip" 
                       basedir="${project.build.directory}" 
                       includes="*.jar" />
                </target>
            </configuration>
        </execution>
    </executions>
</plugin>
```

要执行插件，我们运行命令:

```java
mvn package
```

因为我们声明我们的插件在 Maven 的`package`阶段运行，运行 Maven 的`package`目标将执行我们上面的插件配置。

## 3。使用`build.xml`文件的例子

除了允许我们在插件配置中定义 Ant 目标，我们还可以使用现有的 Ant `build.xml `文件。

### 3.1.`build.xml`

下面是一个项目的 Ant `build.xml`文件的示例，其中定义了一个目标，将 zip 文件从项目的基本目录上传到 FTP 服务器:

```java
<project name="MyProject" default="dist" basedir=".">
   <description>Project Description</description>

   ...

    <target name="ftpArtifact">
        <ftp 
          server="${ftp.host}" 
          userid="${ftp.user}" 
          password="${ftp.password}">
            <fileset dir="${project.basedir}>
                <include name="**/*.zip" />
            </fileset>
        </ftp>
    </target>
</project>
```

### 3.2.插件配置

为了使用上面的`build.xml`文件，我们在插件声明中定义了执行:

```java
<execution>
    <id>deploy-artifact</id>
    <phase>install</phase>
    <goals>
        <goal>run</goal>
    </goals>
    <configuration>
        <target>
            <ant antfile="${basedir}/build.xml">
                <target name="ftpArtifact"/>
            </ant>
        </target>
    </configuration>
</execution>
```

由于`ftp`任务不包含在`ant.jar`中，我们需要将 Ant 的可选依赖项添加到我们的插件配置中:

```java
<plugin>
    <executions>
       ...
    </executions>
    <dependencies>
        <dependency>
            <groupId>commons-net</groupId>
            <artifactId>commons-net</artifactId>
            <version>1.4.1</version>
        </dependency>
        <dependency>
            <groupId>org.apache.ant</groupId>
            <artifactId>ant-commons-net</artifactId>
            <version>1.8.1</version>
        </dependency>
    </dependencies>
</plugin>
```

要执行插件，我们运行命令:

```java
mvn install
```

## 4。结论

在这篇短文中，我们讨论了用 Maven 的`AntRun`插件运行 Ant 任务。尽管它是一个非常简单的插件，只有一个目标，但这个插件可以被证明在喜欢使用 Ant 进行特定构建指令的项目和团队中是有效的。

如果你想了解更多关于 ant 和 Maven 的知识，你可以阅读我们的[文章](/web/20221129013036/https://www.baeldung.com/ant-maven-gradle)，和 Gradle 一起比较这两者。