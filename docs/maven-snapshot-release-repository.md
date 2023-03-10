# Maven 快照存储库与发布存储库

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/maven-snapshot-release-repository>

## 1.概观

在本教程中，我们将解释 Maven 快照和发布库之间的区别。

## 2.Maven 仓库

Maven 存储库包含一组预编译的工件，我们可以在应用程序中将其作为依赖项使用。对于传统的 Java 应用程序，这些通常是`.jar`文件。

通常，有两种类型的存储库:本地和远程。

本地存储库是 Maven 在构建它的计算机上创建的存储库。它通常位于 [`$HOME/.m2/repository`](/web/20220909162234/https://www.baeldung.com/maven-local-repository) 目录下。

当我们构建一个应用程序时，Maven 将在我们的本地存储库中搜索依赖关系。如果没有找到某个依赖项，Maven 将在远程存储库中搜索它(在`[settings.xml](/web/20220909162234/https://www.baeldung.com/maven-settings-xml)`或`pom.xml`文件中定义)。此外，它会将依赖项复制到我们的本地存储库中以备将来使用。

远程存储库是包含工件的外部存储库。一旦 Maven 从远程存储库中下载了一个工件，它就会倾向于在本地存储库中查找工件，以限制工件的下载。

此外，我们可以根据快照和发布存储库之间的工件类型来区分存储库。

## 3.快照存储库

快照存储库是一个用于增量的、未发布的工件版本的存储库。

快照版本是尚未发布的版本。总体思路是在发布版本之前有一个快照版本。它允许我们增量地部署相同的临时版本，而不需要项目升级他们正在使用的工件版本。这些项目可以使用相同的版本来获得更新的快照版本。

例如，在发布版本`1.0.0`之前，我们可以有它的快照版本。快照版本在版本后面有一个`SNAPSHOT`后缀(例如`1.0.0-SNAPSHOT`)。

### 3.1.部署一个工件

持续开发通常使用快照版本控制。使用快照版本，我们可以部署一个工件，其编号由时间戳和构建号组成。

让我们假设我们有一个正在开发的项目，它有一个`SNAPSHOT`版本:

```java
<groupId>com.baeldung</groupId>
<artifactId>maven-snapshot-repository</artifactId>
<version>1.0.0-SNAPSHOT</version>
```

我们将把项目部署到一个自托管的 [Nexus](/web/20220909162234/https://www.baeldung.com/maven-deploy-nexus) 存储库中。

首先，让我们定义我们想要部署工件的发布存储库信息。我们可以使用分发管理插件:

```java
<distributionManagement>
    <snapshotRepository>
        <id>nexus</id>
        <name>nexus-snapshot</name>
        <url>http://localhost:8081/repository/maven-snapshots/</url>
    </snapshotRepository>
</distributionManagement>
```

此外，我们将使用`mvn deploy`命令部署我们的项目。

**在部署之后，实际的工件版本将包含一个时间戳值，而不是`SNAPSHOT`值。**例如，当我们部署`1.0.0-SNAPSHOT`时，实际值将包含当前时间戳和构建号(例如，`1.0.0-20220709.063105-3`)。

时间戳值是在工件部署期间计算的。Maven 生成校验和，并上传带有相同时间戳的工件文件。

`maven-metadata.xml`文件包含关于快照版本的精确信息及其与最新时间戳值的链接:

```java
<metadata modelVersion="1.1.0">
    <groupId>com.baeldung</groupId>
    <artifactId>maven-snapshot-repository</artifactId>
    <version>1.0.0-SNAPSHOT</version>
    <versioning>
        <snapshot>
            <timestamp>20220709.063105</timestamp>
            <buildNumber>3</buildNumber>
        </snapshot>
        <lastUpdated>20220709063105</lastUpdated>
        <snapshotVersions>
            <snapshotVersion>
                <extension>jar</extension>
                <value>1.0.0-20220709.063105-3</value>
                <updated>20220709063105</updated>
            </snapshotVersion>
            <snapshotVersion>
                <extension>pom</extension>
                <value>1.0.0-20220709.063105-3</value>
                <updated>20220709063105</updated>
            </snapshotVersion>
        </snapshotVersions>
    </versioning>
</metadata>
```

元数据文件有助于管理从快照版本到时间戳值的转换。

每次我们在同一个快照版本下部署项目时，Maven 都会生成包含新时间戳值和新构建号的版本。

### 3.2.下载神器

在下载快照工件之前，Maven 下载其关联的`maven-metadata.xml`文件。这样，Maven 可以根据时间戳值和内部版本号检查是否有更新的版本。

**这样一个神器的检索仍然可以使用`SNAPSHOT`版本。**

要从存储库中下载工件，首先，我们需要定义一个依赖关系存储库:

```java
<repositories>
    <repository>
        <id>nexus</id>
        <name>nexus-snapshot</name>
        <url>http://localhost:8081/repository/maven-snapshots/</url>
        <snapshots>
            <enabled>true</enabled>
        </snapshots>
        <releases>
            <enabled>false</enabled>
        </releases>
    </repository>
</repositories>
```

**默认情况下不启用快照版本。**我们需要手动启用它们:

```java
<snapshots>
    <enabled>true</enabled>
</snapshots>
```

通过启用快照，我们可以定义多久检查一次`SNAPSHOT`工件的新版本。但是，默认更新策略设置为每天一次。我们可以通过设置不同的更新策略来覆盖此行为:

```java
<snapshots>
    <enabled>true</enabled>
    <updatePolicy>always</updatePolicy>
</snapshots>
```

我们可以在`updatePolicy`元素中放置四个不同的值:

*   `always` —每次检查是否有更新的版本
*   `daily`(默认值)—每天检查一次更新版本
*   `interval:mm` —根据以分钟为单位设置的时间间隔检查更新版本
*   永远不要尝试获取新版本(与我们本地已有的版本相比)

此外，我们可以通过在命令中传递参数`-U`来强制更新所有快照工件，而不是定义`updatePolicy`:

```java
mvn install -U
```

此外，如果依赖项已经被下载，并且校验和与我们在本地存储库中已经有的校验和相同，则不会被重新下载。

接下来，我们可以向我们的项目添加一个工件的快照版本:

```java
<dependencies>
    <dependency>
        <groupId>com.baeldung</groupId>
        <artifactId>maven-snapshot-repository</artifactId>
        <version>1.0.0-SNAPSHOT</version>
    </dependency>
</dependencies>
```

**在开发阶段使用快照版本可以防止工件有多个版本。**我们可以使用相同的`SNAPSHOT`版本，其构建将包含我们在给定时间的代码快照。

## 4.发布存储库

发布存储库包含工件的最终版本(发布)。简单地说，一个发布工件代表一个其内容不应该被修改的工件。

默认情况下，在我们的`settings.xml`或`pom.xml`文件中定义的所有存储库都启用发布存储库。

### 4.1.部署一个工件

现在，让我们在本地 Nexus 存储库中部署项目。让我们假设我们已经完成了开发，并准备发布项目:

```java
<groupId>com.baeldung</groupId>
<artifactId>maven-release-repository</artifactId>
<version>1.0.0</version>
```

让我们在分发管理器中定义发布存储库:

```java
<distributionManagement>
    <repository>
        <id>nexus</id>
        <name>nexus-release</name>
        <url>http://localhost:8081/repository/maven-releases/</url>
    </repository>
    <snapshotRepository>
        <id>nexus</id>
        <name>nexus-snapshot</name>
        <url>http://localhost:8081/repository/maven-snapshots/</url>
    </snapshotRepository>
</distributionManagement>
```

一旦我们从项目版本中删除了单词`SNAPSHOT`,在部署期间将自动选择发布存储库而不是快照存储库。

此外，如果我们想要在相同的版本下重新部署工件，我们可能会得到一个错误:“`Repository does not allow updating assets`”。**一旦我们部署了发布的工件版本，我们就不能改变它的内容。**因此，要解决这个问题，我们需要发布下一个版本。

### 4.2.下载神器

Maven 默认从 [Maven 中央存储库](https://web.archive.org/web/20220909162234/https://repo1.maven.org/maven2)中寻找组件。默认情况下，该存储库使用发布版本策略。

发布库将只解析发布的工件。换句话说，它应该只包含发布的工件版本，这些版本的内容在将来不应该改变。

如果我们想要下载发布的工件，我们需要定义存储库:

```java
<repository>
    <id>nexus</id>
    <name>nexus-release</name>
    <url>http://localhost:8081/repository/maven-releases/</url>
</repository>
```

最后，让我们简单地将发布版本添加到我们的项目中:

```java
<dependencies>
    <dependency>
        <groupId>com.baeldung</groupId>
        <artifactId>maven-release-repository</artifactId>
        <version>1.0.0</version>
    </dependency>
</dependencies>
```

## 5.结论

在本教程中，我们学习了 Maven 快照和发布库之间的区别。总的来说，我们应该为仍在开发中的项目使用快照存储库，为已经准备好投入生产的项目使用发布存储库。与往常一样，GitHub 上的[提供了示例的源代码。](https://web.archive.org/web/20220909162234/https://github.com/eugenp/tutorials/tree/master/maven-modules)