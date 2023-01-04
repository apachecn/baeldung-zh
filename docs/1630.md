# 将 Gradle 构建文件转换为 Maven POM

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/gradle-build-to-maven-pom>

## 1.介绍

在本教程中，我们将看看如何将 Gradle 构建文件转换为 Maven POM 文件。我们将使用 Gradle 版作为例子，我们还将探索一些可用的定制选项。

## 2.Gradle 构建文件

让我们从一个标准的 Gradle Java 项目`gradle-to-maven`开始，使用下面的`build.gradle`文件:

```
repositories {
    mavenCentral()
}

group = 'com.baeldung'
version = '0.0.1'

apply plugin: 'java'

dependencies {
    implementation 'org.slf4j:slf4j-api:1.7.25'
    testImplementation 'junit:junit:4.12'
}
```

## 3。Maven 插件

Gradle 附带了一个 [Maven 插件](https://web.archive.org/web/20220926191334/https://docs.gradle.org/current/userguide/maven_plugin.html)，它增加了将 Gradle 文件转换为 Maven POM 文件的支持。它还可以将工件部署到 Maven 存储库中。

要使用它，让我们将 Maven 发布插件添加到我们的`build.gradle` 文件中:

```
apply plugin: 'maven-publish'
```

插件使用 Gradle 文件中的`group`和`version`并将它们添加到 POM 文件中。此外，它会自动从目录名中提取`artifactId`。

插件也会自动添加`publish`任务。因此，为了进行转换，让我们将发布的基本定义添加到 POM 文件中:

```
publishing {
    publications {
        customLibrary(MavenPublication) {
            from components.java
        }
    }

    repositories {
        maven {
            name = 'sampleRepo'
            url = layout.buildDirectory.dir("repo")
        }
    }
}
```

现在，我们可以将我们的`customLibrary`发布到一个本地的基于目录的存储库，用于演示目的:

```
gradle publish
```

运行上面的命令会创建一个包含这些子目录的`build`目录:

*   `libs –` 包含名为`${artifactId}-${version}.jar`的罐子
*   **`publications/customLibrary` `–` 包含转换后的 POM 文件，文件名为`pom-default.xml`**
*   `tmp/jar –` 包含载货单的
*   `repo`–用作包含已发布工件的存储库的文件系统位置

生成的 POM 文件将如下所示:

```
<?xml version="1.0" encoding="UTF-8"?>
<project xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd" 

    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance">
  <modelVersion>4.0.0</modelVersion>
  <groupId>com.baeldung</groupId>
  <artifactId>gradle-to-maven</artifactId>
  <version>0.0.1</version>
  <dependencies>
    <dependency>
      <groupId>org.slf4j</groupId>
      <artifactId>slf4j-api</artifactId>
      <version>1.7.25</version>
      <scope>runtime</scope>
    </dependency>
  </dependencies>
</project>
```

请注意，**测试范围依赖项不包括在 POM 中，默认的`runtime`范围被分配给所有其他依赖项**。

`publish`任务还将这个 POM 文件和 JAR 上传到指定的存储库。

## 4.定制 Maven 插件

在某些情况下，在生成的 POM 文件中自定义项目信息可能会很有用。让我们来看看。

### 4.1.`groupId,` `artifactId,`和`version`

改变 POM 的`groupId`、`artifactId`和`version`可以在`publications/{publicationName}`块中操作:

```
publishing {
    publications {
        customLibrary(MavenPublishing) {
            groupId = 'com.baeldung.sample'
            artifactId = 'gradle-maven-converter'
            version = '0.0.1-maven'
        }
    }
}
```

运行 `publish`任务现在会产生带有上面提供的信息的 POM 文件:

```
<groupId>com.baeldung.sample</groupId>
<artifactId>gradle-maven-converter</artifactId>
<version>0.0.1-maven</version>
```

### 4.2.自动生成的内容

Maven 插件还使得更改任何生成的 POM 元素变得简单。例如，要将默认的`runtime`范围改为`compile`，我们可以将下面的闭包添加到`pom.withXml`方法中:

```
pom.withXml {
    asNode()
      .dependencies
      .dependency
      .findAll { dependency ->
        // find all dependencies with runtime scope
        dependency.scope.text() == 'runtime'
      }
      .each { dependency ->
        // set the scope to 'compile'
        dependency.scope*.value = 'compile'
      }
}
```

这将更改生成的 POM 文件中所有依赖项的范围:

```
<dependency>
  <groupId>org.slf4j</groupId>
  <artifactId>slf4j-api</artifactId>
  <version>1.7.25</version>
  <scope>compile</scope>
</dependency>
```

### 4.3.附加说明

最后，如果我们想要添加额外的信息，**我们可以在`pom` 函数中包含这种 Maven 支持的元素。**

让我们添加一些许可证信息:

```
...
pom {
    licenses {
        license {
            name = 'The Apache License, Version 2.0'
            url = 'http://www.apache.org/licenses/LICENSE-2.0.txt'
        }
    }
}
...
```

我们现在可以看到添加到 POM 中的许可证信息:

```
... 
<licenses>
  <license>
    <name>The Apache License, Version 2.0</name>
    <url>http://www.apache.org/licenses/LICENSE-2.0.txt</url>
  </license>
</licenses>
...
```

## 5.结论

在这个快速教程中，我们学习了如何将 Gradle 构建文件转换为 Maven POM。

和往常一样，这篇文章的源代码可以在 GitHub 上找到[。](https://web.archive.org/web/20220926191334/https://github.com/eugenp/tutorials/tree/master/gradle-modules/gradle/gradle-to-maven)